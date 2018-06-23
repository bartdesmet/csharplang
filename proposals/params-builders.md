# Generalized `params`

* [x] Proposed
* [ ] Prototype
* [ ] Implementation
* [ ] Specification

## Summary
[summary]: #summary

C# has support for `params` parameters in parameter lists on methods, constructors, and indexers. The type of a `params` parameter is restricted to single-dimensional array types.

Support for `params IEnumerable<T>` has been suggested before, but a more general pattern starts to show up when constructing APIs that build immutable data structures. A good example is the `System.Linq.Expressions` API where factory methods accept a `params Expression[]` but need to create a defensive copy of the array to ensure immutability (using an internal `TrueReadOnlyCollection<T>` type). This causes excessive allocations, even if the `params` array was constructed by the compiler, which ensures only the callee can observe the reference to the array, so external mutation by the caller is not possible.

This proposal suggests a collection initialization builder pattern that is supported by `params` parameters "arrays".

## Detailed design
[design]: #detailed-design

In the section on [Method parameters](../spec/classes.md#method-parameters), the grammar production for *parameter_array* is altered as follows:

```antlr
parameter_array
    : attributes? 'params' type identifier
    ;
```

where *type* was substituted for *array_type*.

The following paragraph is changed:

> A *parameter_array* consists of an optional set of *attributes* ([Attributes](../spec/attributes.md)), a `params` modifier, an *array_type*, and an *identifier*. A parameter array declares a single parameter of the given array type with the given name. The *array_type* of a parameter array must be a single-dimensional array type ([Array types](../spec/arrays.md#array-types)). In a method invocation, a parameter array permits either a single argument of the given array type to be specified, or it permits zero or more arguments of the array element type to be specified. Parameter arrays are described further in [Parameter arrays](../spec/classes.md#parameter-arrays).

to

> A ***parameter array type*** is either single-dimensional array type ([Array types](../spec/arrays.md#array-types)) or a type that has an associated ***parameter array builder type***.
>
> The ***parameter array builder type*** `B` is derived from a candidate parameter array type `T` by obtaining the type specified in the single argument on the `System.Runtime.CompilerServices.CollectionBuilderAtribute` custom attribute applied to `T`, if `T` is a non-generic type, or on the open generic type definition of `T` if `T` is a closed generic type. Base classes are not considered to locate this custom attribute.
>
> If `T` is a closed generic type with `N` type arguments, `B` should be an open generic type with `N` type parameters. If this condition is not met, `T` is not classified as a parameter array type. Otherwise, we redefine `B` to be the closed generic instantiation using the type arguments of `T` for the remainder of this section.
>
> `B` is a `class` or a `struct` and has the following public methods:
>
> ```csharp
> {
>     public static I Create(int length);
> }
> ```
>
> where `I` is an intermediate type which may be different from `B`. `I` is a `class` or a `struct` and has the following public methods:
>
> ```csharp
> {
>     public void Add(E element);
>     public void R GetResult();
> }
> ```
>
> The type `E` on the `Add` method is the inferred ***parameter array element type***. If multiple overloads of `Add` exist with 1 parameter, `E` cannot be determined uniquely and `T` is not classified as a parameter array type.
>
> Note that `E` can never be an open generic type because `Add` and `Create` are not allowed to have type parameters, and `B` is either non-generic or a closed generic type using the type arguments of `T`.
>
> The return type `R` of `GetResult` should be implicitly convertible to `T` ([Implicit conversions](../spec/conversions.md#implicit-conversions)).
>
> If any of the preceding requirements is not met, `T` is not classified as a parameter array type.
>
> A *parameter_array* consists of an optional set of *attributes* ([Attributes](../spec/attributes.md)), a `params` modifier, a *type*, and an *identifier*. A parameter array declares a single parameter of the given type with the given name. The *type* of a parameter array must be a parameter array type. In a method invocation, a parameter array permits either a single argument of the given array type to be specified, or it permits zero or more arguments of the array element type to be specified. Parameter arrays are described further in [Parameter arrays](../spec/classes.md#parameter-arrays).

Note that the section on [Parameter arrays](../spec/classes.md#parameter-arrays) refers to the "element type of the parameter array" which is either:

* the element type of the array type, if the ***parameter array type*** is a single-dimensional array type, or,
* the inferred ***parameter array element type*** `E` otherwise.

The rules for [Better function member](../spec/expressions.md#better-function-member) need to be reviewed, because the set of applicable function members can grow because more function members with a parameter array can become applicable in their expanded form. Betterness between two expanded forms may have to be revisited.

Note that there is no immediate breaking change potential by introducing parameter array support for types other than single-dimensional arrays, because it's currently invalid to use the `params` modifier on types other than single-dimensional arrays.

## Example

An example of a parameter array builder type for `ImmutableArray<T>` is shown below:

```csharp
[CollectionBuilder(typeof(ImmutableArrayBuilder<>))]
public struct ImmutableArray<T>
{
    internal ImmutableArray(T[] items) { ... }
}

public struct ImmutableArrayBuilder<T>
{
    private readonly T[] _array;
    private int _index;

    public ImmutableArrayBuilder<T>(int length)
    {
        _array = new T[length];
        _index = 0;
    }

    public static ImmutableArrayBuilder<T> Create(int length) =>
        new ImmutableArrayBuilder<T>(length);

    public void Add(T element) => [_index++] = element;

    public ImmutableArray<T> GetResult() => new ImmutableArray<T>(_array);
}
```

This enables a user to write the following method:

```csharp
void F(int a, params ImmutableArray<int> array) { ... }
```

and call it like this:

```csharp
F(x(), y(), z());
```

which results in the following code generation:

```csharp
var t0 = x();
var t1 = y();
var t2 = z();

var t3 = ImmutableArrayBuilder<int>.Create(2);
t3.Add(t1);
t3.Add(t2);

F(t0, t3.GetResult());
```

where variables `t0` to `t3` are compiler-generated.

Note that the use of this feature works well with immutable collection types but is not exclusive to such types. Even mutable collection types such as lists could benefit from `params` support with the added benefit of inferring the initial capacity at compile time. This helps to optimize code like this:

```csharp
Initialize(new List<int> { 1, 2, 3, 4, 5 })
```

which does not specify an initial capacity, or code like this:

```csharp
Initialize(new List<int>(4) { 1, 2, 3, 4, 5 })
```

where someone added an element and forgot to update the capacity. If a type such as `List<T>` were to add support for a `params` builder type, both issues can be avoided and the call-site syntax gets cleaner:

```csharp
Initialize(1, 2, 3, 4, 5)
```

## Open questions

Should a change to collection initializer expressions be considered to support builder types? For example:

```csharp
new List<int> { 1, 2, 3, 4, 5 }
```

could benefit from the builder approach because the capacity can be statically determined at compile time. An example with immutable arrays would become even more obvious:

```csharp
new ImmutableArray<int> { 1, 2, 3, 4, 5 }
```

because there is no similar concise way of creating these without undesirable allocations, for example:

```csharp
// a params int[] allocation, and a defensive copy
ImmutableArray.Create<int>(1, 2, 3, 4, 5)

// a builder allocation, a params int[] allocation, and a defensive copy
ImmutableArray.CreateBuilder<int>(5).AddRange(1, 2, 3, 4, 5).ToImmutable()

// a builder allocation and incredibly verbose
var t = ImmutableArray.CreateBuilder<int>(5);
t.Add(1);
t.Add(2);
t.Add(3);
t.Add(4);
t.Add(t);
var array = t.ToImmutable();

// NOT POSSIBLE because the nested Builder type has an internal constructor
// a builder allocation and lots of decoration
new ImmutableArray<T>.Builder(5) { 1, 2, 3, 4, 5 }.ToImmutable()
```

The last form is almost exactly what the proposal's code generation does, if one substitutes the `Builder` constructor invocation for a `Create` method invocation, unrolls the collection initializer to `Add` methods, and replaces `ToImmutable` by `GetResult`.

There are some concerns to be addressed. First, adding a builder type to an existing type can cause subtle change of behavior. In the examples above, the second case of `ImmutableArray<int>` was never possible due to the lack of an `Add` method, so there's no issue there. For the first case of `List<int>`, the behavior changes from invoking the default constructor on the type to emitting code against the associated builder type.

One could argue that library writers should ensure that adding a collection builder type to an existing type does not cause any change in observable behavior when used with a collection initializer. For the case of `List<T>` this would be very achievable.

Second, collection initializers also support concise initialization syntax using `Add` methods with more than 1 parameter.

```csharp
new Dictionary<string, int> { { "a", 1 } }
```

Supporting this in combination with builder types would require revisting the definitions earlier in this proposal such that `Add` has element types `E_1` to `E_N` as parameters, and usage in the context of `params` assignment always picks the overload of `Add` that has 1 parameter. We'd have to define what it means to have multiple overloads of `Add` with 1 parameter. This combination would make the following valid:

```csharp
[CollectionBuilder(typeof(DictionaryBuilder<,>))]
public class Dictionary<K, V>
{
    ...
}

public struct DictionaryBuilder<K, V>
{
    private readonly Dictionary<K, V> _dictionary;

    public DictionaryBuilder(int length)
    {
        _dictionary = new Dictionary<K, V>(length);
    }

    public static DictionaryBuilder<K, V> Create(int length) =>
        new DictionaryBuilder<K, V>(length);

    public void Add(KeyValuePair<K, V> element) =>
        _dictionary.Add(element.Key, element.Value);

    public void Add(K key, V value) =>
        _dictionary.Add(key, value);

    public Dictionary<K, V> GetResult() => _dictionary;
}
```

Collection initializer expressions would be able to bind to either `Add` method overload:

```csharp
new Dictionary<string, int> {
    { "a", 1 },
    new KeyValuePair<string, int>("b", 2)
}
```

Assignment to a `params` array would work equally well:

```csharp
InitializeDictionary<string, int>(
    new KeyValuePair<string, int>("a", 1),
    new KeyValuePair<string, int>("b", 2)
)
```

but only binding to the `Add` method with 1 parameter. Thinking about the proposal in this way, one could argue that `params` is a shorthand inline array initializer and the previous example is equivalent to:

```csharp
InitializeDictionary<string, int>(new Dictionary<string, int> {
    new KeyValuePair<string, int>("a", 1),
    new KeyValuePair<string, int>("b", 2)
})
```

just like

```csharp
Sum(1, 2, 3)
```

is equivalent to:

```csharp
Sum(new int[] { 1, 2, 3 })
```

and both examples are a means to convert from the expanded form to the normal form by introducing an object creation expression. This could be the more elegant way to formulate `params` collection builder support in terms of a desugaring "lowering" step to a collection initializer expression.

Making the following work is likely a bridge too far:

```csharp
InitializeDictionary<string, int>(
    { "a", 1 },
    { "b", 2 }
)
```

where collection initialization syntax can be used when binding to `params` in the expanded form, supporting binding to `Add` methods with more than 1 parameter. It's not at all clear how candidate members would be selected because these element initializers are not expressions and don't contribute any types. However, if one adds an `Add` method with a tuple:

```csharp
    public void Add((K key, V value) element) =>
        _dictionary.Add(element.key, element.value);
```

almost the same concise syntax would work:

```csharp
InitializeDictionary<string, int>(
    ("a", 1),
    ("b", 2)
)
```

because the element type `E` can be inferred as a `(string, int)` tuple type. The question on having multiple overloads of `Add` with 1 parameter still comes up though.

It'd also be interesting to consider target-typed `new` with collection initializers and builders. Would the following work as an abbreviated target-typed collection initializer expression variant?

```csharp
new
{
    { "a", 1 },
    { "b", 2 }
}
```

if the type of the `new` expression is inferred to be `Dictionary<string, int>`? If so, one could still write:

```csharp
InitializeDictionary<string, int>(new {
    { "a", 1 },
    { "b", 2 }
})
```

which would not involve any `params` expanded form binding, but the collection initializer expression may still bind to a builder.
