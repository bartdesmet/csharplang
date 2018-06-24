# Expressions

An expression is a sequence of operators and operands. This chapter defines the syntax, order of evaluation of operands and operators, and meaning of expressions.

## Expression classifications

An expression is classified as one of the following:

*  A value. Every value has an associated type.
*  A variable. Every variable has an associated type, namely the declared type of the variable.
*  A namespace. An expression with this classification can only appear as the left hand side of a *member_access* ([Member access](expressions.md#member-access)). In any other context, an expression classified as a namespace causes a compile-time error.
*  A type. An expression with this classification can only appear as the left hand side of a *member_access* ([Member access](expressions.md#member-access)), or as an operand for the `as` operator ([The as operator](expressions.md#the-as-operator)), the `is` operator ([The is operator](expressions.md#the-is-operator)), or the `typeof` operator ([The typeof operator](expressions.md#the-typeof-operator)). In any other context, an expression classified as a type causes a compile-time error.
*  A method group, which is a set of overloaded methods resulting from a member lookup ([Member lookup](expressions.md#member-lookup)). A method group may have an associated instance expression and an associated type argument list. When an instance method is invoked, the result of evaluating the instance expression becomes the instance represented by `this` ([This access](expressions.md#this-access)). A method group is permitted in an *invocation_expression* ([Invocation expressions](expressions.md#invocation-expressions)) , a *delegate_creation_expression* ([Delegate creation expressions](expressions.md#delegate-creation-expressions)) and as the left hand side of an is operator, and can be implicitly converted to a compatible delegate type ([Method group conversions](conversions.md#method-group-conversions)). In any other context, an expression classified as a method group causes a compile-time error.
*  A null literal. An expression with this classification can be implicitly converted to a reference type or nullable type.
*  An anonymous function. An expression with this classification can be implicitly converted to a compatible delegate type or expression tree type.
*  A property access. Every property access has an associated type, namely the type of the property. Furthermore, a property access may have an associated instance expression. When an accessor (the `get` or `set` block) of an instance property access is invoked, the result of evaluating the instance expression becomes the instance represented by `this` ([This access](expressions.md#this-access)).
*  An event access. Every event access has an associated type, namely the type of the event. Furthermore, an event access may have an associated instance expression. An event access may appear as the left hand operand of the `+=` and `-=` operators ([Event assignment](expressions.md#event-assignment)). In any other context, an expression classified as an event access causes a compile-time error.
*  An indexer access. Every indexer access has an associated type, namely the element type of the indexer. Furthermore, an indexer access has an associated instance expression and an associated argument list. When an accessor (the `get` or `set` block) of an indexer access is invoked, the result of evaluating the instance expression becomes the instance represented by `this` ([This access](expressions.md#this-access)), and the result of evaluating the argument list becomes the parameter list of the invocation.
*  Nothing. This occurs when the expression is an invocation of a method with a return type of `void`. An expression classified as nothing is only valid in the context of a *statement_expression* ([Expression statements](statements.md#expression-statements)).

The final result of an expression is never a namespace, type, method group, or event access. Rather, as noted above, these categories of expressions are intermediate constructs that are only permitted in certain contexts.

A property access or indexer access is always reclassified as a value by performing an invocation of the *get accessor* or the *set accessor*. The particular accessor is determined by the context of the property or indexer access: If the access is the target of an assignment, the *set accessor* is invoked to assign a new value ([Simple assignment](expressions.md#simple-assignment)). Otherwise, the *get accessor* is invoked to obtain the current value ([Values of expressions](expressions.md#values-of-expressions)).

### Values of expressions

Most of the constructs that involve an expression ultimately require the expression to denote a ***value***. In such cases, if the actual expression denotes a namespace, a type, a method group, or nothing, a compile-time error occurs. However, if the expression denotes a property access, an indexer access, or a variable, the value of the property, indexer, or variable is implicitly substituted:

*  The value of a variable is simply the value currently stored in the storage location identified by the variable. A variable must be considered definitely assigned ([Definite assignment](variables.md#definite-assignment)) before its value can be obtained, or otherwise a compile-time error occurs.
*  The value of a property access expression is obtained by invoking the *get accessor* of the property. If the property has no *get accessor*, a compile-time error occurs. Otherwise, a function member invocation ([Compile-time checking of dynamic overload resolution](expressions.md#compile-time-checking-of-dynamic-overload-resolution)) is performed, and the result of the invocation becomes the value of the property access expression.
*  The value of an indexer access expression is obtained by invoking the *get accessor* of the indexer. If the indexer has no *get accessor*, a compile-time error occurs. Otherwise, a function member invocation ([Compile-time checking of dynamic overload resolution](expressions.md#compile-time-checking-of-dynamic-overload-resolution)) is performed with the argument list associated with the indexer access expression, and the result of the invocation becomes the value of the indexer access expression.

## Static and Dynamic Binding

The process of determining the meaning of an operation based on the type or value of constituent expressions (arguments, operands, receivers) is often referred to as ***binding***. For instance the meaning of a method call is determined based on the type of the receiver and arguments. The meaning of an operator is determined based on the type of its operands.

In C# the meaning of an operation is usually determined at compile-time, based on the compile-time type of its constituent expressions. Likewise, if an expression contains an error, the error is detected and reported by the compiler. This approach is known as ***static binding***.

However, if an expression is a dynamic expression (i.e. has the type `dynamic`) this indicates that any binding that it participates in should be based on its run-time type (i.e. the actual type of the object it denotes at run-time) rather than the type it has at compile-time. The binding of such an operation is therefore deferred until the time where the operation is to be executed during the running of the program. This is referred to as ***dynamic binding***.

When an operation is dynamically bound, little or no checking is performed by the compiler. Instead if the run-time binding fails, errors are reported as exceptions at run-time.

The following operations in C# are subject to binding:

*  Member access: `e.M`
*  Method invocation: `e.M(e1, ..., eN)`
*  Delegate invocation:`e(e1, ..., eN)`
*  Element access: `e[e1, ..., eN]`
*  Object creation: `new C(e1, ..., eN)`
*  Overloaded unary operators: `+`, `-`, `!`, `~`, `++`, `--`, `true`, `false`
*  Overloaded binary operators: `+`, `-`, `*`, `/`, `%`, `&`, `&&`, `|`, `||`, `??`, `^`, `<<`, `>>`, `==`,`!=`, `>`, `<`, `>=`, `<=`
*  Assignment operators: `=`, `+=`, `-=`, `*=`, `/=`, `%=`, `&=`, `|=`, `^=`, `<<=`, `>>=`
*  Implicit and explicit conversions

When no dynamic expressions are involved, C# defaults to static binding, which means that the compile-time types of constituent expressions are used in the selection process. However, when one of the constituent expressions in the operations listed above is a dynamic expression, the operation is instead dynamically bound.

### Binding-time

Static binding takes place at compile-time, whereas dynamic binding takes place at run-time. In the following sections, the term ***binding-time*** refers to either compile-time or run-time, depending on when the binding takes place.

The following example illustrates the notions of static and dynamic binding and of binding-time:
```csharp
object  o = 5;
dynamic d = 5;

Console.WriteLine(5);  // static  binding to Console.WriteLine(int)
Console.WriteLine(o);  // static  binding to Console.WriteLine(object)
Console.WriteLine(d);  // dynamic binding to Console.WriteLine(int)
```

The first two calls are statically bound: the overload of `Console.WriteLine` is picked based on the compile-time type of their argument. Thus, the binding-time is compile-time.

The third call is dynamically bound: the overload of `Console.WriteLine` is picked based on the run-time type of its argument. This happens because the argument is a dynamic expression -- its compile-time type is `dynamic`. Thus, the binding-time for the third call is run-time.

### Dynamic binding

The purpose of dynamic binding is to allow C# programs to interact with ***dynamic objects***, i.e. objects that do not follow the normal rules of the C# type system. Dynamic objects may be objects from other programming languages with different types systems, or they may be objects that are programmatically setup to implement their own binding semantics for different operations.

The mechanism by which a dynamic object implements its own semantics is implementation defined. A given interface -- again implementation defined -- is implemented by dynamic objects to signal to the C# run-time that they have special semantics. Thus, whenever operations on a dynamic object are dynamically bound, their own binding semantics, rather than those of C# as specified in this document, take over.

While the purpose of dynamic binding is to allow interoperation with dynamic objects, C# allows dynamic binding on all objects, whether they are dynamic or not. This allows for a smoother integration of dynamic objects, as the results of operations on them may not themselves be dynamic objects, but are still of a type unknown to the programmer at compile-time. Also dynamic binding can help eliminate error-prone reflection-based code even when no objects involved are dynamic objects.

The following sections describe for each construct in the language exactly when dynamic binding is applied, what compile time checking -- if any -- is applied, and what the compile-time result and expression classification is.

### Types of constituent expressions

When an operation is statically bound, the type of a constituent expression (e.g. a receiver, and argument, an index or an operand) is always considered to be the compile-time type of that expression.

When an operation is dynamically bound, the type of a constituent expression is determined in different ways depending on the compile-time type of the constituent expression:

*  A constituent expression of compile-time type `dynamic` is considered to have the type of the actual value that the expression evaluates to at runtime
*  A constituent expression whose compile-time type is a type parameter is considered to have the type which the type parameter is bound to at runtime
*  Otherwise the constituent expression is considered to have its compile-time type.

## Operators

Expressions are constructed from ***operands*** and ***operators***. The operators of an expression indicate which operations to apply to the operands. Examples of operators include `+`, `-`, `*`, `/`, and `new`. Examples of operands include literals, fields, local variables, and expressions.

There are three kinds of operators:

*  Unary operators. The unary operators take one operand and use either prefix notation (such as `--x`) or postfix notation (such as `x++`).
*  Binary operators. The binary operators take two operands and all use infix notation (such as `x + y`).
*  Ternary operator. Only one ternary operator, `?:`, exists; it takes three operands and uses infix notation (`c ? x : y`).

The order of evaluation of operators in an expression is determined by the ***precedence*** and ***associativity*** of the operators ([Operator precedence and associativity](expressions.md#operator-precedence-and-associativity)).

Operands in an expression are evaluated from left to right. For example, in `F(i) + G(i++) * H(i)`, method `F` is called using the old value of `i`, then method `G` is called with the old value of `i`, and, finally, method `H` is called with the new value of `i`. This is separate from and unrelated to operator precedence.

Certain operators can be ***overloaded***. Operator overloading permits user-defined operator implementations to be specified for operations where one or both of the operands are of a user-defined class or struct type ([Operator overloading](expressions.md#operator-overloading)).

### Operator precedence and associativity

When an expression contains multiple operators, the ***precedence*** of the operators controls the order in which the individual operators are evaluated. For example, the expression `x + y * z` is evaluated as `x + (y * z)` because the `*` operator has higher precedence than the binary `+` operator. The precedence of an operator is established by the definition of its associated grammar production. For example, an *additive_expression* consists of a sequence of *multiplicative_expression*s separated by `+` or `-` operators, thus giving the `+` and `-` operators lower precedence than the `*`, `/`, and `%` operators.

The following table summarizes all operators in order of precedence from highest to lowest:

| __Section__                                                                                   | __Category__                | __Operators__ | 
|-----------------------------------------------------------------------------------------------|-----------------------------|---------------|
| [Primary expressions](expressions.md#primary-expressions)                                     | Primary                     | `x.y`  `f(x)`  `a[x]`  `x++`  `x--`  `new`  `typeof`  `default`  `checked`  `unchecked`  `delegate` | 
| [Unary operators](expressions.md#unary-operators)                                             | Unary                       | `+`  `-`  `!`  `~`  `++x`  `--x`  `(T)x` | 
| [Arithmetic operators](expressions.md#arithmetic-operators)                                   | Multiplicative              | `*`  `/`  `%` | 
| [Arithmetic operators](expressions.md#arithmetic-operators)                                   | Additive                    | `+`  `-`      | 
| [Shift operators](expressions.md#shift-operators)                                             | Shift                       | `<<`  `>>`    | 
| [Relational and type-testing operators](expressions.md#relational-and-type-testing-operators) | Relational and type testing | `<`  `>`  `<=`  `>=`  `is`  `as` | 
| [Relational and type-testing operators](expressions.md#relational-and-type-testing-operators) | Equality                    | `==`  `!=`    | 
| [Logical operators](expressions.md#logical-operators)                                         | Logical AND                 | `&`           | 
| [Logical operators](expressions.md#logical-operators)                                         | Logical XOR                 | `^`           | 
| [Logical operators](expressions.md#logical-operators)                                         | Logical OR                  | `\|`          |
| [Conditional logical operators](expressions.md#conditional-logical-operators)                 | Conditional AND             | `&&`          | 
| [Conditional logical operators](expressions.md#conditional-logical-operators)                 | Conditional OR              | `\|\|`        | 
| [The null coalescing operator](expressions.md#the-null-coalescing-operator)                   | Null coalescing             | `??`          | 
| [Conditional operator](expressions.md#conditional-operator)                                   | Conditional                 | `?:`          | 
| [Assignment operators](expressions.md#assignment-operators), [Anonymous function expressions](expressions.md#anonymous-function-expressions)  | Assignment and lambda expression | `=`  `*=`  `/=`  `%=`  `+=`  `-=`  `<<=`  `>>=`  `&=`  `^=`  `\|=`  `=>` | 

When an operand occurs between two operators with the same precedence, the associativity of the operators controls the order in which the operations are performed:

*  Except for the assignment operators and the null coalescing operator, all binary operators are ***left-associative***, meaning that operations are performed from left to right. For example, `x + y + z` is evaluated as `(x + y) + z`.
*  The assignment operators, the null coalescing operator and the conditional operator (`?:`) are ***right-associative***, meaning that operations are performed from right to left. For example, `x = y = z` is evaluated as `x = (y = z)`.

Precedence and associativity can be controlled using parentheses. For example, `x + y * z` first multiplies `y` by `z` and then adds the result to `x`, but `(x + y) * z` first adds `x` and `y` and then multiplies the result by `z`.

### Operator overloading

All unary and binary operators have predefined implementations that are automatically available in any expression. In addition to the predefined implementations, user-defined implementations can be introduced by including `operator` declarations in classes and structs ([Operators](classes.md#operators)). User-defined operator implementations always take precedence over predefined operator implementations: Only when no applicable user-defined operator implementations exist will the predefined operator implementations be considered, as described in [Unary operator overload resolution](expressions.md#unary-operator-overload-resolution) and [Binary operator overload resolution](expressions.md#binary-operator-overload-resolution).

The ***overloadable unary operators*** are:
```csharp
+   -   !   ~   ++   --   true   false
```

Although `true` and `false` are not used explicitly in expressions (and therefore are not included in the precedence table in [Operator precedence and associativity](expressions.md#operator-precedence-and-associativity)), they are considered operators because they are invoked in several expression contexts: boolean expressions ([Boolean expressions](expressions.md#boolean-expressions)) and expressions involving the conditional ([Conditional operator](expressions.md#conditional-operator)), and conditional logical operators ([Conditional logical operators](expressions.md#conditional-logical-operators)).

The ***overloadable binary operators*** are:
```csharp
+   -   *   /   %   &   |   ^   <<   >>   ==   !=   >   <   >=   <=
```

Only the operators listed above can be overloaded. In particular, it is not possible to overload member access, method invocation, or the `=`, `&&`, `||`, `??`, `?:`, `=>`, `checked`, `unchecked`, `new`, `typeof`, `default`, `as`, and `is` operators.

When a binary operator is overloaded, the corresponding assignment operator, if any, is also implicitly overloaded. For example, an overload of operator `*` is also an overload of operator `*=`. This is described further in [Compound assignment](expressions.md#compound-assignment). Note that the assignment operator itself (`=`) cannot be overloaded. An assignment always performs a simple bit-wise copy of a value into a variable.

Cast operations, such as `(T)x`, are overloaded by providing user-defined conversions ([User-defined conversions](conversions.md#user-defined-conversions)).

Element access, such as `a[x]`, is not considered an overloadable operator. Instead, user-defined indexing is supported through indexers ([Indexers](classes.md#indexers)).

In expressions, operators are referenced using operator notation, and in declarations, operators are referenced using functional notation. The following table shows the relationship between operator and functional notations for unary and binary operators. In the first entry, *op* denotes any overloadable unary prefix operator. In the second entry, *op* denotes the unary postfix `++` and `--` operators. In the third entry, *op* denotes any overloadable binary operator.


| __Operator notation__ | __Functional notation__ |
|-----------------------|-------------------------|
| `op x`                | `operator op(x)`        | 
| `x op`                | `operator op(x)`        | 
| `x op y`              | `operator op(x,y)`      | 

User-defined operator declarations always require at least one of the parameters to be of the class or struct type that contains the operator declaration. Thus, it is not possible for a user-defined operator to have the same signature as a predefined operator.

User-defined operator declarations cannot modify the syntax, precedence, or associativity of an operator. For example, the `/` operator is always a binary operator, always has the precedence level specified in [Operator precedence and associativity](expressions.md#operator-precedence-and-associativity), and is always left-associative.

While it is possible for a user-defined operator to perform any computation it pleases, implementations that produce results other than those that are intuitively expected are strongly discouraged. For example, an implementation of `operator ==` should compare the two operands for equality and return an appropriate `bool` result.

The descriptions of individual operators in [Primary expressions](expressions.md#primary-expressions) through [Conditional logical operators](expressions.md#conditional-logical-operators) specify the predefined implementations of the operators and any additional rules that apply to each operator. The descriptions make use of the terms ***unary operator overload resolution***, ***binary operator overload resolution***, and ***numeric promotion***, definitions of which are found in the following sections.

### Unary operator overload resolution

An operation of the form `op x` or `x op`, where `op` is an overloadable unary operator, and `x` is an expression of type `X`, is processed as follows:

*  The set of candidate user-defined operators provided by `X` for the operation `operator op(x)` is determined using the rules of [Candidate user-defined operators](expressions.md#candidate-user-defined-operators).
*  If the set of candidate user-defined operators is not empty, then this becomes the set of candidate operators for the operation. Otherwise, the predefined unary `operator op` implementations, including their lifted forms, become the set of candidate operators for the operation. The predefined implementations of a given operator are specified in the description of the operator ([Primary expressions](expressions.md#primary-expressions) and [Unary operators](expressions.md#unary-operators)).
*  The overload resolution rules of [Overload resolution](expressions.md#overload-resolution) are applied to the set of candidate operators to select the best operator with respect to the argument list `(x)`, and this operator becomes the result of the overload resolution process. If overload resolution fails to select a single best operator, a binding-time error occurs.

### Binary operator overload resolution

An operation of the form `x op y`, where `op` is an overloadable binary operator, `x` is an expression of type `X`, and `y` is an expression of type `Y`, is processed as follows:

*  The set of candidate user-defined operators provided by `X` and `Y` for the operation `operator op(x,y)` is determined. The set consists of the union of the candidate operators provided by `X` and the candidate operators provided by `Y`, each determined using the rules of [Candidate user-defined operators](expressions.md#candidate-user-defined-operators). If `X` and `Y` are the same type, or if `X` and `Y` are derived from a common base type, then shared candidate operators only occur in the combined set once.
*  If the set of candidate user-defined operators is not empty, then this becomes the set of candidate operators for the operation. Otherwise, the predefined binary `operator op` implementations, including their lifted forms,  become the set of candidate operators for the operation. The predefined implementations of a given operator are specified in the description of the operator ([Arithmetic operators](expressions.md#arithmetic-operators) through [Conditional logical operators](expressions.md#conditional-logical-operators)). For predefined enum and delegate operators, the only operators considered are those defined by an enum or delegate type that is the binding-time type of one of the operands.
*  The overload resolution rules of [Overload resolution](expressions.md#overload-resolution) are applied to the set of candidate operators to select the best operator with respect to the argument list `(x,y)`, and this operator becomes the result of the overload resolution process. If overload resolution fails to select a single best operator, a binding-time error occurs.

### Candidate user-defined operators

Given a type `T` and an operation `operator op(A)`, where `op` is an overloadable operator and `A` is an argument list, the set of candidate user-defined operators provided by `T` for `operator op(A)` is determined as follows:

*  Determine the type `T0`. If `T` is a nullable type, `T0` is its underlying type, otherwise `T0` is equal to `T`.
*  For all `operator op` declarations in `T0` and all lifted forms of such operators, if at least one operator is applicable ([Applicable function member](expressions.md#applicable-function-member)) with respect to the argument list `A`, then the set of candidate operators consists of all such applicable operators in `T0`.
*  Otherwise, if `T0` is `object`, the set of candidate operators is empty.
*  Otherwise, the set of candidate operators provided by `T0` is the set of candidate operators provided by the direct base class of `T0`, or the effective base class of `T0` if `T0` is a type parameter.

### Numeric promotions

Numeric promotion consists of automatically performing certain implicit conversions of the operands of the predefined unary and binary numeric operators. Numeric promotion is not a distinct mechanism, but rather an effect of applying overload resolution to the predefined operators. Numeric promotion specifically does not affect evaluation of user-defined operators, although user-defined operators can be implemented to exhibit similar effects.

As an example of numeric promotion, consider the predefined implementations of the binary `*` operator:

```csharp
int operator *(int x, int y);
uint operator *(uint x, uint y);
long operator *(long x, long y);
ulong operator *(ulong x, ulong y);
float operator *(float x, float y);
double operator *(double x, double y);
decimal operator *(decimal x, decimal y);
```

When overload resolution rules ([Overload resolution](expressions.md#overload-resolution)) are applied to this set of operators, the effect is to select the first of the operators for which implicit conversions exist from the operand types. For example, for the operation `b * s`, where `b` is a `byte` and `s` is a `short`, overload resolution selects `operator *(int,int)` as the best operator. Thus, the effect is that `b` and `s` are converted to `int`, and the type of the result is `int`. Likewise, for the operation `i * d`, where `i` is an `int` and `d` is a `double`, overload resolution selects `operator *(double,double)` as the best operator.

#### Unary numeric promotions

Unary numeric promotion occurs for the operands of the predefined `+`, `-`, and `~` unary operators. Unary numeric promotion simply consists of converting operands of type `sbyte`, `byte`, `short`, `ushort`, or `char` to type `int`. Additionally, for the unary `-` operator, unary numeric promotion converts operands of type `uint` to type `long`.

#### Binary numeric promotions

Binary numeric promotion occurs for the operands of the predefined `+`, `-`, `*`, `/`, `%`, `&`, `|`, `^`, `==`, `!=`, `>`, `<`, `>=`, and `<=` binary operators. Binary numeric promotion implicitly converts both operands to a common type which, in case of the non-relational operators, also becomes the result type of the operation. Binary numeric promotion consists of applying the following rules, in the order they appear here:

*  If either operand is of type `decimal`, the other operand is converted to type `decimal`, or a binding-time error occurs if the other operand is of type `float` or `double`.
*  Otherwise, if either operand is of type `double`, the other operand is converted to type `double`.
*  Otherwise, if either operand is of type `float`, the other operand is converted to type `float`.
*  Otherwise, if either operand is of type `ulong`, the other operand is converted to type `ulong`, or a binding-time error occurs if the other operand is of type `sbyte`, `short`, `int`, or `long`.
*  Otherwise, if either operand is of type `long`, the other operand is converted to type `long`.
*  Otherwise, if either operand is of type `uint` and the other operand is of type `sbyte`, `short`, or `int`, both operands are converted to type `long`.
*  Otherwise, if either operand is of type `uint`, the other operand is converted to type `uint`.
*  Otherwise, both operands are converted to type `int`.

Note that the first rule disallows any operations that mix the `decimal` type with the `double` and `float` types. The rule follows from the fact that there are no implicit conversions between the `decimal` type and the `double` and `float` types.

Also note that it is not possible for an operand to be of type `ulong` when the other operand is of a signed integral type. The reason is that no integral type exists that can represent the full range of `ulong` as well as the signed integral types.

In both of the above cases, a cast expression can be used to explicitly convert one operand to a type that is compatible with the other operand.

In the example
```csharp
decimal AddPercent(decimal x, double percent) {
    return x * (1.0 + percent / 100.0);
}
```
a binding-time error occurs because a `decimal` cannot be multiplied by a `double`. The error is resolved by explicitly converting the second operand to `decimal`, as follows:

```csharp
decimal AddPercent(decimal x, double percent) {
    return x * (decimal)(1.0 + percent / 100.0);
}
```

### Lifted operators

***Lifted operators*** permit predefined and user-defined operators that operate on non-nullable value types to also be used with nullable forms of those types. Lifted operators are constructed from predefined and user-defined operators that meet certain requirements, as described in the following:

*   For the unary operators

    ```csharp
    +  ++  -  --  !  ~
    ```

    a lifted form of an operator exists if the operand and result types are both non-nullable value types. The lifted form is constructed by adding a single `?` modifier to the operand and result types. The lifted operator produces a null value if the operand is null. Otherwise, the lifted operator unwraps the operand, applies the underlying operator, and wraps the result.

*   For the binary operators

    ```csharp
    +  -  *  /  %  &  |  ^  <<  >>
    ```

    a lifted form of an operator exists if the operand and result types are all non-nullable value types. The lifted form is constructed by adding a single `?` modifier to each operand and result type. The lifted operator produces a null value if one or both operands are null (an exception being the `&` and `|` operators of the `bool?` type, as described in [Boolean logical operators](expressions.md#boolean-logical-operators)). Otherwise, the lifted operator unwraps the operands, applies the underlying operator, and wraps the result.

*   For the equality operators

    ```csharp
    ==  !=
    ```

    a lifted form of an operator exists if the operand types are both non-nullable value types and if the result type is `bool`. The lifted form is constructed by adding a single `?` modifier to each operand type. The lifted operator considers two null values equal, and a null value unequal to any non-null value. If both operands are non-null, the lifted operator unwraps the operands and applies the underlying operator to produce the `bool` result.

*   For the relational operators

    ```csharp
    <  >  <=  >=
    ```

    a lifted form of an operator exists if the operand types are both non-nullable value types and if the result type is `bool`. The lifted form is constructed by adding a single `?` modifier to each operand type. The lifted operator produces the value `false` if one or both operands are null. Otherwise, the lifted operator unwraps the operands and applies the underlying operator to produce the `bool` result.

## Member lookup

A member lookup is the process whereby the meaning of a name in the context of a type is determined. A member lookup can occur as part of evaluating a *simple_name* ([Simple names](expressions.md#simple-names)) or a *member_access* ([Member access](expressions.md#member-access)) in an expression. If the *simple_name* or *member_access* occurs as the *primary_expression* of an *invocation_expression* ([Method invocations](expressions.md#method-invocations)), the member is said to be invoked.

If a member is a method or event, or if it is a constant, field or property of either a delegate type ([Delegates](delegates.md)) or the type `dynamic` ([The dynamic type](types.md#the-dynamic-type)), then the member is said to be *invocable*.

Member lookup considers not only the name of a member but also the number of type parameters the member has and whether the member is accessible. For the purposes of member lookup, generic methods and nested generic types have the number of type parameters indicated in their respective declarations and all other members have zero type parameters.

A member lookup of a name `N` with `K` type parameters in a type `T` is processed as follows:

*  First, a set of accessible members named `N` is determined:
    * If `T` is a type parameter, then the set is the union of the sets of accessible members named `N` in each of the types specified as a primary constraint or secondary constraint ([Type parameter constraints](classes.md#type-parameter-constraints)) for `T`, along with the set of accessible members named `N` in `object`.
    * Otherwise, the set consists of all accessible ([Member access](basic-concepts.md#member-access)) members named `N` in `T`, including inherited members and the accessible members named `N` in `object`. If `T` is a constructed type, the set of members is obtained by substituting type arguments as described in [Members of constructed types](classes.md#members-of-constructed-types). Members that include an `override` modifier are excluded from the set.
*  Next, if `K` is zero, all nested types whose declarations include type parameters are removed. If `K` is not zero, all members with a different number of type parameters are removed. Note that when `K` is zero, methods having type parameters are not removed, since the type inference process ([Type inference](expressions.md#type-inference)) might be able to infer the type arguments.
*  Next, if the member is *invoked*, all non-*invocable* members are removed from the set.
*  Next, members that are hidden by other members are removed from the set. For every member `S.M` in the set, where `S` is the type in which the member `M` is declared, the following rules are applied:
    * If `M` is a constant, field, property, event, or enumeration member, then all members declared in a base type of `S` are removed from the set.
    * If `M` is a type declaration, then all non-types declared in a base type of `S` are removed from the set, and all type declarations with the same number of type parameters as `M` declared in a base type of `S` are removed from the set.
    * If `M` is a method, then all non-method members declared in a base type of `S` are removed from the set.
*  Next, interface members that are hidden by class members are removed from the set. This step only has an effect if `T` is a type parameter and `T` has both an effective base class other than `object` and a non-empty effective interface set ([Type parameter constraints](classes.md#type-parameter-constraints)). For every member `S.M` in the set, where `S` is the type in which the member `M` is declared, the following rules are applied if `S` is a class declaration other than `object`:
    * If `M` is a constant, field, property, event, enumeration member, or type declaration, then all members declared in an interface declaration are removed from the set.
    * If `M` is a method, then all non-method members declared in an interface declaration are removed from the set, and all methods with the same signature as `M` declared in an interface declaration are removed from the set.
*  Finally, having removed hidden members, the result of the lookup is determined:
    * If the set consists of a single member that is not a method, then this member is the result of the lookup.
    * Otherwise, if the set contains only methods, then this group of methods is the result of the lookup.
    * Otherwise, the lookup is ambiguous, and a binding-time error occurs.

For member lookups in types other than type parameters and interfaces, and member lookups in interfaces that are strictly single-inheritance (each interface in the inheritance chain has exactly zero or one direct base interface), the effect of the lookup rules is simply that derived members hide base members with the same name or signature. Such single-inheritance lookups are never ambiguous. The ambiguities that can possibly arise from member lookups in multiple-inheritance interfaces are described in [Interface member access](interfaces.md#interface-member-access).

### Base types

For purposes of member lookup, a type `T` is considered to have the following base types:

*  If `T` is `object`, then `T` has no base type.
*  If `T` is an *enum_type*, the base types of `T` are the class types `System.Enum`, `System.ValueType`, and `object`.
*  If `T` is a *struct_type*, the base types of `T` are the class types `System.ValueType` and `object`.
*  If `T` is a *class_type*, the base types of `T` are the base classes of `T`, including the class type `object`.
*  If `T` is an *interface_type*, the base types of `T` are the base interfaces of `T` and the class type `object`.
*  If `T` is an *array_type*, the base types of `T` are the class types `System.Array` and `object`.
*  If `T` is a *delegate_type*, the base types of `T` are the class types `System.Delegate` and `object`.

## Function members

Function members are members that contain executable statements. Function members are always members of types and cannot be members of namespaces. C# defines the following categories of function members:

*  Methods
*  Properties
*  Events
*  Indexers
*  User-defined operators
*  Instance constructors
*  Static constructors
*  Destructors

Except for destructors and static constructors (which cannot be invoked explicitly), the statements contained in function members are executed through function member invocations. The actual syntax for writing a function member invocation depends on the particular function member category.

The argument list ([Argument lists](expressions.md#argument-lists)) of a function member invocation provides actual values or variable references for the parameters of the function member.

Invocations of generic methods may employ type inference to determine the set of type arguments to pass to the method. This process is described in [Type inference](expressions.md#type-inference).

Invocations of methods, indexers, operators and instance constructors employ overload resolution to determine which of a candidate set of function members to invoke. This process is described in [Overload resolution](expressions.md#overload-resolution).

Once a particular function member has been identified at binding-time, possibly through overload resolution, the actual run-time process of invoking the function member is described in [Compile-time checking of dynamic overload resolution](expressions.md#compile-time-checking-of-dynamic-overload-resolution).

The following table summarizes the processing that takes place in constructs involving the six categories of function members that can be explicitly invoked. In the table, `e`, `x`, `y`, and `value` indicate expressions classified as variables or values, `T` indicates an expression classified as a type, `F` is the simple name of a method, and `P` is the simple name of a property.


| __Construct__     | __Example__    | __Description__ |
|-------------------|----------------|-----------------|
| Method invocation | `F(x,y)`       | Overload resolution is applied to select the best method `F` in the containing class or struct. The method is invoked with the argument list `(x,y)`. If the method is not `static`, the instance expression is `this`. | 
|                   | `T.F(x,y)`     | Overload resolution is applied to select the best method `F` in the class or struct `T`. A binding-time error occurs if the method is not `static`. The method is invoked with the argument list `(x,y)`. | 
|                   | `e.F(x,y)`     | Overload resolution is applied to select the best method F in the class, struct, or interface given by the type of `e`. A binding-time error occurs if the method is `static`. The method is invoked with the instance expression `e` and the argument list `(x,y)`. | 
| Property access   | `P`            | The `get` accessor of the property `P` in the containing class or struct is invoked. A compile-time error occurs if `P` is write-only. If `P` is not `static`, the instance expression is `this`. | 
|                   | `P = value`    | The `set` accessor of the property `P` in the containing class or struct is invoked with the argument list `(value)`. A compile-time error occurs if `P` is read-only. If `P` is not `static`, the instance expression is `this`. | 
|                   | `T.P`          | The `get` accessor of the property `P` in the class or struct `T` is invoked. A compile-time error occurs if `P` is not `static` or if `P` is write-only. | 
|                   | `T.P = value`  | The `set` accessor of the property `P` in the class or struct `T` is invoked with the argument list `(value)`. A compile-time error occurs if `P` is not `static` or if `P` is read-only. | 
|                   | `e.P`          | The `get` accessor of the property `P` in the class, struct, or interface given by the type of `e` is invoked with the instance expression `e`. A binding-time error occurs if `P` is `static` or if `P` is write-only. | 
|                   | `e.P = value`  | The `set` accessor of the property `P` in the class, struct, or interface given by the type of `e` is invoked with the instance expression `e` and the argument list `(value)`. A binding-time error occurs if `P` is `static` or if `P` is read-only. | 
| Event access      | `E += value`   | The `add` accessor of the event `E` in the containing class or struct is invoked. If `E` is not static, the instance expression is `this`. | 
|                   | `E -= value`   | The `remove` accessor of the event `E` in the containing class or struct is invoked. If `E` is not static, the instance expression is `this`. | 
|                   | `T.E += value` | The `add` accessor of the event `E` in the class or struct `T` is invoked. A binding-time error occurs if `E` is not static. | 
|                   | `T.E -= value` | The `remove` accessor of the event `E` in the class or struct `T` is invoked. A binding-time error occurs if `E` is not static. | 
|                   | `e.E += value` | The `add` accessor of the event `E` in the class, struct, or interface given by the type of `e` is invoked with the instance expression `e`. A binding-time error occurs if `E` is static. | 
|                   | `e.E -= value` | The `remove` accessor of the event `E` in the class, struct, or interface given by the type of `e` is invoked with the instance expression `e`. A binding-time error occurs if `E` is static. | 
| Indexer access    | `e[x,y]`       | Overload resolution is applied to select the best indexer in the class, struct, or interface given by the type of e. The `get` accessor of the indexer is invoked with the instance expression `e` and the argument list `(x,y)`. A binding-time error occurs if the indexer is write-only. | 
|                   | `e[x,y] = value` | Overload resolution is applied to select the best indexer in the class, struct, or interface given by the type of `e`. The `set` accessor of the indexer is invoked with the instance expression `e` and the argument list `(x,y,value)`. A binding-time error occurs if the indexer is read-only. | 
| Operator invocation | `-x`         | Overload resolution is applied to select the best unary operator in the class or struct given by the type of `x`. The selected operator is invoked with the argument list `(x)`. | 
|                     | `x + y`      | Overload resolution is applied to select the best binary operator in the classes or structs given by the types of `x` and `y`. The selected operator is invoked with the argument list `(x,y)`. | 
| Instance constructor invocation | `new T(x,y)` | Overload resolution is applied to select the best instance constructor in the class or struct `T`. The instance constructor is invoked with the argument list `(x,y)`. | 

### Argument lists

Every function member and delegate invocation includes an argument list which provides actual values or variable references for the parameters of the function member. The syntax for specifying the argument list of a function member invocation depends on the function member category:

*  For instance constructors, methods, indexers and delegates, the arguments are specified as an *argument_list*, as described below. For indexers, when invoking the `set` accessor, the argument list additionally includes the expression specified as the right operand of the assignment operator.
*  For properties, the argument list is empty when invoking the `get` accessor, and consists of the expression specified as the right operand of the assignment operator when invoking the `set` accessor.
*  For events, the argument list consists of the expression specified as the right operand of the `+=` or `-=` operator.
*  For user-defined operators, the argument list consists of the single operand of the unary operator or the two operands of the binary operator.

The arguments of properties ([Properties](classes.md#properties)), events ([Events](classes.md#events)), and user-defined operators ([Operators](classes.md#operators)) are always passed as value parameters ([Value parameters](classes.md#value-parameters)). The arguments of indexers ([Indexers](classes.md#indexers)) are always passed as value parameters ([Value parameters](classes.md#value-parameters)) or parameter arrays ([Parameter arrays](classes.md#parameter-arrays)). Reference and output parameters are not supported for these categories of function members.

The arguments of an instance constructor, method, indexer or delegate invocation are specified as an *argument_list*:

```antlr
argument_list
    : argument (',' argument)*
    ;

argument
    : argument_name? argument_value
    ;

argument_name
    : identifier ':'
    ;

argument_value
    : expression
    | 'ref' variable_reference
    | 'out' variable_reference
    ;
```

An *argument_list* consists of one or more *argument*s, separated by commas. Each argument consists of an optional  *argument_name* followed by an *argument_value*. An *argument* with an *argument_name* is referred to as a ***named argument***, whereas an *argument* without an *argument_name* is a ***positional argument***. It is an error for a positional argument to appear after a named argument in an *argument_list*.

The *argument_value* can take one of the following forms:

*  An *expression*, indicating that the argument is passed as a value parameter ([Value parameters](classes.md#value-parameters)).
*  The keyword `ref` followed by a *variable_reference* ([Variable references](variables.md#variable-references)), indicating that the argument is passed as a reference parameter ([Reference parameters](classes.md#reference-parameters)). A variable must be definitely assigned ([Definite assignment](variables.md#definite-assignment)) before it can be passed as a reference parameter. The keyword `out` followed by a *variable_reference* ([Variable references](variables.md#variable-references)), indicating that the argument is passed as an output parameter ([Output parameters](classes.md#output-parameters)). A variable is considered definitely assigned ([Definite assignment](variables.md#definite-assignment)) following a function member invocation in which the variable is passed as an output parameter.

> __Expression Tree Conversion Translation Steps__
>
> For purposes of conversion to an expression tree, an *argument_list*
>
> ```antlr
> argument_list
>     : argument (',' argument)*
>     ;
> ```
>
> is preprocessed into
>
> ```antlr
> argument_list
>     : no_params_argument_list
>     | raw_argument_list? params_argument_list
>     ;
>
> no_params_argument_list
>     : raw_argument_list
>     ;
>
> params_argument_list
>     : 'new' array_type params_array_initializer
>     ;
>
> params_array_initializer
>     : '{' params_array_argument_list? '}'
>     ;
>
> params_array_argument_list
>     : argument_expression (',' argument_expression)*
>     ;
>
> raw_argument_list
>     : argument (',' argument)*
>     ;
>
> argument
>     : argument_name? argument_value
>     ;
>
> argument_value
>     : argument_expression
>     | 'ref' variable_reference
>     | 'out' variable_reference
>     ;
>
> argument_expression
>     : converted_argument_expression
>     | expression
>     ;
>
> converted_argument_expression
>     : '(' type ')' '(' expression ')'
>     ;
> ```
>
> by inspecting the bound parent of the argument list for:
>
> * translation of a [parameter array](classes.md#parameter-arrays) binding using the expanded form (see [Applicable function member](#applicable-function-member) and [
Run-time evaluation of argument lists](#run-time-evaluation-of-argument-lists)) to *params_argument_list*, where *array_type* is the array type of the corresponding `params` parameter (see [Corresponding parameters](#corresponding-parameters)), and,
> * translation of any implicit conversion used for binding an argument to a parameter to *converted_argument_expression* where *type* is the conversion target type, following the rules in [Applicable function member](#applicable-function-member).
>
> For example, given
>
> ```csharp
> class C
> {
>     public static void F(int x, params long[] ys) {}
> }
> ```
>
> an argument list `i1, i2, i3` with expressions `i1`, `i2`, and `i3` of type `int` used in a method invocation expression `C.F(i1, i2, i3)` is preprocessed to `i1, new long[] { (long)i2, (long)i3 }`.
>
> This pre-processing step effectively makes implicit conversions and the use of `params` explicit. Productions `params_argument_list` and `converted_argument_expression` are classified as ***compiler-generated*** and their translation to generalized expression trees will include a `Q.Flags.CompilerGenerated` flag.
>
> After applying this pre-processing step, which may introduce an [array creation expression](#array-creation-expressions) and zero or more [cast expressions](#cast-expression), translation to expression trees proceeds as follows.
>
> For all expression trees, an *argument_value* is translated by translating the *expression* or *variable_reference*.
>
> An *argument*
>
> ```antlr
> argument_name? argument_value
> ```
>
> for purposes of conversion to a query expression tree, is translated by translating *argument_value*, unless an *argument_name* is specified, in which case an error is reported.
>
> For purposes of conversion to a generalized expression tree, it is translated into
>
> ```csharp
> argument_value_expr
> ```
>
> if the containing *argument_list* is bound to an [array access](#array-access), or translated into
>
> ```csharp
> Q.ParameterBinding(flags, method, index, argument_value_expr)
> ```
>
> if the containing *argument_list* is bound to a method, constructor, delegate, or indexer, where
>
> * `flags` is the bitwise `|` combination of any of the following:
>   * `Q.ParameterBindingFlags.Named` if *argument_name* is specified,
>   * `Q.ParameterBindingFlags.Ref` if *argument_value* is `ref` *variable_reference*,
>   * `Q.ParameterBindingFlags.Out` if *argument_value* is `out` *variable_reference*,
>   * `Q.ParameterBindingFlags.Params` if the argument is assigned to a `params` parameter that was constructed from the expanded form,
>   * or `default(Q.ParameterBindingFlags)` if no flags apply;
> * `method` is an expression of type `MethodBase` representing the method to which the argument list is bound:
>    * for [method invocations](#method-invocations) and [extension method invocations](#extension-method-invocations), `method` is the bound method,
>    * for [delegate invocations](#delegate-invocations), `method` corresponds to the `Invoke` instance method defined on the target delegate type,
>    * for [indexer access](#indexer-access), `method` corresponds to the *get accessor* on the bound indexer, and,
>    * for [object creation](#object-creation-expressions), `method` corresponds to the bound constructor;
> * `index` is the zero-based index of the parameter corresponding to the argument (see [Corresponding parameters](#corresponding-parameters));
> * `argument_value_expr` is the result of translating *argument_value*,
>
> or translated into
>
> ```csharp
> Q.ParameterBinding(argument_info, argument_value_expr)
> ```
>
> if the containing *argument_list* is used for a dynamic operation (see [Dynamic binding](#dynamic-binding)), where
>
> * `argument_info` is an expression of type `Microsoft.CSharp.RuntimeBinder.CSharpArgumentInfo`;
> * `argument_value_expr` is the result of translating *argument_value*.
>
> Finally, a *raw_argument_list*, for purposes of conversion to a query expression tree is translated into
>
> ```csharp
> new System.Linq.Expressions.Expression[] { argument_expr (',' argument_expr)* }
> ```
>
> or, for purposes of conversion to a generalized expression tree, is translated into a syntactic fragment
>
> ```csharp
> Q.ArgumentList(argument_expr (',' argument_expr))
> ```
>
> where `argument_expr` is the result of translating the corresponding `argument`.
>
> Note that the translation for a generalized expression tree produces an invocation to `Q.ArgumentList`. In particular, the translation does not produce an array creation expression. This serves two purposes:
>
> * The use of an implicitly typed array creation expression would require [Finding the best common type of a set of expressions](expressions.md#finding-the-best-common-type-of-a-set-of-expressions) rules to be applied immediately to the resulting expression representation of an argument list.
> * Representation of argument lists with few arguments cannot be represented in a more efficient manner than an array. For instance, a method invocation expression involving zero to four arguments could bind to optimized overloads of `Q.ArgumentList`, while invocations with more arguments could bind to a `params` array overload.
>
> ***TODO***
>
> * Check implicit conversion behavior on arguments, e.g. do we need to filter out identity conversions, reference conversions, etc.?
> * Create a specification for `argument_info` used in dynamic operations.
> * Review an alternative where generalizd expression trees produce a syntactic fragment `(',' argument_expr)*` that's embedded in the parent node's expression tree representation. That is, a world where argument lists are not first-class nodes (as was the case for query expression trees). Note that many nodes can benefit from a first-class representation of argument lists.


#### Corresponding parameters

For each argument in an argument list there has to be a corresponding parameter in the function member or delegate being invoked.

The parameter list used in the following is determined as follows:

*  For virtual methods and indexers defined in classes, the parameter list is picked from the most specific declaration or override of the function member, starting with the static type of the receiver, and searching through its base classes.
*  For interface methods and indexers, the parameter list is picked form the most specific definition of the member, starting with the interface type and searching through the base interfaces. If no unique parameter list is found, a parameter list with inaccessible names and no optional parameters is constructed, so that invocations cannot use named parameters or omit optional arguments.
*  For partial methods, the parameter list of the defining partial method declaration is used.
*  For all other function members and delegates there is only a single parameter list, which is the one used.

The position of an argument or parameter is defined as the number of arguments or parameters preceding it in the argument list or parameter list.

The corresponding parameters for function member arguments are established as follows:

*  Arguments in the *argument_list* of instance constructors, methods, indexers and delegates:
    * A positional argument where a fixed parameter occurs at the same position in the parameter list corresponds to that parameter.
    * A positional argument of a function member with a parameter array invoked in its normal form corresponds to the parameter  array, which must occur at the same position in the parameter list.
    * A positional argument of a function member with a parameter array invoked in its expanded form, where no fixed parameter occurs at the same position in the parameter list, corresponds to an element in the parameter array.
    * A named argument corresponds to the parameter of the same name in the parameter list.
    * For indexers, when invoking the `set` accessor, the expression specified as the right operand of the assignment operator corresponds to the implicit `value` parameter of the `set` accessor declaration.
*  For properties, when invoking the `get` accessor there are no arguments. When invoking the `set` accessor, the expression specified as the right operand of the assignment operator corresponds to the implicit `value` parameter of the `set` accessor declaration.
*  For user-defined unary operators (including conversions), the single operand corresponds to the single parameter of the operator declaration.
*  For user-defined binary operators, the left operand corresponds to the first parameter, and the right operand corresponds to the second parameter of the operator declaration.

#### Run-time evaluation of argument lists

During the run-time processing of a function member invocation ([Compile-time checking of dynamic overload resolution](expressions.md#compile-time-checking-of-dynamic-overload-resolution)), the expressions or variable references of an argument list are evaluated in order, from left to right, as follows:

*  For a value parameter, the argument expression is evaluated and an implicit conversion ([Implicit conversions](conversions.md#implicit-conversions)) to the corresponding parameter type is performed. The resulting value becomes the initial value of the value parameter in the function member invocation.
*  For a reference or output parameter, the variable reference is evaluated and the resulting storage location becomes the storage location represented by the parameter in the function member invocation. If the variable reference given as a reference or output parameter is an array element of a *reference_type*, a run-time check is performed to ensure that the element type of the array is identical to the type of the parameter. If this check fails, a `System.ArrayTypeMismatchException` is thrown.

Methods, indexers, and instance constructors may declare their right-most parameter to be a parameter array ([Parameter arrays](classes.md#parameter-arrays)). Such function members are invoked either in their normal form or in their expanded form depending on which is applicable ([Applicable function member](expressions.md#applicable-function-member)):

*  When a function member with a parameter array is invoked in its normal form, the argument given for the parameter array must be a single expression that is implicitly convertible ([Implicit conversions](conversions.md#implicit-conversions)) to the parameter array type. In this case, the parameter array acts precisely like a value parameter.
*  When a function member with a parameter array is invoked in its expanded form, the invocation must specify zero or more positional arguments for the parameter array, where each argument is an expression that is implicitly convertible ([Implicit conversions](conversions.md#implicit-conversions)) to the element type of the parameter array. In this case, the invocation creates an instance of the parameter array type with a length corresponding to the number of arguments, initializes the elements of the array instance with the given argument values, and uses the newly created array instance as the actual argument.

The expressions of an argument list are always evaluated in the order they are written. Thus, the example
```csharp
class Test
{
    static void F(int x, int y = -1, int z = -2) {
        System.Console.WriteLine("x = {0}, y = {1}, z = {2}", x, y, z);
    }

    static void Main() {
        int i = 0;
        F(i++, i++, i++);
        F(z: i++, x: i++);
    }
}
```
produces the output
```
x = 0, y = 1, z = 2
x = 4, y = -1, z = 3
```

The array co-variance rules ([Array covariance](arrays.md#array-covariance)) permit a value of an array type `A[]` to be a reference to an instance of an array type `B[]`, provided an implicit reference conversion exists from `B` to `A`. Because of these rules, when an array element of a *reference_type* is passed as a reference or output parameter, a run-time check is required to ensure that the actual element type of the array is identical to that of the parameter. In the example
```csharp
class Test
{
    static void F(ref object x) {...}

    static void Main() {
        object[] a = new object[10];
        object[] b = new string[10];
        F(ref a[0]);        // Ok
        F(ref b[1]);        // ArrayTypeMismatchException
    }
}
```
the second invocation of `F` causes a `System.ArrayTypeMismatchException` to be thrown because the actual element type of `b` is `string` and not `object`.

When a function member with a parameter array is invoked in its expanded form, the invocation is processed exactly as if an array creation expression with an array initializer ([Array creation expressions](expressions.md#array-creation-expressions)) was inserted around the expanded parameters. For example, given the declaration
```csharp
void F(int x, int y, params object[] args);
```
the following invocations of the expanded form of the method
```csharp
F(10, 20);
F(10, 20, 30, 40);
F(10, 20, 1, "hello", 3.0);
```
correspond exactly to
```csharp
F(10, 20, new object[] {});
F(10, 20, new object[] {30, 40});
F(10, 20, new object[] {1, "hello", 3.0});
```

In particular, note that an empty array is created when there are zero arguments given for the parameter array.

When arguments are omitted from a function member with corresponding optional parameters, the default arguments of the function member declaration are implicitly passed. Because these are always constant, their evaluation will not impact the evaluation order of the remaining arguments.

### Type inference

When a generic method is called without specifying type arguments, a ***type inference*** process attempts to infer type arguments for the call. The presence of type inference allows a more convenient syntax to be used for calling a generic method, and allows the programmer to avoid specifying redundant type information. For example, given the method declaration:
```csharp
class Chooser
{
    static Random rand = new Random();

    public static T Choose<T>(T first, T second) {
        return (rand.Next(2) == 0)? first: second;
    }
}
```
it is possible to invoke the `Choose` method without explicitly specifying a type argument:
```csharp
int i = Chooser.Choose(5, 213);                 // Calls Choose<int>

string s = Chooser.Choose("foo", "bar");        // Calls Choose<string>
```

Through type inference, the type arguments `int` and `string` are determined from the arguments to the method.

Type inference occurs as part of the binding-time processing of a method invocation ([Method invocations](expressions.md#method-invocations)) and takes place before the overload resolution step of the invocation. When a particular method group is specified in a method invocation, and no type arguments are specified as part of the method invocation, type inference is applied to each generic method in the method group. If type inference succeeds, then the inferred type arguments are used to determine the types of arguments for subsequent overload resolution. If overload resolution chooses a generic method as the one to invoke, then the inferred type arguments are used as the actual type arguments for the invocation. If type inference for a particular method fails, that method does not participate in overload resolution. The failure of type inference, in and of itself, does not cause a binding-time error. However, it often leads to a binding-time error when overload resolution then fails to find any applicable methods.

If the supplied number of arguments is different than the number of parameters in the method, then inference immediately fails. Otherwise, assume that the generic method has the following signature:
```csharp
Tr M<X1,...,Xn>(T1 x1, ..., Tm xm)
```

With a method call of the form `M(E1...Em)` the task of type inference is to find unique type arguments `S1...Sn` for each of the type parameters `X1...Xn` so that the call `M<S1...Sn>(E1...Em)` becomes valid.

During the process of inference each type parameter `Xi` is either *fixed* to a particular type `Si` or *unfixed* with an associated set of *bounds*. Each of the bounds is some type `T`. Initially each type variable `Xi` is unfixed with an empty set of bounds.

Type inference takes place in phases. Each phase will try to infer type arguments for more type variables based on the findings of the previous phase. The first phase makes some initial inferences of bounds, whereas the second phase fixes type variables to specific types and infers further bounds. The second phase may have to be repeated a number of times.

*Note:* Type inference takes place not only when a generic method is called. Type inference for conversion of method groups is described in [Type inference for conversion of method groups](expressions.md#type-inference-for-conversion-of-method-groups) and finding the best common type of a set of expressions is described in [Finding the best common type of a set of expressions](expressions.md#finding-the-best-common-type-of-a-set-of-expressions).

#### The first phase

For each of the method arguments `Ei`:

*   If `Ei` is an anonymous function, an *explicit parameter type inference* ([Explicit parameter type inferences](expressions.md#explicit-parameter-type-inferences)) is made from `Ei` to `Ti`
*   Otherwise, if `Ei` has a type `U` and `xi` is a value parameter then a *lower-bound inference* is made *from* `U` *to* `Ti`.
*   Otherwise, if `Ei` has a type `U` and `xi` is a `ref` or `out` parameter then an *exact inference* is made *from* `U` *to* `Ti`.
*   Otherwise, no inference is made for this argument.


#### The second phase

The second phase proceeds as follows:

*   All *unfixed* type variables `Xi` which do not *depend on* ([Dependence](expressions.md#dependence)) any `Xj` are fixed ([Fixing](expressions.md#fixing)).
*   If no such type variables exist, all *unfixed* type variables `Xi` are *fixed* for which all of the following hold:
    *   There is at least one type variable `Xj` that depends on `Xi`
    *   `Xi` has a non-empty set of bounds
*   If no such type variables exist and there are still *unfixed* type variables, type inference fails.
*   Otherwise, if no further *unfixed* type variables exist, type inference succeeds.
*   Otherwise, for all arguments `Ei` with corresponding parameter type `Ti` where the *output types* ([Output types](expressions.md#output-types)) contain *unfixed* type variables `Xj` but the *input types* ([Input types](expressions.md#input-types)) do not, an *output type inference* ([Output type inferences](expressions.md#output-type-inferences)) is made *from* `Ei` *to* `Ti`. Then the second phase is repeated.

#### Input types

If `E` is a method group or implicitly typed anonymous function and `T` is a delegate type or expression tree type then all the parameter types of `T` are *input types* of `E` *with type* `T`.

####  Output types

If `E` is a method group or an anonymous function and `T` is a delegate type or expression tree type then the return type of `T` is an *output type of* `E` *with type* `T`.

#### Dependence

An *unfixed* type variable `Xi` *depends directly on* an unfixed type variable `Xj` if for some argument `Ek` with type `Tk` `Xj` occurs in an *input type* of `Ek` with type `Tk` and `Xi` occurs in an *output type* of `Ek` with type `Tk`.

`Xj` *depends on* `Xi` if `Xj` *depends directly on* `Xi` or if `Xi` *depends directly on* `Xk` and `Xk` *depends on* `Xj`. Thus "depends on" is the transitive but not reflexive closure of "depends directly on".

#### Output type inferences

An *output type inference* is made *from* an expression `E` *to* a type `T` in the following way:

*  If `E` is an anonymous function with inferred return type  `U` ([Inferred return type](expressions.md#inferred-return-type)) and `T` is a delegate type or expression tree type with return type `Tb`, then a *lower-bound inference* ([Lower-bound inferences](expressions.md#lower-bound-inferences)) is made *from* `U` *to* `Tb`.
*  Otherwise, if `E` is a method group and `T` is a delegate type or expression tree type with parameter types `T1...Tk` and return type `Tb`, and overload resolution of `E` with the types `T1...Tk` yields a single method with return type `U`, then a *lower-bound inference* is made *from* `U` *to* `Tb`.
*  Otherwise, if `E` is an expression with type `U`, then a *lower-bound inference* is made *from* `U` *to* `T`.
*  Otherwise, no inferences are made.

#### Explicit parameter type inferences

An *explicit parameter type inference* is made *from* an expression `E` *to* a type `T` in the following way:

*  If `E` is an explicitly typed anonymous function with parameter types `U1...Uk` and `T` is a delegate type or expression tree type with parameter types `V1...Vk` then for each `Ui` an *exact inference* ([Exact inferences](expressions.md#exact-inferences)) is made *from* `Ui` *to* the corresponding `Vi`.

#### Exact inferences

An *exact inference* *from* a type `U` *to* a type `V` is made as follows:

*  If `V` is one of the *unfixed* `Xi` then `U` is added to the set of exact bounds for `Xi`.

*  Otherwise, sets `V1...Vk` and `U1...Uk` are determined by checking if any of the following cases apply:

   *  `V` is an array type `V1[...]` and `U` is an array type `U1[...]`  of the same rank
   *  `V` is the type `V1?` and `U` is the type `U1?`
   *  `V` is a constructed type `C<V1...Vk>`and `U` is a constructed type `C<U1...Uk>`

   If any of these cases apply then an *exact inference* is made *from* each `Ui` *to* the corresponding `Vi`.

*  Otherwise no inferences are made.

#### Lower-bound inferences

A *lower-bound inference* *from* a type `U` *to* a type `V` is made as follows:

*  If `V` is one of the *unfixed* `Xi` then `U` is added to the set of lower bounds for `Xi`.
*  Otherwise, if `V` is the type `V1?`and `U` is the type `U1?` then a lower bound inference is made from `U1` to `V1`.
*  Otherwise, sets `U1...Uk` and `V1...Vk` are determined by checking if any of the following cases apply:
   *  `V` is an array type `V1[...]` and `U` is an array type `U1[...]` (or a type parameter whose effective base type is `U1[...]`) of the same rank
   *  `V` is one of `IEnumerable<V1>`, `ICollection<V1>` or `IList<V1>` and `U` is a one-dimensional array type `U1[]`(or a type parameter whose effective base type is `U1[]`)
   *  `V` is a constructed class, struct, interface or delegate type `C<V1...Vk>` and there is a unique type `C<U1...Uk>` such that `U` (or, if `U` is a type parameter, its effective base class or any member of its effective interface set) is identical to, inherits from (directly or indirectly), or implements (directly or indirectly) `C<U1...Uk>`.

      (The "uniqueness" restriction means that in the case interface `C<T> {} class U: C<X>, C<Y> {}`, then no inference is made when inferring from `U` to `C<T>` because `U1` could be `X` or `Y`.)

   If any of these cases apply then an inference is made *from* each `Ui` *to* the corresponding `Vi` as follows:

   *  If `Ui` is not known to be a reference type then an *exact inference* is made
   *  Otherwise, if `U` is an array type then a *lower-bound inference* is made
   *  Otherwise, if `V` is `C<V1...Vk>` then inference depends on the i-th type parameter of `C`:
      *  If it is covariant then a *lower-bound inference* is made.
      *  If it is contravariant then an *upper-bound inference* is made.
      *  If it is invariant then an *exact inference* is made.
*  Otherwise, no inferences are made.

#### Upper-bound inferences

An *upper-bound inference* *from* a type `U` *to* a type `V` is made as follows:

*  If `V` is one of the *unfixed* `Xi` then `U` is added to the set of upper bounds for `Xi`.
*  Otherwise, sets `V1...Vk` and `U1...Uk` are determined by checking if any of the following cases apply:
   *  `U` is an array type `U1[...]` and `V` is an array type `V1[...]` of the same rank
   *  `U` is one of `IEnumerable<Ue>`, `ICollection<Ue>` or `IList<Ue>` and `V` is a one-dimensional array type `Ve[]`
   *  `U` is the type `U1?` and `V` is the type `V1?`
   *  `U` is constructed class, struct, interface or delegate type `C<U1...Uk>` and `V` is a class, struct, interface or delegate type which is identical to, inherits from (directly or indirectly), or implements (directly or indirectly) a unique type `C<V1...Vk>`

      (The "uniqueness" restriction means that if we have `interface C<T>{} class V<Z>: C<X<Z>>, C<Y<Z>>{}`, then no inference is made when inferring from `C<U1>` to `V<Q>`. Inferences are not made from `U1` to either `X<Q>` or `Y<Q>`.)

   If any of these cases apply then an inference is made *from* each `Ui` *to* the corresponding `Vi` as follows:
   *  If  `Ui` is not known to be a reference type then an *exact inference* is made
   *  Otherwise, if `V` is an array type then an *upper-bound inference* is made
   *  Otherwise, if `U` is `C<U1...Uk>` then inference depends on the i-th type parameter of `C`:
      *  If it is covariant then an *upper-bound inference* is made.
      *  If it is contravariant then a *lower-bound inference* is made.
      *  If it is invariant then an *exact inference* is made.
*  Otherwise, no inferences are made.   

#### Fixing

An *unfixed* type variable `Xi` with a set of bounds is *fixed* as follows:

*  The set of *candidate types* `Uj` starts out as the set of all types in the set of bounds for `Xi`.
*  We then examine each bound for `Xi` in turn: For each exact bound `U` of `Xi` all types `Uj` which are not identical to `U` are removed from the candidate set. For each lower bound `U` of `Xi` all types `Uj` to which there is *not* an implicit conversion from `U` are removed from the candidate set. For each upper bound `U` of `Xi` all types `Uj` from which there is *not* an implicit conversion to `U` are removed from the candidate set.
*  If among the remaining candidate types `Uj` there is a unique type `V` from which there is an implicit conversion to all the other candidate types, then `Xi` is fixed to `V`.
*  Otherwise, type inference fails.

#### Inferred return type

The inferred return type of an anonymous function `F` is used during type inference and overload resolution. The inferred return type can only be determined for an anonymous function where all parameter types are known, either because they are explicitly given, provided through an anonymous function conversion or inferred during type inference on an enclosing generic method invocation.

The ***inferred result type*** is determined as follows:

*  If the body of `F` is an *expression* that has a type, then the inferred result type of `F` is the type of that expression.
*  If the body of `F` is a *block* and the set of expressions in the block's `return` statements has a best common type `T` ([Finding the best common type of a set of expressions](expressions.md#finding-the-best-common-type-of-a-set-of-expressions)), then the inferred result type of `F` is `T`.
*  Otherwise, a result type cannot be inferred for `F`.

The ***inferred return type*** is determined as follows:

*  If `F` is async and the body of `F` is either an expression classified as nothing ([Expression classifications](expressions.md#expression-classifications)), or a statement block where no return statements have expressions, the inferred return type is `System.Threading.Tasks.Task`
*  If `F` is async and has an inferred result type `T`, the inferred return type is `System.Threading.Tasks.Task<T>`.
*  If `F` is non-async and has an inferred result type `T`, the inferred return type is `T`.
*  Otherwise a return type cannot be inferred for `F`.

As an example of type inference involving anonymous functions, consider the `Select` extension method declared in the `System.Linq.Enumerable` class:
```csharp
namespace System.Linq
{
    public static class Enumerable
    {
        public static IEnumerable<TResult> Select<TSource,TResult>(
            this IEnumerable<TSource> source,
            Func<TSource,TResult> selector)
        {
            foreach (TSource element in source) yield return selector(element);
        }
    }
}
```

Assuming the `System.Linq` namespace was imported with a `using` clause, and given a class `Customer` with a `Name` property of type `string`, the `Select` method can be used to select the names of a list of customers:
```csharp
List<Customer> customers = GetCustomerList();
IEnumerable<string> names = customers.Select(c => c.Name);
```

The extension method invocation ([Extension method invocations](expressions.md#extension-method-invocations)) of `Select` is processed by rewriting the invocation to a static method invocation:
```csharp
IEnumerable<string> names = Enumerable.Select(customers, c => c.Name);
```

Since type arguments were not explicitly specified, type inference is used to infer the type arguments. First, the `customers` argument is related to the `source` parameter, inferring `T` to be `Customer`. Then, using the anonymous function type inference process described above, `c` is given type `Customer`, and the expression `c.Name` is related to the return type of the `selector` parameter, inferring `S` to be `string`. Thus, the invocation is equivalent to
```csharp
Sequence.Select<Customer,string>(customers, (Customer c) => c.Name)
```
and the result is of type `IEnumerable<string>`.

The following example demonstrates how anonymous function type inference allows type information to "flow" between arguments in a generic method invocation. Given the method:
```csharp
static Z F<X,Y,Z>(X value, Func<X,Y> f1, Func<Y,Z> f2) {
    return f2(f1(value));
}
```

Type inference for the invocation:
```csharp
double seconds = F("1:15:30", s => TimeSpan.Parse(s), t => t.TotalSeconds);
```
proceeds as follows: First, the argument `"1:15:30"` is related to the `value` parameter, inferring `X` to be `string`. Then, the parameter of the first anonymous function, `s`, is given the inferred type `string`, and the expression `TimeSpan.Parse(s)` is related to the return type of `f1`, inferring `Y` to be `System.TimeSpan`. Finally, the parameter of the second anonymous function, `t`, is given the inferred type `System.TimeSpan`, and the expression `t.TotalSeconds` is related to the return type of `f2`, inferring `Z` to be `double`. Thus, the result of the invocation is of type `double`.

#### Type inference for conversion of method groups

Similar to calls of generic methods, type inference must also be applied when a method group `M` containing a generic method is converted to a given delegate type `D` ([Method group conversions](conversions.md#method-group-conversions)). Given a method
```csharp
Tr M<X1...Xn>(T1 x1 ... Tm xm)
```
and the method group `M` being assigned to the delegate type `D` the task of type inference is to find type arguments `S1...Sn` so that the expression:
```csharp
M<S1...Sn>
```
becomes compatible ([Delegate declarations](delegates.md#delegate-declarations)) with `D`.

Unlike the type inference algorithm for generic method calls, in this case there are only argument *types*, no argument *expressions*. In particular, there are no anonymous functions and hence no need for multiple phases of inference.

Instead, all `Xi` are considered *unfixed*, and a *lower-bound inference* is made *from* each argument type `Uj` of `D` *to* the corresponding parameter type `Tj` of `M`. If for any of the `Xi` no bounds were found, type inference fails. Otherwise, all `Xi` are *fixed* to corresponding `Si`, which are the result of type inference.

#### Finding the best common type of a set of expressions

In some cases, a common type needs to be inferred for a set of expressions. In particular, the element types of implicitly typed arrays and the return types of anonymous functions with *block* bodies are found in this way.

Intuitively, given a set of expressions `E1...Em` this inference should be equivalent to calling a method
```csharp
Tr M<X>(X x1 ... X xm)
```
with the `Ei` as arguments.

More precisely, the inference starts out with an *unfixed* type variable `X`. *Output type inferences* are then made *from* each `Ei` *to* `X`. Finally, `X` is *fixed* and, if successful, the resulting type `S` is the resulting best common type for the expressions. If no such `S` exists, the expressions have no best common type.

### Overload resolution

Overload resolution is a binding-time mechanism for selecting the best function member to invoke given an argument list and a set of candidate function members. Overload resolution selects the function member to invoke in the following distinct contexts within C#:

*  Invocation of a method named in an *invocation_expression* ([Method invocations](expressions.md#method-invocations)).
*  Invocation of an instance constructor named in an *object_creation_expression* ([Object creation expressions](expressions.md#object-creation-expressions)).
*  Invocation of an indexer accessor through an *element_access* ([Element access](expressions.md#element-access)).
*  Invocation of a predefined or user-defined operator referenced in an expression ([Unary operator overload resolution](expressions.md#unary-operator-overload-resolution) and [Binary operator overload resolution](expressions.md#binary-operator-overload-resolution)).

Each of these contexts defines the set of candidate function members and the list of arguments in its own unique way, as described in detail in the sections listed above. For example, the set of candidates for a method invocation does not include methods marked `override` ([Member lookup](expressions.md#member-lookup)), and methods in a base class are not candidates if any method in a derived class is applicable ([Method invocations](expressions.md#method-invocations)).

Once the candidate function members and the argument list have been identified, the selection of the best function member is the same in all cases:

*  Given the set of applicable candidate function members, the best function member in that set is located. If the set contains only one function member, then that function member is the best function member. Otherwise, the best function member is the one function member that is better than all other function members with respect to the given argument list, provided that each function member is compared to all other function members using the rules in [Better function member](expressions.md#better-function-member). If there is not exactly one function member that is better than all other function members, then the function member invocation is ambiguous and a binding-time error occurs.

The following sections define the exact meanings of the terms ***applicable function member*** and ***better function member***.

#### Applicable function member

A function member is said to be an ***applicable function member*** with respect to an argument list `A` when all of the following are true:

*  Each argument in `A` corresponds to a parameter in the function member declaration as described in [Corresponding parameters](expressions.md#corresponding-parameters), and any parameter to which no argument corresponds is an optional parameter.
*  For each argument in `A`, the parameter passing mode of the argument (i.e., value, `ref`, or `out`) is identical to the parameter passing mode of the corresponding parameter, and
   *  for a value parameter or a parameter array, an implicit conversion ([Implicit conversions](conversions.md#implicit-conversions)) exists from the argument to the type of the corresponding parameter, or
   *  for a `ref` or `out` parameter, the type of the argument is identical to the type of the corresponding parameter. After all, a `ref` or `out` parameter is an alias for the argument passed.

For a function member that includes a parameter array, if the function member is applicable by the above rules, it is said to be applicable in its ***normal form***. If a function member that includes a parameter array is not applicable in its normal form, the function member may instead be applicable in its ***expanded form***:

*  The expanded form is constructed by replacing the parameter array in the function member declaration with zero or more value parameters of the element type of the parameter array such that the number of arguments in the argument list `A` matches the total number of parameters. If `A` has fewer arguments than the number of fixed parameters in the function member declaration, the expanded form of the function member cannot be constructed and is thus not applicable.
*  Otherwise, the expanded form is applicable if for each argument in `A` the parameter passing mode of the argument is identical to the parameter passing mode of the corresponding parameter, and
   *  for a fixed value parameter or a value parameter created by the expansion, an implicit conversion ([Implicit conversions](conversions.md#implicit-conversions)) exists from the type of the argument to the type of the corresponding parameter, or
   *  for a `ref` or `out` parameter, the type of the argument is identical to the type of the corresponding parameter.

#### Better function member

For the purposes of determining the better function member, a stripped-down argument list A is constructed containing just the argument expressions themselves in the order they appear in the original argument list.

Parameter lists for each of the candidate function members are constructed in the following way:

*  The expanded form is used if the function member was applicable only in the expanded form.
*  Optional parameters with no corresponding arguments are removed from the parameter list
*  The parameters are reordered so that they occur at the same position as the corresponding argument in the argument list.

Given an argument list `A` with a set of argument expressions `{E1, E2, ..., En}` and two applicable function members `Mp` and `Mq` with parameter types `{P1, P2, ..., Pn}` and `{Q1, Q2, ..., Qn}`, `Mp` is defined to be a ***better function member*** than `Mq` if

*  for each argument, the implicit conversion from `Ex` to `Qx` is not better than the implicit conversion from `Ex` to `Px`, and
*  for at least one argument, the conversion from `Ex` to `Px` is better than the conversion from `Ex` to `Qx`.

When performing this evaluation, if `Mp` or `Mq` is applicable in its expanded form, then `Px` or `Qx` refers to a parameter in the expanded form of the parameter list.

In case the parameter type sequences `{P1, P2, ..., Pn}` and `{Q1, Q2, ..., Qn}` are equivalent (i.e. each `Pi` has an identity conversion to the corresponding `Qi`), the following tie-breaking rules are applied, in order, to determine the better function member.

*  If `Mp` is a non-generic method and `Mq` is a generic method, then `Mp` is better than `Mq`.
*  Otherwise, if `Mp` is applicable in its normal form and `Mq` has a `params` array and is applicable only in its expanded form, then `Mp` is better than `Mq`.
*  Otherwise, if `Mp` has more declared parameters than `Mq`, then `Mp` is better than `Mq`. This can occur if both methods have `params` arrays and are applicable only in their expanded forms.
*  Otherwise if all parameters of `Mp` have a corresponding argument whereas default arguments need to be substituted for at least one optional parameter in `Mq` then `Mp` is better than `Mq`.
*  Otherwise, if `Mp` has more specific parameter types than `Mq`, then `Mp` is better than `Mq`. Let `{R1, R2, ..., Rn}` and `{S1, S2, ..., Sn}` represent the uninstantiated and unexpanded parameter types of `Mp` and `Mq`. `Mp`'s parameter types are more specific than `Mq`'s if, for each parameter, `Rx` is not less specific than `Sx`, and, for at least one parameter, `Rx` is more specific than `Sx`:
   *  A type parameter is less specific than a non-type parameter.
   *  Recursively, a constructed type is more specific than another constructed type (with the same number of type arguments) if at least one type argument is more specific and no type argument is less specific than the corresponding type argument in the other.
   *  An array type is more specific than another array type (with the same number of dimensions) if the element type of the first is more specific than the element type of the second.
*  Otherwise if one member is a non-lifted operator and  the other is a lifted operator, the non-lifted one is better.
*  Otherwise, neither function member is better.

#### Better conversion from expression

Given an implicit conversion `C1` that converts from an expression `E` to a type `T1`, and an implicit conversion `C2` that converts from an expression `E` to a type `T2`, `C1` is a ***better conversion*** than `C2` if `E` does not exactly match `T2` and at least one of the following holds:

* `E` exactly matches `T1` ([Exactly matching Expression](expressions.md#exactly-matching-expression))
* `T1` is a better conversion target than `T2` ([Better conversion target](expressions.md#better-conversion-target))

#### Exactly matching Expression

Given an expression `E` and a type `T`, `E` exactly matches `T` if one of the following holds:

*  `E` has a type `S`, and an identity conversion exists from `S` to `T`
*  `E` is an anonymous function, `T` is either a delegate type `D` or an expression tree type `Expression<D>` and one of the following holds:
   *  An inferred return type `X` exists for `E` in the context of the parameter list of `D` ([Inferred return type](expressions.md#inferred-return-type)), and an identity conversion exists from `X` to the return type of `D`
   *  Either `E` is non-async and `D` has a return type `Y` or `E` is async and `D` has a return type `Task<Y>`, and one of the following holds:
      * The body of `E` is an expression that exactly matches `Y`
      * The body of `E` is a statement block where every return statement returns an expression that exactly matches `Y`

#### Better conversion target

Given two different types `T1` and `T2`, `T1` is a better conversion target than `T2` if no implicit conversion from `T2` to `T1` exists, and at least one of the following holds:

*  An implicit conversion from `T1` to `T2` exists
*  `T1` is either a delegate type `D1` or an expression tree type `Expression<D1>`, `T2` is either a delegate type `D2` or an expression tree type `Expression<D2>`, `D1` has a return type `S1` and one of the following holds:
   * `D2` is void returning
   * `D2` has a return type `S2`, and `S1` is a better conversion target than `S2`
*  `T1` is `Task<S1>`, `T2` is `Task<S2>`, and `S1` is a better conversion target than `S2`
*  `T1` is `S1` or `S1?` where `S1` is a signed integral type, and `T2` is `S2` or `S2?` where `S2` is an unsigned integral type. Specifically:
   * `S1` is `sbyte` and `S2` is `byte`, `ushort`, `uint`, or `ulong`
   * `S1` is `short` and `S2` is `ushort`, `uint`, or `ulong`
   * `S1` is `int` and `S2` is `uint`, or `ulong`
   * `S1` is `long` and `S2` is `ulong`

#### Overloading in generic classes

While signatures as declared must be unique, it is possible that substitution of type arguments results in identical signatures. In such cases, the tie-breaking rules of overload resolution above will pick the most specific member.

The following examples show overloads that are valid and invalid according to this rule:

```csharp
interface I1<T> {...}

interface I2<T> {...}

class G1<U>
{
    int F1(U u);                  // Overload resolution for G<int>.F1
    int F1(int i);                // will pick non-generic

    void F2(I1<U> a);             // Valid overload
    void F2(I2<U> a);
}

class G2<U,V>
{
    void F3(U u, V v);            // Valid, but overload resolution for
    void F3(V v, U u);            // G2<int,int>.F3 will fail

    void F4(U u, I1<V> v);        // Valid, but overload resolution for    
    void F4(I1<V> v, U u);        // G2<I1<int>,int>.F4 will fail

    void F5(U u1, I1<V> v2);      // Valid overload
    void F5(V v1, U u2);

    void F6(ref U u);             // valid overload
    void F6(out V v);
}
```

### Compile-time checking of dynamic overload resolution

For most dynamically bound operations the set of possible candidates for resolution is unknown at compile-time. In certain cases, however the candidate set is known at compile-time:

*  Static method calls with dynamic arguments
*  Instance method calls where the receiver is not a dynamic expression
*  Indexer calls where the receiver is not a dynamic expression
*  Constructor calls with dynamic arguments

In these cases a limited compile-time check is performed for each candidate to see if any of them could possibly apply at run-time.This check consists of the following steps:

*  Partial type inference: Any type argument that does not depend directly or indirectly on an argument of type `dynamic` is inferred using the rules of [Type inference](expressions.md#type-inference). The remaining type arguments are unknown.
*  Partial applicability check: Applicability is checked according to [Applicable function member](expressions.md#applicable-function-member), but ignoring parameters whose types are unknown.
*  If no candidate passes this test, a compile-time error occurs.

### Function member invocation

This section describes the process that takes place at run-time to invoke a particular function member. It is assumed that a binding-time process has already determined the particular member to invoke, possibly by applying overload resolution to a set of candidate function members.

For purposes of describing the invocation process, function members are divided into two categories:

*  Static function members. These are instance constructors, static methods, static property accessors, and user-defined operators. Static function members are always non-virtual.
*  Instance function members. These are instance methods, instance property accessors, and indexer accessors. Instance function members are either non-virtual or virtual, and are always invoked on a particular instance. The instance is computed by an instance expression, and it becomes accessible within the function member as `this` ([This access](expressions.md#this-access)).

The run-time processing of a function member invocation consists of the following steps, where `M` is the function member and, if `M` is an instance member, `E` is the instance expression:

*  If `M` is a static function member:
   * The argument list is evaluated as described in [Argument lists](expressions.md#argument-lists).
   * `M` is invoked.

*  If `M` is an instance function member declared in a *value_type*:
   * `E` is evaluated. If this evaluation causes an exception, then no further steps are executed.
   * If `E` is not classified as a variable, then a temporary local variable of `E`'s type is created and the value of `E` is assigned to that variable. `E` is then reclassified as a reference to that temporary local variable. The temporary variable is accessible as `this` within `M`, but not in any other way. Thus, only when `E` is a true variable is it possible for the caller to observe the changes that `M` makes to `this`.
   * The argument list is evaluated as described in [Argument lists](expressions.md#argument-lists).
   * `M` is invoked. The variable referenced by `E` becomes the variable referenced by `this`.

*  If `M` is an instance function member declared in a *reference_type*:
   * `E` is evaluated. If this evaluation causes an exception, then no further steps are executed.
   * The argument list is evaluated as described in [Argument lists](expressions.md#argument-lists).
   * If the type of `E` is a *value_type*, a boxing conversion ([Boxing conversions](types.md#boxing-conversions)) is performed to convert `E` to type `object`, and `E` is considered to be of type `object` in the following steps. In this case, `M` could only be a member of `System.Object`.
   * The value of `E` is checked to be valid. If the value of `E` is `null`, a `System.NullReferenceException` is thrown and no further steps are executed.
   * The function member implementation to invoke is determined:
     * If the binding-time type of `E` is an interface, the function member to invoke is the implementation of `M` provided by the run-time type of the instance referenced by `E`. This function member is determined by applying the interface mapping rules ([Interface mapping](interfaces.md#interface-mapping)) to determine the implementation of `M` provided by the run-time type of the instance referenced by `E`.
     * Otherwise, if `M` is a virtual function member, the function member to invoke is the implementation of `M` provided by the run-time type of the instance referenced by `E`. This function member is determined by applying the rules for determining the most derived implementation ([Virtual methods](classes.md#virtual-methods)) of `M` with respect to the run-time type of the instance referenced by `E`.
     * Otherwise, `M` is a non-virtual function member, and the function member to invoke is `M` itself.
   * The function member implementation determined in the step above is invoked. The object referenced by `E` becomes the object referenced by `this`.

#### Invocations on boxed instances

A function member implemented in a *value_type* can be invoked through a boxed instance of that *value_type* in the following situations:

*  When the function member is an `override` of a method inherited from type `object` and is invoked through an instance expression of type `object`.
*  When the function member is an implementation of an interface function member and is invoked through an instance expression of an *interface_type*.
*  When the function member is invoked through a delegate.

In these situations, the boxed instance is considered to contain a variable of the *value_type*, and this variable becomes the variable referenced by `this` within the function member invocation. In particular, this means that when a function member is invoked on a boxed instance, it is possible for the function member to modify the value contained in the boxed instance.

## Primary expressions

Primary expressions include the simplest forms of expressions.

```antlr
primary_expression
    : primary_no_array_creation_expression
    | array_creation_expression
    ;

primary_no_array_creation_expression
    : literal
    | interpolated_string_expression
    | simple_name
    | parenthesized_expression
    | member_access
    | invocation_expression
    | element_access
    | this_access
    | base_access
    | post_increment_expression
    | post_decrement_expression
    | object_creation_expression
    | delegate_creation_expression
    | anonymous_object_creation_expression
    | typeof_expression
    | checked_expression
    | unchecked_expression
    | default_value_expression
    | nameof_expression
    | anonymous_method_expression
    | primary_no_array_creation_expression_unsafe
    ;
```

Primary expressions are divided between *array_creation_expression*s and *primary_no_array_creation_expression*s. Treating array-creation-expression in this way, rather than listing it along with the other simple expression forms, enables the grammar to disallow potentially confusing code such as
```csharp
object o = new int[3][1];
```
which would otherwise be interpreted as
```csharp
object o = (new int[3])[1];
```

### Literals

A *primary_expression* that consists of a *literal* ([Literals](lexical-structure.md#literals)) is classified as a value.

> __Expression Tree Conversion Translation Steps__
>
> A *literal* expression
>
> ```csharp
> literal
> ```
>
> when subject to conversion to a query expression tree, is translated into
>
> ```csharp
> Expression.Constant(literal, type)
> ```
>
> and when subject to conversion to a generalized expression tree, is translated into
>
> ```csharp
> Q.Constant(flags, literal, type)
> ```
>
> where `type` is an expression of type `Type` representing the static type of the value of the literal, and where `flags` is an expression of type `Q.Flags`:
>
> * `default(Q.Flags)` if no flags apply, or,
> * `Q.CompilerGenerated` if the node was compiler-generated (for instance, when translating an optional parameter).


### Interpolated strings

An *interpolated_string_expression* consists of a `$` sign followed by a regular or verbatim string literal, wherein holes, delimited by `{` and `}`, enclose expressions and formatting specifications. An interpolated string expression is the result of an *interpolated_string_literal* that has been broken up into individual tokens, as described in [Interpolated string literals](lexical-structure.md#interpolated-string-literals).

```antlr
interpolated_string_expression
    : '$' interpolated_regular_string
    | '$' interpolated_verbatim_string
    ;

interpolated_regular_string
    : interpolated_regular_string_whole
    | interpolated_regular_string_start interpolated_regular_string_body interpolated_regular_string_end
    ;

interpolated_regular_string_body
    : interpolation (interpolated_regular_string_mid interpolation)*
    ;

interpolation
    : expression
    | expression ',' constant_expression
    ;

interpolated_verbatim_string
    : interpolated_verbatim_string_whole
    | interpolated_verbatim_string_start interpolated_verbatim_string_body interpolated_verbatim_string_end
    ;

interpolated_verbatim_string_body
    : interpolation (interpolated_verbatim_string_mid interpolation)+
    ;
```

The *constant_expression* in an interpolation must have an implicit conversion to `int`.

An *interpolated_string_expression* is classified as a value. If it is immediately converted to `System.IFormattable` or `System.FormattableString` with an implicit interpolated string conversion ([Implicit interpolated string conversions](conversions.md#implicit-interpolated-string-conversions)), the interpolated string expression has that type. Otherwise, it has the type `string`.

If the type of an interpolated string is `System.IFormattable` or `System.FormattableString`, the meaning is a call to `System.Runtime.CompilerServices.FormattableStringFactory.Create`. If the type is `string`, the meaning of the expression is a call to `string.Format`. In both cases, the argument list of the call consists of a format string literal with placeholders for each interpolation, and an argument for each expression corresponding to the place holders.

The format string literal is constructed as follows, where `N` is the number of interpolations in the *interpolated_string_expression*:

*  If an *interpolated_regular_string_whole* or an *interpolated_verbatim_string_whole* follows the `$` sign, then the format string literal is that token.
*  Otherwise, the format string literal consists of: 
   *  First the *interpolated_regular_string_start* or *interpolated_verbatim_string_start*
   *  Then for each number `I` from `0` to `N-1`: 
      * The decimal representation of `I`
      * Then, if the corresponding *interpolation* has a *constant_expression*, a `,` (comma) followed by the decimal representation of the value of the *constant_expression*
      * Then the *interpolated_regular_string_mid*, *interpolated_regular_string_end*, *interpolated_verbatim_string_mid* or *interpolated_verbatim_string_end* immediately following the corresponding interpolation.

The subsequent arguments are simply the *expressions* from the *interpolations* (if any), in order.

TODO: examples.

> __Expression Tree Conversion Translation Steps__
>
> When subject to conversion to a query expression tree, an interpolated string expression is pre-processed by first rewriting the expression to a method invocation expression of `string.Format` or `System.Runtime.CompilerServices.FormattableStringFactory.Create`, as described above. The resulting expression is then translated to an expression tree, resulting in an `Expression.Call` factory invocation.
>
> A few things are worth making explicit:
>
> * Query expression trees representing interpolated strings are always represented using a `Call` expression node, without any further `Convert` node applied to it. Therefore, the type of the expression node will match the return type of the bound method, which is either `string` or `System.FormattableString`. In particular, an interpolated string converted to `System.IFormattable` will have the more derived `System.FormattableString` type on its expression tree representation.
> * Overload resolution rules apply after the rewrite step. For instance, an interpolated string `$"{a}{b}"` will bind to the `string.Format(string, object, object)` overload, but interpolated string `$"{a}{b}{c}{d}"` will bind to the `string.Format(string, object[])` overload, thus resulting in the use of a `NewArrayInit` expression node to construct the `params` array.
> * After binding to any of the methods listed above, regular conversion rules apply for argument assignment. For instance, an interpolated string `$"{42}"` will have the boxing conversion of `42` represented by a `Convert` expression node. No conversions are emitted for interpolations that have a reference type, because an implicit conversion to `object` exists.
> * Interpolated strings containing interpolations with constant expressions may be subject to implementation-dependent compile-time optimizations prior to being rewritten to a method invocation expression. For instance, an interpolated string `$"A{"B"}C"` may get optimized to a constant string with value `"ABC"` which gets further translated to a `Cosntant` expression node.
>
> When subject to conversion to a query expression tree, an interpolated string expression is translated as follows. First, each interpolation of the form
>
> ```
> {<interpolatedExpression>[,<alignment>][:<formatString>]}
> ```
>
> is translated by first translating `interpolatedExpression` to an expression tree `intExpr`, and (if present) translating `alignment` to an expression tree `alignExpr`. The interpolation is then translated into
>
> ```csharp
> Q.InterpolationExpression(info, intExpr)
> ```
>
> if no `alignment` is specified, or
>
> ```csharp
> Q.InterpolationExpression(info, intExpr, alignmentExpr)
> ```
>
> otherwise, where `info` is
>
> ```csharp
> Q.InterpolationExpressionInfo(formatString)
> ```
>
> if a `formatString` is specified, and where `formatString` is represented as an expression of type `string` containing the string representation of the format string, or
>
> ```csharp
> Q.InterpolationExpressionInfo()
> ```
>
> otherwise.
>
> Note that `alignment` is translated to an expression tree, and not to an expression of type `int` passed to `InterpolationInfo`. Generalized expression tree conversion suppresses compile-time evaluation of constant expressions in order to retain the original syntactic structure of the code, offering the highest degree of flexibility to expression tree libraries. Note that `formatString` is translated to an expression of type `string` passed to `InterpolationInfo` because the lexical structure of interpolated strings only allows literal strings for that part.
>
> Next, each literal string fragment occurring before the first interpolation, in between any two interpolations, and after the last interpolation is translated into
>
> ```csharp
> Q.InterpolationLiteral(str)
> ```
>
> where `str` is an expression of type `string` containing the string representation of the literal string fragment (in particular, escape sequences have been processed).
>
> Finally, the interpolated string expression is converted to an expression tree representation
>
> ```csharp
> Q.InterpolatedString(type, args)
> ```
>
> where `type` is an expression of type `Type` representing any of `string`, `System.FormattableString`, or `System.IFormattable`, and `args` is an argument list
>
> ```csharp
> i1, ..., iN
> ```
>
> where `i1` through `iN` correspond to the expression tree translation of the literal string fragments and interpolations occurring in the interpolated string expression in lexical order.
>
> If the interpolated string is empty (i.e. any of `$""` or `$@""`), a single interpolation literal is produced for use in `args`:
>
> ```csharp
> Q.InterpolationLiteral("")
> ```
>
> Note that generalized expression trees retain the structure of the interpolated string expression. This enables expression tree libraries to avoid having to parse a .NET format string, for instance when translating an expression tree to a target language that supports interpolation as well.



### Simple names

A *simple_name* consists of an identifier, optionally followed by a type argument list:

```antlr
simple_name
    : identifier type_argument_list?
    ;
```

A *simple_name* is either of the form `I` or of the form `I<A1,...,Ak>`, where `I` is a single identifier and `<A1,...,Ak>` is an optional *type_argument_list*. When no *type_argument_list* is specified, consider `K` to be zero. The *simple_name* is evaluated and classified as follows:

*  If `K` is zero and the *simple_name* appears within a *block* and if the *block*'s (or an enclosing *block*'s) local variable declaration space ([Declarations](basic-concepts.md#declarations)) contains a local variable, parameter or constant with name `I`, then the *simple_name* refers to that local variable, parameter or constant and is classified as a variable or value.
*  If `K` is zero and the *simple_name* appears within the body of a generic method declaration and if that declaration includes a type parameter with name `I`, then the *simple_name* refers to that type parameter.
*  Otherwise, for each instance type `T` ([The instance type](classes.md#the-instance-type)), starting with the instance type of the immediately enclosing type declaration and continuing with the instance type of each enclosing class or struct declaration (if any):
   *  If `K` is zero and the declaration of `T` includes a type parameter with name `I`, then the *simple_name* refers to that type parameter.
   *  Otherwise, if a member lookup ([Member lookup](expressions.md#member-lookup)) of `I` in `T` with `K` type arguments produces a match:
      * If `T` is the instance type of the immediately enclosing class or struct type and the lookup identifies one or more methods, the result is a method group with an associated instance expression of `this`. If a type argument list was specified, it is used in calling a generic method ([Method invocations](expressions.md#method-invocations)).
      * Otherwise, if `T` is the instance type of the immediately enclosing class or struct type, if the lookup identifies an instance member, and if the reference occurs within the body of an instance constructor, an instance method, or an instance accessor, the result is the same as a member access ([Member access](expressions.md#member-access)) of the form `this.I`. This can only happen when `K` is zero.
      * Otherwise, the result is the same as a member access ([Member access](expressions.md#member-access)) of the form `T.I` or `T.I<A1,...,Ak>`. In this case, it is a binding-time error for the *simple_name* to refer to an instance member.

*  Otherwise, for each namespace `N`, starting with the namespace in which the *simple_name* occurs, continuing with each enclosing namespace (if any), and ending with the global namespace, the following steps are evaluated until an entity is located:
   *  If `K` is zero and `I` is the name of a namespace in `N`, then:
      * If the location where the *simple_name* occurs is enclosed by a namespace declaration for `N` and the namespace declaration contains an *extern_alias_directive* or *using_alias_directive* that associates the name `I` with a namespace or type, then the *simple_name* is ambiguous and a compile-time error occurs.
      * Otherwise, the *simple_name* refers to the namespace named `I` in `N`.
   *  Otherwise, if `N` contains an accessible type having name `I` and `K` type parameters, then:
      * If `K` is zero and the location where the *simple_name* occurs is enclosed by a namespace declaration for `N` and the namespace declaration contains an *extern_alias_directive* or *using_alias_directive* that associates the name `I` with a namespace or type, then the *simple_name* is ambiguous and a compile-time error occurs.
      * Otherwise, the *namespace_or_type_name* refers to the type constructed with the given type arguments.
   *  Otherwise, if the location where the *simple_name* occurs is enclosed by a namespace declaration for `N`:
      * If `K` is zero and the namespace declaration contains an *extern_alias_directive* or *using_alias_directive* that associates the name `I` with an imported namespace or type, then the *simple_name* refers to that namespace or type.
      * Otherwise, if the namespaces and type declarations imported by the *using_namespace_directive*s and *using_static_directive*s of the namespace declaration contain exactly one accessible type or non-extension static member having name `I` and `K` type parameters, then the *simple_name* refers to that type or member constructed with the given type arguments.
      * Otherwise, if the namespaces and types imported by the *using_namespace_directive*s of the namespace declaration contain more than one accessible type or non-extension-method static member having name `I` and `K` type parameters, then the *simple_name* is ambiguous and an error occurs.

   Note that this entire step is exactly parallel to the corresponding step in the processing of a *namespace_or_type_name* ([Namespace and type names](basic-concepts.md#namespace-and-type-names)).

*  Otherwise, the *simple_name* is undefined and a compile-time error occurs.


### Parenthesized expressions

A *parenthesized_expression* consists of an *expression* enclosed in parentheses.

```antlr
parenthesized_expression
    : '(' expression ')'
    ;
```

A *parenthesized_expression* is evaluated by evaluating the *expression* within the parentheses. If the *expression* within the parentheses denotes a namespace or type, a compile-time error occurs. Otherwise, the result of the *parenthesized_expression* is the result of the evaluation of the contained *expression*.

> __Expression Tree Conversion Translation Steps__
>
> A *parenthesized_expression* expression
>
> ```csharp
> ( expression )
> ```
>
> when subject to conversion to an expression tree, is translated by applying the expression tree translation steps to `expression`.

### Member access

A *member_access* consists of a *primary_expression*, a *predefined_type*, or a *qualified_alias_member*, followed by a "`.`" token, followed by an *identifier*, optionally followed by a *type_argument_list*.

```antlr
member_access
    : primary_expression '.' identifier type_argument_list?
    | predefined_type '.' identifier type_argument_list?
    | qualified_alias_member '.' identifier
    ;

predefined_type
    : 'bool'   | 'byte'  | 'char'  | 'decimal' | 'double' | 'float' | 'int' | 'long'
    | 'object' | 'sbyte' | 'short' | 'string'  | 'uint'   | 'ulong' | 'ushort'
    ;
```

The *qualified_alias_member* production is defined in [Namespace alias qualifiers](namespaces.md#namespace-alias-qualifiers).

A *member_access* is either of the form `E.I` or of the form `E.I<A1, ..., Ak>`, where `E` is a primary-expression, `I` is a single identifier and `<A1, ..., Ak>` is an optional *type_argument_list*. When no *type_argument_list* is specified, consider `K` to be zero.

A *member_access* with a *primary_expression* of type `dynamic` is dynamically bound ([Dynamic binding](expressions.md#dynamic-binding)). In this case the compiler classifies the member access as a property access of type `dynamic`. The rules below to determine the meaning of the *member_access* are then applied at run-time, using the run-time type instead of the compile-time type of the *primary_expression*. If this run-time classification leads to a method group, then the member access must be the *primary_expression* of an *invocation_expression*.

The *member_access* is evaluated and classified as follows:

*  If `K` is zero and `E` is a namespace and `E` contains a nested namespace with name `I`, then the result is that namespace.
*  Otherwise, if `E` is a namespace and `E` contains an accessible type having name `I` and `K` type parameters, then the result is that type constructed with the given type arguments.
*  If `E` is a *predefined_type* or a *primary_expression* classified as a type, if `E` is not a type parameter, and if a member lookup ([Member lookup](expressions.md#member-lookup)) of `I` in `E` with `K` type parameters produces a match, then `E.I` is evaluated and classified as follows:
   *  If `I` identifies a type, then the result is that type constructed with the given type arguments.
   *  If `I` identifies one or more methods, then the result is a method group with no associated instance expression. If a type argument list was specified, it is used in calling a generic method ([Method invocations](expressions.md#method-invocations)).
   *  If `I` identifies a `static` property, then the result is a property access with no associated instance expression.
   *  If `I` identifies a `static` field:
      * If the field is `readonly` and the reference occurs outside the static constructor of the class or struct in which the field is declared, then the result is a value, namely the value of the static field `I` in `E`.
      * Otherwise, the result is a variable, namely the static field `I` in `E`.
   *  If `I` identifies a `static` event:
      * If the reference occurs within the class or struct in which the event is declared, and the event was declared without *event_accessor_declarations* ([Events](classes.md#events)), then `E.I` is processed exactly as if `I` were a static field.
      * Otherwise, the result is an event access with no associated instance expression.
   *  If `I` identifies a constant, then the result is a value, namely the value of that constant.
    * If `I` identifies an enumeration member, then the result is a value, namely the value of that enumeration member.
    * Otherwise, `E.I` is an invalid member reference, and a compile-time error occurs.
*  If `E` is a property access, indexer access, variable, or value, the type of which is `T`, and a member lookup ([Member lookup](expressions.md#member-lookup)) of `I` in `T` with `K` type arguments produces a match, then `E.I` is evaluated and classified as follows:
   *  First, if `E` is a property or indexer access, then the value of the property or indexer access is obtained ([Values of expressions](expressions.md#values-of-expressions)) and `E` is reclassified as a value.
   *  If `I` identifies one or more methods, then the result is a method group with an associated instance expression of `E`. If a type argument list was specified, it is used in calling a generic method ([Method invocations](expressions.md#method-invocations)).
   *  If `I` identifies an instance property,
      * If `E` is `this`, `I` identifies an automatically implemented property ([Automatically implemented properties](classes.md#automatically-implemented-properties)) without a setter, and the reference occurs within an instance constructor for a class or struct type `T`, then the result is a variable, namely the hidden backing field for the auto-property given by `I` in the instance of `T` given by `this`.
      * Otherwise, the result is a property access with an associated instance expression of `E`.
   *  If `T` is a *class_type* and `I` identifies an instance field of that *class_type*:
      * If the value of `E` is `null`, then a `System.NullReferenceException` is thrown.
      * Otherwise, if the field is `readonly` and the reference occurs outside an instance constructor of the class in which the field is declared, then the result is a value, namely the value of the field `I` in the object referenced by `E`.
      * Otherwise, the result is a variable, namely the field `I` in the object referenced by `E`.
   *  If `T` is a *struct_type* and `I` identifies an instance field of that *struct_type*:
      * If `E` is a value, or if the field is `readonly` and the reference occurs outside an instance constructor of the struct in which the field is declared, then the result is a value, namely the value of the field `I` in the struct instance given by `E`.
      * Otherwise, the result is a variable, namely the field `I` in the struct instance given by `E`.
   *  If `I` identifies an instance event:
      * If the reference occurs within the class or struct in which the event is declared, and the event was declared without *event_accessor_declarations* ([Events](classes.md#events)), and the reference does not occur as the left-hand side of a `+=` or `-=` operator, then `E.I` is processed exactly as if `I` was an instance field.
      * Otherwise, the result is an event access with an associated instance expression of `E`.
*  Otherwise, an attempt is made to process `E.I` as an extension method invocation ([Extension method invocations](expressions.md#extension-method-invocations)). If this fails, `E.I` is an invalid member reference, and a binding-time error occurs.

> __Expression Tree Conversion Translation Steps__
>
> A *member_access* of the form `E.I` is converted to an expression tree in the following cases:
>
> * if `I` identifies an instance property or an instance field, and `E` represents an expression, which may be of type `dynamic`, or,
> * if `I` identifies a static property or a static field, and `E` represents a type `T`.
>
> In all other cases, the *member_access* is not represented as a separate expression tree, but is translated as part of an ancestor expression or statement. For instance, if the parent expression is a method invocation, the *member_access* is deconstructed into an instance expression (if the method invocation is bound to an instance method) and a `MethodInfo` representing the bound method or a dynamic binder object.
>
> Expression tree conversion proceeds as follows.
>
> ### Static property access
>
> If `I` represents a static property, construct an expression `method` of type `MethodInfo` representing the `get` accessor of the property. When converted to a query expression tree, the *member_access* is translated into:
>
> ```csharp
> Expression.Property((Expression)null, method)
> ```
>
> When converted to a generalized expression tree, the *member_access* is translated into:
>
> ```csharp
> Q.MemberAccess(info)
> ```
>
> where `info` is
>
> ```csharp
> Q.MemberAccessInfo(default(Q.Flags), method)
> ```
>
> ### Static field access
>
> If `I` represents a static field, construct an expression `field` of type `FieldInfo` representing the field. When converted to a query expression tree, the *member_access* is translated into:
>
> ```csharp
> Expression.Field((Expression)null, field)
> ```
>
> When converted to a generalized expression tree, the *member_access* is translated into:
>
> ```csharp
> Q.MemberAccess(info)
> ```
>
> where `info` is
>
> ```csharp
> Q.MemberAccessInfo(default(Q.Flags), field)
> ```
>
> ***TODO***
> * Describe behavior for `const` fields.
>
> ### Instance property access
>
> If `I` represents an instance property, and `E` is of type `dynamic`, a compile-time error occurs. Otherwise, construct an expression `expr` by converting `E` to an expression tree, and construct an expression `method` of type `MethodInfo` representing the `get` accessor of the property. When converted to a query expression tree, the *member_access* is translated into:
>
> ```csharp
> Expression.Property(expr, method)
> ```
>
> When converted to a generalized expression tree, the *member_access* is translated into:
>
> ```csharp
> Q.MemberAccess(info, expr)
> ```
>
> where `info` is
>
> ```csharp
> Q.MemberAccessInfo(default(Q.Flags), binder)
> ```
>
> if `E` is of type `dynamic` and where `binder` is an expression of type `Microsoft.CSharp.RuntimeBinder.CallSiteBinder` representing the dynamic operation, or
>
> ```csharp
> Q.MemberAccessInfo(default(Q.Flags), method)
> ```
>
> otherwise.
>
> ***TODO***
> * Describe quirk https://github.com/dotnet/roslyn/issues/4471 for query expression trees.
>
> ### Instance field access
>
> If `I` represents an instance field, construct an expression `expr` by converting `E` to an expression tree, and construct an expression `field` of type `FieldInfo` representing the field. When converted to a query expression tree, the *member_access* is translated into:
>
> ```csharp
> Expression.Field(expr, field)
> ```
>
> When converted to a generalized expression tree, the *member_access* is translated into:
>
> ```csharp
> Q.MemberAccess(info, expr)
> ```
>
> where `info` is
>
> ```csharp
> Q.MemberAccessInfo(default(Q.Flags), field)
> ```
>
> Note that a *member_access* applied to an expression `E` of type `dynamic` is classified as a property access. Therefore, this section does not cover the dynamically bound case which is covered in the section on instance property access.

#### Identical simple names and type names

In a member access of the form `E.I`, if `E` is a single identifier, and if the meaning of `E` as a *simple_name* ([Simple names](expressions.md#simple-names)) is a constant, field, property, local variable, or parameter with the same type as the meaning of `E` as a *type_name* ([Namespace and type names](basic-concepts.md#namespace-and-type-names)), then both possible meanings of `E` are permitted. The two possible meanings of `E.I` are never ambiguous, since `I` must necessarily be a member of the type `E` in both cases. In other words, the rule simply permits access to the static members and nested types of `E` where a compile-time error would otherwise have occurred. For example:
```csharp
struct Color
{
    public static readonly Color White = new Color(...);
    public static readonly Color Black = new Color(...);

    public Color Complement() {...}
}

class A
{
    public Color Color;                // Field Color of type Color

    void F() {
        Color = Color.Black;           // References Color.Black static member
        Color = Color.Complement();    // Invokes Complement() on Color field
    }

    static void G() {
        Color c = Color.White;         // References Color.White static member
    }
}
```

#### Grammar ambiguities

The productions for *simple_name* ([Simple names](expressions.md#simple-names)) and *member_access* ([Member access](expressions.md#member-access)) can give rise to ambiguities in the grammar for expressions. For example, the statement:
```
F(G<A,B>(7));
```
could be interpreted as a call to `F` with two arguments, `G < A` and `B > (7)`. Alternatively, it could be interpreted as a call to `F` with one argument, which is a call to a generic method `G` with two type arguments and one regular argument.

If a sequence of tokens can be parsed (in context) as a *simple_name* ([Simple names](expressions.md#simple-names)), *member_access* ([Member access](expressions.md#member-access)), or *pointer_member_access* ([Pointer member access](unsafe-code.md#pointer-member-access)) ending with a *type_argument_list* ([Type arguments](types.md#type-arguments)), the token immediately following the closing `>` token is examined. If it is one of
```csharp
(  )  ]  }  :  ;  ,  .  ?  ==  !=  |  ^
```
then the *type_argument_list* is retained as part of the *simple_name*, *member_access* or *pointer_member_access* and any other possible parse of the sequence of tokens is discarded. Otherwise, the *type_argument_list* is not considered to be part of the *simple_name*, *member_access* or *pointer_member_access*, even if there is no other possible parse of the sequence of tokens. Note that these rules are not applied when parsing a *type_argument_list* in a *namespace_or_type_name* ([Namespace and type names](basic-concepts.md#namespace-and-type-names)). The statement
```csharp
F(G<A,B>(7));
```
will, according to this rule, be interpreted as a call to `F` with one argument, which is a call to a generic method `G` with two type arguments and one regular argument. The statements
```csharp
F(G < A, B > 7);
F(G < A, B >> 7);
```
will each be interpreted as a call to `F` with two arguments. The statement
```csharp
x = F < A > +y;
```
will be interpreted as a less than operator, greater than operator, and unary plus operator, as if the statement had been written `x = (F < A) > (+y)`, instead of as a *simple_name* with a *type_argument_list* followed by a binary plus operator. In the statement
```csharp
x = y is C<T> + z;
```
the tokens `C<T>` are interpreted as a *namespace_or_type_name* with a *type_argument_list*.

### Invocation expressions

An *invocation_expression* is used to invoke a method.

```antlr
invocation_expression
    : primary_expression '(' argument_list? ')'
    ;
```

An *invocation_expression* is dynamically bound ([Dynamic binding](expressions.md#dynamic-binding)) if at least one of the following holds:

* The *primary_expression* has compile-time type `dynamic`.
* At least one argument of the optional *argument_list* has compile-time type `dynamic` and the *primary_expression* does not have a delegate type.

In this case the compiler classifies the *invocation_expression* as a value of type `dynamic`. The rules below to determine the meaning of the *invocation_expression* are then applied at run-time, using the run-time type instead of the compile-time type of those of the *primary_expression* and arguments which have the compile-time type `dynamic`. If the *primary_expression* does not have compile-time type `dynamic`, then the method invocation undergoes a limited compile time check as described in [Compile-time checking of dynamic overload resolution](expressions.md#compile-time-checking-of-dynamic-overload-resolution).

The *primary_expression* of an *invocation_expression* must be a method group or a value of a *delegate_type*. If the *primary_expression* is a method group, the *invocation_expression* is a method invocation ([Method invocations](expressions.md#method-invocations)). If the *primary_expression* is a value of a *delegate_type*, the *invocation_expression* is a delegate invocation ([Delegate invocations](expressions.md#delegate-invocations)). If the *primary_expression* is neither a method group nor a value of a *delegate_type*, a binding-time error occurs.

The optional *argument_list* ([Argument lists](expressions.md#argument-lists)) provides values or variable references for the parameters of the method.

The result of evaluating an *invocation_expression* is classified as follows:

*  If the *invocation_expression* invokes a method or delegate that returns `void`, the result is nothing. An expression that is classified as nothing is permitted only in the context of a *statement_expression* ([Expression statements](statements.md#expression-statements)) or as the body of a *lambda_expression* ([Anonymous function expressions](expressions.md#anonymous-function-expressions)). Otherwise a binding-time error occurs.
*  Otherwise, the result is a value of the type returned by the method or delegate.

#### Method invocations

For a method invocation, the *primary_expression* of the *invocation_expression* must be a method group. The method group identifies the one method to invoke or the set of overloaded methods from which to choose a specific method to invoke. In the latter case, determination of the specific method to invoke is based on the context provided by the types of the arguments in the *argument_list*.

The binding-time processing of a method invocation of the form `M(A)`, where `M` is a method group (possibly including a *type_argument_list*), and `A` is an optional *argument_list*, consists of the following steps:

*  The set of candidate methods for the method invocation is constructed. For each method `F` associated with the method group `M`:
   *  If `F` is non-generic, `F` is a candidate when:
      * `M` has no type argument list, and
      * `F` is applicable with respect to `A` ([Applicable function member](expressions.md#applicable-function-member)).
   *  If `F` is generic and `M` has no type argument list, `F` is a candidate when:
      * Type inference ([Type inference](expressions.md#type-inference)) succeeds, inferring a list of type arguments for the call, and
      * Once the inferred type arguments are substituted for the corresponding method type parameters, all constructed types in the parameter list of F satisfy their constraints ([Satisfying constraints](types.md#satisfying-constraints)), and the parameter list of `F` is applicable with respect to `A` ([Applicable function member](expressions.md#applicable-function-member)).
   *  If `F` is generic and `M` includes a type argument list, `F` is a candidate when:
      * `F` has the same number of method type parameters as were supplied in the type argument list, and
      * Once the type arguments are substituted for the corresponding method type parameters, all constructed types in the parameter list of F satisfy their constraints ([Satisfying constraints](types.md#satisfying-constraints)), and the parameter list of `F` is applicable with respect to `A` ([Applicable function member](expressions.md#applicable-function-member)).
*  The set of candidate methods is reduced to contain only methods from the most derived types: For each method `C.F` in the set, where `C` is the type in which the method `F` is declared, all methods declared in a base type of `C` are removed from the set. Furthermore, if `C` is a class type other than `object`, all methods declared in an interface type are removed from the set. (This latter rule only has affect when the method group was the result of a member lookup on a type parameter having an effective base class other than object and a non-empty effective interface set.)
*  If the resulting set of candidate methods is empty, then further processing along the following steps are abandoned, and instead an attempt is made to process the invocation as an extension method invocation ([Extension method invocations](expressions.md#extension-method-invocations)). If this fails, then no applicable methods exist, and a binding-time error occurs.
*  The best method of the set of candidate methods is identified using the overload resolution rules of [Overload resolution](expressions.md#overload-resolution). If a single best method cannot be identified, the method invocation is ambiguous, and a binding-time error occurs. When performing overload resolution, the parameters of a generic method are considered after substituting the type arguments (supplied or inferred) for the corresponding method type parameters.
*  Final validation of the chosen best method is performed:
   * The method is validated in the context of the method group: If the best method is a static method, the method group must have resulted from a *simple_name* or a *member_access* through a type. If the best method is an instance method, the method group must have resulted from a *simple_name*, a *member_access* through a variable or value, or a *base_access*. If neither of these requirements is true, a binding-time error occurs.
   * If the best method is a generic method, the type arguments (supplied or inferred) are checked against the constraints ([Satisfying constraints](types.md#satisfying-constraints)) declared on the generic method. If any type argument does not satisfy the corresponding constraint(s) on the type parameter, a binding-time error occurs.

Once a method has been selected and validated at binding-time by the above steps, the actual run-time invocation is processed according to the rules of function member invocation described in [Compile-time checking of dynamic overload resolution](expressions.md#compile-time-checking-of-dynamic-overload-resolution).

The intuitive effect of the resolution rules described above is as follows: To locate the particular method invoked by a method invocation, start with the type indicated by the method invocation and proceed up the inheritance chain until at least one applicable, accessible, non-override method declaration is found. Then perform type inference and overload resolution on the set of applicable, accessible, non-override methods declared in that type and invoke the method thus selected. If no method was found, try instead to process the invocation as an extension method invocation.

> __Expression Tree Conversion Translation Steps__
>
> Method invocation expressions
>
> ```csharp
> T.M(args)
> instance.M(args)
> ```
>
> are translated to an expression tree by first translating `instance` to an expression tree `instanceExpr`, if `M` is bound to an instance method, and by translating the argument list `args` to `argsExpr`, if it exists. Note than translation of argument lists may insert cast expressions for required implicit conversions, and/or array initialization expressions for assignment to `params` parameters, classifying these expressions as compiler-generated (see [Argument lists](expressions.md#argument-lists)). Translation to an expression tree then proceeds as follows.
>
> For purposes of conversion to a query expression tree, if the method invocation is dynamically bound, translation fails and an error is reported. Otherwise, it is translated into
>
> ```csharp
> Expression.Call(instanceExpr, method, argsExpr)
> ```
>
> if an argument list `args` exists, or
>
> ```csharp
> Expression.Call(instanceExpr, method, empty)
> ```
>
> where `instanceExpr` is `default(Expression)` if `M` is bound to a static method, `method` is an expression of type `MethodInfo` representing the bound method `M`, and `empty` is an expression of type `Expression[]` with zero elements.
>
> Note that the conversion of the argument list `args` to `argsExpr` for query expression trees yields an expression of type `Expression[]`, see [Argument lists](expressions.md#argument-lists). Therefore, overload resolution rules applied to bind the `Call` factory method invocation will always pick the `Call(Expression, MethodInfo, Expression[])` overload, even if specialized overloads for low argument counts are available.
>
> Also note that the conversion for a method invocation expression without any arguments uses an expression representing an empty array `empty` rather than `new Expression[0]`. This provides the implementation the flexibility to use a shared instance of an empty array (e.g. `Array.Empty<Expression>()`).
>
> For purposes of conversion to a generalized expression tree, it is translated into
>
> ```csharp
> Q.Call(info, argsExpr)
> ```
>
> or, if no argument list is present,
>
> ```csharp
> Q.Call(info)
> ```
>
> if `M` is bound to a static method, and into
>
> ```csharp
> Q.Call(info, instanceExpr, argsExpr)
> ```
>
> or, if no argument list is present,
>
> ```csharp
> Q.Call(info, instanceExpr)
> ```
>
> otherwise, where `info` is
>
> ```csharp
> Q.CallInfo(binder)
> ```
>
> if the method invocation is dynamically bound, where `binder` is an expression of type `Microsoft.CSharp.RuntimeBinder.CallSiteBinder` representing the dynamic operation, or
>
> ```csharp
> Q.CallInfo(flags, method)
> ```
>
> otherwise, where `method` is an expression of type `MethodInfo` representing the bound method `M`, and `flags` is
>
> * `Q.Flags.ResultDiscarded` if the expression occurs in an *expression_statement* ([Expression statements](statements.md#expression-statements)), or,
> * `default(Q.Flags)` otherwise.
>
> ***TODO***
> * Consider whether `argsExpr` should be a syntactic fragment of shape `(, argExpr)*` that gets concatenated to the argument list containing `info` and `fExpr`, thus flattening the first-class representation of an argument list and "inlining" it on the `Call` factory (much like query expression trees does).
> * Evaluate options for the dynamic case:
>   * create a specification for the construction of `binder` (can also be used to specify `dynamic` behavior outside the context of expression trees) and consider making more properties on the `Microsoft.CSharp.RuntimeBinder` library types publicly accessable so expression tree libraries can inspect this info, or,
>   * steer away from reusing `Microsoft.CSharp` and introduce a `CallInfo` factory overload for dynamic binding (e.g. a `(Q.DynamicFlags flags, Type context, string name, Type[] typeArgs, params A[] argumentInfo)` overload, where `A` is built through factory invocations as well).

#### Extension method invocations

In a method invocation ([Invocations on boxed instances](expressions.md#invocations-on-boxed-instances)) of one of the forms
```csharp
expr . identifier ( )

expr . identifier ( args )

expr . identifier < typeargs > ( )

expr . identifier < typeargs > ( args )
```
if the normal processing of the invocation finds no applicable methods, an attempt is made to process the construct as an extension method invocation. If *expr* or any of the *args* has compile-time type `dynamic`, extension methods will not apply.

The objective is to find the best *type_name* `C`, so that the corresponding static method invocation can take place:
```csharp
C . identifier ( expr )

C . identifier ( expr , args )

C . identifier < typeargs > ( expr )

C . identifier < typeargs > ( expr , args )
```

An extension method `Ci.Mj` is ***eligible*** if:

*  `Ci` is a non-generic, non-nested class
*  The name of `Mj` is *identifier*
*  `Mj` is accessible and applicable when applied to the arguments as a static method as shown above
*  An implicit identity, reference or boxing conversion exists from *expr* to the type of the first parameter of `Mj`.

The search for `C` proceeds as follows:

*  Starting with the closest enclosing namespace declaration, continuing with each enclosing namespace declaration, and ending with the containing compilation unit, successive attempts are made to find a candidate set of extension methods:
   * If the given namespace or compilation unit directly contains non-generic type declarations `Ci` with eligible extension methods `Mj`, then the set of those extension methods is the candidate set.
   * If types `Ci` imported by *using_static_declarations* and directly declared in namespaces imported by *using_namespace_directive*s in the given namespace or compilation unit directly contain eligible extension methods `Mj`, then the set of those extension methods is the candidate set.
*  If no candidate set is found in any enclosing namespace declaration or compilation unit, a compile-time error occurs.
*  Otherwise, overload resolution is applied to the candidate set as described in ([Overload resolution](expressions.md#overload-resolution)). If no single best method is found, a compile-time error occurs.
*  `C` is the type within which the best method is declared as an extension method.

Using `C` as a target, the method call is then processed as a static method invocation ([Compile-time checking of dynamic overload resolution](expressions.md#compile-time-checking-of-dynamic-overload-resolution)).

The preceding rules mean that instance methods take precedence over extension methods, that extension methods available in inner namespace declarations take precedence over extension methods available in outer namespace declarations, and that extension methods declared directly in a namespace take precedence over extension methods imported into that same namespace with a using namespace directive. For example:
```csharp
public static class E
{
    public static void F(this object obj, int i) { }

    public static void F(this object obj, string s) { }
}

class A { }

class B
{
    public void F(int i) { }
}

class C
{
    public void F(object obj) { }
}

class X
{
    static void Test(A a, B b, C c) {
        a.F(1);              // E.F(object, int)
        a.F("hello");        // E.F(object, string)

        b.F(1);              // B.F(int)
        b.F("hello");        // E.F(object, string)

        c.F(1);              // C.F(object)
        c.F("hello");        // C.F(object)
    }
}
```

In the example, `B`'s method takes precedence over the first extension method, and `C`'s method takes precedence over both extension methods.

```csharp
public static class C
{
    public static void F(this int i) { Console.WriteLine("C.F({0})", i); }
    public static void G(this int i) { Console.WriteLine("C.G({0})", i); }
    public static void H(this int i) { Console.WriteLine("C.H({0})", i); }
}

namespace N1
{
    public static class D
    {
        public static void F(this int i) { Console.WriteLine("D.F({0})", i); }
        public static void G(this int i) { Console.WriteLine("D.G({0})", i); }
    }
}

namespace N2
{
    using N1;

    public static class E
    {
        public static void F(this int i) { Console.WriteLine("E.F({0})", i); }
    }

    class Test
    {
        static void Main(string[] args)
        {
            1.F();
            2.G();
            3.H();
        }
    }
}
```

The output of this example is:
```
E.F(1)
D.G(2)
C.H(3)
```
`D.G` takes precedence over `C.G`, and `E.F` takes precedence over both `D.F` and `C.F`.

> __Expression Tree Conversion Translation Steps__
>
> An extension method invocation expression
>
> ```csharp
> instance.M(args)
> ```
>
> is translated to a query expression tree by first rewriting the expression to a static method invocation expression using the rules described above, and translating the result of this rewrite.
>
> For purposes of conversion to a generalized expression tree, it is translated into
>
> ```csharp
> Q.Call(info, instanceExpr, argsExpr)
> ```
>
> or, if no argument list is present,
>
> ```csharp
> Q.Call(info, instanceExpr)
> ```
>
> where `info` is
>
> ```csharp
> Q.CallInfo(flags, method)
> ```
>
> where `method` is an expression of type `MethodInfo` representing the bound method `M`, and `flags` is the bitwise `|` combination of the following:
>
> * `Q.Flags.InvokeExtensionMethod`, and
> * `Q.Flags.ResultDiscarded` if the expression occurs in an *expression_statement* ([Expression statements](statements.md#expression-statements)).
>
> Note that the use of `Q.Flags.InvokeExtensionMethod` enables expression tree libraries to distinguish between invoking an extension method using instance method invocation syntax and invoking it using static method invocation syntax. Expression tree libraries can trivially rewrite extension method invocation expressions into static method invocation expression by inspecting this flag, for instance for purposes of evaluating the expression.

#### Delegate invocations

For a delegate invocation, the *primary_expression* of the *invocation_expression* must be a value of a *delegate_type*. Furthermore, considering the *delegate_type* to be a function member with the same parameter list as the *delegate_type*, the *delegate_type* must be applicable ([Applicable function member](expressions.md#applicable-function-member)) with respect to the *argument_list* of the *invocation_expression*.

The run-time processing of a delegate invocation of the form `D(A)`, where `D` is a *primary_expression* of a *delegate_type* and `A` is an optional *argument_list*, consists of the following steps:

*  `D` is evaluated. If this evaluation causes an exception, no further steps are executed.
*  The value of `D` is checked to be valid. If the value of `D` is `null`, a `System.NullReferenceException` is thrown and no further steps are executed.
*  Otherwise, `D` is a reference to a delegate instance. Function member invocations ([Compile-time checking of dynamic overload resolution](expressions.md#compile-time-checking-of-dynamic-overload-resolution)) are performed on each of the callable entities in the invocation list of the delegate. For callable entities consisting of an instance and instance method, the instance for the invocation is the instance contained in the callable entity.

> __Expression Tree Conversion Translation Steps__
>
> A delegate invocation expression
>
> ```csharp
> f(args)
> ```
>
> is translated to an expression tree by first translating `f` to an expression tree `fExpr` and by translating the argument list `args` to `argsExpr`, if it exists. Note than translation of argument lists may insert cast expressions for required implicit conversions, and/or array initialization expressions for assignment to `params` parameters, classifying these expressions as compiler-generated (see [Argument lists](expressions.md#argument-lists)) Translation to an expression tree then proceeds as follows.
>
> For purposes of conversion to a query expression tree, if the delegate invocation is dynamically bound, translation fails and an error is reported. Otherwise, it is translated into
>
> ```csharp
> Expression.Invoke(fExpr, argsExpr)
> ```
>
> if an argument list `args` exists, or
>
> ```csharp
> Expression.Invoke(fExpr, empty)
> ```
>
> where `empty` is an expression of type `Expression[]` with zero elements otherwise.
>
> Note that the conversion of the argument list `args` to `argsExpr` for query expression trees yields an expression of type `Expression[]`, see [Argument lists](expressions.md#argument-lists). Therefore, overload resolution rules applied to bind the `Invoke` factory method invocation will always pick the `Invoke(Expression, Expression[])` overload, even if specialized overloads for low argument counts are available.
>
> Also note that the conversion for a delegate invocation expression without any arguments uses an expression representing an empty array `empty` rather than `new Expression[0]`. This provides the implementation the flexibility to use a shared instance of an empty array (e.g. `Array.Empty<Expression>()`).
>
> For purposes of conversion to a generalized expression tree, it is translated into
>
> ```csharp
> Q.Invoke(info, fExpr, argsExpr)
> ```
>
> or, if no argument list is present,
>
> ```csharp
> Q.Invoke(info, fExpr)
> ```
>
> where `info` is
>
> ```csharp
> Q.InvokeInfo(binder)
> ```
>
> if the delegate invocation is dynamically bound, where `binder` is an expression of type `Microsoft.CSharp.RuntimeBinder.CallSiteBinder` representing the dynamic operation, or
>
> ```csharp
> Q.InvokeInfo(flags)
> ```
>
> otherwise, where `flags` is
>
> * `Q.Flags.ResultDiscarded` if the expression occurs in an *expression_statement* ([Expression statements](statements.md#expression-statements)), or,
> * `default(Q.Flags)` otherwise.
>
> ***TODO***
> * Consider whether `argsExpr` should be a syntactic fragment of shape `(, argExpr)*` that gets concatenated to the argument list containing `info` and `fExpr`, thus flattening the first-class representation of an argument list and "inlining" it on the `Invoke` factory (much like query expression trees does).
> * Evaluate options for the dynamic case:
>   * create a specification for the construction of `binder` (can also be used to specify `dynamic` behavior outside the context of expression trees) and consider making more properties on the `Microsoft.CSharp.RuntimeBinder` library types publicly accessable so expression tree libraries can inspect this info, or,
>   * steer away from reusing `Microsoft.CSharp` and introduce a `InvokeInfo` factory overload for dynamic binding (e.g. a `(Q.DynamicFlags flags, Type context, params A[] argumentInfo)` overload, where `A` is built through factory invocations as well).

### Element access

An *element_access* consists of a *primary_no_array_creation_expression*, followed by a "`[`" token, followed by an *argument_list*, followed by a "`]`" token. The *argument_list* consists of one or more *argument*s, separated by commas.

```antlr
element_access
    : primary_no_array_creation_expression '[' expression_list ']'
    ;
```

The *argument_list* of an *element_access* is not allowed to contain `ref` or `out` arguments.

An *element_access* is dynamically bound ([Dynamic binding](expressions.md#dynamic-binding)) if at least one of the following holds:

* The *primary_no_array_creation_expression* has compile-time type `dynamic`.
* At least one expression of the *argument_list* has compile-time type `dynamic` and the *primary_no_array_creation_expression* does not have an array type.

In this case the compiler classifies the *element_access* as a value of type `dynamic`. The rules below to determine the meaning of the *element_access* are then applied at run-time, using the run-time type instead of the compile-time type of those of the *primary_no_array_creation_expression* and *argument_list* expressions which have the compile-time type `dynamic`. If the *primary_no_array_creation_expression* does not have compile-time type `dynamic`, then the element access undergoes a limited compile time check as described in [Compile-time checking of dynamic overload resolution](expressions.md#compile-time-checking-of-dynamic-overload-resolution).

If the *primary_no_array_creation_expression* of an *element_access* is a value of an *array_type*, the *element_access* is an array access ([Array access](expressions.md#array-access)). Otherwise, the *primary_no_array_creation_expression* must be a variable or value of a class, struct, or interface type that has one or more indexer members, in which case the *element_access* is an indexer access ([Indexer access](expressions.md#indexer-access)).

#### Array access

For an array access, the *primary_no_array_creation_expression* of the *element_access* must be a value of an *array_type*. Furthermore, the *argument_list* of an array access is not allowed to contain named arguments.The number of expressions in the *argument_list* must be the same as the rank of the *array_type*, and each expression must be of type `int`, `uint`, `long`, `ulong`, or must be implicitly convertible to one or more of these types.

The result of evaluating an array access is a variable of the element type of the array, namely the array element selected by the value(s) of the expression(s) in the *argument_list*.

The run-time processing of an array access of the form `P[A]`, where `P` is a *primary_no_array_creation_expression* of an *array_type* and `A` is an *argument_list*, consists of the following steps:

*  `P` is evaluated. If this evaluation causes an exception, no further steps are executed.
*  The index expressions of the *argument_list* are evaluated in order, from left to right. Following evaluation of each index expression, an implicit conversion ([Implicit conversions](conversions.md#implicit-conversions)) to one of the following types is performed: `int`, `uint`, `long`, `ulong`. The first type in this list for which an implicit conversion exists is chosen. For instance, if the index expression is of type `short` then an implicit conversion to `int` is performed, since implicit conversions from `short` to `int` and from `short` to `long` are possible. If evaluation of an index expression or the subsequent implicit conversion causes an exception, then no further index expressions are evaluated and no further steps are executed.
*  The value of `P` is checked to be valid. If the value of `P` is `null`, a `System.NullReferenceException` is thrown and no further steps are executed.
*  The value of each expression in the *argument_list* is checked against the actual bounds of each dimension of the array instance referenced by `P`. If one or more values are out of range, a `System.IndexOutOfRangeException` is thrown and no further steps are executed.
*  The location of the array element given by the index expression(s) is computed, and this location becomes the result of the array access.

> __Expression Tree Conversion Translation Steps__
>
> An array access expression
>
> ```csharp
> array[args]
> ```
>
> is translated to an expression tree by first pre-processing the expression by rewriting each `argExpr` in the `args` argument list to
>
> ```csharp
> checked((T)(argExpr))
> ```
>
> where `T` is `int` if the expression is translated to a query expression tree and `argExpr` is not of type `int`, or where `T` is any of the types `int`, `uint`, `long`, or `ulong` if the expression is translated to a generalized expression tree and an implicit conversion is needed ([Implicit conversions](conversions.md#implicit-conversions)). If no conversions are required, `argExpr` remains unchanged. Note this step may introduce a cast expression which will be classified as compiler-generated. This rewritten index expression will subsequently be referred to as `indexExpr`, and the resulting argument list will be referred to as `indices`. Finally, convert the argument list `indices` to an expression tree `indicesExpr`. Translation to an expression tree then proceeds as follows.
>
> For purposes of conversion to a query expression tree, it is translated into
>
> ```csharp
> Expression.ArrayIndex(arrayExpr, indicesExpr)
> ```
>
> if the argument list `indices` has more than one argument, or
>
> ```csharp
> Expression.ArrayIndex(arrayExpr, indexExpr)
> ```
>
> where `indexExpr` is the result of translating the single index expression to an expression tree.
>
> Note that the conversion of the argument list `indices` to `indicesExpr` for query expression trees yields an expression of type `Expression[]`, see [Argument lists](expressions.md#argument-lists). Therefore, binding to the `ArrayIndex` factory method for indexers with more than one index expression will always pick the `ArrayIndex(Expression, Expression[])` overload, even if specialized overloads for low argument counts are available. For historical reasons, query expression tree factory methods have distinguished between one indexer versus more than one.
>
> It is also worth pointing out that the `ArrayIndex(Expression, Expression[])` overload returns an expression of type `MethodCallExpression` where the target method is the `Get` method on the multi-dimensional array type. This design is historical and also prevents the use of this node for purposes of assignment.
>
> For purposes of conversion to a generalized expression tree, it is translated into
>
> ```csharp
> Q.ArrayIndex(info, arrayExpr, indicesExpr)
> ```
>
> where `info` is
>
> ```csharp
> Q.ArrayIndexInfo(default(Q.Flags))
> ```
>
> where the first flags argument is reserved for future use.

#### Indexer access

For an indexer access, the *primary_no_array_creation_expression* of the *element_access* must be a variable or value of a class, struct, or interface type, and this type must implement one or more indexers that are applicable with respect to the *argument_list* of the *element_access*.

The binding-time processing of an indexer access of the form `P[A]`, where `P` is a *primary_no_array_creation_expression* of a class, struct, or interface type `T`, and `A` is an *argument_list*, consists of the following steps:

*  The set of indexers provided by `T` is constructed. The set consists of all indexers declared in `T` or a base type of `T` that are not `override` declarations and are accessible in the current context ([Member access](basic-concepts.md#member-access)).
*  The set is reduced to those indexers that are applicable and not hidden by other indexers. The following rules are applied to each indexer `S.I` in the set, where `S` is the type in which the indexer `I` is declared:
   * If `I` is not applicable with respect to `A` ([Applicable function member](expressions.md#applicable-function-member)), then `I` is removed from the set.
   * If `I` is applicable with respect to `A` ([Applicable function member](expressions.md#applicable-function-member)), then all indexers declared in a base type of `S` are removed from the set.
   * If `I` is applicable with respect to `A` ([Applicable function member](expressions.md#applicable-function-member)) and `S` is a class type other than `object`, all indexers declared in an interface are removed from the set.
*  If the resulting set of candidate indexers is empty, then no applicable indexers exist, and a binding-time error occurs.
*  The best indexer of the set of candidate indexers is identified using the overload resolution rules of [Overload resolution](expressions.md#overload-resolution). If a single best indexer cannot be identified, the indexer access is ambiguous, and a binding-time error occurs.
*  The index expressions of the *argument_list* are evaluated in order, from left to right. The result of processing the indexer access is an expression classified as an indexer access. The indexer access expression references the indexer determined in the step above, and has an associated instance expression of `P` and an associated argument list of `A`.

Depending on the context in which it is used, an indexer access causes invocation of either the *get accessor* or the *set accessor* of the indexer. If the indexer access is the target of an assignment, the *set accessor* is invoked to assign a new value ([Simple assignment](expressions.md#simple-assignment)). In all other cases, the *get accessor* is invoked to obtain the current value ([Values of expressions](expressions.md#values-of-expressions)).

> __Expression Tree Conversion Translation Steps__
>
> A indexer access expression
>
> ```csharp
> obj[args]
> ```
>
> is translated to an expression tree by first translating `obj` to an expression tree `objExpr` and by translating the argument list `args` to `argsExpr`. Note than translation of argument lists may insert cast expressions for required implicit conversions, and/or array initialization expressions for assignment to `params` parameters, classifying these expressions as compiler-generated (see [Argument lists](expressions.md#argument-lists)). Translation to an expression tree then proceeds as follows.
>
> For purposes of conversion to a query expression tree, if the indexer access expression is dynamically bound, translation fails and an error is reported. Otherwise, it is translated into
>
> ```csharp
> Expression.Call(objExpr, accessorMethod, argsExpr)
> ```
>
> where `accessorMethod` is an expression of type `MethodInfo` representing the `get` accessor method of the bound indexer. The `get` accessor can either be declared on the type of `objExpr`, or can be inherited from a base class. To locate the bound `get` accessor method, the class hierarchy is walked starting from the type of `objExpr`.
>
> Note that for purposes of compatibility, translation of query expression trees does not bind to `Expression.MakeIndex` factory methods, which return an `IndexExpression` that supports assignment.
>
> For purposes of conversion to a generalized expression tree, it is translated into
>
> ```csharp
> Q.Index(info, objExpr, argsExpr)
> ```
>
> where `info` is
>
> ```csharp
> Q.IndexInfo(binder)
> ```
>
> or
>
> ```csharp
> Q.IndexInfo(getBinder, setBinder)
> ```
>
> if the indexer access is dynamically bound, where `binder`, `getBinder`, and `setBinder` are expressions of type `Microsoft.CSharp.RuntimeBinder.CallSiteBinder` representing the dynamic operation(s) involved depending on the use site of the indexer access expression (in particular, two binders are needed for compound assignment), or
>
> ```csharp
> Q.IndexInfo(default(Q.Flags), accessorMethod)
> ```
>
> where the first flags argument is reserved for future use, and where `accessorMethod` is an expression of type `MethodInfo` representing the `get` or `set` accessor method of the bound indexer.
>
> If both a `get` and `set` accessor are available, it is left to the implementation which accessor method is passed to `IndexInfo`. This situation can arise for compound assignment expressions where the indexer is used both to read and to assign. Expression tree libraries should use the `MethodInfo` to locate the `PropertyInfo` of the corresponding accessor.
>
>> The limitation of passing a `MethodInfo` rather than a `PropertyInfo` object is due to restrictions in code-generation. In particular, it is easy to obtain a `MethodInfo` object through a `ldtoken` instruction and a call to `MethodBase.GetMethodFromHandle`. However, it is non-trivial to locate the corresponding `PropertyInfo` without performing complex code generation against `System.Reflection` APIs.
>
> Note that generalized expression trees may refer to an indexer using the `set` accessor. This supports `set`-only indexers used in assignment expressions.
>
> ***TODO***:
> * Evaluate options for the dynamic case:
>   * create a specification for the construction of `binder` (can also be used to specify `dynamic` behavior outside the context of expression trees) and consider making more properties on the `Microsoft.CSharp.RuntimeBinder` library types publicly accessable so expression tree libraries can inspect this info, or,
>   * steer away from reusing `Microsoft.CSharp` and introduce a `IndexInfo` factory overload for dynamic binding (e.g. a `(Q.DynamicFlags flags, Type context, params A[] argumentInfo)` overload, where `A` is built through factory invocations as well), and,
>   * define how to deal with compound assignment which requires a `GetIndex` and `SetIndex` binder; do we need both?

### This access

A *this_access* consists of the reserved word `this`.

```antlr
this_access
    : 'this'
    ;
```

A *this_access* is permitted only in the *block* of an instance constructor, an instance method, or an instance accessor. It has one of the following meanings:

*  When `this` is used in a *primary_expression* within an instance constructor of a class, it is classified as a value. The type of the value is the instance type ([The instance type](classes.md#the-instance-type)) of the class within which the usage occurs, and the value is a reference to the object being constructed.
*  When `this` is used in a *primary_expression* within an instance method or instance accessor of a class, it is classified as a value. The type of the value is the instance type ([The instance type](classes.md#the-instance-type)) of the class within which the usage occurs, and the value is a reference to the object for which the method or accessor was invoked.
*  When `this` is used in a *primary_expression* within an instance constructor of a struct, it is classified as a variable. The type of the variable is the instance type ([The instance type](classes.md#the-instance-type)) of the struct within which the usage occurs, and the variable represents the struct being constructed. The `this` variable of an instance constructor of a struct behaves exactly the same as an `out` parameter of the struct type—in particular, this means that the variable must be definitely assigned in every execution path of the instance constructor.
*  When `this` is used in a *primary_expression* within an instance method or instance accessor of a struct, it is classified as a variable. The type of the variable is the instance type ([The instance type](classes.md#the-instance-type)) of the struct within which the usage occurs.
   * If the method or accessor is not an iterator ([Iterators](classes.md#iterators)), the `this` variable represents the struct for which the method or accessor was invoked, and behaves exactly the same as a `ref` parameter of the struct type.
   * If the method or accessor is an iterator, the `this` variable represents a copy of the struct for which the method or accessor was invoked, and behaves exactly the same as a value parameter of the struct type.

Use of `this` in a *primary_expression* in a context other than the ones listed above is a compile-time error. In particular, it is not possible to refer to `this` in a static method, a static property accessor, or in a *variable_initializer* of a field declaration.

> __Expression Tree Conversion Translation Steps__
>
> A *this_access* expression
>
> ```csharp
> this
> ```
>
> is translated to a query expression tree as
>
> ```csharp
> Expression.Constant(this, type)
> ```
>
> and is translated to a generalized expression tree as
>
> ```csharp
> Q.Constant(flags, literal, type)
> ```
>
> where `flags` is an expression of type `Q.Flags` with value `Q.Flags.This` optionally bitwise `|` combined with `Q.CompilerGenerated` if the node was compiler-generated (for instance when `this` was not user-specified in an instance method invocation),
>
> and where `type` is an expression of type `Type` representing the instance type ([The instance type](classes.md#the-instance-type)) of the class within which the usage occurs.
>
> Note that generalized expression trees retain information about use of `this` using a flag passed to the `Constant` factory method.

### Base access

A *base_access* consists of the reserved word `base` followed by either a "`.`" token and an identifier or an *argument_list* enclosed in square brackets:

```antlr
base_access
    : 'base' '.' identifier
    | 'base' '[' expression_list ']'
    ;
```

A *base_access* is used to access base class members that are hidden by similarly named members in the current class or struct. A *base_access* is permitted only in the *block* of an instance constructor, an instance method, or an instance accessor. When `base.I` occurs in a class or struct, `I` must denote a member of the base class of that class or struct. Likewise, when `base[E]` occurs in a class, an applicable indexer must exist in the base class.

At binding-time, *base_access* expressions of the form `base.I` and `base[E]` are evaluated exactly as if they were written `((B)this).I` and `((B)this)[E]`, where `B` is the base class of the class or struct in which the construct occurs. Thus, `base.I` and `base[E]` correspond to `this.I` and `this[E]`, except `this` is viewed as an instance of the base class.

When a *base_access* references a virtual function member (a method, property, or indexer), the determination of which function member to invoke at run-time ([Compile-time checking of dynamic overload resolution](expressions.md#compile-time-checking-of-dynamic-overload-resolution)) is changed. The function member that is invoked is determined by finding the most derived implementation ([Virtual methods](classes.md#virtual-methods)) of the function member with respect to `B` (instead of with respect to the run-time type of `this`, as would be usual in a non-base access). Thus, within an `override` of a `virtual` function member, a *base_access* can be used to invoke the inherited implementation of the function member. If the function member referenced by a *base_access* is abstract, a binding-time error occurs.

> __Expression Tree Conversion Translation Steps__
>
> A *base_access* expression is not supported in expression trees.

### Postfix increment and decrement operators

```antlr
post_increment_expression
    : primary_expression '++'
    ;

post_decrement_expression
    : primary_expression '--'
    ;
```

The operand of a postfix increment or decrement operation must be an expression classified as a variable, a property access, or an indexer access. The result of the operation is a value of the same type as the operand.

If the *primary_expression* has the compile-time type `dynamic` then the operator is dynamically bound ([Dynamic binding](expressions.md#dynamic-binding)), the *post_increment_expression* or *post_decrement_expression* has the compile-time type `dynamic` and the following rules are applied at run-time using the run-time type of the *primary_expression*.

If the operand of a postfix increment or decrement operation is a property or indexer access, the property or indexer must have both a `get` and a `set` accessor. If this is not the case, a binding-time error occurs.

Unary operator overload resolution ([Unary operator overload resolution](expressions.md#unary-operator-overload-resolution)) is applied to select a specific operator implementation. Predefined `++` and `--` operators exist for the following types: `sbyte`, `byte`, `short`, `ushort`, `int`, `uint`, `long`, `ulong`, `char`, `float`, `double`, `decimal`, and any enum type. The predefined `++` operators return the value produced by adding 1 to the operand, and the predefined `--` operators return the value produced by subtracting 1 from the operand. In a `checked` context, if the result of this addition or subtraction is outside the range of the result type and the result type is an integral type or enum type, a `System.OverflowException` is thrown.

The run-time processing of a postfix increment or decrement operation of the form `x++` or `x--` consists of the following steps:

*   If `x` is classified as a variable:
    * `x` is evaluated to produce the variable.
    * The value of `x` is saved.
    * The selected operator is invoked with the saved value of `x` as its argument.
    * The value returned by the operator is stored in the location given by the evaluation of `x`.
    * The saved value of `x` becomes the result of the operation.
*   If `x` is classified as a property or indexer access:
    * The instance expression (if `x` is not `static`) and the argument list (if `x` is an indexer access) associated with `x` are evaluated, and the results are used in the subsequent `get` and `set` accessor invocations.
    * The `get` accessor of `x` is invoked and the returned value is saved.
    * The selected operator is invoked with the saved value of `x` as its argument.
    * The `set` accessor of `x` is invoked with the value returned by the operator as its `value` argument.
    * The saved value of `x` becomes the result of the operation.

The `++` and `--` operators also support prefix notation ([Prefix increment and decrement operators](expressions.md#prefix-increment-and-decrement-operators)). Typically, the result of `x++` or `x--` is the value of `x` before the operation, whereas the result of `++x` or `--x` is the value of `x` after the operation. In either case, `x` itself has the same value after the operation.

An `operator ++` or `operator --` implementation can be invoked using either postfix or prefix notation. It is not possible to have separate operator implementations for the two notations.

> __Expression Tree Conversion Translation Steps__
>
> A `++` and `--` postfix increment and decrement expressions
>
> ```csharp
> expression++
> expression--
> ```
>
> are translated into an expression tree by first translating `expression` to `expr`, and by optionally constructing an expression `method` of type `MethodInfo` representing the user-defined `operator ++` or `operator --` implementation, if the operator is bound to a user-defined operator. Expression tree translation then proceeds as follows.
>
> When subject to conversion to a query expression tree, an error is reported.
>
> When subject to conversion to a generalized expression tree, it is translated into either of
>
> ```csharp
> Q.PostIncrement(info, expr)
> Q.PostDecrement(info, expr)
> ```
>
> where `info` is either of
>
> ```csharp
> Q.PostIncrementInfo(binder)
> Q.PostDecrementInfo(binder)
> ```
>
> if `expression` is of type `dynamic`, where `binder` is an expression of type `Microsoft.CSharp.RuntimeBinder.CallSiteBinder` representing the dynamic operation, or
>
> ```csharp
> Q.PostIncrementInfo(flags, method)
> Q.PostDecrementInfo(flags, method)
> ```
>
> if the operator is bound to a user-defined implementation of the `operator ++` or `operator --`, or
>
> ```csharp
> Q.PostIncrementInfo(flags)
> Q.PostDecrementInfo(flags)
> ```
>
> otherwise, where `flags` is the bitwise `|` combination of
>
> * `Q.Flags.ResultDiscarded` if the expression occurs in an *expression_statement* ([Expression statements](statements.md#expression-statements)), and,
> * `Q.Flags.CheckedContext` if the prefix increment or decrement expression occurs in a checked context, or,
> * `default(Q.Flags)` if none of the flags apply.
>
> Note that the behavior of a user-defined increment or decrement operator is typically not affected by the use in a checked context. For purposes of generalized expression trees, this information is retained for all operators, and passed as a flag. The bound expression tree library is free to ignore this information.
>
> ***TODO***:
> * Evaluate options for the dynamic case:
>   * create a specification for the construction of `binder` (can also be used to specify `dynamic` behavior outside the context of expression trees) and consider making more properties on the `Microsoft.CSharp.RuntimeBinder` library types publicly accessable so expression tree libraries can inspect this info, or,
>   * steer away from reusing `Microsoft.CSharp` and introduce `Post*Info` factory overloads for dynamic binding (e.g. a `(Q.DynamicFlags flags, Type context, A argumentInfo)` overload, where `A` is built through factory invocations as well).

### The new operator

The `new` operator is used to create new instances of types.

There are three forms of `new` expressions:

*  Object creation expressions are used to create new instances of class types and value types.
*  Array creation expressions are used to create new instances of array types.
*  Delegate creation expressions are used to create new instances of delegate types.

The `new` operator implies creation of an instance of a type, but does not necessarily imply dynamic allocation of memory. In particular, instances of value types require no additional memory beyond the variables in which they reside, and no dynamic allocations occur when `new` is used to create instances of value types.

#### Object creation expressions

An *object_creation_expression* is used to create a new instance of a *class_type* or a *value_type*.

```antlr
object_creation_expression
    : 'new' type '(' argument_list? ')' object_or_collection_initializer?
    | 'new' type object_or_collection_initializer
    ;

object_or_collection_initializer
    : object_initializer
    | collection_initializer
    ;
```

The *type* of an *object_creation_expression* must be a *class_type*, a *value_type* or a *type_parameter*. The *type* cannot be an `abstract` *class_type*.

The optional *argument_list* ([Argument lists](expressions.md#argument-lists)) is permitted only if the *type* is a *class_type* or a *struct_type*.

An object creation expression can omit the constructor argument list and enclosing parentheses provided it includes an object initializer or collection initializer. Omitting the constructor argument list and enclosing parentheses is equivalent to specifying an empty argument list.

Processing of an object creation expression that includes an object initializer or collection initializer consists of first processing the instance constructor and then processing the member or element initializations specified by the object initializer ([Object initializers](expressions.md#object-initializers)) or collection initializer ([Collection initializers](expressions.md#collection-initializers)).

If any of the arguments in the optional *argument_list* has the compile-time type `dynamic` then the *object_creation_expression* is dynamically bound ([Dynamic binding](expressions.md#dynamic-binding)) and the following rules are applied at run-time using the run-time type of those arguments of the *argument_list* that have the compile time type `dynamic`. However, the object creation undergoes a limited compile time check as described in [Compile-time checking of dynamic overload resolution](expressions.md#compile-time-checking-of-dynamic-overload-resolution).

The binding-time processing of an *object_creation_expression* of the form `new T(A)`, where `T` is a *class_type* or a *value_type* and `A` is an optional *argument_list*, consists of the following steps:

*   If `T` is a *value_type* and `A` is not present:
    * The *object_creation_expression* is a default constructor invocation. The result of the *object_creation_expression* is a value of type `T`, namely the default value for `T` as defined in [The System.ValueType type](types.md#the-systemvaluetype-type).
*   Otherwise, if `T` is a *type_parameter* and `A` is not present:
    * If no value type constraint or constructor constraint ([Type parameter constraints](classes.md#type-parameter-constraints)) has been specified for `T`, a binding-time error occurs.
    * The result of the *object_creation_expression* is a value of the run-time type that the type parameter has been bound to, namely the result of invoking the default constructor of that type. The run-time type may be a reference type or a value type.
*   Otherwise, if `T` is a *class_type* or a *struct_type*:
    * If `T` is an `abstract` *class_type*, a compile-time error occurs.
    * The instance constructor to invoke is determined using the overload resolution rules of [Overload resolution](expressions.md#overload-resolution). The set of candidate instance constructors consists of all accessible instance constructors declared in `T` which are applicable with respect to `A` ([Applicable function member](expressions.md#applicable-function-member)). If the set of candidate instance constructors is empty, or if a single best instance constructor cannot be identified, a binding-time error occurs.
    * The result of the *object_creation_expression* is a value of type `T`, namely the value produced by invoking the instance constructor determined in the step above.
*  Otherwise, the *object_creation_expression* is invalid, and a binding-time error occurs.

Even if the *object_creation_expression* is dynamically bound, the compile-time type is still `T`.

The run-time processing of an *object_creation_expression* of the form `new T(A)`, where `T` is *class_type* or a *struct_type* and `A` is an optional *argument_list*, consists of the following steps:

*   If `T` is a *class_type*:
    * A new instance of class `T` is allocated. If there is not enough memory available to allocate the new instance, a `System.OutOfMemoryException` is thrown and no further steps are executed.
    * All fields of the new instance are initialized to their default values ([Default values](variables.md#default-values)).
    * The instance constructor is invoked according to the rules of function member invocation ([Compile-time checking of dynamic overload resolution](expressions.md#compile-time-checking-of-dynamic-overload-resolution)). A reference to the newly allocated instance is automatically passed to the instance constructor and the instance can be accessed from within that constructor as `this`.
*   If `T` is a *struct_type*:
    * An instance of type `T` is created by allocating a temporary local variable. Since an instance constructor of a *struct_type* is required to definitely assign a value to each field of the instance being created, no initialization of the temporary variable is necessary.
    * The instance constructor is invoked according to the rules of function member invocation ([Compile-time checking of dynamic overload resolution](expressions.md#compile-time-checking-of-dynamic-overload-resolution)). A reference to the newly allocated instance is automatically passed to the instance constructor and the instance can be accessed from within that constructor as `this`.

> __Expression Tree Conversion Translation Steps__
>
> A `new` object creation expression
>
> ```csharp
> new T(args)
> ```
>
> with no *object_initializer* (see [Object initializer](expressions.md#object-initializer)) and no *collection_initializer* (see [Collection initializer](expressions.md#collection-initializer)) is translated into an expression tree by first translating the argument list `args` to `argsExpr`. Note than translation of argument lists may insert cast expressions for required implicit conversions, and/or array initialization expressions for assignment to `params` parameters, classifying these expressions as compiler-generated (see [Argument lists](expressions.md#argument-lists)). Expression tree translation then proceeds as follows.
>
> When subject to conversion to a query expression tree, if the object creation operation is dynamically bound, translation fails and an error is reported. Otherwise, it is translated into
>
> ```csharp
> Expression.New(type)
> ```
>
> where `type` is an expression of type `Type` if no argument list is specified. Note that this rule applies regardless of whether `T` is a struct, a class, or a generic parameter type with a `new()` constraint. The `Expression.New` factory does infer a `ConstructorInfo` object if `T` is not a struct.
>
> Otherwise, if an argument list is specified, it is translated into
>
> ```csharp
> Expression.New(constructor, argsExpr)
> ```
>
> where `constructor` is an object of type `ConstructorInfo` representing the constructor bound by for the object creation expression.
>
> Note that the conversion of the argument list `args` to `argsExpr` for query expression trees yields an expression of type `Expression[]`, see [Argument lists](expressions.md#argument-lists). Therefore, overload resolution rules applied to bind the `New` factory method invocation will always pick the `New(ConstructorInfo, Expression[])` overload, even if specialized overloads for low argument counts are available.
>
> When subject to conversion to a generalized expression tree, it is translated into
>
> ```csharp
> Q.New(info, argsExpr)
> ```
>
> or, if no argument list is present,
>
> ```csharp
> Q.New(info)
> ```
>
> where `info` is either of
>
> ```csharp
> Q.NewInfo(type, binder)
> ```
>
> if the object creation expression is dynamically bound, where `binder` is an expression of type `Microsoft.CSharp.RuntimeBinder.CallSiteBinder` representing the dynamic operation, and where `type` is an expression of type `Type` representing the type `T`, or
>
> ```csharp
> Q.NewInfo(flags, type)
> ```
>
> where `type` is an expression of type `Type`, if `T` is a struct type and no argument list is present, or if `T` is a generic parameter type with a `new()` constraint, or
>
> ```csharp
> Q.NewInfo(flags, constructor)
> ```
>
> otherwise, where `constructor` is an object of type `ConstructorInfo` representing the constructor bound by for the object creation expression, and `flags` is
>
> * `Q.Flags.ResultDiscarded` if the expression is occurs in an *expression_statement* ([Expression statements](statements.md#expression-statements)), or,
> * `default(Q.Flags)` otherwise.
>
> Note that generalized expression trees prefer binding to a `NewInfo` factory with a `ConstructorInfo` if one is available, even if the object creation expression has no argument list specified. This reduces the need for expression tree libraries to use reflection to find the parameterless constructor.
>
> ***TODO***:
> * Evaluate options for the dynamic case:
>   * create a specification for the construction of `binder` (can also be used to specify `dynamic` behavior outside the context of expression trees) and consider making more properties on the `Microsoft.CSharp.RuntimeBinder` library types publicly accessable so expression tree libraries can inspect this info, or,
>   * steer away from reusing `Microsoft.CSharp` and introduce a `NewInfo` factory overload for dynamic binding (e.g. a `(Q.DynamicFlags flags, Type context, params A[] argumentInfo)` overload, where `A` is built through factory invocations as well).
>
>
> An object creation expression with an *object_initializer*
>
> ```csharp
> newObj { member_init_1, ..., member_init_N }
> ```
>
> where `newObj` is either `new T` or `new T(args)` is translated to an expression tree by first rewriting `new T` to `new T()` and translating the resulting expression as an object creation expression to an expression tree `newExpr`. Next, each *member_initializer* `member_init_i` (for `1 <= i <= N`) is translated to an expression tree `initExpr_i` (see [Object initializers](expressions.md#object-initializers)). Expression tree conversion then proceeds as follows.
>
> When converted to a query expression tree, it is translated into
>
> ```csharp
> newExpr
> ```
>
> if the member initializer list is empty (that is, `N` is `0`), or
>
> ```csharp
> Expression.MemberInit(newExpr, memberBindings)
> ```
>
> where `memberBindings` is an expression of type `System.Linq.Expressions.MemberBinding[]` representing an array of length `N` with elements `initExpr_1` to `initExpr_N` otherwise.
>
> When converted to a generalized expression tree, it is translated into
>
> ```csharp
> Q.NewObjectInit(flags, newExpr, memberBindings)
> ```
>
> where `flags` is
>
> * `Q.Flags.ResultDiscarded` if the expression is occurs in an *expression_statement* ([Expression statements](statements.md#expression-statements)), or,
> * `default(Q.Flags)` otherwise,
>
> and where `memberBindings` is
>
> ```csharp
> Q.MemberBindings(initExpr_1, ..., initExpr_N)
> ```
>
> Note that generalized expression trees do not take away empty object initializer lists. Also note that member bindings are represented using a first-class expression node, allowing reuse for ***nested object initializers*** (see [Object initializers](expressions.md#object-initializers)). 
>
>
>
> An object creation expression with a *collection_initializer*
>
> ```csharp
> newObj { element_init_1, ..., element_init_N }
> ```
>
> where `newObj` is either `new T` or `new T(args)` is translated to an expression tree by first rewriting `new T` to `new T()` and translating the resulting expression as an object creation expression to an expression tree `newExpr`. Next, each *element_initializer* `element_init_i` (for `1 <= i <= N`) is translated to an expression tree `initExpr_i` (see [Collection initializers](expressions.md#collection-initializers)). Expression tree conversion then proceeds as follows.
>
> When converted to a query expression tree, it is translated into
>
> ```csharp
> Expression.ListInit(newExpr, elementInitializers)
> ```
>
> where `elementInitializers` is an expression of type `System.Linq.Expressions.ElementInit[]` representing an array of length `N` with elements `initExpr_1` to `initExpr_N` otherwise.
>
> When converted to a generalized expression tree, it is translated into
>
> ```csharp
> Q.NewCollectionInit(flags, newExpr, elementInitializers)
> ```
>
> where `flags` is
>
> * `Q.Flags.ResultDiscarded` if the expression is occurs in an *expression_statement* ([Expression statements](statements.md#expression-statements)), or,
> * `default(Q.Flags)` otherwise,
>
> and where `elementInitializers` is
>
> ```csharp
> Q.ElementInitializers(initExpr_1, ..., initExpr_N)
> ```
>
> Note that element initializers are represented using a first-class expression node, allowing reuse for ***nested object initializers*** (see [Object initializers](expressions.md#object-initializers)). 

#### Object initializers

An ***object initializer*** specifies values for zero or more fields, properties or indexed elements of an object.

```antlr
object_initializer
    : '{' member_initializer_list? '}'
    | '{' member_initializer_list ',' '}'
    ;

member_initializer_list
    : member_initializer (',' member_initializer)*
    ;

member_initializer
    : initializer_target '=' initializer_value
    ;

initializer_target
    : identifier
    | '[' argument_list ']'
    ;

initializer_value
    : expression
    | object_or_collection_initializer
    ;
```

An object initializer consists of a sequence of member initializers, enclosed by `{` and `}` tokens and separated by commas. Each *member_initializer* designates a target for the initialization. An *identifier* must name an accessible field or property of the object being initialized, whereas an *argument_list* enclosed in square brackets must specify arguments for an accessible indexer on the object being initialized. It is an error for an object initializer to include more than one member initializer for the same field or property.

Each *initializer_target* is followed by an equals sign and either an expression, an object initializer or a collection initializer. It is not possible for expressions within the object initializer to refer to the newly created object it is initializing.

A member initializer that specifies an expression after the equals sign is processed in the same way as an assignment ([Simple assignment](expressions.md#simple-assignment)) to the target.

A member initializer that specifies an object initializer after the equals sign is a ***nested object initializer***, i.e. an initialization of an embedded object. Instead of assigning a new value to the field or property, the assignments in the nested object initializer are treated as assignments to members of the field or property. Nested object initializers cannot be applied to properties with a value type, or to read-only fields with a value type.

A member initializer that specifies a collection initializer after the equals sign is an initialization of an embedded collection. Instead of assigning a new collection to the target field, property or indexer, the elements given in the initializer are added to the collection referenced by the target. The target must be of a collection type that satisfies the requirements specified in [Collection initializers](expressions.md#collection-initializers).

The arguments to an index initializer will always be evaluated exactly once. Thus, even if the arguments end up never getting used (e.g. because of an empty nested initializer), they will be evaluated for their side effects.

The following class represents a point with two coordinates:
```csharp
public class Point
{
    int x, y;

    public int X { get { return x; } set { x = value; } }
    public int Y { get { return y; } set { y = value; } }
}
```

An instance of `Point` can be created and initialized as follows:
```csharp
Point a = new Point { X = 0, Y = 1 };
```
which has the same effect as
```csharp
Point __a = new Point();
__a.X = 0;
__a.Y = 1; 
Point a = __a;
```
where `__a` is an otherwise invisible and inaccessible temporary variable. The following class represents a rectangle created from two points:
```csharp
public class Rectangle
{
    Point p1, p2;

    public Point P1 { get { return p1; } set { p1 = value; } }
    public Point P2 { get { return p2; } set { p2 = value; } }
}
```

An instance of `Rectangle` can be created and initialized as follows:
```csharp
Rectangle r = new Rectangle {
    P1 = new Point { X = 0, Y = 1 },
    P2 = new Point { X = 2, Y = 3 }
};
```
which has the same effect as
```csharp
Rectangle __r = new Rectangle();
Point __p1 = new Point();
__p1.X = 0;
__p1.Y = 1;
__r.P1 = __p1;
Point __p2 = new Point();
__p2.X = 2;
__p2.Y = 3;
__r.P2 = __p2; 
Rectangle r = __r;
```
where `__r`, `__p1` and `__p2` are temporary variables that are otherwise invisible and inaccessible.

If `Rectangle`'s constructor allocates the two embedded `Point` instances
```csharp
public class Rectangle
{
    Point p1 = new Point();
    Point p2 = new Point();

    public Point P1 { get { return p1; } }
    public Point P2 { get { return p2; } }
}
```
the following construct can be used to initialize the embedded `Point` instances instead of assigning new instances:
```csharp
Rectangle r = new Rectangle {
    P1 = { X = 0, Y = 1 },
    P2 = { X = 2, Y = 3 }
};
```
which has the same effect as
```csharp
Rectangle __r = new Rectangle();
__r.P1.X = 0;
__r.P1.Y = 1;
__r.P2.X = 2;
__r.P2.Y = 3;
Rectangle r = __r;
```

Given an appropriate definition of C, the following example:
```csharp
var c = new C {
    x = true,
    y = { a = "Hello" },
    z = { 1, 2, 3 },
    ["x"] = 5,
    [0,0] = { "a", "b" },
    [1,2] = {}
};
```
is equivalent to this series of assignments:
```csharp
C __c = new C();
__c.x = true;
__c.y.a = "Hello";
__c.z.Add(1); 
__c.z.Add(2);
__c.z.Add(3);
string __i1 = "x";
__c[__i1] = 5;
int __i2 = 0, __i3 = 0;
__c[__i2,__i3].Add("a");
__c[__i2,__i3].Add("b");
int __i4 = 1, __i5 = 2;
var c = __c;
```
where `__c`, etc., are generated variables that are invisible and inaccessible to the source code. Note that the arguments for `[0,0]` are evaluated only once, and the arguments for `[1,2]` are evaluated once even though they are never used.

> __Expression Tree Conversion Translation Steps__
>
> Translation of a *member_initializer*
>
> ```antlr
> member_initializer
>     : initializer_target '=' initializer_value
>     ;
>
> initializer_target
>     : identifier
>     | '[' argument_list ']'
>     ;
>
> initializer_value
>     : expression
>     | object_or_collection_initializer
>     ;
> ```
>
> to an expression tree proceeds as follows.
>
> First, create an expression `member` of type `MemberInfo` representing the bound *initializer_target*:
>
> * for an *identifier*, either a `FieldInfo` if *identifier* binds to a field, or a `MethodInfo` for the `set` accessor of the property bound by *identifier*,
> * for an indexer assignment, the `MethodInfo` for the `set` accessor of the indexer bound by the initializer target. Expression tree translation then proceeds as follows.
>
> If *initializer_value* is an expression, create an expression `expr` by
>
> * translating `(T)(expression)` to an expression tree, if assignment ([Simple assignment](expressions.md#simple-assignment)) to the target requires an implicit conversion to type `T` which will be classified as compiler-generated, or
> * translating `expression` to an expression tree, otherwise.
>
> When converting to a query expression tree, if an indexer assignment is used, a compile-time error occurs. Otherwise, it is translated to:
>
> ```csharp
> Expression.Bind(member, expr)
> ```
>
> if *initializer_value* is an expression, or
>
> ```csharp
> Expression.MemberBind(member, bindings)
> ```
>
> if *initializer_value* is an object initializer, where `bindings` is an expression of type `System.Linq.Expressions.MemberBinding[]` with elements containing the result of translating each nested *member_initializer* to an expression tree, or
>
> ```csharp
> Expression.ListBind(member, initializers)
> ```
>
> if *initializer_value* is a collection initializer, where `initializers` is an expression of type `System.Linq.Expressions.ElementInit[]` with elements containing the result of translating each nested *element_initializer* to an expression tree.
>
> When converted to a generalized expression tree, first define an expression `target` as
>
> ```csharp
> Q.NewObjectInitTarget(info)
> ```
>
> if *initializer_target* is an *identifier*, or
>
> ```csharp
> Q.NewObjectInitTarget(info, args)
> ```
>
> if *initializer_target* is bound to an indexer, where `args` is the result of translating the *argument_list* to an expression tree, and where `info` is
>
> ```csharp
> Q.NewObjectInitTarget(default(Q.Flags), member)
> ```
>
> where the first flags argument is reserved for future use (for instance, when `member` may represent a generalized extension member).
>
> The member initializer is then translated into:
>
> ```csharp
> Q.NewObjectMemberInit(default(Q.Flags), target, expr)
> ```
>
> if *initializer_value* is an expression, where `expr` is the result of translating the expression to an expression tree, or
>
> ```csharp
> Q.NewObjectMemberInit(default(Q.Flags), target, bindings)
> ```
>
> if *initializer_value* is an object initializer, where `bindings` is
>
> ```csharp
> Q.MemberBindings(initExpr_1, ..., initExpr_N)
> ```
>
> where `initExpr_i` is the result of translating the `i`th nested *member_initializer* to an expression tree, or
>
> ```csharp
> Q.NewObjectMemberInit(default(Q.Flags), target, initializers)
> ```
>
> if *initializer_value* is an collection initializer, where `initializers` is
>
> ```csharp
> Q.ElementInitializers(initExpr_1, ..., initExpr_N)
> ```
>
> where `initExpr_i` is the result of translating the `i`th nested *element_initializer* to an expression tree.
>
> ***TODO***:
> * Review the factory method names. The introduction of indexer initializers has led to the requirement to add a construct modeling the assignment target, separate from the "binding". The most natural name for a binding is to have `MemberInit` in it, conform the C# specification nomenclature. As such, the original `MemberInit` becomes confusing and has been substituted for `NewObjectInit`.
> * If we decide that generalized expression trees can benefit from improved naming (e.g. `ListInit` incorrectly implies a list rather than any collection type), we may as well want to:
>   * make all names derived from the C# grammar, and,
>   * reduce flattening or inlining of language constructs when specifying the factory method invocations (i.e. making more grammar productions first-class nodes).
> * Review the use of `info` nodes across the board.

#### Collection initializers

A collection initializer specifies the elements of a collection.

```antlr
collection_initializer
    : '{' element_initializer_list '}'
    | '{' element_initializer_list ',' '}'
    ;

element_initializer_list
    : element_initializer (',' element_initializer)*
    ;

element_initializer
    : non_assignment_expression
    | '{' expression_list '}'
    ;

expression_list
    : expression (',' expression)*
    ;
```

A collection initializer consists of a sequence of element initializers, enclosed by `{` and `}` tokens and separated by commas. Each element initializer specifies an element to be added to the collection object being initialized, and consists of a list of expressions enclosed by `{` and `}` tokens and separated by commas.  A single-expression element initializer can be written without braces, but cannot then be an assignment expression, to avoid ambiguity with member initializers. The *non_assignment_expression* production is defined in [Expression](expressions.md#expression).

The following is an example of an object creation expression that includes a collection initializer:
```csharp
List<int> digits = new List<int> { 0, 1, 2, 3, 4, 5, 6, 7, 8, 9 };
```

The collection object to which a collection initializer is applied must be of a type that implements `System.Collections.IEnumerable` or a compile-time error occurs. For each specified element in order, the collection initializer invokes an `Add` method on the target object with the expression list of the element initializer as argument list, applying normal member lookup and overload resolution for each invocation. Thus, the collection object must have an applicable instance or extension method with the name `Add` for each element initializer.

The following class represents a contact with a name and a list of phone numbers:
```csharp
public class Contact
{
    string name;
    List<string> phoneNumbers = new List<string>();

    public string Name { get { return name; } set { name = value; } }

    public List<string> PhoneNumbers { get { return phoneNumbers; } }
}
```

A `List<Contact>` can be created and initialized as follows:
```csharp
var contacts = new List<Contact> {
    new Contact {
        Name = "Chris Smith",
        PhoneNumbers = { "206-555-0101", "425-882-8080" }
    },
    new Contact {
        Name = "Bob Harris",
        PhoneNumbers = { "650-555-0199" }
    }
};
```
which has the same effect as
```csharp
var __clist = new List<Contact>();
Contact __c1 = new Contact();
__c1.Name = "Chris Smith";
__c1.PhoneNumbers.Add("206-555-0101");
__c1.PhoneNumbers.Add("425-882-8080");
__clist.Add(__c1);
Contact __c2 = new Contact();
__c2.Name = "Bob Harris";
__c2.PhoneNumbers.Add("650-555-0199");
__clist.Add(__c2);
var contacts = __clist;
```
where `__clist`, `__c1` and `__c2` are temporary variables that are otherwise invisible and inaccessible.

> __Expression Tree Conversion Translation Steps__
>
> Translation of a *collection_initializer*
>
> ```antlr
> collection_initializer
>     : '{' element_initializer_list '}'
>     | '{' element_initializer_list ',' '}'
>     ;
>
> element_initializer_list
>     : element_initializer (',' element_initializer)*
>     ;
>
> element_initializer
>     : non_assignment_expression
>     | '{' expression_list '}'
>     ;
>
> expression_list
>     : expression (',' expression)*
>     ;
> ```
>
> to an expression tree proceeds as follows.
>
> First, create an expression `method` of type `MethodInfo` representing the bound `Add` method associated with the *collection_initializer*. Note this method may be an extension method.
>
> Next, if the *element_initializer* is a *non_assignment_expression*, convert this expression to an expression tree `expr`. Otherwise, if *element_initializer* contains an *expression_list*, convert the expressions in this list to expression trees `expr_1` to `expr_N`, in lexical order, by
>
> * translating `(T)(expression)` to an expression tree, if assignment to the corresponding parameter on `Add` requires an implicit conversion to type `T` which will be classified as compiler-generated, or
> * translating `expression` to an expression tree, otherwise.
>
> When converting to a query expression tree, if `method` refers to a static method, a compile-time error occurs. Otherwise, it is translated to:
>
> ```csharp
> Expression.ElementInit(method, args)
> ```
>
> where `args` is
>
> ```csharp
> new Expression[] { expr }
> ```
>
> if the *element_initializer* is a *non_assignment_expression*, or
>
> ```csharp
> new Expression[] { expr_1, ..., expr_N }
> ```
>
> otherwise.
>
> When converting a generalized expression tree, it is translated to:
>
> ```csharp
> Q.ElementInit(info, args)
> ```
>
> where `args` is
>
> ```csharp
> expr
> ```
>
> if the *element_initializer* is a *non_assignment_expression*, or
>
> ```csharp
> expr_1, ..., expr_N
> ```
>
> otherwise, and where `info` is
>
> ```csharp
> Q.ElementInitInfo(flags, method)
> ```
>
> where `flags` is `Q.Flags.InvokeExtensionMethod` is the specified method is invoked as an extension method, or `default(Q.Flags)` otherwise.
>
> Note that generalized expression trees use an argument list containing the expression trees representing the expressions used in the element initializer, thus resulting in overload resolution on `ElementInit`. This enables expression trees libraries to introduce optimized overloads for common cases, for instance an initializer consisting of a single expression.
>
> Also note that generalized expression trees support extension methods for 

#### Array creation expressions

An *array_creation_expression* is used to create a new instance of an *array_type*.

```antlr
array_creation_expression
    : 'new' non_array_type '[' expression_list ']' rank_specifier* array_initializer?
    | 'new' array_type array_initializer
    | 'new' rank_specifier array_initializer
    ;
```

An array creation expression of the first form allocates an array instance of the type that results from deleting each of the individual expressions from the expression list. For example, the array creation expression `new int[10,20]` produces an array instance of type `int[,]`, and the array creation expression `new int[10][,]` produces an array of type `int[][,]`. Each expression in the expression list must be of type `int`, `uint`, `long`, or `ulong`, or implicitly convertible to one or more of these types. The value of each expression determines the length of the corresponding dimension in the newly allocated array instance. Since the length of an array dimension must be nonnegative, it is a compile-time error to have a *constant_expression* with a negative value in the expression list.

Except in an unsafe context ([Unsafe contexts](unsafe-code.md#unsafe-contexts)), the layout of arrays is unspecified.

If an array creation expression of the first form includes an array initializer, each expression in the expression list must be a constant and the rank and dimension lengths specified by the expression list must match those of the array initializer.

In an array creation expression of the second or third form, the rank of the specified array type or rank specifier must match that of the array initializer. The individual dimension lengths are inferred from the number of elements in each of the corresponding nesting levels of the array initializer. Thus, the expression
```csharp
new int[,] {{0, 1}, {2, 3}, {4, 5}}
```
exactly corresponds to
```csharp
new int[3, 2] {{0, 1}, {2, 3}, {4, 5}}
```

An array creation expression of the third form is referred to as an ***implicitly typed array creation expression***. It is similar to the second form, except that the element type of the array is not explicitly given, but determined as the best common type ([Finding the best common type of a set of expressions](expressions.md#finding-the-best-common-type-of-a-set-of-expressions)) of the set of expressions in the array initializer. For a multidimensional array, i.e., one where the *rank_specifier* contains at least one comma, this set comprises all *expression*s found in nested *array_initializer*s.

Array initializers are described further in [Array initializers](arrays.md#array-initializers).

The result of evaluating an array creation expression is classified as a value, namely a reference to the newly allocated array instance. The run-time processing of an array creation expression consists of the following steps:

*  The dimension length expressions of the *expression_list* are evaluated in order, from left to right. Following evaluation of each expression, an implicit conversion ([Implicit conversions](conversions.md#implicit-conversions)) to one of the following types is performed: `int`, `uint`, `long`, `ulong`. The first type in this list for which an implicit conversion exists is chosen. If evaluation of an expression or the subsequent implicit conversion causes an exception, then no further expressions are evaluated and no further steps are executed.
*  The computed values for the dimension lengths are validated as follows. If one or more of the values are less than zero, a `System.OverflowException` is thrown and no further steps are executed.
*  An array instance with the given dimension lengths is allocated. If there is not enough memory available to allocate the new instance, a `System.OutOfMemoryException` is thrown and no further steps are executed.
*  All elements of the new array instance are initialized to their default values ([Default values](variables.md#default-values)).
*  If the array creation expression contains an array initializer, then each expression in the array initializer is evaluated and assigned to its corresponding array element. The evaluations and assignments are performed in the order the expressions are written in the array initializer—in other words, elements are initialized in increasing index order, with the rightmost dimension increasing first. If evaluation of a given expression or the subsequent assignment to the corresponding array element causes an exception, then no further elements are initialized (and the remaining elements will thus have their default values).

An array creation expression permits instantiation of an array with elements of an array type, but the elements of such an array must be manually initialized. For example, the statement
```csharp
int[][] a = new int[100][];
```
creates a single-dimensional array with 100 elements of type `int[]`. The initial value of each element is `null`. It is not possible for the same array creation expression to also instantiate the sub-arrays, and the statement
```csharp
int[][] a = new int[100][5];        // Error
```
results in a compile-time error. Instantiation of the sub-arrays must instead be performed manually, as in
```csharp
int[][] a = new int[100][];
for (int i = 0; i < 100; i++) a[i] = new int[5];
```

When an array of arrays has a "rectangular" shape, that is when the sub-arrays are all of the same length, it is more efficient to use a multi-dimensional array. In the example above, instantiation of the array of arrays creates 101 objects—one outer array and 100 sub-arrays. In contrast,
```csharp
int[,] = new int[100, 5];
```
creates only a single object, a two-dimensional array, and accomplishes the allocation in a single statement.

The following are examples of implicitly typed array creation expressions:
```csharp
var a = new[] { 1, 10, 100, 1000 };                       // int[]

var b = new[] { 1, 1.5, 2, 2.5 };                         // double[]

var c = new[,] { { "hello", null }, { "world", "!" } };   // string[,]

var d = new[] { 1, "one", 2, "two" };                     // Error
```

The last expression causes a compile-time error because neither `int` nor `string` is implicitly convertible to the other, and so there is no best common type. An explicitly typed array creation expression must be used in this case, for example specifying the type to be `object[]`. Alternatively, one of the elements can be cast to a common base type, which would then become the inferred element type.

Implicitly typed array creation expressions can be combined with anonymous object initializers ([Anonymous object creation expressions](expressions.md#anonymous-object-creation-expressions)) to create anonymously typed data structures. For example:
```csharp
var contacts = new[] {
    new {
        Name = "Chris Smith",
        PhoneNumbers = new[] { "206-555-0101", "425-882-8080" }
    },
    new {
        Name = "Bob Harris",
        PhoneNumbers = new[] { "650-555-0199" }
    }
};
```

> __Expression Tree Conversion Translation Steps__
>
> Translation of an *array_creation_expression* is performed by first pre-processing the expression as follows.
>
> If *array_creation_expression* is an ***implicitly typed array creation expression*** of the form
>
> ```antlr
> 'new' rank_specifier array_initializer
> ```
>
> it is first translated to
>
> ```antlr
> 'new' array_type array_initializer
> ```
>
> where `array_type` is the array type inferred as described above. The result of this translation is a new *array_creation_expression* which subsumes the original one from this point on.
>
> Note that the preceding step may introduce an `array_type` for which no valid syntax is available, because it involves anonymous types. For purposes of expression tree conversion, these types only occur transiently as part of the translation steps, and are ultimately converted to expressions of type `Type`. A concrete implementation of this specification will most likely not involve syntactic transformations but operate on an intermediate representation of the code, where bound types are represented symbolically rather than syntactically.
>
> If *array_creation_expression* is of the form
>
> ```antlr
> 'new' array_type array_initializer
> ```
>
> it is translated to
>
> ```antlr
> 'new' non_array_type '[' expression_list ']' rank_specifier* array_initializer
> ```
>
> where `non_array_type` is the element type of `array_type`, and `expression_list` is constructed by inferring the individual dimension lengths from the number of elements in each of the corresponding nesting levels of the *array_initializer* and by representing these lenghts as numeric literals that can be converted to `int`, `uint`, `long`, or `ulong`. The result of this translation is a new *array_creation_expression* which subsumes any preceding one from this point on.
>
> After the preceding translation steps, all forms of *array_creation_expression* have been transformed to the following form
>
> ```antlr
> 'new' non_array_type '[' expression_list ']' rank_specifier* array_initializer?
> ```
>
> Expression tree translation then proceeds by constructing an expression `elementType` of type `Type` representing the *non_array_type* type, and by constructing expressions `lenExpr_1` to `lenExpr_N` from each dimension length *expression* in *expression_list* by first translating *expression* to
>
> * `(T)(expression)`, if the dimension length is implicitly converted to `T` where `T` is `int`, `uint`, `long`, or `ulong`, and classifying the cast expression as compiler-generated, or,
> * `expression` otherwise,
>
> and subsequently converting the resulting expression to an expression tree.
>
> Finally, if specified, convert *array_initializer* to an expression `initExpr` using the expression tree conversion steps described in [Array initializers](arrays.md#array-initializers). Expression tree translation then proceeds as follows.
>
> When converted to a query expression tree, a compile-time error is reported if the array type is a multi-dimensional array and an array initializer is specified. Otherwise, it is translated to
>
> ```csharp
> Expression.NewArrayInit(elementType, initExpr)
> ```
>
> if an array initializer is specified, or to
>
> ```csharp
> Expression.NewArrayBounds(elementType, bounds)
> ```
>
> otherwise, where `bounds` is an expression of type `Expression[]` representing an array of length `N` containing the expressions `lenExpr_1` to `lenExpr_N`.
>
> When converted to a generalized expression tree, it is translated to
>
> ```csharp
> Q.NewArrayInit(info, initExpr)
> ```
>
> if an array initializer is specified, where `info` is
>
> ```csharp
> Q.NewArrayInitInfo(default(Q.Flags), elementType)
> ```
>
> or to
>
> ```csharp
> Q.NewArrayBounds(info, lenExpr_1, ..., lenExpr_N)
> ```
>
> otherwise, where `info` is
>
> ```csharp
> Q.NewArrayBoundsInfo(default(Q.Flags), elementType)
> ```
>
> Note that generalized expression trees specified the dimension length expressions as an argument list, thus resulting in overload resolution of the `NewArrayBounds` method.

#### Delegate creation expressions

A *delegate_creation_expression* is used to create a new instance of a *delegate_type*.

```antlr
delegate_creation_expression
    : 'new' delegate_type '(' expression ')'
    ;
```

The argument of a delegate creation expression must be a method group, an anonymous function or a value of either the compile time type `dynamic` or a *delegate_type*. If the argument is a method group, it identifies the method and, for an instance method, the object for which to create a delegate. If the argument is an anonymous function it directly defines the parameters and method body of the delegate target. If the argument is a value it identifies a delegate instance of which to create a copy.

If the *expression* has the compile-time type `dynamic`, the *delegate_creation_expression* is dynamically bound ([Dynamic binding](expressions.md#dynamic-binding)), and the rules below are applied at run-time using the run-time type of the *expression*. Otherwise the rules are applied at compile-time.

The binding-time processing of a *delegate_creation_expression* of the form `new D(E)`, where `D` is a *delegate_type* and `E` is an *expression*, consists of the following steps:

*  If `E` is a method group, the delegate creation expression is processed in the same way as a method group conversion ([Method group conversions](conversions.md#method-group-conversions)) from `E` to `D`.
*  If `E` is an anonymous function, the delegate creation expression is processed in the same way as an anonymous function conversion ([Anonymous function conversions](conversions.md#anonymous-function-conversions)) from `E` to `D`.
*  If `E` is a value, `E` must be compatible ([Delegate declarations](delegates.md#delegate-declarations)) with `D`, and the result is a reference to a newly created delegate of type `D` that refers to the same invocation list as `E`. If `E` is not compatible with `D`, a compile-time error occurs.

The run-time processing of a *delegate_creation_expression* of the form `new D(E)`, where `D` is a *delegate_type* and `E` is an *expression*, consists of the following steps:

*   If `E` is a method group, the delegate creation expression is evaluated as a method group conversion ([Method group conversions](conversions.md#method-group-conversions)) from `E` to `D`.
*   If `E` is an anonymous function, the delegate creation is evaluated as an anonymous function conversion from `E` to `D` ([Anonymous function conversions](conversions.md#anonymous-function-conversions)).
*   If `E` is a value of a *delegate_type*:
    * `E` is evaluated. If this evaluation causes an exception, no further steps are executed.
    * If the value of `E` is `null`, a `System.NullReferenceException` is thrown and no further steps are executed.
    * A new instance of the delegate type `D` is allocated. If there is not enough memory available to allocate the new instance, a `System.OutOfMemoryException` is thrown and no further steps are executed.
    * The new delegate instance is initialized with the same invocation list as the delegate instance given by `E`.

The invocation list of a delegate is determined when the delegate is instantiated and then remains constant for the entire lifetime of the delegate. In other words, it is not possible to change the target callable entities of a delegate once it has been created. When two delegates are combined or one is removed from another ([Delegate declarations](delegates.md#delegate-declarations)), a new delegate results; no existing delegate has its contents changed.

It is not possible to create a delegate that refers to a property, indexer, user-defined operator, instance constructor, destructor, or static constructor.

As described above, when a delegate is created from a method group, the formal parameter list and return type of the delegate determine which of the overloaded methods to select. In the example
```csharp
delegate double DoubleFunc(double x);

class A
{
    DoubleFunc f = new DoubleFunc(Square);

    static float Square(float x) {
        return x * x;
    }

    static double Square(double x) {
        return x * x;
    }
}
```
the `A.f` field is initialized with a delegate that refers to the second `Square` method because that method exactly matches the formal parameter list and return type of `DoubleFunc`. Had the second `Square` method not been present, a compile-time error would have occurred.

> __Expression Tree Conversion Translation Steps__
>
> Translation of an *delegate_creation_expression* of the form
>
> ```csharp
> new D(E)
> ```
>
> starts by constructing an expression `delegateType` of type `Type` representing the type `D`.
>
> When converted to a query expression tree, a compile-time error is produced if `E` is of type `dynamic`. Otherwise, it is translated to:
>
> ```csharp
> expr
> ```
>
> where `expr` is the result of translating `E`, if `E` is anonynmous function expression.
>
> Otherwise, construct an expression `method` of type `MethodInfo` representing
>
> * the method represented by `E` if `E` is a method group, or,
> * the `Invoke` method on delegate type `D` if `E` is a value of a delegate type, otherwise,
>
> and, construct an expression `receiver` to be
>
> * if `E` is a method group referring to bound method `M`,
>   * if `M` is a static method, a `null` expression implicitly typed as `object`, or
>   * if `E` has an expression `instance` representing the invocation target,
>     * `instance` if the type of `instance` is a reference type, or
>     * `(object)(instance)` otherwise, or,
> * if `E` represents a value of a delegate type, the original expression `E`.
>
> Expression tree conversion then proceeds by first checking the existence of the `System.Reflection.MethodInfo.CreateDelegate` instance method. If it exists, the expression tree is represented by translating
>
> ```csharp
> (D)method.CreateDelegate(delegateType, receiver)
> ```
>
> to an expression tree, or by translating
>
> ```csharp
> (D)System.Delegate.CreateDelegate(delegateType, receiver, method)
> ```
>
> to an expression tree otherwise. Note that these translations will result in `Expression.Convert` and `Expression.Call` factory invocations.
>
> When converted to a generalized expression tree, it is translated to
>
> ```csharp
> Q.NewDelegate(info)
> ```
>
> if `E` is a method group referring to a static method, or
>
> ```csharp
> Q.NewDelegate(info, expr)
> ```
>
> otherwise, where `expr` is either the result of translating the target instance if `E` is a method group, or the result of translating `E` if `E` is a value or an anonymous function expression, and where `info` is
>
> ```csharp
> Q.NewDelegateInfo(default(Q.Flags), typeof(D), method)
> ```
>
> if `E` is a method group, where `method` is an expression of type `MethodInfo` representing the method `M` bound by the method group in `E`, or
>
> ```csharp
> Q.NewDelegateInfo(default(Q.Flags), typeof(D), binder)
> ```
>
> if `E` has a `dynamic` type, where `binder` is an expression of type `Microsoft.CSharp.RuntimeBinder.CallSiteBinder` representing the dynamic operation used to convert to `D`, or
>
> ```csharp
> Q.NewDelegateInfo(default(Q.Flags), typeof(D))
> ```
>
> otherwise.
>
> ***TODO***:
> * Evaluate if the dynamic case should simply be lowered to `(D)E` (diverging from the user's syntax).
> * Evaluate if the anonymous method case should omit `NewDelegate` (diverging from the user's syntax).

#### Anonymous object creation expressions

An *anonymous_object_creation_expression* is used to create an object of an anonymous type.

```antlr
anonymous_object_creation_expression
    : 'new' anonymous_object_initializer
    ;

anonymous_object_initializer
    : '{' member_declarator_list? '}'
    | '{' member_declarator_list ',' '}'
    ;

member_declarator_list
    : member_declarator (',' member_declarator)*
    ;

member_declarator
    : simple_name
    | member_access
    | base_access
    | null_conditional_member_access
    | identifier '=' expression
    ;
```

An anonymous object initializer declares an anonymous type and returns an instance of that type. An anonymous type is a nameless class type that inherits directly from `object`. The members of an anonymous type are a sequence of read-only properties inferred from the anonymous object initializer used to create an instance of the type. Specifically, an anonymous object initializer of the form
```csharp
new { p1 = e1, p2 = e2, ..., pn = en }
```
declares an anonymous type of the form
```csharp
class __Anonymous1
{
    private readonly T1 f1;
    private readonly T2 f2;
    ...
    private readonly Tn fn;

    public __Anonymous1(T1 a1, T2 a2, ..., Tn an) {
        f1 = a1;
        f2 = a2;
        ...
        fn = an;
    }

    public T1 p1 { get { return f1; } }
    public T2 p2 { get { return f2; } }
    ...
    public Tn pn { get { return fn; } }

    public override bool Equals(object __o) { ... }
    public override int GetHashCode() { ... }
}
```
where each `Tx` is the type of the corresponding expression `ex`. The expression used in a *member_declarator* must have a type. Thus, it is a compile-time error for an expression in a *member_declarator* to be null or an anonymous function. It is also a compile-time error for the expression to have an unsafe type.

The names of an anonymous type and of the parameter to its `Equals` method are automatically generated by the compiler and cannot be referenced in program text.

Within the same program, two anonymous object initializers that specify a sequence of properties of the same names and compile-time types in the same order will produce instances of the same anonymous type.

In the example
```csharp
var p1 = new { Name = "Lawnmower", Price = 495.00 };
var p2 = new { Name = "Shovel", Price = 26.95 };
p1 = p2;
```
the assignment on the last line is permitted because `p1` and `p2` are of the same anonymous type.

The `Equals` and `GetHashcode` methods on anonymous types override the methods inherited from `object`, and are defined in terms of the `Equals` and `GetHashcode` of the properties, so that two instances of the same anonymous type are equal if and only if all their properties are equal.

A member declarator can be abbreviated to a simple name ([Type inference](expressions.md#type-inference)), a member access ([Compile-time checking of dynamic overload resolution](expressions.md#compile-time-checking-of-dynamic-overload-resolution)), a base access ([Base access](expressions.md#base-access)) or a null-conditional member access ([Null-conditional expressions as projection initializers](expressions.md#null-conditional-expressions-as-projection-initializers)). This is called a ***projection initializer*** and is shorthand for a declaration of and assignment to a property with the same name. Specifically, member declarators of the forms
```csharp
identifier
expr.identifier
```
are precisely equivalent to the following, respectively:
```csharp
identifier = identifier
identifier = expr.identifier
```

Thus, in a projection initializer the *identifier* selects both the value and the field or property to which the value is assigned. Intuitively, a projection initializer projects not just a value, but also the name of the value.

> __Expression Tree Conversion Translation Steps__
>
> An anonymous object creation expression
>
> ```csharp
> new { memberDecl_1, ..., memberDecl_N }
> ```
>
> by first constructing the following expressions:
>
> * an expression `constructor` of type `ConstructorInfo` representing the constructor on the generated anonymous type used to construct the anonymous object at runtime,
> * expressions `member_i` (for `1 <= i <= N`) of type `MethodInfo` representing the `get` accessor method of the property on the anonymous type generated for `memberDecl_i`, and,
> * expressions `expr_i` (for `1 <= i <= N`) obtained by translating the expression occurring in `memberDecl_i`.
>
> Translation to an expression tree then proceeds as follows.
>
> When converted to a query expression tree, it is translated into
>
> ```csharp
> Expression.New(constructor, (IEnumerable<Expression>)exprs, members)
> ```
>
> where `exprs` is an expression of type `Expression[]` representing an array of length `N` with elements `expr_1` to `expr_N`, and where `members` is an  expression of type `MemberInfo[]` representing an array of length `N` with elements `member_1` to `member_N`.
>
> When converted to a generalized expression tree, it is translated into
>
> ```csharp
> Q.NewAnonymous(info, declExpr_1, ..., declExpr_N)
> ```
>
> where `declExpr_i` (for `1 <= i <= N`) is
>
> ```csharp
> Q.MemberDeclaration(member_i, expr_i)
> ```
>
> and `info` is
>
> ```csharp
> Q.NewAnonymousInfo(flags, constructor)
> ```
>
> where `flags` is either:
>
> * `Q.Flags.CompilerGenerated` if the anonymous object creation expression was compiler-generated (for instance, for [Transparent identifiers](expressions.md#transparent-identifiers)), or,
> * `default(Q.Flags)` otherwise.
>
> Note that generalized expression trees model anonymous object creation expressions using `MemberDeclaration` nodes that pair up `MethodInfo` and expression node objects for each member declaration. This reduces the need for expression tree libraries to zip an expression node collection with a members collection when analyzing an expression tree.

### The typeof operator

The `typeof` operator is used to obtain the `System.Type` object for a type.

```antlr
typeof_expression
    : 'typeof' '(' type ')'
    | 'typeof' '(' unbound_type_name ')'
    | 'typeof' '(' 'void' ')'
    ;

unbound_type_name
    : identifier generic_dimension_specifier?
    | identifier '::' identifier generic_dimension_specifier?
    | unbound_type_name '.' identifier generic_dimension_specifier?
    ;

generic_dimension_specifier
    : '<' comma* '>'
    ;

comma
    : ','
    ;
```

The first form of *typeof_expression* consists of a `typeof` keyword followed by a parenthesized *type*. The result of an expression of this form is the `System.Type` object for the indicated type. There is only one `System.Type` object for any given type. This means that for a type `T`, `typeof(T) == typeof(T)` is always true. The *type* cannot be `dynamic`.

The second form of *typeof_expression* consists of a `typeof` keyword followed by a parenthesized *unbound_type_name*. An *unbound_type_name* is very similar to a *type_name* ([Namespace and type names](basic-concepts.md#namespace-and-type-names)) except that an *unbound_type_name* contains *generic_dimension_specifier*s where a *type_name* contains *type_argument_list*s. When the operand of a *typeof_expression* is a sequence of tokens that satisfies the grammars of both *unbound_type_name* and *type_name*, namely when it contains neither a *generic_dimension_specifier* nor a *type_argument_list*, the sequence of tokens is considered to be a *type_name*. The meaning of an *unbound_type_name* is determined as follows:

*  Convert the sequence of tokens to a *type_name* by replacing each *generic_dimension_specifier* with a *type_argument_list* having the same number of commas and the keyword `object` as each *type_argument*.
*  Evaluate the resulting *type_name*, while ignoring all type parameter constraints.
*  The *unbound_type_name* resolves to the unbound generic type associated with the resulting constructed type ([Bound and unbound types](types.md#bound-and-unbound-types)).

The result of the *typeof_expression* is the `System.Type` object for the resulting unbound generic type.

The third form of *typeof_expression* consists of a `typeof` keyword followed by a parenthesized `void` keyword. The result of an expression of this form is the `System.Type` object that represents the absence of a type. The type object returned by `typeof(void)` is distinct from the type object returned for any type. This special type object is useful in class libraries that allow reflection onto methods in the language, where those methods wish to have a way to represent the return type of any method, including void methods, with an instance of `System.Type`.

The `typeof` operator can be used on a type parameter. The result is the `System.Type` object for the run-time type that was bound to the type parameter. The `typeof` operator can also be used on a constructed type or an unbound generic type ([Bound and unbound types](types.md#bound-and-unbound-types)). The `System.Type` object for an unbound generic type is not the same as the `System.Type` object of the instance type. The instance type is always a closed constructed type at run-time so its `System.Type` object depends on the run-time type arguments in use, while the unbound generic type has no type arguments.

The example
```csharp
using System;

class X<T>
{
    public static void PrintTypes() {
        Type[] t = {
            typeof(int),
            typeof(System.Int32),
            typeof(string),
            typeof(double[]),
            typeof(void),
            typeof(T),
            typeof(X<T>),
            typeof(X<X<T>>),
            typeof(X<>)
        };
        for (int i = 0; i < t.Length; i++) {
            Console.WriteLine(t[i]);
        }
    }
}

class Test
{
    static void Main() {
        X<int>.PrintTypes();
    }
}
```
produces the following output:
```
System.Int32
System.Int32
System.String
System.Double[]
System.Void
System.Int32
X`1[System.Int32]
X`1[X`1[System.Int32]]
X`1[T]
```

Note that `int` and `System.Int32` are the same type.

Also note that the result of `typeof(X<>)` does not depend on the type argument but the result of `typeof(X<T>)` does.

> __Expression Tree Conversion Translation Steps__
>
> A `typeof` expression
>
> ```csharp
> typeof(T)
> ```
>
> when subject to conversion to a query expression tree, is translated into
>
> ```csharp
> Expression.Constant(typeof(T), typeof(Type))
> ```
>
> and when subject to conversion to a generalized expression tree, is translated into
>
> ```csharp
> Q.TypeOf(default(Q.Flags), type)
> ```
>
> where `type` is an expression of type `Type` representing the type `T`, and where the first flags parameter is reserved for future use.

### The checked and unchecked operators

The `checked` and `unchecked` operators are used to control the ***overflow checking context*** for integral-type arithmetic operations and conversions.

```antlr
checked_expression
    : 'checked' '(' expression ')'
    ;

unchecked_expression
    : 'unchecked' '(' expression ')'
    ;
```

The `checked` operator evaluates the contained expression in a checked context, and the `unchecked` operator evaluates the contained expression in an unchecked context. A *checked_expression* or *unchecked_expression* corresponds exactly to a *parenthesized_expression* ([Parenthesized expressions](expressions.md#parenthesized-expressions)), except that the contained expression is evaluated in the given overflow checking context.

The overflow checking context can also be controlled through the `checked` and `unchecked` statements ([The checked and unchecked statements](statements.md#the-checked-and-unchecked-statements)).

The following operations are affected by the overflow checking context established by the `checked` and `unchecked` operators and statements:

*  The predefined `++` and `--` unary operators ([Postfix increment and decrement operators](expressions.md#postfix-increment-and-decrement-operators) and [Prefix increment and decrement operators](expressions.md#prefix-increment-and-decrement-operators)), when the operand is of an integral type.
*  The predefined `-` unary operator ([Unary minus operator](expressions.md#unary-minus-operator)), when the operand is of an integral type.
*  The predefined `+`, `-`, `*`, and `/` binary operators ([Arithmetic operators](expressions.md#arithmetic-operators)), when both operands are of integral types.
*  Explicit numeric conversions ([Explicit numeric conversions](conversions.md#explicit-numeric-conversions)) from one integral type to another integral type, or from `float` or `double` to an integral type.

When one of the above operations produce a result that is too large to represent in the destination type, the context in which the operation is performed controls the resulting behavior:

*  In a `checked` context, if the operation is a constant expression ([Constant expressions](expressions.md#constant-expressions)), a compile-time error occurs. Otherwise, when the operation is performed at run-time, a `System.OverflowException` is thrown.
*  In an `unchecked` context, the result is truncated by discarding any high-order bits that do not fit in the destination type.

For non-constant expressions (expressions that are evaluated at run-time) that are not enclosed by any `checked` or `unchecked` operators or statements, the default overflow checking context is `unchecked` unless external factors (such as compiler switches and execution environment configuration) call for `checked` evaluation.

For constant expressions (expressions that can be fully evaluated at compile-time), the default overflow checking context is always `checked`. Unless a constant expression is explicitly placed in an `unchecked` context, overflows that occur during the compile-time evaluation of the expression always cause compile-time errors.

The body of an anonymous function is not affected by `checked` or `unchecked` contexts in which the anonymous function occurs.

In the example
```csharp
class Test
{
    static readonly int x = 1000000;
    static readonly int y = 1000000;

    static int F() {
        return checked(x * y);      // Throws OverflowException
    }

    static int G() {
        return unchecked(x * y);    // Returns -727379968
    }

    static int H() {
        return x * y;               // Depends on default
    }
}
```
no compile-time errors are reported since neither of the expressions can be evaluated at compile-time. At run-time, the `F` method throws a `System.OverflowException`, and the `G` method returns -727379968 (the lower 32 bits of the out-of-range result). The behavior of the `H` method depends on the default overflow checking context for the compilation, but it is either the same as `F` or the same as `G`.

In the example
```csharp
class Test
{
    const int x = 1000000;
    const int y = 1000000;

    static int F() {
        return checked(x * y);      // Compile error, overflow
    }

    static int G() {
        return unchecked(x * y);    // Returns -727379968
    }

    static int H() {
        return x * y;               // Compile error, overflow
    }
}
```
the overflows that occur when evaluating the constant expressions in `F` and `H` cause compile-time errors to be reported because the expressions are evaluated in a `checked` context. An overflow also occurs when evaluating the constant expression in `G`, but since the evaluation takes place in an `unchecked` context, the overflow is not reported.

The `checked` and `unchecked` operators only affect the overflow checking context for those operations that are textually contained within the "`(`" and "`)`" tokens. The operators have no effect on function members that are invoked as a result of evaluating the contained expression. In the example
```csharp
class Test
{
    static int Multiply(int x, int y) {
        return x * y;
    }

    static int F() {
        return checked(Multiply(1000000, 1000000));
    }
}
```
the use of `checked` in `F` does not affect the evaluation of `x * y` in `Multiply`, so `x * y` is evaluated in the default overflow checking context.

The `unchecked` operator is convenient when writing constants of the signed integral types in hexadecimal notation. For example:
```csharp
class Test
{
    public const int AllBits = unchecked((int)0xFFFFFFFF);

    public const int HighBit = unchecked((int)0x80000000);
}
```

Both of the hexadecimal constants above are of type `uint`. Because the constants are outside the `int` range, without the `unchecked` operator, the casts to `int` would produce compile-time errors.

The `checked` and `unchecked` operators and statements allow programmers to control certain aspects of some numeric calculations. However, the behavior of some numeric operators depends on their operands' data types. For example, multiplying two decimals always results in an exception on overflow even within an explicitly `unchecked` construct. Similarly, multiplying two floats never results in an exception on overflow even within an explicitly `checked` construct. In addition, other operators are never affected by the mode of checking, whether default or explicit.

> __Expression Tree Conversion Translation Steps__
>
> For purposes of conversion to a query expression tree, no separate nodes are constructed to represent an `unchecked` or a `checked` context. Instead, `Checked` variants of query expression tree factories are used.
>
> For purposes of conversion to a generalized expression tree, `Q.Flags.CheckedContext` flags are passed to generalized expression tree factory invocations for expressions occurring within a `checked` context, and a node is constructed to retain the syntactic structure of the original code.
>
> Expression tree conversion starts by constructing an expression `expr` from converting the expression operand to an expression tree.
>
> When translating a `checked` operator, it is translated into
>
> ```csharp
> Q.Checked(info, expr)
> ```
>
> where `info` is
>
> ```csharp
> Q.CheckedInfo(flags)
> ```
>
> and when translating an `unchecked` operator, it is translated into
>
> ```csharp
> Q.Unchecked(info, expr)
> ```
> where `info` is
>
> ```csharp
> Q.CheckedInfo(flags)
> ```
>
> and `flags` is `Q.Flags.CompilerGenerated` if the operator was compiler-generated, or `default(Q.Flags)` otherwise.
>
> Note that the use of a separate node to represent `checked` and `unchecked` operators enables expression tree libraries to distinguish between a user explicitly opting in to checked or unchecked operations, and a compiler configuration flag influencing this behavior.
>
> ***TODO***
> * Determine if syntactic WYSIWYG expression trees are desirable, because it introduces more node types. However, it's much easier to explain the correspondence between grammar productions and node types.
> * Determine if `Q.Flags.CheckedContext` should be passed to **all** nodes constructed within a `checked` context. In particular, pass this flag not only to `Add`, `Multiple`, `Subtract`, and `Negate` factory methods. This could enable expression tree libraries to interpret `checked` more broadly, for instance when the user calls `Math.Pow` and an expression tree library supports a checked variant for well-known methods when evaluating or translating such an expression.

### Default value expressions

A default value expression is used to obtain the default value ([Default values](variables.md#default-values)) of a type. Typically a default value expression is used for type parameters, since it may not be known if the type parameter is a value type or a reference type. (No conversion exists from the `null` literal to a type parameter unless the type parameter is known to be a reference type.)

```antlr
default_value_expression
    : 'default' '(' type ')'
    ;
```

If the *type* in a *default_value_expression* evaluates at run-time to a reference type, the result is `null` converted to that type. If the *type* in a *default_value_expression* evaluates at run-time to a value type, the result is the *value_type*'s default value ([Default constructors](types.md#default-constructors)).

A *default_value_expression* is a constant expression ([Constant expressions](expressions.md#constant-expressions)) if the type is a reference type or a type parameter that is known to be a reference type ([Type parameter constraints](classes.md#type-parameter-constraints)). In addition, a *default_value_expression* is a constant expression if the type is one of the following value types: `sbyte`, `byte`, `short`, `ushort`, `int`, `uint`, `long`, `ulong`, `char`, `float`, `double`, `decimal`, `bool`, or any enumeration type.

> __Expression Tree Conversion Translation Steps__
>
> A `default` expression
>
> ```csharp
> default(T)
> ```
>
> when subject to conversion to a query expression tree, is translated into
>
> ```csharp
> Expression.Constant(default(T), typeof(T))
> ```
>
> and when subject to conversion to a generalized expression tree, is translated into
>
> ```csharp
> Q.Default(default(Q.Flags), type)
> ```
>
> where `type` is an expression of type `Type` representing the type `T`, and where the first flags parameter is reserved for future use.
>
> Note that generalized expression trees introduce a special node for default expressions, allowing expression tree libraries to distinguish a default literal (for instance `0`) from a default expression (for instance `default(int)`).

### Nameof expressions

A *nameof_expression* is used to obtain the name of a program entity as a constant string.

```antlr
nameof_expression
    : 'nameof' '(' named_entity ')'
    ;

named_entity
    : simple_name
    | named_entity_target '.' identifier type_argument_list?
    ;

named_entity_target
    : 'this'
    | 'base'
    | named_entity 
    | predefined_type 
    | qualified_alias_member
    ;
```

Grammatically speaking, the *named_entity* operand is always an expression. Because `nameof` is not a reserved keyword, a nameof expression is always syntactically ambiguous with an invocation of the simple name `nameof`. For compatibility reasons, if a name lookup ([Simple names](expressions.md#simple-names)) of the name `nameof` succeeds, the expression is treated as an *invocation_expression* -- regardless of whether the invocation is legal. Otherwise it is a *nameof_expression*.

The meaning of the *named_entity* of a *nameof_expression* is the meaning of it as an expression; that is, either as a *simple_name*, a *base_access* or a *member_access*. However, where the lookup described in [Simple names](expressions.md#simple-names) and [Member access](expressions.md#member-access) results in an error because an instance member was found in a static context, a *nameof_expression* produces no such error.

It is a compile-time error for a *named_entity* designating a method group to have a *type_argument_list*. It is a compile time error for a *named_entity_target* to have the type `dynamic`.

A *nameof_expression* is a constant expression of type `string`, and has no effect at runtime. Specifically, its *named_entity* is not evaluated, and is ignored for the purposes of definite assignment analysis ([General rules for simple expressions](variables.md#general-rules-for-simple-expressions)). Its value is the last identifier of the *named_entity* before the optional final *type_argument_list*, transformed in the following way:

* The prefix "`@`", if used, is removed.
* Each *unicode_escape_sequence* is transformed into its corresponding Unicode character.
* Any *formatting_characters* are removed.

These are the same transformations applied in [Identifiers](lexical-structure.md#identifiers) when testing equality between identifiers.

TODO: examples

> __Expression Tree Conversion Translation Steps__
>
> A `nameof` expression
>
> ```csharp
> nameof(named_entity)
> ```
>
> when subject to conversion to a query expression tree, is translated into
>
> ```csharp
> Expression.Constant(name, typeof(string))
> ```
>
> and when subject to conversion to a generalized expression tree, is translated into
>
> ```csharp
> Q.Constant(Q.Flags.CompilerGenerated, name, typeof(string))
> ```
>
> where `name` is an expression of type `string` containing the value of the `nameof` expression as described above.
>
> ***TODO***:
> * Should generalized expression trees use a `Q.NameOf` factory instead?
>   * This conveys the original intent better., thus allowing
>      * expression tree libraries to make use of `nameof` resilient when they carry out tree transformations (e.g. renaming a variable, rebind types and members, etc.)
>   * Should overloads of `NameOf` accept
>     * a parameter of type `Type`, if the named entity refers to a type, or,
>     * a parameter of type `T`, where `T` is the return type of `Q.Variable` or `Q.Parameter`, if the named entity refers to a variable, or,
>     * a parameter of type `MemberInfo`, if the named entity refers to an unambiguous member, which may
>       * be questionable because introduction of ambiguity, e.g. by adding an overload, would cause the expression tree to start binding to another overload of `NameOf`, but it wouldn't regress to calling `Constant`), or,
>       * require a more sophisticated parameter list that tries to retain as much semantic info as possible (e.g. `Type, string` to refer to a member on a given type),
>     * a parameter of type `string` for all other cases.
>
> Given the above observations, a possible alternative formulation is:
>
>> and when subject to conversion to a generalized expression tree, is translated into
>>
>> ```csharp
>> Q.NameOf(default(Q.Flags), type)
>> ```
>>
>> if `named_entity` uniquely identifies a type, where `type` is an expression of type `Type` representing this type, or
>>
>> ```csharp
>> Q.NameOf(default(Q.Flags), type, name)
>> ```
>>
>> if `named_entity` identifies a member on a type, where `type` is an expression of type `Type` representing this type, and `name` is an expression of type `string` representing the name of the member, or
>>
>> ```csharp
>> Q.NameOf(default(Q.Flags), expr)
>> ```
>>
>> if `named_entity` refers to a variable or a parameter that occurs in the expression tree being translated, where `expr` is an expression tree node representing the variable or parameter being referenced, or
>>
>> ```csharp
>> Q.NameOf(Q.Flags.CompilerGenerated, name)
>> ```
>>
>> otherwise, where `name` is an expression of type `string` containing the value of the `nameof` expression as described above.
>>
>> ***TODO***: Note that the `nameof` expression does not allow referring to a label as a named entity. If this were added, the third case could be altered to include referring to an expression tree node representing a label.

### Anonymous method expressions

An *anonymous_method_expression* is one of two ways of defining an anonymous function. These are further described in [Anonymous function expressions](expressions.md#anonymous-function-expressions).

## Unary operators

The `?`, `+`, `-`, `!`, `~`, `++`, `--`, cast, and `await` operators are called the unary operators.

```antlr
unary_expression
    : primary_expression
    | null_conditional_expression
    | '+' unary_expression
    | '-' unary_expression
    | '!' unary_expression
    | '~' unary_expression
    | pre_increment_expression
    | pre_decrement_expression
    | cast_expression
    | await_expression
    | unary_expression_unsafe
    ;
```

If the operand of a *unary_expression* has the compile-time type `dynamic`, it is dynamically bound ([Dynamic binding](expressions.md#dynamic-binding)). In this case the compile-time type of the *unary_expression* is `dynamic`, and the resolution described below will take place at run-time using the run-time type of the operand.

### Null-conditional operator

The null-conditional operator applies a list of operations to its operand only if that operand is non-null. Otherwise the result of applying the operator is `null`.

```antlr
null_conditional_expression
    : primary_expression null_conditional_operations
    ;

null_conditional_operations
    : null_conditional_operations? '?' '.' identifier type_argument_list?
    | null_conditional_operations? '?' '[' argument_list ']'
    | null_conditional_operations '.' identifier type_argument_list?
    | null_conditional_operations '[' argument_list ']'
    | null_conditional_operations '(' argument_list? ')'
    ;
```

The list of operations can include member access and element access operations (which may themselves be null-conditional), as well as invocation.

For example, the expression `a.b?[0]?.c()` is a *null_conditional_expression* with a *primary_expression* `a.b` and *null_conditional_operations* `?[0]` (null-conditional element access), `?.c` (null-conditional member access) and `()` (invocation).

For a *null_conditional_expression* `E` with a *primary_expression* `P`, let `E0` be the expression obtained by textually removing the leading `?` from each of the *null_conditional_operations* of `E` that have one. Conceptually, `E0` is the expression that will be evaluated if none of the null checks represented by the `?`s do find a `null`.

Also, let `E1` be the expression obtained by textually removing the leading `?` from just the first of the *null_conditional_operations* in `E`. This may lead to a *primary-expression* (if there was just one `?`) or to another *null_conditional_expression*.

For example, if `E` is the expression `a.b?[0]?.c()`, then `E0` is the expression `a.b[0].c()` and `E1` is the expression `a.b[0]?.c()`.

If `E0` is classified as nothing, then `E` is classified as nothing. Otherwise E is classified as a value.

`E0` and `E1` are used to determine the meaning of `E`:

*  If `E` occurs as a *statement_expression* the meaning of `E` is the same as the statement

   ```csharp
   if ((object)P != null) E1;
   ```

   except that P is evaluated only once.

*  Otherwise, if `E0` is classified as nothing a compile-time error occurs.

*  Otherwise, let `T0` be the type of `E0`.

   *  If `T0` is a type parameter that is not known to be a reference type or a non-nullable value type, a compile-time error occurs.

   *  If `T0` is a non-nullable value type, then the type of `E` is `T0?`, and the meaning of `E` is the same as

      ```csharp
      ((object)P == null) ? (T0?)null : E1
      ```

      except that `P` is evaluated only once.

   *  Otherwise the type of `E` is `T0`, and the meaning of `E` is the same as

      ```csharp
      ((object)P == null) ? null : E1
      ```

      except that `P` is evaluated only once.

If `E1` is itself a *null_conditional_expression*, then these rules are applied again, nesting the tests for `null` until there are no further `?`'s, and the expression has been reduced all the way down to the primary-expression `E0`.

For example, if the expression `a.b?[0]?.c()` occurs as a statement-expression, as in the statement:
```csharp
a.b?[0]?.c();
```
its meaning is equivalent to:
```csharp
if (a.b != null) a.b[0]?.c();
```
which again is equivalent to:
```csharp
if (a.b != null) if (a.b[0] != null) a.b[0].c();
```
Except that `a.b` and `a.b[0]` are evaluated only once.

If it occurs in a context where its value is used, as in:
```csharp
var x = a.b?[0]?.c();
```
and assuming that the type of the final invocation is not a non-nullable value type, its meaning is equivalent to:
```csharp
var x = (a.b == null) ? null : (a.b[0] == null) ? null : a.b[0].c();
```
except that `a.b` and `a.b[0]` are evaluated only once.

> __Expression Tree Conversion Translation Steps__
>
> A *null_conditional_expression*
>
> ```antlr
> null_conditional_expression
>     : primary_expression null_conditional_operations
>     ;
>
> null_conditional_operations
>     : null_conditional_operations? '?' '.' identifier type_argument_list?
>     | null_conditional_operations? '?' '[' argument_list ']'
>     | null_conditional_operations '.' identifier type_argument_list?
>     | null_conditional_operations '[' argument_list ']'
>     | null_conditional_operations '(' argument_list? ')'
>     ;
> ```
>
> is translated to an expression tree as follows.
>
> If none of the *null_conditional_operation* contains a `?` token, the expression does not involve any null-conditional evaluation and is converted to an expression tree using the rules for member access ([Member access](#member-access)), element access ([Element access](#element-access)), or invocation expressions ([Invocation expressions](#invocation-expressions)). Otherwise, the rules below apply.
>
> When converted to a query expression tree, a compile-time error occurs.
>
> When converted to a generalized expression tree, first translate a *null_conditional_expression* by textually splitting the expression at the first occurrence of `?`:
>
> * `R` is the expression constructed from the portion preceding the first `?` token in the expression, and,
> * `C` is the expression constructed from a compiler-generated identifier `t` (which is otherwise invisible) and from the portion `O` succeeding the first `?` token in the expression, such that `C` is of the form `tO`.
>
> For example, an expression `a.b?[0]?.c()` is split into `a.b` (for `R`) and `t[0]?c()` (for `C`).
>
> Next, construct an expression `expr` by converting expression `R` to an expression tree, and construct an expression `type` of of type `Type` representing type `N` which is the non-nullable variant of `T`, where `T` is the type of `R`:
>
> * If `T` is a nullable value type `U?`, `N` is defined to be `U`.
> * Otherwise, if `T` is a non-nullable value type or a reference type, `N` is defined to be `T`.
>
> Expression tree conversion then proceeds as follows. First, construct a node representing the null-conditional receiver
>
> ```csharp
> var r = Q.ConditionalReceiver(default(Q.Flags), type);
> ```
>
> where `r` is a compiler-generated identifier that is otherwise invisible. Next, convert the expression `C` to an expression tree `nonNullExpr`, converting the single occurrence of identifier `t` to `r`, as defined above. Finally, translate the *null_conditional_expression* to
>
> ```csharp
> Q.ConditionalAccess(default(Q.Flags), expr, r, nonNullExpr)
> ```
>
> This translation can be thought of as introducing a variable `t`, holding the result of evaluating `R` to a non-null value, in the scope of `C`.

#### Null-conditional expressions as projection initializers

A null-conditional expression is only allowed as a *member_declarator* in an *anonymous_object_creation_expression* ([Anonymous object creation expressions](expressions.md#anonymous-object-creation-expressions)) if it ends with an (optionally null-conditional) member access. Grammatically, this requirement can be expressed as:

```antlr
null_conditional_member_access
    : primary_expression null_conditional_operations? '?' '.' identifier type_argument_list?
    | primary_expression null_conditional_operations '.' identifier type_argument_list?
    ;
```

This is a special case of the grammar for *null_conditional_expression* above. The production for *member_declarator* in [Anonymous object creation expressions](expressions.md#anonymous-object-creation-expressions) then includes only *null_conditional_member_access*.

#### Null-conditional expressions as statement expressions

A null-conditional expression is only allowed as a *statement_expression* ([Expression statements](statements.md#expression-statements)) if it ends with an invocation. Grammatically, this requirement can be expressed as:

```antlr
null_conditional_invocation_expression
    : primary_expression null_conditional_operations '(' argument_list? ')'
    ;
```

This is a special case of the grammar for *null_conditional_expression* above. The production for *statement_expression* in [Expression statements](statements.md#expression-statements) then includes only *null_conditional_invocation_expression*.


### Unary plus operator

For an operation of the form `+x`, unary operator overload resolution ([Unary operator overload resolution](expressions.md#unary-operator-overload-resolution)) is applied to select a specific operator implementation. The operand is converted to the parameter type of the selected operator, and the type of the result is the return type of the operator. The predefined unary plus operators are:

```csharp
int operator +(int x);
uint operator +(uint x);
long operator +(long x);
ulong operator +(ulong x);
float operator +(float x);
double operator +(double x);
decimal operator +(decimal x);
```

For each of these operators, the result is simply the value of the operand.

> __Expression Tree Conversion Translation Steps__
>
> A `+` unary plus expression
>
> ```csharp
> +expression
> ```
>
> is translated into an expression tree by first translating `expression` to `expr`, and by optionally constructing an expression `method` of type `MethodInfo` representing the user-defined `operator +` implementation, if the operator is bound to a user-defined unary operator. Expression tree translation then proceeds as follows.
>
> When subject to conversion to a query expression tree, and `expression` has type `dynamic`, translation fails and an error is reported. Otherwise, it is translated into
>
> ```csharp
> Expression.UnaryPlus(expr, method)
> ```
>
> if the operator is bound to a user-defined implementation of the unary `operator +`, or into
>
> ```csharp
> expr
> ```
>
> otherwise. That is, the unary plus operator is optimized away in this case.
>
> When subject to conversion to a generalized expression tree, it is translated into
>
> ```csharp
> Q.UnaryPlus(info, expr)
> ```
>
> where `info` is
>
> ```csharp
> Q.UnaryPlusInfo(binder)
> ```
>
> if `expression` is of type `dynamic`, where `binder` is an expression of type `Microsoft.CSharp.RuntimeBinder.CallSiteBinder` representing the dynamic operation, or
>
> ```csharp
> Q.UnaryPlusInfo(flags, method)
> ```
>
> if the operator is bound to a user-defined implementation of the unary `operator +`, or
>
> ```csharp
> Q.UnaryPlusInfo(flags)
> ```
>
> otherwise, where `flags` is either:
>
> * `Q.Flags.CheckedContext` if the unary plus expression occurs in a checked context, or,
> * `default(Q.Flags)` otherwise.
>
> Note that the behavior of the unary plus operator is typically not affected by the use in a checked context. For purposes of generalized expression trees, this information is retained for all operators, and passed as a flag. The bound expression tree library is free to ignore this information.
>
> ***TODO***:
> * Evaluate options for the dynamic case:
>   * create a specification for the construction of `binder` (can also be used to specify `dynamic` behavior outside the context of expression trees) and consider making more properties on the `Microsoft.CSharp.RuntimeBinder` library types publicly accessable so expression tree libraries can inspect this info, or,
>   * steer away from reusing `Microsoft.CSharp` and introduce a `UnaryPlusInfo` factory overload for dynamic binding (e.g. a `(Q.DynamicFlags flags, Type context, A argumentInfo)` overload, where `A` is built through factory invocations as well).

### Unary minus operator

For an operation of the form `-x`, unary operator overload resolution ([Unary operator overload resolution](expressions.md#unary-operator-overload-resolution)) is applied to select a specific operator implementation. The operand is converted to the parameter type of the selected operator, and the type of the result is the return type of the operator. The predefined negation operators are:

*  Integer negation:

   ```csharp
   int operator -(int x);
   long operator -(long x);
   ```

   The result is computed by subtracting `x` from zero. If the value of of `x` is the smallest representable value of the operand type (-2^31 for `int` or -2^63 for `long`), then the mathematical negation of `x` is not representable within the operand type. If this occurs within a `checked` context, a `System.OverflowException` is thrown; if it occurs within an `unchecked` context, the result is the value of the operand and the overflow is not reported.

   If the operand of the negation operator is of type `uint`, it is converted to type `long`, and the type of the result is `long`. An exception is the rule that permits the `int` value -2147483648 (-2^31) to be written as a decimal integer literal ([Integer literals](lexical-structure.md#integer-literals)).

   If the operand of the negation operator is of type `ulong`, a compile-time error occurs. An exception is the rule that permits the `long` value -9223372036854775808 (-2^63) to be written as a decimal integer literal ([Integer literals](lexical-structure.md#integer-literals)).

*  Floating-point negation:

   ```csharp
   float operator -(float x);
   double operator -(double x);
   ```

   The result is the value of `x` with its sign inverted. If `x` is NaN, the result is also NaN.

*  Decimal negation:

   ```csharp
   decimal operator -(decimal x);
   ```

   The result is computed by subtracting `x` from zero. Decimal negation is equivalent to using the unary minus operator of type `System.Decimal`.

> __Expression Tree Conversion Translation Steps__
>
> A `-` unary minus expression
>
> ```csharp
> -expression
> ```
>
> is translated into an expression tree by first translating `expression` to `expr`, and by optionally constructing an expression `method` of type `MethodInfo` representing the user-defined `operator -` implementation, if the operator is bound to a user-defined unary operator. Expression tree translation then proceeds as follows.
>
> When subject to conversion to a query expression tree, and `expression` has type `dynamic`, translation fails and an error is reported. Otherwise, it is translated into
>
> ```csharp
> Expression.Negate(expr, method)
> ```
>
> if the operator is bound to a user-defined implementation of the unary `operator -`. Otherwise, it is translated into
>
> ```csharp
> Expression.NegateChecked(expr)
> ```
>
> if the unary minus expression occurs in a checked context, or
>
> ```csharp
> Expression.Negate(expr)
> ```
>
> otherwise.
>
> When subject to conversion to a generalized expression tree, it is translated into
>
> ```csharp
> Q.Negate(info, expr)
> ```
>
> where `info` is
>
> ```csharp
> Q.NegateInfo(binder)
> ```
>
> if `expression` is of type `dynamic`, where `binder` is an expression of type `Microsoft.CSharp.RuntimeBinder.CallSiteBinder` representing the dynamic operation, or
>
> ```csharp
> Q.NegateInfo(flags, method)
> ```
>
> if the operator is bound to a user-defined implementation of the unary `operator -`, or
>
> ```csharp
> Q.NegateInfo(flags)
> ```
>
> otherwise, where `flags` is either:
>
> * `Q.Flags.CheckedContext` if the unary minus expression occurs in a checked context, or,
> * `default(Q.Flags)` otherwise.
>
> Note that the behavior of a user-defined unary negation operator is typically not affected by the use in a checked context. For purposes of generalized expression trees, this information is retained for all operators, and passed as a flag. The bound expression tree library is free to ignore this information.
>
> ***TODO***:
> * Evaluate options for the dynamic case:
>   * create a specification for the construction of `binder` (can also be used to specify `dynamic` behavior outside the context of expression trees) and consider making more properties on the `Microsoft.CSharp.RuntimeBinder` library types publicly accessable so expression tree libraries can inspect this info, or,
>   * steer away from reusing `Microsoft.CSharp` and introduce a `NegateInfo` factory overload for dynamic binding (e.g. a `(Q.DynamicFlags flags, Type context, A argumentInfo)` overload, where `A` is built through factory invocations as well).

### Logical negation operator

For an operation of the form `!x`, unary operator overload resolution ([Unary operator overload resolution](expressions.md#unary-operator-overload-resolution)) is applied to select a specific operator implementation. The operand is converted to the parameter type of the selected operator, and the type of the result is the return type of the operator. Only one predefined logical negation operator exists:
```csharp
bool operator !(bool x);
```

This operator computes the logical negation of the operand: If the operand is `true`, the result is `false`. If the operand is `false`, the result is `true`.

> __Expression Tree Conversion Translation Steps__
>
> A `!` unary logical negation expression
>
> ```csharp
> !expression
> ```
>
> is translated into an expression tree by first translating `expression` to `expr`, and by optionally constructing an expression `method` of type `MethodInfo` representing the user-defined `operator !` implementation, if the operator is bound to a user-defined unary operator. Expression tree translation then proceeds as follows.
>
> When subject to conversion to a query expression tree, and `expression` has type `dynamic`, translation fails and an error is reported. Otherwise, it is translated into
>
> ```csharp
> Expression.Not(expr, method)
> ```
>
> if the operator is bound to a user-defined implementation of the unary `operator !`. Otherwise, it is translated into
>
> ```csharp
> Expression.Not(expr)
> ```
>
> When subject to conversion to a generalized expression tree, it is translated into
>
> ```csharp
> Q.Not(info, expr)
> ```
>
> where `info` is
>
> ```csharp
> Q.NotInfo(binder)
> ```
>
> if `expression` is of type `dynamic`, where `binder` is an expression of type `Microsoft.CSharp.RuntimeBinder.CallSiteBinder` representing the dynamic operation, or
>
> ```csharp
> Q.NotInfo(flags, method)
> ```
>
> if the operator is bound to a user-defined implementation of the unary `operator !`, or
>
> ```csharp
> Q.NotInfo(flags)
> ```
>
> otherwise, where `flags` is either:
>
> * `Q.Flags.CheckedContext` if the unary logical negation expression occurs in a checked context, or,
> * `default(Q.Flags)` otherwise.
>
> Note that the behavior of a unary logical negation operator is typically not affected by the use in a checked context. For purposes of generalized expression trees, this information is retained for all operators, and passed as a flag. The bound expression tree library is free to ignore this information.
>
> ***TODO***:
> * Evaluate options for the dynamic case:
>   * create a specification for the construction of `binder` (can also be used to specify `dynamic` behavior outside the context of expression trees) and consider making more properties on the `Microsoft.CSharp.RuntimeBinder` library types publicly accessable so expression tree libraries can inspect this info, or,
>   * steer away from reusing `Microsoft.CSharp` and introduce a `NotInfo` factory overload for dynamic binding (e.g. a `(Q.DynamicFlags flags, Type context, A argumentInfo)` overload, where `A` is built through factory invocations as well).

### Bitwise complement operator

For an operation of the form `~x`, unary operator overload resolution ([Unary operator overload resolution](expressions.md#unary-operator-overload-resolution)) is applied to select a specific operator implementation. The operand is converted to the parameter type of the selected operator, and the type of the result is the return type of the operator. The predefined bitwise complement operators are:
```csharp
int operator ~(int x);
uint operator ~(uint x);
long operator ~(long x);
ulong operator ~(ulong x);
```

For each of these operators, the result of the operation is the bitwise complement of `x`.

Every enumeration type `E` implicitly provides the following bitwise complement operator:

```csharp
E operator ~(E x);
```

The result of evaluating `~x`, where `x` is an expression of an enumeration type `E` with an underlying type `U`, is exactly the same as evaluating `(E)(~(U)x)`, except that the conversion to `E` is always performed as if in an `unchecked` context ([The checked and unchecked operators](expressions.md#the-checked-and-unchecked-operators)).

> __Expression Tree Conversion Translation Steps__
>
> A `~` unary bitwise complement expression
>
> ```csharp
> ~expression
> ```
>
> is translated into an expression tree by first translating `expression` to `expr`, and by optionally constructing an expression `method` of type `MethodInfo` representing the user-defined `operator ~` implementation, if the operator is bound to a user-defined unary operator. Expression tree translation then proceeds as follows.
>
> When subject to conversion to a query expression tree, and `expression` has type `dynamic`, translation fails and an error is reported. Otherwise, it is translated into
>
> ```csharp
> Expression.Not(expr, method)
> ```
>
> if the operator is bound to a user-defined implementation of the unary `operator ~`. Otherwise, it is translated into
>
> ```csharp
> Expression.Not(expr)
> ```
>
> Note that query expression trees are not using a `OnesComplement` factory for purposes of compatibility.
>
> When subject to conversion to a generalized expression tree, it is translated into
>
> ```csharp
> Q.OnesComplement(info, expr)
> ```
>
> where `info` is
>
> ```csharp
> Q.OnesComplementInfo(binder)
> ```
>
> if `expression` is of type `dynamic`, where `binder` is an expression of type `Microsoft.CSharp.RuntimeBinder.CallSiteBinder` representing the dynamic operation, or
>
> ```csharp
> Q.OnesComplementInfo(flags, method)
> ```
>
> if the operator is bound to a user-defined implementation of the unary `operator ~`, or
>
> ```csharp
> Q.OnesComplementInfo(flags)
> ```
>
> otherwise, where `flags` is either:
>
> * `Q.Flags.CheckedContext` if the unary ones complement expression occurs in a checked context, or,
> * `default(Q.Flags)` otherwise.
>
> Note that the behavior of a unary bitwise complement operator is typically not affected by the use in a checked context. For purposes of generalized expression trees, this information is retained for all operators, and passed as a flag. The bound expression tree library is free to ignore this information.
>
> ***TODO***:
> * Evaluate options for the dynamic case:
>   * create a specification for the construction of `binder` (can also be used to specify `dynamic` behavior outside the context of expression trees) and consider making more properties on the `Microsoft.CSharp.RuntimeBinder` library types publicly accessable so expression tree libraries can inspect this info, or,
>   * steer away from reusing `Microsoft.CSharp` and introduce a `OnesComplementInfo` factory overload for dynamic binding (e.g. a `(Q.DynamicFlags flags, Type context, A argumentInfo)` overload, where `A` is built through factory invocations as well).

### Prefix increment and decrement operators

```antlr
pre_increment_expression
    : '++' unary_expression
    ;

pre_decrement_expression
    : '--' unary_expression
    ;
```

The operand of a prefix increment or decrement operation must be an expression classified as a variable, a property access, or an indexer access. The result of the operation is a value of the same type as the operand.

If the operand of a prefix increment or decrement operation is a property or indexer access, the property or indexer must have both a `get` and a `set` accessor. If this is not the case, a binding-time error occurs.

Unary operator overload resolution ([Unary operator overload resolution](expressions.md#unary-operator-overload-resolution)) is applied to select a specific operator implementation. Predefined `++` and `--` operators exist for the following types: `sbyte`, `byte`, `short`, `ushort`, `int`, `uint`, `long`, `ulong`, `char`, `float`, `double`, `decimal`, and any enum type. The predefined `++` operators return the value produced by adding 1 to the operand, and the predefined `--` operators return the value produced by subtracting 1 from the operand. In a `checked` context, if the result of this addition or subtraction is outside the range of the result type and the result type is an integral type or enum type, a `System.OverflowException` is thrown.

The run-time processing of a prefix increment or decrement operation of the form `++x` or `--x` consists of the following steps:

*   If `x` is classified as a variable:
    * `x` is evaluated to produce the variable.
    * The selected operator is invoked with the value of `x` as its argument.
    * The value returned by the operator is stored in the location given by the evaluation of `x`.
    * The value returned by the operator becomes the result of the operation.
*   If `x` is classified as a property or indexer access:
    * The instance expression (if `x` is not `static`) and the argument list (if `x` is an indexer access) associated with `x` are evaluated, and the results are used in the subsequent `get` and `set` accessor invocations.
    * The `get` accessor of `x` is invoked.
    * The selected operator is invoked with the value returned by the `get` accessor as its argument.
    * The `set` accessor of `x` is invoked with the value returned by the operator as its `value` argument.
    * The value returned by the operator becomes the result of the operation.

The `++` and `--` operators also support postfix notation ([Postfix increment and decrement operators](expressions.md#postfix-increment-and-decrement-operators)). Typically, the result of `x++` or `x--` is the value of `x` before the operation, whereas the result of `++x` or `--x` is the value of `x` after the operation. In either case, `x` itself has the same value after the operation.

An `operator++` or `operator--` implementation can be invoked using either postfix or prefix notation. It is not possible to have separate operator implementations for the two notations.

> __Expression Tree Conversion Translation Steps__
>
> A `++` and `--` unary prefix increment and decrement expressions
>
> ```csharp
> ++expression
> --expression
> ```
>
> are translated into an expression tree by first translating `expression` to `expr`, and by optionally constructing an expression `method` of type `MethodInfo` representing the user-defined `operator ++` or `operator --` implementation, if the operator is bound to a user-defined operator. Expression tree translation then proceeds as follows.
>
> When subject to conversion to a query expression tree, an error is reported.
>
> When subject to conversion to a generalized expression tree, it is translated into either of
>
> ```csharp
> Q.PreIncrement(info, expr)
> Q.PreDecrement(info, expr)
> ```
>
> where `info` is either of
>
> ```csharp
> Q.PreIncrementInfo(binder)
> Q.PreDecrementInfo(binder)
> ```
>
> if `expression` is of type `dynamic`, where `binder` is an expression of type `Microsoft.CSharp.RuntimeBinder.CallSiteBinder` representing the dynamic operation, or
>
> ```csharp
> Q.PreIncrementInfo(flags, method)
> Q.PreDecrementInfo(flags, method)
> ```
>
> if the operator is bound to a user-defined implementation of the `operator ++` or `operator --`, or
>
> ```csharp
> Q.PreIncrementInfo(flags)
> Q.PreDecrementInfo(flags)
> ```
>
> otherwise, where `flags` is the bitwise `|` combination of
>
> * `Q.Flags.ResultDiscarded` if the expression occurs in an *expression_statement* ([Expression statements](statements.md#expression-statements)), or,
> * `Q.Flags.CheckedContext` if the prefix increment or decrement expression occurs in a checked context, or,
> * `default(Q.Flags)` if none of the flags apply.
>
> Note that the behavior of a user-defined increment or decrement operator is typically not affected by the use in a checked context. For purposes of generalized expression trees, this information is retained for all operators, and passed as a flag. The bound expression tree library is free to ignore this information.
>
> ***TODO***:
> * Evaluate options for the dynamic case:
>   * create a specification for the construction of `binder` (can also be used to specify `dynamic` behavior outside the context of expression trees) and consider making more properties on the `Microsoft.CSharp.RuntimeBinder` library types publicly accessable so expression tree libraries can inspect this info, or,
>   * steer away from reusing `Microsoft.CSharp` and introduce a `Pre*Info` factory overload for dynamic binding (e.g. a `(Q.DynamicFlags flags, Type context, A argumentInfo)` overload, where `A` is built through factory invocations as well).

### Cast expressions

A *cast_expression* is used to explicitly convert an expression to a given type.

```antlr
cast_expression
    : '(' type ')' unary_expression
    ;
```

A *cast_expression* of the form `(T)E`, where `T` is a *type* and `E` is a *unary_expression*, performs an explicit conversion ([Explicit conversions](conversions.md#explicit-conversions)) of the value of `E` to type `T`. If no explicit conversion exists from `E` to `T`, a binding-time error occurs. Otherwise, the result is the value produced by the explicit conversion. The result is always classified as a value, even if `E` denotes a variable.

The grammar for a *cast_expression* leads to certain syntactic ambiguities. For example, the expression `(x)-y` could either be interpreted as a *cast_expression* (a cast of `-y` to type `x`) or as an *additive_expression* combined with a *parenthesized_expression* (which computes the value `x - y)`.

To resolve *cast_expression* ambiguities, the following rule exists: A sequence of one or more *token*s ([White space](lexical-structure.md#white-space)) enclosed in parentheses is considered the start of a *cast_expression* only if at least one of the following are true:

*  The sequence of tokens is correct grammar for a *type*, but not for an *expression*.
*  The sequence of tokens is correct grammar for a *type*, and the token immediately following the closing parentheses is the token "`~`", the token "`!`", the token "`(`", an *identifier* ([Unicode character escape sequences](lexical-structure.md#unicode-character-escape-sequences)), a *literal* ([Literals](lexical-structure.md#literals)), or any *keyword* ([Keywords](lexical-structure.md#keywords)) except `as` and `is`.

The term "correct grammar" above means only that the sequence of tokens must conform to the particular grammatical production. It specifically does not consider the actual meaning of any constituent identifiers. For example, if `x` and `y` are identifiers, then `x.y` is correct grammar for a type, even if `x.y` doesn't actually denote a type.

From the disambiguation rule it follows that, if `x` and `y` are identifiers, `(x)y`, `(x)(y)`, and `(x)(-y)` are *cast_expression*s, but `(x)-y` is not, even if `x` identifies a type. However, if `x` is a keyword that identifies a predefined type (such as `int`), then all four forms are *cast_expression*s (because such a keyword could not possibly be an expression by itself).

> __Expression Tree Conversion Translation Steps__
>
> A cast expression
>
> ```csharp
> (T)expression
> ```
>
> is translated into an expression tree by first translating `expression` to `expr`, by constructing an expression `type` of type `Type` from `typeof(T)` if `T` is not `dynamic` or `typeof(object)` if `T` is `dynamic`, and by optionally constructing an expression `method` of type `MethodInfo` representing the user-defined `implicit operator T` or `explicit operator T` implementation, if the operator is bound to a user-defined unary operator. Expression tree translation then proceeds as follows.
>
> When subject to conversion to a query expression tree, and `expression` has type `dynamic`, translation fails and an error is reported. Otherwise, it is translated into
>
> ```csharp
> Expression.Convert(expr, type, method)
> ```
>
> if the operator is bound to a user-defined implementation of `implicit operator T` or `explicit operator T`, or
>
> ```csharp
> Expression.ConvertChecked(expr, type)
> ```
>
> if the conversion occurs in a checked context, or
>
> ```csharp
> Expression.Convert(expr, type)
> ```
>
> otherwise.
>
> When subject to conversion to a generalized expression tree, it is translated into
>
> ```csharp
> Q.Convert(info, expr)
> ```
>
> where `info` is
>
> ```csharp
> Q.ConvertInfo(type, binder)
> ```
>
> if `expression` is of type `dynamic`, where `binder` is an expression of type `Microsoft.CSharp.RuntimeBinder.CallSiteBinder` representing the dynamic operation, or
>
> ```csharp
> Q.ConvertInfo(flags, type, method)
> ```
>
> if the operator is bound to a user-defined implementation of `implicit operator T` or `explicit operator T`, or
>
> ```csharp
> Q.ConvertInfo(flags, type)
> ```
>
> otherwise, where `flags` is a bitwise `|` of any of the following:
>
> * `default(Q.Flags)`,
> * `Q.Flags.CheckedContext` if the cast expression occurs in a checked context,
> * `Q.Flags.CompilerGenerated` if the cast expression was inserted (for instance, when translating implicit conversions).
>
> Note that the behavior of a user-defined cast operator is typically not affected by the use in a checked context. For purposes of generalized expression trees, this information is retained for all operators, and passed as a flag. The bound expression tree library is free to ignore this information.
>
> ***TODO***:
> * Evaluate options for the dynamic case:
>   * create a specification for the construction of `binder` (can also be used to specify `dynamic` behavior outside the context of expression trees) and consider making more properties on the `Microsoft.CSharp.RuntimeBinder` library types publicly accessable so expression tree libraries can inspect this info, or,
>   * steer away from reusing `Microsoft.CSharp` and introduce a `ConvertInfo` factory overload for dynamic binding (e.g. a `(Q.DynamicFlags flags, Type context, Type type)` overload).

### Await expressions

The await operator is used to suspend evaluation of the enclosing async function until the asynchronous operation represented by the operand has completed.

```antlr
await_expression
    : 'await' unary_expression
    ;
```

An *await_expression* is only allowed in the body of an async function ([Iterators](classes.md#iterators)). Within the nearest enclosing async function, an *await_expression* may not occur in these places:

*  Inside a nested (non-async) anonymous function
*  Inside the block of a *lock_statement*
*  In an unsafe context

Note that an *await_expression* cannot occur in most places within a *query_expression*, because those are syntactically transformed to use non-async lambda expressions.

Inside of an async function, `await` cannot be used as an identifier. There is therefore no syntactic ambiguity between await-expressions and various expressions involving identifiers. Outside of async functions, `await` acts as a normal identifier.

The operand of an *await_expression* is called the ***task***. It represents an asynchronous operation that may or may not be complete at the time the *await_expression* is evaluated. The purpose of the await operator is to suspend execution of the enclosing async function until the awaited task is complete, and then obtain its outcome.

#### Awaitable expressions

The task of an await expression is required to be ***awaitable***. An expression `t` is awaitable if one of the following holds:

*  `t` is of compile time type `dynamic`
*  `t` has an accessible instance or extension method called `GetAwaiter` with no parameters and no type parameters, and a return type `A` for which all of the following hold:
   * `A` implements the interface `System.Runtime.CompilerServices.INotifyCompletion` (hereafter known as `INotifyCompletion` for brevity)
   * `A` has an accessible, readable instance property `IsCompleted` of type `bool`
   * `A` has an accessible instance method `GetResult` with no parameters and no type parameters

The purpose of the `GetAwaiter` method is to obtain an ***awaiter*** for the task. The type `A` is called the ***awaiter type*** for the await expression.

The purpose of the `IsCompleted` property is to determine if the task is already complete. If so, there is no need to suspend evaluation.

The purpose of the `INotifyCompletion.OnCompleted` method is to sign up a "continuation" to the task; i.e. a delegate (of type `System.Action`) that will be invoked once the task is complete.

The purpose of the `GetResult` method is to obtain the outcome of the task once it is complete. This outcome may be successful completion, possibly with a result value, or it may be an exception which is thrown by the `GetResult` method.

#### Classification of await expressions

The expression `await t` is classified the same way as the expression `(t).GetAwaiter().GetResult()`. Thus, if the return type of `GetResult` is `void`, the *await_expression* is classified as nothing. If it has a non-void return type `T`, the *await_expression* is classified as a value of type `T`.

#### Runtime evaluation of await expressions

At runtime, the expression `await t` is evaluated as follows:

*  An awaiter `a` is obtained by evaluating the expression `(t).GetAwaiter()`.
*  A `bool` `b` is obtained by evaluating the expression `(a).IsCompleted`.
*  If `b` is `false` then evaluation depends on whether `a` implements the interface `System.Runtime.CompilerServices.ICriticalNotifyCompletion` (hereafter known as `ICriticalNotifyCompletion` for brevity). This check is done at binding time; i.e. at runtime if `a` has the compile time type `dynamic`, and at compile time otherwise. Let `r` denote the resumption delegate ([Iterators](classes.md#iterators)):
    * If `a` does not implement `ICriticalNotifyCompletion`, then the expression 
`(a as (INotifyCompletion)).OnCompleted(r)` is evaluated.
    * If `a` does implement `ICriticalNotifyCompletion`, then the expression 
`(a as (ICriticalNotifyCompletion)).UnsafeOnCompleted(r)` is evaluated.
    * Evaluation is then suspended, and control is returned to the current caller of the async function.
*  Either immediately after (if `b` was `true`), or upon later invocation of the resumption delegate (if `b` was `false`), the expression `(a).GetResult()` is evaluated. If it returns a value, that value is the result of the *await_expression*. Otherwise the result is nothing.

An awaiter's implementation of the interface methods `INotifyCompletion.OnCompleted` and `ICriticalNotifyCompletion.UnsafeOnCompleted` should cause the delegate `r` to be invoked at most once. Otherwise, the behavior of the enclosing async function is undefined.

> __Expression Tree Conversion Translation Steps__
>
> An await expression
>
> ```csharp
> await expression
> ```
>
> is translated into an expression tree by first translating `expression` to `expr`, and by constructing the following if the type of `expression` is not `dynamic`:
>
> * an expression `getAwaiter` of type `MethodInfo` representing the bound `GetAwaiter` method,
> * an expression `isCompleted` of type `MethodInfo` representing the get accessor method of the bound `IsCompleted` property, and,
> * an expression `getResult` of type `MethodInfo` representing the bound `GetResult` method.
>
> Expression tree translation then proceeds as follows.
>
> When subject to conversion to a query expression tree, an error is reported.
>
> When subject to conversion to a generalized expression tree, it is translated into
>
> ```csharp
> Q.Await(info, expr)
> ```
>
> where `info` is
>
> ```csharp
> Q.AwaitInfo(flags, context)
> ```
>
> if `expression` is of type `dynamic`, where `context` is an expression of type `Type` representing the nearest parent type where the await expression is used (for purposes of runtime binding), or
>
> ```csharp
> Q.AwaitInfo(flags, getAwaiter, isCompleted, getResult)
> ```
>
> otherwise, where `flags` is
>
> * `Q.Flags.ResultDiscarded` if the expression occurs in an *expression_statement* ([Expression statements](statements.md#expression-statements)), or,
> * `default(Q.Flags)` otherwise.
>
> Note that the expression tree library can trivially infer any of the following from the supplied parameters to `AwaitInfo`:
>
> * whether `GetAwaiter` is an extension method by checking if the method is `static`,
> * the awaiter type `A` by checking the return type of the `GetAwaiter` method,
> * the type of the await expression by checking the return type of the `GetResult` method, and,
> * whether the awaiter type `A` implements `ICriticalNotifyCompletion`.
>
> That is, `AwaitInfo` gets passed all bound members used to evaluate the await expression, which are hard or impossible to look up at runtime using the rules described in this specification. For example, `GetAwaiter` cannot be determined at runtime if it is an extension method, due to the lack of knowledge about imported namespaces.

## Arithmetic operators

The `*`, `/`, `%`, `+`, and `-` operators are called the arithmetic operators.

```antlr
multiplicative_expression
    : unary_expression
    | multiplicative_expression '*' unary_expression
    | multiplicative_expression '/' unary_expression
    | multiplicative_expression '%' unary_expression
    ;

additive_expression
    : multiplicative_expression
    | additive_expression '+' multiplicative_expression
    | additive_expression '-' multiplicative_expression
    ;
```

If an operand of an arithmetic operator has the compile-time type `dynamic`, then the expression is dynamically bound ([Dynamic binding](expressions.md#dynamic-binding)). In this case the compile-time type of the expression is `dynamic`, and the resolution described below will take place at run-time using the run-time type of those operands that have the compile-time type `dynamic`.

> __Expression Tree Conversion Translation Steps__
>
> Expression tree conversion of arithmetic expressions of the form
>
> ```csharp
> l op r
> ```
>
> where `l` and `r` are expressions, and `op` is any of the `*`, `/`, `%`, `+`, or `-` operators, proceeds by first defining an identifier `M` with value
>
> * `Add` if `op` is `+`, or,
> * `Subtract` if `op` is `-`, or,
> * `Multiply` if `op` is `*`, or,
> * `Divide` if `op` is `/`, or,
> * `Modulo` if `op` is `%`,
>
> and by optionally rewriting `l` and `r` to `(L)(l)` and `(R)(r)` if either operand requires an implicit conversion, for instance due to numeric promotion ([Numeric promotion](#numeric-promotions)), or more generally overload resolution ([Overload resolution](#overload-resolution)), or lifting ([Lifted operators](#lifted-operators)), or conversions described in any of the subsections below. Any cast expression introduced as part of this rewrite step is classified as compiler-generated.
>
> After the rewrite step, construct expressions `lExpr` and `rExpr` by translating the result of rewriting `l` and `r` to expression trees, and optionally construct expression `method` of type `MethodInfo` representing the static method implementing the operator, which can be any of
>
> * a method implementing a user-defined operator, or,
> * a method defined in any of the subsections below.
>
> Expression tree conversion then proceeds as follows.
>
> When converted to a query expression tree, if the operation is dynamically bound, a compile-time error occurs. Otherwise, it is translated into
>
> ```csharp
> Expression.M(lExpr, rExpr, method)
> ```
>
> if `method` is defined, or
>
> ```csharp
> Expression.MChecked(lExpr, rExpr)
> ```
>
> otherwise, if all of the following conditions hold:
>
> * the operation occurs in a checked context, and,
> * the operator is `+`, `-`, or `*`, and,
> * any of the following conditions hold:
>   * the operation is `+` or `-` defined on enum types, or,
>   * the operation is predefined on `int`, `uint`, `long`, or `ulong`, or
>   * the operation involves numeric promotion of any of the above, or,
>   * the operation is the lifted variant of any of the above,
>
> or
>
> ```csharp
> Expression.M(lExpr, rExpr)
> ```
>
> otherwise. A few examples can help to clarify these rules:
>
> * `unchecked(a + b)` on operands of type `int` uses method `Add`,
> * `checked(a - b)` on operands of type `int` uses method `SutbractChecked`,
> * `checked(a * b)` on operands of type `decimal` uses method `Multiply` because `method` is defined,
> * `checked(a / b)` uses method `Divide` because `/` does not have a checked variant.
>
> When converted to a a generalized expression tree, it is translated into
>
> ```csharp
> Q.M(info, lExpr, rExpr)
> ```
>
> where `info` is
>
> ```csharp
> Q.MInfo(binder)
> ```
>
> if the operation is dynamically bound, where where `binder` is an expression of type `Microsoft.CSharp.RuntimeBinder.CallSiteBinder` representing the dynamic operation, or
>
> ```csharp
> Q.MInfo(flags, method)
> ```
>
> if `method` is defined, or,
>
> ```csharp
> Q.MInfo(flags)
> ```
>
> where flags is the bitwise `|` combination of any of the following:
>
> * `Q.Flags.Checked` if the operation occurs in a checked context,
> * `Q.Flags.IsLifted` if the operation involves lifting to null,
> * `default(Q.Flags)` if none of these apply.
>
> Note that generalized expression trees retain information about the checked context under all circumstances and use a flag to represent this information. This enables expression tree libraries to interpret the meaning of checked arithmetic which may be relevant to expression evaluation involving user-defined operators. For instance, a `+` operator defined on `Matrix` values may benefit from applying checked arithmetic to the `+` operation applied to the elements of the matrices.

### Multiplication operator

For an operation of the form `x * y`, binary operator overload resolution ([Binary operator overload resolution](expressions.md#binary-operator-overload-resolution)) is applied to select a specific operator implementation. The operands are converted to the parameter types of the selected operator, and the type of the result is the return type of the operator.

The predefined multiplication operators are listed below. The operators all compute the product of `x` and `y`.

*  Integer multiplication:

   ```csharp
   int operator *(int x, int y);
   uint operator *(uint x, uint y);
   long operator *(long x, long y);
   ulong operator *(ulong x, ulong y);
   ```

   In a `checked` context, if the product is outside the range of the result type, a `System.OverflowException` is thrown. In an `unchecked` context, overflows are not reported and any significant high-order bits outside the range of the result type are discarded.


*  Floating-point multiplication:

   ```csharp
   float operator *(float x, float y);
   double operator *(double x, double y);
   ```

   The product is computed according to the rules of IEEE 754 arithmetic. The following table lists the results of all possible combinations of nonzero finite values, zeros, infinities, and NaN's. In the table, `x` and `y` are positive finite values. `z` is the result of `x * y`. If the result is too large for the destination type, `z` is infinity. If the result is too small for the destination type, `z` is zero.

   |      |      |      |     |     |      |      |     |
   |:----:|-----:|:----:|:---:|:---:|:----:|:----:|:----|
   |      | +y   | -y   | +0  | -0  | +inf | -inf | NaN | 
   | +x   | +z   | -z   | +0  | -0  | +inf | -inf | NaN | 
   | -x   | -z   | +z   | -0  | +0  | -inf | +inf | NaN | 
   | +0   | +0   | -0   | +0  | -0  | NaN  | NaN  | NaN | 
   | -0   | -0   | +0   | -0  | +0  | NaN  | NaN  | NaN | 
   | +inf | +inf | -inf | NaN | NaN | +inf | -inf | NaN | 
   | -inf | -inf | +inf | NaN | NaN | -inf | +inf | NaN | 
   | NaN  | NaN  | NaN  | NaN | NaN | NaN  | NaN  | NaN | 

*  Decimal multiplication:

   ```csharp
   decimal operator *(decimal x, decimal y);
   ```

   If the resulting value is too large to represent in the `decimal` format, a `System.OverflowException` is thrown. If the result value is too small to represent in the `decimal` format, the result is zero. The scale of the result, before any rounding, is the sum of the scales of the two operands.

   Decimal multiplication is equivalent to using the multiplication operator of type `System.Decimal`.

   > __Expression Tree Conversion Translation Steps__
   >
   > For purposes of expression tree conversion of decimal multiplication, `method` is defined as the `op_Multiply` method defined on `System.Decimal`.


### Division operator

For an operation of the form `x / y`, binary operator overload resolution ([Binary operator overload resolution](expressions.md#binary-operator-overload-resolution)) is applied to select a specific operator implementation. The operands are converted to the parameter types of the selected operator, and the type of the result is the return type of the operator.

The predefined division operators are listed below. The operators all compute the quotient of `x` and `y`.

*  Integer division:

   ```csharp
   int operator /(int x, int y);
   uint operator /(uint x, uint y);
   long operator /(long x, long y);
   ulong operator /(ulong x, ulong y);
   ```

   If the value of the right operand is zero, a `System.DivideByZeroException` is thrown.

   The division rounds the result towards zero. Thus the absolute value of the result is the largest possible integer that is less than or equal to the absolute value of the quotient of the two operands. The result is zero or positive when the two operands have the same sign and zero or negative when the two operands have opposite signs.

   If the left operand is the smallest representable `int` or `long` value and the right operand is `-1`, an overflow occurs. In a `checked` context, this causes a `System.ArithmeticException` (or a subclass thereof) to be thrown. In an `unchecked` context, it is implementation-defined as to whether a `System.ArithmeticException` (or a subclass thereof) is thrown or the overflow goes unreported with the resulting value being that of the left operand.

*  Floating-point division:

   ```csharp
   float operator /(float x, float y);
   double operator /(double x, double y);
   ```

   The quotient is computed according to the rules of IEEE 754 arithmetic. The following table lists the results of all possible combinations of nonzero finite values, zeros, infinities, and NaN's. In the table, `x` and `y` are positive finite values. `z` is the result of `x / y`. If the result is too large for the destination type, `z` is infinity. If the result is too small for the destination type, `z` is zero.

   |      |      |      |      |      |      |      |      |
   |:----:|:----:|:----:|:----:|:----:|:----:|:----:|:----:|
   |      | +y   | -y   | +0   | -0   | +inf | -inf | NaN  | 
   | +x   | +z   | -z   | +inf | -inf | +0   | -0   | NaN  | 
   | -x   | -z   | +z   | -inf | +inf | -0   | +0   | NaN  | 
   | +0   | +0   | -0   | NaN  | NaN  | +0   | -0   | NaN  | 
   | -0   | -0   | +0   | NaN  | NaN  | -0   | +0   | NaN  | 
   | +inf | +inf | -inf | +inf | -inf | NaN  | NaN  | NaN  | 
   | -inf | -inf | +inf | -inf | +inf | NaN  | NaN  | NaN  | 
   | NaN  | NaN  | NaN  | NaN  | NaN  | NaN  | NaN  | NaN  | 

*  Decimal division:

   ```csharp
   decimal operator /(decimal x, decimal y);
   ```

   If the value of the right operand is zero, a `System.DivideByZeroException` is thrown. If the resulting value is too large to represent in the `decimal` format, a `System.OverflowException` is thrown. If the result value is too small to represent in the `decimal` format, the result is zero. The scale of the result is the smallest scale that will preserve a result equal to the nearest representable decimal value to the true mathematical result.

   Decimal division is equivalent to using the division operator of type `System.Decimal`.

   > __Expression Tree Conversion Translation Steps__
   >
   > For purposes of expression tree conversion of decimal division, `method` is defined as the `op_Division` method defined on `System.Decimal`.


### Remainder operator

For an operation of the form `x % y`, binary operator overload resolution ([Binary operator overload resolution](expressions.md#binary-operator-overload-resolution)) is applied to select a specific operator implementation. The operands are converted to the parameter types of the selected operator, and the type of the result is the return type of the operator.

The predefined remainder operators are listed below. The operators all compute the remainder of the division between `x` and `y`.

*  Integer remainder:

   ```csharp
   int operator %(int x, int y);
   uint operator %(uint x, uint y);
   long operator %(long x, long y);
   ulong operator %(ulong x, ulong y);
   ```

   The result of `x % y` is the value produced by `x - (x / y) * y`. If `y` is zero, a `System.DivideByZeroException` is thrown.

   If the left operand is the smallest `int` or `long` value and the right operand is `-1`, a `System.OverflowException` is thrown. In no case does `x % y` throw an exception where `x / y` would not throw an exception.

*  Floating-point remainder:

   ```csharp
   float operator %(float x, float y);
   double operator %(double x, double y);
   ```

   The following table lists the results of all possible combinations of nonzero finite values, zeros, infinities, and NaN's. In the table, `x` and `y` are positive finite values. `z` is the result of `x % y` and is computed as `x - n * y`, where `n` is the largest possible integer that is less than or equal to `x / y`. This method of computing the remainder is analogous to that used for integer operands, but differs from the IEEE 754 definition (in which `n` is the integer closest to `x / y`).

   |      |      |      |      |      |      |      |      |
   |:----:|:----:|:----:|:----:|:----:|:----:|:----:|:----:|
   |      | +y   | -y   | +0   | -0   | +inf | -inf | NaN  | 
   | +x   | +z   | +z   | NaN  | NaN  | x    | x    | NaN  | 
   | -x   | -z   | -z   | NaN  | NaN  | -x   | -x   | NaN  | 
   | +0   | +0   | +0   | NaN  | NaN  | +0   | +0   | NaN  | 
   | -0   | -0   | -0   | NaN  | NaN  | -0   | -0   | NaN  | 
   | +inf | NaN  | NaN  | NaN  | NaN  | NaN  | NaN  | NaN  | 
   | -inf | NaN  | NaN  | NaN  | NaN  | NaN  | NaN  | NaN  | 
   | NaN  | NaN  | NaN  | NaN  | NaN  | NaN  | NaN  | NaN  | 

*  Decimal remainder:

   ```csharp
   decimal operator %(decimal x, decimal y);
   ```

   If the value of the right operand is zero, a `System.DivideByZeroException` is thrown. The scale of the result, before any rounding, is the larger of the scales of the two operands, and the sign of the result, if non-zero, is the same as that of `x`.

   Decimal remainder is equivalent to using the remainder operator of type `System.Decimal`.

   > __Expression Tree Conversion Translation Steps__
   >
   > For purposes of expression tree conversion of decimal remainder, `method` is defined as the `op_Modulus` method defined on `System.Decimal`.


### Addition operator

For an operation of the form `x + y`, binary operator overload resolution ([Binary operator overload resolution](expressions.md#binary-operator-overload-resolution)) is applied to select a specific operator implementation. The operands are converted to the parameter types of the selected operator, and the type of the result is the return type of the operator.

The predefined addition operators are listed below. For numeric and enumeration types, the predefined addition operators compute the sum of the two operands. When one or both operands are of type string, the predefined addition operators concatenate the string representation of the operands.

*  Integer addition:

   ```csharp
   int operator +(int x, int y);
   uint operator +(uint x, uint y);
   long operator +(long x, long y);
   ulong operator +(ulong x, ulong y);
   ```

   In a `checked` context, if the sum is outside the range of the result type, a `System.OverflowException` is thrown. In an `unchecked` context, overflows are not reported and any significant high-order bits outside the range of the result type are discarded.

*  Floating-point addition:

   ```csharp
   float operator +(float x, float y);
   double operator +(double x, double y);
   ```

   The sum is computed according to the rules of IEEE 754 arithmetic. The following table lists the results of all possible combinations of nonzero finite values, zeros, infinities, and NaN's. In the table, `x` and `y` are nonzero finite values, and `z` is the result of `x + y`. If `x` and `y` have the same magnitude but opposite signs, `z` is positive zero. If `x + y` is too large to represent in the destination type, `z` is an infinity with the same sign as `x + y`.

   |      |      |      |      |      |      |      |
   |:----:|:----:|:----:|:----:|:----:|:----:|:----:|
   |      | y    | +0   | -0   | +inf | -inf | NaN  | 
   | x    | z    | x    | x    | +inf | -inf | NaN  | 
   | +0   | y    | +0   | +0   | +inf | -inf | NaN  | 
   | -0   | y    | +0   | -0   | +inf | -inf | NaN  | 
   | +inf | +inf | +inf | +inf | +inf | NaN  | NaN  | 
   | -inf | -inf | -inf | -inf | NaN  | -inf | NaN  | 
   | NaN  | NaN  | NaN  | NaN  | NaN  | NaN  | NaN  | 

*  Decimal addition:

   ```csharp
   decimal operator +(decimal x, decimal y);
   ```

   If the resulting value is too large to represent in the `decimal` format, a `System.OverflowException` is thrown. The scale of the result, before any rounding, is the larger of the scales of the two operands.

   Decimal addition is equivalent to using the addition operator of type `System.Decimal`.

   > __Expression Tree Conversion Translation Steps__
   >
   > For purposes of expression tree conversion of decimal addition, `method` is defined as the `op_Addition` method defined on `System.Decimal`.

*  Enumeration addition. Every enumeration type implicitly provides the following predefined operators, where `E` is the enum type, and `U` is the underlying type of `E`:

   ```csharp
   E operator +(E x, U y);
   E operator +(U x, E y);
   ```

   At run-time these operators are evaluated exactly as `(E)((U)x + (U)y)`.

   > __Expression Tree Conversion Translation Steps__
   >
   > Expression tree conversion for enum addition uses an expansion similar to the one shown above, but with some differences to support nullable lifting and promotion of numeric underlying types.
   >
   > Consider an expression of the form `e + i` or `i + e`, where `e` has type `E` which is an enum type or a nullable enum type, and `i` has type `I` which is an integer type, a nullable integer type, or is implicitly convertible to an integer type or a nullable integer type.
   >
   > If `E` is a non-nullable enum type, let `U` be the underlying type of `E`. If `U` is `int`, `uint`, `long`, or `ulong`, rewrite `e` to `(U)e` and optionally rewrite `i` to `(U)i` if an implicit conversion from `I` to `U` is defined. Otherwise, if `U` is `byte`, `sbyte`, `short`, or `ushort`, rewrite `e` to `(int)e` and rewrite `i` to `(int)(U)i` if an implicit conversion from `I` to `U` is defined, or rewrite `i` to `(int)i` otherwise.
   >
   > If `E` is a nullable type `E0?` where `E0` is an enum type, let `U` be the underlying type of `E0`. If `U` is `int`, `uint`, `long`, or `ulong`, rewrite `e` to `(U?)e` and optionally rewrite `i` to `(U?)i` if an implicit conversion from `I` to `U` or from `I?` to `U?` is defined. Otherwise, if `U` is `byte`, `sbyte`, `short`, or `ushort`, rewrite `e` to `(int?)e` and rewrite `i` to `(int?)(U)i` or `(int?)(U?)i` if an implicit conversion from `I` to `U` or from `I?` to `U?` is defined, or rewrite `i` to `(int?)i` otherwise.
   >
   > Expression tree conversion then proceeds by translating `(E)(e + i)` or `(E)(i + e)`. All conversions introduced in the preceding steps are classified as compiler-generated.
   >
   > Also note that all conversion and addition operations in the resulting expression are subject to checked arithmetic if they occur in a checked context. For example, in query expression trees this will result in the use of `AddChecked` and `ConvertChecked`; in generalized expression trees this will result in the use of the `Q.Flags.CheckedContext` flags.
   >
   > For example, given
   >
   > ```csharp
   > enum IntEnum : int {}
   > enum ByteEnum : byte {}
   >
   > struct ToByte
   > {
   >     public static implicit operator byte(ToByte b) => ...
   > }
   > ```
   >
   > the following conversions take place prior to expression tree translation:
   >
   > * an expression `ie + i` where `ie` is of type `IntEnum` and `i` is of type `int` is converted to `(IntEnum)((int)ie + i)`,
   > * an expression `be + b` where `be` is of type `ByteEnum` and `b` is of type `byte` is converted to `(ByteEnum)((int)be + (int)b)`,
   > * an expression `nbe + nb` where `nbe` is of type `ByteEnum?` and `nb` is of type `byte?` is converted to `(ByteEnum?)((int?)nbe + (int?)nb)`,
   > * an expression `be + tb` where `be` is of type `ByteEnum` and `tb` is of type `ToByte` is converted to `(ByteEnum)((int)be + (int)(byte)tb)`.
   >
   > ***TODO***
   > * These rules are very awkward because they try to capture the current behavior of the C# compiler when translating enum operations to expression trees, which is folding many conversions together.
   > * For generalized expression trees, we should likely revisit these rules:
   >   * We could state that all conversions prompted by the C# specification are inserted as-is.
   >     * This can reduce the amount of expression tree-specific lingo in the C# specification.
   >     * For example, `be + b` (as defined above) would translate to `(ByteEnum)((byte)be + (byte)b)` where the last conversion is redundant, but the C# specification here inserts it. Subsequent promotion from `byte` to `int` for the addition would further rewrite it to `(ByteEnum)((int)(byte)be + (int)(byte)b)`. Similar rules for nullable lifting would apply.
   >   * Given the above, we could either
   >     * leave the resulting expression as is, marking all these conversions as compiler-generated (using `Q.Flags.CompilerGenerated`), or,
   >     * have a well-defined finite fixed set of optimizations that *will* be (thus, guaranteed to be) performed as part of expression tree translation, such as:
   >       * remove identity conversion (e.g. `(byte)b` becomes `b`),
   >       * coalesce numeric conversions (e.g. `(int)(byte)b` becomes `(int)b`)
   >       * etc.

*  String concatenation:

   ```csharp
   string operator +(string x, string y);
   string operator +(string x, object y);
   string operator +(object x, string y);
   ```

   These overloads of the binary `+` operator perform string concatenation. If an operand of string concatenation is `null`, an empty string is substituted. Otherwise, any non-string argument is converted to its string representation by invoking the virtual `ToString` method inherited from type `object`. If `ToString` returns `null`, an empty string is substituted.

   ```csharp
   using System;
   
   class Test
   {
       static void Main() {
           string s = null;
           Console.WriteLine("s = >" + s + "<");        // displays s = ><
           int i = 1;
           Console.WriteLine("i = " + i);               // displays i = 1
           float f = 1.2300E+15F;
           Console.WriteLine("f = " + f);               // displays f = 1.23E+15
           decimal d = 2.900m;
           Console.WriteLine("d = " + d);               // displays d = 2.900
       }
   }
   ```

   The result of the string concatenation operator is a string that consists of the characters of the left operand followed by the characters of the right operand. The string concatenation operator never returns a `null` value. A `System.OutOfMemoryException` may be thrown if there is not enough memory available to allocate the resulting string.

   > __Expression Tree Conversion Translation Steps__
   >
   > Expression tree conversion for string concatenation defines `method` as any of the following methods on `System.String`:
   >
   > * `Concat(string, string)`, if both operands are of type `string`, or
   > * `Concat(object, object)`, otherwise,
   >
   > and introduces boxing conversions, which are classified as compiler-generated, on operands with a value type. For instance, the expression
   >
   > ```csharp
   > s + x
   > ```
   >
   > where `s` is an expression of type `string` and `x` is an expression of type `int` is translated to
   >
   > ```csharp
   > Expression.Add(
   >     sExpr,
   >     Expression.Convert(xExpr, typeof(object)),
   >     method
   > )
   > ```
   >
   > for query expression trees, and to
   >
   > ```csharp
   > Q.Add(
   >     Q.AddInfo(default(Q.Flags), method),
   >     sExpr,
   >     Q.Convert(
   >         Q.ConvertInfo(Q.Flags.CompilerGenerated, typeof(object)),
   >         xExpr
   >     )
   > )
   > ```
   >
   > for generalized expression trees, where `sExpr` and `xExpr` are the result of translating `s` and `x` to expression trees.

*  Delegate combination. Every delegate type implicitly provides the following predefined operator, where `D` is the delegate type:

   ```csharp
   D operator +(D x, D y);
   ```

   The binary `+` operator performs delegate combination when both operands are of some delegate type `D`. (If the operands have different delegate types, a binding-time error occurs.) If the first operand is `null`, the result of the operation is the value of the second operand (even if that is also `null`). Otherwise, if the second operand is `null`, then the result of the operation is the value of the first operand. Otherwise, the result of the operation is a new delegate instance that, when invoked, invokes the first operand and then invokes the second operand. For examples of delegate combination, see [Subtraction operator](expressions.md#subtraction-operator) and [Delegate invocation](delegates.md#delegate-invocation). Since `System.Delegate` is not a delegate type, `operator` `+` is not defined for it.

   > __Expression Tree Conversion Translation Steps__
   >
   > Expression tree conversion for delegate combination defines `method` as the `Combine(Delegate, Delegate)` method on `System.Delegate` and introduces a cast expression to `D` on the result, which is classified as compiler-generated. For instance, the expression
   >
   > ```csharp
   > f1 + f2
   > ```
   >
   > where `f1` and `f2` are expressions of delegate type `Func<int>` is translated to
   >
   > ```csharp
   > Expression.Convert(
   >     Expression.Add(f1Expr, f2Expr, method),
   >     typeof(Func<int>)
   > )
   > ```
   >
   > for query expression trees, and to
   >
   > ```csharp
   > Q.Convert(
   >     Q.ConvertInfo(Q.Flags.CompilerGenerated, typeof(Func<int>))
   >     Q.Add(
   >         Q.AddInfo(default(Q.Flags), method),
   >         f1Expr, f2Expr
   >     )
   > )
   > ```
   >
   > for generalized expression trees, where `f1Expr` and `f2Expr` are the result of translating `f1` and `f2` to expression trees. Note that these conversions do not introduce a cast from `Func<int>` to `Delegate`; the only introduced conversion occurs on the result.

### Subtraction operator

For an operation of the form `x - y`, binary operator overload resolution ([Binary operator overload resolution](expressions.md#binary-operator-overload-resolution)) is applied to select a specific operator implementation. The operands are converted to the parameter types of the selected operator, and the type of the result is the return type of the operator.

The predefined subtraction operators are listed below. The operators all subtract `y` from `x`.

*  Integer subtraction:

   ```csharp
   int operator -(int x, int y);
   uint operator -(uint x, uint y);
   long operator -(long x, long y);
   ulong operator -(ulong x, ulong y);
   ```

   In a `checked` context, if the difference is outside the range of the result type, a `System.OverflowException` is thrown. In an `unchecked` context, overflows are not reported and any significant high-order bits outside the range of the result type are discarded.

*  Floating-point subtraction:

   ```csharp
   float operator -(float x, float y);
   double operator -(double x, double y);
   ```

   The difference is computed according to the rules of IEEE 754 arithmetic. The following table lists the results of all possible combinations of nonzero finite values, zeros, infinities, and NaNs. In the table, `x` and `y` are nonzero finite values, and `z` is the result of `x - y`. If `x` and `y` are equal, `z` is positive zero. If `x - y` is too large to represent in the destination type, `z` is an infinity with the same sign as `x - y`.

   |      |      |      |      |      |      |     |
   |:----:|:----:|:----:|:----:|:----:|:----:|:---:|
   | NaN  | y    | +0   | -0   | +inf | -inf | NaN | 
   | x    | z    | x    | x    | -inf | +inf | NaN | 
   | +0   | -y   | +0   | +0   | -inf | +inf | NaN | 
   | -0   | -y   | -0   | +0   | -inf | +inf | NaN | 
   | +inf | +inf | +inf | +inf | NaN  | +inf | NaN | 
   | -inf | -inf | -inf | -inf | -inf | NaN  | NaN | 
   | NaN  | NaN  | NaN  | NaN  | NaN  | NaN  | NaN | 

*  Decimal subtraction:

   ```csharp
   decimal operator -(decimal x, decimal y);
   ```

   If the resulting value is too large to represent in the `decimal` format, a `System.OverflowException` is thrown. The scale of the result, before any rounding, is the larger of the scales of the two operands.

   Decimal subtraction is equivalent to using the subtraction operator of type `System.Decimal`.

   > __Expression Tree Conversion Translation Steps__
   >
   > For purposes of expression tree conversion of decimal subtraction, `method` is defined as the `op_Subtraction` method defined on `System.Decimal`.

*  Enumeration subtraction. Every enumeration type implicitly provides the following predefined operator, where `E` is the enum type, and `U` is the underlying type of `E`:

   ```csharp
   U operator -(E x, E y);
   ```

   This operator is evaluated exactly as `(U)((U)x - (U)y)`. In other words, the operator computes the difference between the ordinal values of `x` and `y`, and the type of the result is the underlying type of the enumeration.

   ```csharp
   E operator -(E x, U y);
   ```

   This operator is evaluated exactly as `(E)((U)x - y)`. In other words, the operator subtracts a value from the underlying type of the enumeration, yielding a value of the enumeration.

   > __Expression Tree Conversion Translation Steps__
   >
   > Expression tree conversion for enum subtraction uses an expansion similar to the one shown above, but with some differences to support nullable lifting and promotion of numeric underlying types.
   >
   > For an expression of the form `e1 - e2`, where `e1` and `e2` have type `E` which is an enum type or a nullable enum type, rewrite the expression as follows:
   >
   > * If `E` is a non-nullable enum type, let `U` be the underlying type of `E`. If `U` is `int`, `uint`, `long`, or `ulong`, rewrite the expression to `(U)((U)e1 - (U)e2)`. Otherwise, if `U` is `byte`, `sbyte`, `short`, or `ushort`, rewrite the expression to `(U)((int)e1 - (int)e2)`.
   >
   > * If `E` is a nullable type `E0?` where `E0` is an enum type, let `U` be the underlying type of `E0`. If `U` is `int`, `uint`, `long`, or `ulong`, rewrite the expression to `(U?)((U?)e1 - (U?)e2)`. Otherwise, if `U` is `byte`, `sbyte`, `short`, or `ushort`, rewrite the expression to `(U?)((int?)e1 - (int?)e2)`.
   >
   > For an expression of the form `e - i`, where `e` has type `E` which is an enum type or a nullable enum type, and `i` has type `I` which is an integer type, a nullable integer type, or is implicitly convertible to an integer type or a nullable integer type, rewrite the expression as follows:
   >
   > * If `E` is a non-nullable enum type, let `U` be the underlying type of `E`. If `U` is `int`, `uint`, `long`, or `ulong`, rewrite `e` to `(U)e` and optionally rewrite `i` to `(U)i` if an implicit conversion from `I` to `U` is defined. Otherwise, if `U` is `byte`, `sbyte`, `short`, or `ushort`, rewrite `e` to `(int)e` and rewrite `i` to `(int)(U)i` if an implicit conversion from `I` to `U` is defined, or rewrite `i` to `(int)i` otherwise. Finally, rewrite the original expression to `(E)(e - i)`, given the rewritten expressions `e` and `i`.
   >
   > * If `E` is a nullable type `E0?` where `E0` is an enum type, let `U` be the underlying type of `E0`. If `U` is `int`, `uint`, `long`, or `ulong`, rewrite `e` to `(U?)e` and optionally rewrite `i` to `(U?)i` if an implicit conversion from `I` to `U` or from `I?` to `U?` is defined. Otherwise, if `U` is `byte`, `sbyte`, `short`, or `ushort`, rewrite `e` to `(int?)e` and rewrite `i` to `(int?)(U)i` or `(int?)(U?)i` if an implicit conversion from `I` to `U` or from `I?` to `U?` is defined, or rewrite `i` to `(int?)i` otherwise. Finally, rewrite the original expression to `(E)(e - i)`, given the rewritten expressions `e` and `i`.
   >
   > Expression tree conversion then proceeds by translating the resulting expression. All conversions introduced in the preceding steps are classified as compiler-generated.
   >
   > Also note that all conversion and subtraction operations in the resulting expression are subject to checked arithmetic if they occur in a checked context. For example, in query expression trees this will result in the use of `SubtractChecked` and `ConvertChecked`; in generalized expression trees this will result in the use of the `Q.Flags.CheckedContext` flags.
   >
   > ***TODO***
   > * See remarks on enum addition.

*  Delegate removal. Every delegate type implicitly provides the following predefined operator, where `D` is the delegate type:

   ```csharp
   D operator -(D x, D y);
   ```

   The binary `-` operator performs delegate removal when both operands are of some delegate type `D`. If the operands have different delegate types, a binding-time error occurs. If the first operand is `null`, the result of the operation is `null`. Otherwise, if the second operand is `null`, then the result of the operation is the value of the first operand. Otherwise, both operands represent invocation lists ([Delegate declarations](delegates.md#delegate-declarations)) having one or more entries, and the result is a new invocation list consisting of the first operand's list with the second operand's entries removed from it, provided the second operand's list is a proper contiguous sublist of the first's.     (To determine sublist equality, corresponding entries are compared as for the delegate equality operator ([Delegate equality operators](expressions.md#delegate-equality-operators)).) Otherwise, the result is the value of the left operand. Neither of the operands' lists is changed in the process. If the second operand's list matches multiple sublists of contiguous entries in the first operand's list, the right-most matching sublist of contiguous entries is removed. If removal results in an empty list, the result is `null`. For example:

   ```csharp
   delegate void D(int x);
   
   class C
   {
       public static void M1(int i) { /* ... */ }
       public static void M2(int i) { /* ... */ }
   }

   class Test
   {
       static void Main() { 
           D cd1 = new D(C.M1);
           D cd2 = new D(C.M2);
           D cd3 = cd1 + cd2 + cd2 + cd1;   // M1 + M2 + M2 + M1
           cd3 -= cd1;                      // => M1 + M2 + M2
   
           cd3 = cd1 + cd2 + cd2 + cd1;     // M1 + M2 + M2 + M1
           cd3 -= cd1 + cd2;                // => M2 + M1
   
           cd3 = cd1 + cd2 + cd2 + cd1;     // M1 + M2 + M2 + M1
           cd3 -= cd2 + cd2;                // => M1 + M1
   
           cd3 = cd1 + cd2 + cd2 + cd1;     // M1 + M2 + M2 + M1
           cd3 -= cd2 + cd1;                // => M1 + M2
   
           cd3 = cd1 + cd2 + cd2 + cd1;     // M1 + M2 + M2 + M1
           cd3 -= cd1 + cd1;                // => M1 + M2 + M2 + M1
       }
   }
   ```

   > __Expression Tree Conversion Translation Steps__
   >
   > Expression tree conversion for delegate removal defines `method` as the `Remove(Delegate, Delegate)` method on `System.Delegate` and introduces a cast expression to `D` on the result, which is classified as compiler-generated. For instance, the expression
   >
   > ```csharp
   > f1 - f2
   > ```
   >
   > where `f1` and `f2` are expressions of delegate type `Func<int>` is translated to
   >
   > ```csharp
   > Expression.Convert(
   >     Expression.Subtract(f1Expr, f2Expr, method),
   >     typeof(Func<int>)
   > )
   > ```
   >
   > for query expression trees, and to
   >
   > ```csharp
   > Q.Convert(
   >     Q.ConvertInfo(Q.Flags.CompilerGenerated, typeof(Func<int>))
   >     Q.Subtract(
   >         Q.SubtractInfo(default(Q.Flags), method),
   >         f1Expr, f2Expr
   >     )
   > )
   > ```
   >
   > for generalized expression trees, where `f1Expr` and `f2Expr` are the result of translating `f1` and `f2` to expression trees. Note that these conversions do not introduce a cast from `Func<int>` to `Delegate`; the only introduced conversion occurs on the result.

## Shift operators

The `<<` and `>>` operators are used to perform bit shifting operations.

```antlr
shift_expression
    : additive_expression
    | shift_expression '<<' additive_expression
    | shift_expression right_shift additive_expression
    ;
```

If an operand of a *shift_expression* has the compile-time type `dynamic`, then the expression is dynamically bound ([Dynamic binding](expressions.md#dynamic-binding)). In this case the compile-time type of the expression is `dynamic`, and the resolution described below will take place at run-time using the run-time type of those operands that have the compile-time type `dynamic`.

For an operation of the form `x << count` or `x >> count`, binary operator overload resolution ([Binary operator overload resolution](expressions.md#binary-operator-overload-resolution)) is applied to select a specific operator implementation. The operands are converted to the parameter types of the selected operator, and the type of the result is the return type of the operator.

When declaring an overloaded shift operator, the type of the first operand must always be the class or struct containing the operator declaration, and the type of the second operand must always be `int`.

The predefined shift operators are listed below.

*  Shift left:

   ```csharp
   int operator <<(int x, int count);
   uint operator <<(uint x, int count);
   long operator <<(long x, int count);
   ulong operator <<(ulong x, int count);
   ```

   The `<<` operator shifts `x` left by a number of bits computed as described below.

   The high-order bits outside the range of the result type of `x` are discarded, the remaining bits are shifted left, and the low-order empty bit positions are set to zero.

*  Shift right:

   ```csharp
   int operator >>(int x, int count);
   uint operator >>(uint x, int count);
   long operator >>(long x, int count);
   ulong operator >>(ulong x, int count);
   ```

   The `>>` operator shifts `x` right by a number of bits computed as described below.

   When `x` is of type `int` or `long`, the low-order bits of `x` are discarded, the remaining bits are shifted right, and the high-order empty bit positions are set to zero if `x` is non-negative and set to one if `x` is negative.

   When `x` is of type `uint` or `ulong`, the low-order bits of `x` are discarded, the remaining bits are shifted right, and the high-order empty bit positions are set to zero.

For the predefined operators, the number of bits to shift is computed as follows:

*  When the type of `x` is `int` or `uint`, the shift count is given by the low-order five bits of `count`. In other words, the shift count is computed from `count & 0x1F`.
*  When the type of `x` is `long` or `ulong`, the shift count is given by the low-order six bits of `count`. In other words, the shift count is computed from `count & 0x3F`.

If the resulting shift count is zero, the shift operators simply return the value of `x`.

Shift operations never cause overflows and produce the same results in `checked` and `unchecked` contexts.

When the left operand of the `>>` operator is of a signed integral type, the operator performs an arithmetic shift right wherein the value of the most significant bit (the sign bit) of the operand is propagated to the high-order empty bit positions. When the left operand of the `>>` operator is of an unsigned integral type, the operator performs a logical shift right wherein high-order empty bit positions are always set to zero. To perform the opposite operation of that inferred from the operand type, explicit casts can be used. For example, if `x` is a variable of type `int`, the operation `unchecked((int)((uint)x >> y))` performs a logical shift right of `x`.

> __Expression Tree Conversion Translation Steps__
>
> An expression with shift operator `op`
>
> ```csharp
> left op right
> ```
>
> is translated into an expression tree by translating `left` to `leftExpr`, by translating `right` to `rightExpr`, and by optionally constructing an expression `method` of type `MethodInfo` representing the method implementing the operator if the operator is bound to a user-defined operator.
>
> Expression tree translation then proceeds as follows, where `M` is `LeftShift` if the operator is `<<`, and `RightShift` if the operator is `>>`.
>
> When subject to conversion to a query expression tree, if the operation is dynamically bound, an error is produced. Otherwise, it is translated into
>
> ```csharp
> Expression.M(leftExpr, rightExpr, method)
> ```
>
> if the operator is bound to a user-defined operator implementation, or
>
> ```csharp
> Expression.M(leftExpr, rightExpr)
> ```
>
> otherwise.
>
> When subject to conversion to a generalized expression tree it is translated into
>
> ```csharp
> Q.M(info, leftExpr, rightExpr)
> ```
>
> where `info` is
>
> ```csharp
> Q.MInfo(binder)
> ```
>
> if the operation is dynamically bound, where `binder` is an expression of type `Microsoft.CSharp.RuntimeBinder.CallSiteBinder` representing the dynamic operation, and otherwise into
>
> ```csharp
> Q.MInfo(flags, method)
> ```
>
> if the operator is bound to a user-defined operator implementation, or
>
> ```csharp
> Q.MInfo(flags)
> ```
>
> otherwise, where `flags` is any of the following:
>
> * `Q.Flags.Lifted` if the operator is [lifted](#lifted-operators), or,
> * `default(Q.Flags)` otherwise.
>
> Note that conversion to generalized expression trees use a `Lifted` flag to convey this information to the expression tree library. This reduces the burden on the expression tree library to infer this information using reflection.
>
> Also note that expression tree conversion does not include a `&` bitmask on the right operand with values `0x1F` or `0x3F`. Expression tree libraries can choose to apply this mask if desired to do so, for instance for purposes of evaluating the expression.
>
> ***TODO***:
> * Evaluate options for the dynamic case:
>   * create a specification for the construction of `binder` (can also be used to specify `dynamic` behavior outside the context of expression trees) and consider making more properties on the `Microsoft.CSharp.RuntimeBinder` library types publicly accessable so expression tree libraries can inspect this info, or,
>   * steer away from reusing `Microsoft.CSharp` and introduce a `MInfo` factory overload for dynamic binding (e.g. a `(Q.DynamicFlags flags, Type context, A leftArgumentInfo, A rightArgumentInfo)` overload, where `A` is built through factory invocations as well).

## Relational and type-testing operators

The `==`, `!=`, `<`, `>`, `<=`, `>=`, `is` and `as` operators are called the relational and type-testing operators.

```antlr
relational_expression
    : shift_expression
    | relational_expression '<' shift_expression
    | relational_expression '>' shift_expression
    | relational_expression '<=' shift_expression
    | relational_expression '>=' shift_expression
    | relational_expression 'is' type
    | relational_expression 'as' type
    ;

equality_expression
    : relational_expression
    | equality_expression '==' relational_expression
    | equality_expression '!=' relational_expression
    ;
```

The `is` operator is described in [The is operator](expressions.md#the-is-operator) and the `as` operator is described in [The as operator](expressions.md#the-as-operator).

The `==`, `!=`, `<`, `>`, `<=` and `>=` operators are ***comparison operators***.

If an operand of a comparison operator has the compile-time type `dynamic`, then the expression is dynamically bound ([Dynamic binding](expressions.md#dynamic-binding)). In this case the compile-time type of the expression is `dynamic`, and the resolution described below will take place at run-time using the run-time type of those operands that have the compile-time type `dynamic`.

For an operation of the form `x` *op* `y`, where *op* is a comparison operator, overload resolution ([Binary operator overload resolution](expressions.md#binary-operator-overload-resolution)) is applied to select a specific operator implementation. The operands are converted to the parameter types of the selected operator, and the type of the result is the return type of the operator.

The predefined comparison operators are described in the following sections. All predefined comparison operators return a result of type `bool`, as described in the following table.


| __Operation__ | __Result__                                                       |
|---------------|------------------------------------------------------------------|
| `x == y`      | `true` if `x` is equal to `y`, `false` otherwise                 | 
| `x != y`      | `true` if `x` is not equal to `y`, `false` otherwise             | 
| `x < y`       | `true` if `x` is less than `y`, `false` otherwise                | 
| `x > y`       | `true` if `x` is greater than `y`, `false` otherwise             | 
| `x <= y`      | `true` if `x` is less than or equal to `y`, `false` otherwise    | 
| `x >= y`      | `true` if `x` is greater than or equal to `y`, `false` otherwise | 

> __Expression Tree Conversion Translation Steps__
>
> An expression with comparison operator `op`
>
> ```csharp
> left op right
> ```
>
> is translated into an expression tree by translating `left` to `leftExpr`, by translating `right` to `rightExpr`, and by optionally constructing an expression `method` of type `MethodInfo` representing the method implementing the operator if the operator is bound to a user-defined operator.
>
> Expression tree translation then proceeds as follows, where `M` is `Equal` if the operator is `==`, `NotEqual` if the operator is `!=`, `LessThan` if the operator is `<`, `LessThanOrEqual` if the operator is `<=`, `GreaterThan` if the operator is `>`, and `GreaterThanOrEqual` if the operator is `>=`.
>
> When subject to conversion to a query expression tree, if the operation is dynamically bound, an error is produced. Otherwise, it is translated into
>
> ```csharp
> Expression.M(leftExpr, rightExpr, liftToNull, method)
> ```
>
> if the operator is bound to a user-defined operator implementation, where `liftToNull` is an expression of type `bool` with value `true` if the operation is [lifted](#lifted-operators) and the return type of the user-defined operator is not a nullable type, and with value `false` otherwise, or,
>
> ```csharp
> Expression.M(leftExpr, rightExpr)
> ```
>
> otherwise.
>
> When subject to conversion to a generalized expression tree it is translated into
>
> ```csharp
> Q.M(info, leftExpr, rightExpr)
> ```
>
> where `info` is
>
> ```csharp
> Q.MInfo(binder)
> ```
>
> if the operation is dynamically bound, where `binder` is an expression of type `Microsoft.CSharp.RuntimeBinder.CallSiteBinder` representing the dynamic operation, and otherwise into
>
> ```csharp
> Q.MInfo(flags, method)
> ```
>
> if the operator is bound to a user-defined operator implementation, or
>
> ```csharp
> Q.MInfo(flags)
> ```
>
> otherwise, where `flags` is the bitwise `|` combination of any of the following:
>
> * `Q.Flags.Lifted` if the operator is [lifted](#lifted-operators), and,
> * `Q.Flags.LiftedToNull` if the operator is [lifted](#lifted-operators) and the return type of the user-defined operator is not a nullable type, or,
> * `default(Q.Flags)` if no flags apply.
>
> Note that conversion to generalized expression trees use `Lifted` and `LiftedToNull` flags to convey this information to the expression tree library. This reduces the burden on the expression tree library to infer this information using reflection.
>
> ***TODO***:
> * Evaluate options for the dynamic case:
>   * create a specification for the construction of `binder` (can also be used to specify `dynamic` behavior outside the context of expression trees) and consider making more properties on the `Microsoft.CSharp.RuntimeBinder` library types publicly accessable so expression tree libraries can inspect this info, or,
>   * steer away from reusing `Microsoft.CSharp` and introduce a `MInfo` factory overload for dynamic binding (e.g. a `(Q.DynamicFlags flags, Type context, A leftArgumentInfo, A rightArgumentInfo)` overload, where `A` is built through factory invocations as well).

### Integer comparison operators

The predefined integer comparison operators are:
```csharp
bool operator ==(int x, int y);
bool operator ==(uint x, uint y);
bool operator ==(long x, long y);
bool operator ==(ulong x, ulong y);

bool operator !=(int x, int y);
bool operator !=(uint x, uint y);
bool operator !=(long x, long y);
bool operator !=(ulong x, ulong y);

bool operator <(int x, int y);
bool operator <(uint x, uint y);
bool operator <(long x, long y);
bool operator <(ulong x, ulong y);

bool operator >(int x, int y);
bool operator >(uint x, uint y);
bool operator >(long x, long y);
bool operator >(ulong x, ulong y);

bool operator <=(int x, int y);
bool operator <=(uint x, uint y);
bool operator <=(long x, long y);
bool operator <=(ulong x, ulong y);

bool operator >=(int x, int y);
bool operator >=(uint x, uint y);
bool operator >=(long x, long y);
bool operator >=(ulong x, ulong y);
```

Each of these operators compares the numeric values of the two integer operands and returns a `bool` value that indicates whether the particular relation is `true` or `false`.

### Floating-point comparison operators

The predefined floating-point comparison operators are:
```csharp
bool operator ==(float x, float y);
bool operator ==(double x, double y);

bool operator !=(float x, float y);
bool operator !=(double x, double y);

bool operator <(float x, float y);
bool operator <(double x, double y);

bool operator >(float x, float y);
bool operator >(double x, double y);

bool operator <=(float x, float y);
bool operator <=(double x, double y);

bool operator >=(float x, float y);
bool operator >=(double x, double y);
```

The operators compare the operands according to the rules of the IEEE 754 standard:

*  If either operand is NaN, the result is `false` for all operators except `!=`, for which the result is `true`. For any two operands, `x != y` always produces the same result as `!(x == y)`. However, when one or both operands are NaN, the `<`, `>`, `<=`, and `>=` operators do not produce the same results as the logical negation of the opposite operator. For example, if either of `x` and `y` is NaN, then `x < y` is `false`, but `!(x >= y)` is `true`.
*  When neither operand is NaN, the operators compare the values of the two floating-point operands with respect to the ordering

   ```
   -inf < -max < ... < -min < -0.0 == +0.0 < +min < ... < +max < +inf
   ```

   where `min` and `max` are the smallest and largest positive finite values that can be represented in the given floating-point format. Notable effects of this ordering are:
   * Negative and positive zeros are considered equal.
   * A negative infinity is considered less than all other values, but equal to another negative infinity.
   * A positive infinity is considered greater than all other values, but equal to another positive infinity.

### Decimal comparison operators

The predefined decimal comparison operators are:
```csharp
bool operator ==(decimal x, decimal y);
bool operator !=(decimal x, decimal y);
bool operator <(decimal x, decimal y);
bool operator >(decimal x, decimal y);
bool operator <=(decimal x, decimal y);
bool operator >=(decimal x, decimal y);
```

Each of these operators compares the numeric values of the two decimal operands and returns a `bool` value that indicates whether the particular relation is `true` or `false`. Each decimal comparison is equivalent to using the corresponding relational or equality operator of type `System.Decimal`.

> __Expression Tree Conversion Translation Steps__
>
> For purposes of expression tree conversion of decimal comparison, `method` is defined as the static method defined on `System.Decimal` implementing the operator.

### Boolean equality operators

The predefined boolean equality operators are:
```csharp
bool operator ==(bool x, bool y);
bool operator !=(bool x, bool y);
```

The result of `==` is `true` if both `x` and `y` are `true` or if both `x` and `y` are `false`. Otherwise, the result is `false`.

The result of `!=` is `false` if both `x` and `y` are `true` or if both `x` and `y` are `false`. Otherwise, the result is `true`. When the operands are of type `bool`, the `!=` operator produces the same result as the `^` operator.

### Enumeration comparison operators

Every enumeration type implicitly provides the following predefined comparison operators:
```csharp
bool operator ==(E x, E y);
bool operator !=(E x, E y);
bool operator <(E x, E y);
bool operator >(E x, E y);
bool operator <=(E x, E y);
bool operator >=(E x, E y);
```

The result of evaluating `x op y`, where `x` and `y` are expressions of an enumeration type `E` with an underlying type `U`, and `op` is one of the comparison operators, is exactly the same as evaluating `((U)x) op ((U)y)`. In other words, the enumeration type comparison operators simply compare the underlying integral values of the two operands.

> __Expression Tree Conversion Translation Steps__
>
> Expression tree conversion for enum comparison uses an expansion similar to the one shown above, but with some differences to support nullable lifting and promotion of numeric underlying types.
>
> For an expression of the form `e1 op e2`, where `e1` and `e2` have type `E` which is an enum type or a nullable enum type, rewrite the expression as follows:
>
> * If `E` is a non-nullable enum type, let `U` be the underlying type of `E`. If `U` is `int`, `uint`, `long`, or `ulong`, rewrite the expression to `(U)e1 op (U)e2`. Otherwise, if `U` is `byte`, `sbyte`, `short`, or `ushort`, rewrite the expression to `(int)e1 op (int)e2`.
>
> * If `E` is a nullable type `E0?` where `E0` is an enum type, let `U` be the underlying type of `E0`. If `U` is `int`, `uint`, `long`, or `ulong`, rewrite the expression to `(U?)e1 op (U?)e2`. Otherwise, if `U` is `byte`, `sbyte`, `short`, or `ushort`, rewrite the expression to `(int?)e1 op (int?)e2`.
>
> Expression tree conversion then proceeds by translating the resulting expression. All conversions introduced in the preceding steps are classified as compiler-generated.
>
> Also note that all conversion operations in the resulting expression are subject to checked arithmetic if they occur in a checked context. For example, in query expression trees this will result in the use of `ConvertChecked`; in generalized expression trees this will result in the use of the `Q.Flags.CheckedContext` flags.
>
> ***TODO***
> * See remarks on enum addition.

### Reference type equality operators

The predefined reference type equality operators are:
```csharp
bool operator ==(object x, object y);
bool operator !=(object x, object y);
```

The operators return the result of comparing the two references for equality or non-equality.

Since the predefined reference type equality operators accept operands of type `object`, they apply to all types that do not declare applicable `operator ==` and `operator !=` members. Conversely, any applicable user-defined equality operators effectively hide the predefined reference type equality operators.

The predefined reference type equality operators require one of the following:

*  Both operands are a value of a type known to be a *reference_type* or the literal `null`. Furthermore, an explicit reference conversion ([Explicit reference conversions](conversions.md#explicit-reference-conversions)) exists from the type of either operand to the type of the other operand.
*  One operand is a value of type `T` where `T` is a *type_parameter* and the other operand is the literal `null`. Furthermore `T` does not have the value type constraint.

Unless one of these conditions are true, a binding-time error occurs. Notable implications of these rules are:

*  It is a binding-time error to use the predefined reference type equality operators to compare two references that are known to be different at binding-time. For example, if the binding-time types of the operands are two class types `A` and `B`, and if neither `A` nor `B` derives from the other, then it would be impossible for the two operands to reference the same object. Thus, the operation is considered a binding-time error.
*  The predefined reference type equality operators do not permit value type operands to be compared. Therefore, unless a struct type declares its own equality operators, it is not possible to compare values of that struct type.
*  The predefined reference type equality operators never cause boxing operations to occur for their operands. It would be meaningless to perform such boxing operations, since references to the newly allocated boxed instances would necessarily differ from all other references.
*  If an operand of a type parameter type `T` is compared to `null`, and the run-time type of `T` is a value type, the result of the comparison is `false`.

The following example checks whether an argument of an unconstrained type parameter type is `null`.
```csharp
class C<T>
{
    void F(T x) {
        if (x == null) throw new ArgumentNullException();
        ...
    }
}
```

The `x == null` construct is permitted even though `T` could represent a value type, and the result is simply defined to be `false` when `T` is a value type.

For an operation of the form `x == y` or `x != y`, if any applicable `operator ==` or `operator !=` exists, the operator overload resolution ([Binary operator overload resolution](expressions.md#binary-operator-overload-resolution)) rules will select that operator instead of the predefined reference type equality operator. However, it is always possible to select the predefined reference type equality operator by explicitly casting one or both of the operands to type `object`. The example
```csharp
using System;

class Test
{
    static void Main() {
        string s = "Test";
        string t = string.Copy(s);
        Console.WriteLine(s == t);
        Console.WriteLine((object)s == t);
        Console.WriteLine(s == (object)t);
        Console.WriteLine((object)s == (object)t);
    }
}
```
produces the output
```
True
False
False
False
```

The `s` and `t` variables refer to two distinct `string` instances containing the same characters. The first comparison outputs `True` because the predefined string equality operator ([String equality operators](expressions.md#string-equality-operators)) is selected when both operands are of type `string`. The remaining comparisons all output `False` because the predefined reference type equality operator is selected when one or both of the operands are of type `object`.

Note that the above technique is not meaningful for value types. The example
```csharp
class Test
{
    static void Main() {
        int i = 123;
        int j = 123;
        System.Console.WriteLine((object)i == (object)j);
    }
}
```
outputs `False` because the casts create references to two separate instances of boxed `int` values.

> __Expression Tree Conversion Translation Steps__
>
> For purposes of expression tree conversion of object reference equality operators, the `Equal` and `NotEqual` factory methods are used, rather than `ReferenceEqual` and `ReferenceNotEqual` methods which may be defined (as is the case for query expression trees).
>
> ***TODO***
> * Evaluate whether generalized expression trees should make reference equality explicit operators explicit, either through the use of dedicated factory method names, or by using a flag.

### String equality operators

The predefined string equality operators are:
```csharp
bool operator ==(string x, string y);
bool operator !=(string x, string y);
```

Two `string` values are considered equal when one of the following is true:

*  Both values are `null`.
*  Both values are non-null references to string instances that have identical lengths and identical characters in each character position.

The string equality operators compare string values rather than string references. When two separate string instances contain the exact same sequence of characters, the values of the strings are equal, but the references are different. As described in [Reference type equality operators](expressions.md#reference-type-equality-operators), the reference type equality operators can be used to compare string references instead of string values.

> __Expression Tree Conversion Translation Steps__
>
> For purposes of expression tree conversion of string equality, `method` is defined as the static method defined on `System.String` implementing the operator.

### Delegate equality operators

Every delegate type implicitly provides the following predefined comparison operators:

```csharp
bool operator ==(System.Delegate x, System.Delegate y);
bool operator !=(System.Delegate x, System.Delegate y);
```

Two delegate instances are considered equal as follows:

*  If either of the delegate instances is `null`, they are equal if and only if both are `null`.
*  If the delegates have different run-time type they are never equal.
*  If both of the delegate instances have an invocation list ([Delegate declarations](delegates.md#delegate-declarations)), those instances are equal if and only if their invocation lists are the same length, and each entry in one's invocation list is equal (as defined below) to the corresponding entry, in order, in the other's invocation list.

The following rules govern the equality of invocation list entries:

*  If two invocation list entries both refer to the same static method then the entries are equal.
*  If two invocation list entries both refer to the same non-static method on the same target object (as defined by the reference equality operators) then the entries are equal.
*  Invocation list entries produced from evaluation of semantically identical *anonymous_method_expression*s or *lambda_expression*s with the same (possibly empty) set of captured outer variable instances are permitted (but not required) to be equal.

> __Expression Tree Conversion Translation Steps__
>
> For purposes of expression tree conversion of delegate equality, `method` is defined as the static method defined on `System.Delegate` implementing the operator. Note that no conversions are introduced.

### Equality operators and null

The `==` and `!=` operators permit one operand to be a value of a nullable type and the other to be the `null` literal, even if no predefined or user-defined operator (in unlifted or lifted form) exists for the operation.

For an operation of one of the forms
```csharp
x == null
null == x
x != null
null != x
```
where `x` is an expression of a nullable type, if operator overload resolution ([Binary operator overload resolution](expressions.md#binary-operator-overload-resolution)) fails to find an applicable operator, the result is instead computed from the `HasValue` property of `x`. Specifically, the first two forms are translated into `!x.HasValue`, and last two forms are translated into `x.HasValue`.

> __Expression Tree Conversion Translation Steps__
>
> For purposes of expression tree conversion of equality operators on nullable types, an `Equal` or `NotEqual` factory is used. That is, no conversion to `HasValue` member access is performed.

### The is operator

The `is` operator is used to dynamically check if the run-time type of an object is compatible with a given type. The result of the operation `E is T`, where `E` is an expression and `T` is a type, is a boolean value indicating whether `E` can successfully be converted to type `T` by a reference conversion, a boxing conversion, or an unboxing conversion. The operation is evaluated as follows, after type arguments have been substituted for all type parameters:

*  If `E` is an anonymous function, a compile-time error occurs
*  If `E` is a method group or the `null` literal, of if the type of `E` is a reference type or a nullable type and the value of `E` is null, the result is false.
*  Otherwise, let `D` represent the dynamic type of `E` as follows:
   * If the type of `E` is a reference type, `D` is the run-time type of the instance reference by `E`.
   * If the type of `E` is a nullable type, `D` is the underlying type of that nullable type.
   * If the type of `E` is a non-nullable value type, `D` is the type of `E`.
*  The result of the operation depends on `D` and `T` as follows:
   * If `T` is a reference type, the result is true if `D` and `T` are the same type, if `D` is a reference type and an implicit reference conversion from `D` to `T` exists, or if `D` is a value type and a boxing conversion from `D` to `T` exists.
   * If `T` is a nullable type, the result is true if `D` is the underlying type of `T`.
   * If `T` is a non-nullable value type, the result is true if `D` and `T` are the same type.
   * Otherwise, the result is false.

Note that user defined conversions, are not considered by the `is` operator.

> __Expression Tree Conversion Translation Steps__
>
> An `is` expression
>
> ```csharp
> expression is T
> ```
>
> is translated into an expression tree by first translating `expression` to `expr`, and by constructing an expression `type` of type `Type` from `typeof(T)` if `T` is not `dynamic` or `typeof(object)` if `T` is `dynamic`.
>
> Expression tree translation then proceeds as follows.
>
> When subject to conversion to a query expression tree, it is translated into
>
> ```csharp
> Expression.TypeIs(expr, type)
> ```
>
> Note that `expression` is allowed to be of type `dynamic`, causing the translated expression to be equivalent to `expression is object`.
>
> When subject to conversion to a generalized expression tree, it is translated into
>
> ```csharp
> Q.TypeIs(info, expr)
> ```
>
> where `info` is
>
> ```csharp
> Q.TypeIsInfo(default(Q.Flags), type)
> ```
>
> where the first flags parameter is reserved for future use.

### The as operator

The `as` operator is used to explicitly convert a value to a given reference type or nullable type. Unlike a cast expression ([Cast expressions](expressions.md#cast-expressions)), the `as` operator never throws an exception. Instead, if the indicated conversion is not possible, the resulting value is `null`.

In an operation of the form `E as T`, `E` must be an expression and `T` must be a reference type, a type parameter known to be a reference type, or a nullable type. Furthermore, at least one of the following must be true, or otherwise a compile-time error occurs:

*  An identity ([Identity conversion](conversions.md#identity-conversion)), implicit nullable ([Implicit nullable conversions](conversions.md#implicit-nullable-conversions)), implicit reference ([Implicit reference conversions](conversions.md#implicit-reference-conversions)), boxing ([Boxing conversions](conversions.md#boxing-conversions)), explicit nullable ([Explicit nullable conversions](conversions.md#explicit-nullable-conversions)), explicit reference ([Explicit reference conversions](conversions.md#explicit-reference-conversions)), or unboxing ([Unboxing conversions](conversions.md#unboxing-conversions)) conversion exists from `E` to `T`.
*  The type of `E` or `T` is an open type.
*  `E` is the `null` literal.

If the compile-time type of `E` is not `dynamic`, the operation `E as T` produces the same result as
```csharp
E is T ? (T)(E) : (T)null
```
except that `E` is only evaluated once. The compiler can be expected to optimize `E as T` to perform at most one dynamic type check as opposed to the two dynamic type checks implied by the expansion above.

If the compile-time type of `E` is `dynamic`, unlike the cast operator the `as` operator is not dynamically bound ([Dynamic binding](expressions.md#dynamic-binding)). Therefore the expansion in this case is:
```csharp
E is T ? (T)(object)(E) : (T)null
```

Note that some conversions, such as user defined conversions, are not possible with the `as` operator and should instead be performed using cast expressions.

In the example
```csharp
class X
{

    public string F(object o) {
        return o as string;        // OK, string is a reference type
    }

    public T G<T>(object o) where T: Attribute {
        return o as T;             // Ok, T has a class constraint
    }

    public U H<U>(object o) {
        return o as U;             // Error, U is unconstrained 
    }
}
```
the type parameter `T` of `G` is known to be a reference type, because it has the class constraint. The type parameter `U` of `H` is not however; hence the use of the `as` operator in `H` is disallowed.

> __Expression Tree Conversion Translation Steps__
>
> An `as` expression
>
> ```csharp
> expression as T
> ```
>
> is translated into an expression tree by first translating `expression` to `expr`, and by constructing an expression `type` of type `Type` from `typeof(T)` if `T` is not `dynamic` or `typeof(object)` if `T` is `dynamic`.
>
> Expression tree translation then proceeds as follows.
>
> When subject to conversion to a query expression tree, and `T` is `dynamic`, an error is produced. Otherwise, it is translated into
>
> ```csharp
> Expression.TypeAs(expr, type)
> ```
>
> When subject to conversion to a generalized expression tree, it is translated into
>
> ```csharp
> Q.TypeAs(info, expr)
> ```
>
> where `info` is
>
> ```csharp
> Q.TypeAsInfo(default(Q.Flags), type)
> ```
>
> where the first flags parameter is reserved for future use.

## Logical operators

The `&`, `^`, and `|` operators are called the logical operators.

```antlr
and_expression
    : equality_expression
    | and_expression '&' equality_expression
    ;

exclusive_or_expression
    : and_expression
    | exclusive_or_expression '^' and_expression
    ;

inclusive_or_expression
    : exclusive_or_expression
    | inclusive_or_expression '|' exclusive_or_expression
    ;
```

If an operand of a logical operator has the compile-time type `dynamic`, then the expression is dynamically bound ([Dynamic binding](expressions.md#dynamic-binding)). In this case the compile-time type of the expression is `dynamic`, and the resolution described below will take place at run-time using the run-time type of those operands that have the compile-time type `dynamic`.

For an operation of the form `x op y`, where `op` is one of the logical operators, overload resolution ([Binary operator overload resolution](expressions.md#binary-operator-overload-resolution)) is applied to select a specific operator implementation. The operands are converted to the parameter types of the selected operator, and the type of the result is the return type of the operator.

> __Expression Tree Conversion Translation Steps__
>
> An expression with logical operator `op`
>
> ```csharp
> left op right
> ```
>
> is translated into an expression tree by translating `left` to `leftExpr`, by translating `right` to `rightExpr`, and by optionally constructing an expression `method` of type `MethodInfo` representing the method implementing the operator if the operator is bound to a user-defined operator.
>
> Expression tree translation then proceeds as follows, where `M` is `And` if the operator is `+`, `Or` if the operator is `|`, and `ExclusiveOr` if the operator is `^`.
>
> When subject to conversion to a query expression tree, if the operation is dynamically bound, an error is produced. Otherwise, it is translated into
>
> ```csharp
> Expression.M(leftExpr, rightExpr, method)
> ```
>
> if the operator is bound to a user-defined operator implementation, or
>
> ```csharp
> Expression.M(leftExpr, rightExpr)
> ```
>
> otherwise.
>
> When subject to conversion to a generalized expression tree it is translated into
>
> ```csharp
> Q.M(info, leftExpr, rightExpr)
> ```
>
> where `info` is
>
> ```csharp
> Q.MInfo(binder)
> ```
>
> if the operation is dynamically bound, where `binder` is an expression of type `Microsoft.CSharp.RuntimeBinder.CallSiteBinder` representing the dynamic operation, and otherwise into
>
> ```csharp
> Q.MInfo(flags, method)
> ```
>
> if the operator is bound to a user-defined operator implementation, or
>
> ```csharp
> Q.MInfo(flags)
> ```
>
> otherwise, where `flags` is any of the following:
>
> * `Q.Flags.Lifted` if the operator is [lifted](#lifted-operators), or,
> * `default(Q.Flags)` otherwise.
>
> Note that conversion to generalized expression trees use a `Lifted` flag to convey this information to the expression tree library. This reduces the burden on the expression tree library to infer this information using reflection.
>
> ***TODO***:
> * Evaluate options for the dynamic case:
>   * create a specification for the construction of `binder` (can also be used to specify `dynamic` behavior outside the context of expression trees) and consider making more properties on the `Microsoft.CSharp.RuntimeBinder` library types publicly accessable so expression tree libraries can inspect this info, or,
>   * steer away from reusing `Microsoft.CSharp` and introduce a `MInfo` factory overload for dynamic binding (e.g. a `(Q.DynamicFlags flags, Type context, A leftArgumentInfo, A rightArgumentInfo)` overload, where `A` is built through factory invocations as well).

The predefined logical operators are described in the following sections.

### Integer logical operators

The predefined integer logical operators are:
```csharp
int operator &(int x, int y);
uint operator &(uint x, uint y);
long operator &(long x, long y);
ulong operator &(ulong x, ulong y);

int operator |(int x, int y);
uint operator |(uint x, uint y);
long operator |(long x, long y);
ulong operator |(ulong x, ulong y);

int operator ^(int x, int y);
uint operator ^(uint x, uint y);
long operator ^(long x, long y);
ulong operator ^(ulong x, ulong y);
```

The `&` operator computes the bitwise logical `AND` of the two operands, the `|` operator computes the bitwise logical `OR` of the two operands, and the `^` operator computes the bitwise logical exclusive `OR` of the two operands. No overflows are possible from these operations.

### Enumeration logical operators

Every enumeration type `E` implicitly provides the following predefined logical operators:

```csharp
E operator &(E x, E y);
E operator |(E x, E y);
E operator ^(E x, E y);
```

The result of evaluating `x op y`, where `x` and `y` are expressions of an enumeration type `E` with an underlying type `U`, and `op` is one of the logical operators, is exactly the same as evaluating `(E)((U)x op (U)y)`. In other words, the enumeration type logical operators simply perform the logical operation on the underlying type of the two operands.

### Boolean logical operators

The predefined boolean logical operators are:
```csharp
bool operator &(bool x, bool y);
bool operator |(bool x, bool y);
bool operator ^(bool x, bool y);
```

The result of `x & y` is `true` if both `x` and `y` are `true`. Otherwise, the result is `false`.

The result of `x | y` is `true` if either `x` or `y` is `true`. Otherwise, the result is `false`.

The result of `x ^ y` is `true` if `x` is `true` and `y` is `false`, or `x` is `false` and `y` is `true`. Otherwise, the result is `false`. When the operands are of type `bool`, the `^` operator computes the same result as the `!=` operator.

### Nullable boolean logical operators

The nullable boolean type `bool?` can represent three values, `true`, `false`, and `null`, and is conceptually similar to the three-valued type used for boolean expressions in SQL. To ensure that the results produced by the `&` and `|` operators for `bool?` operands are consistent with SQL's three-valued logic, the following predefined operators are provided:

```csharp
bool? operator &(bool? x, bool? y);
bool? operator |(bool? x, bool? y);
```

The following table lists the results produced by these operators for all combinations of the values `true`, `false`, and `null`.

| `x`     | `y`     | `x & y` | `x \| y`|
|:-------:|:-------:|:-------:|:-------:|
| `true`  | `true`  | `true`  | `true`  | 
| `true`  | `false` | `false` | `true`  | 
| `true`  | `null`  | `null`  | `true`  | 
| `false` | `true`  | `false` | `true`  | 
| `false` | `false` | `false` | `false` | 
| `false` | `null`  | `false` | `null`  | 
| `null`  | `true`  | `null`  | `true`  | 
| `null`  | `false` | `false` | `null`  | 
| `null`  | `null`  | `null`  | `null`  | 

## Conditional logical operators

The `&&` and `||` operators are called the conditional logical operators. They are also called the "short-circuiting" logical operators.

```antlr
conditional_and_expression
    : inclusive_or_expression
    | conditional_and_expression '&&' inclusive_or_expression
    ;

conditional_or_expression
    : conditional_and_expression
    | conditional_or_expression '||' conditional_and_expression
    ;
```

The `&&` and `||` operators are conditional versions of the `&` and `|` operators:

*  The operation `x && y` corresponds to the operation `x & y`, except that `y` is evaluated only if `x` is not `false`.
*  The operation `x || y` corresponds to the operation `x | y`, except that `y` is evaluated only if `x` is not `true`.

If an operand of a conditional logical operator has the compile-time type `dynamic`, then the expression is dynamically bound ([Dynamic binding](expressions.md#dynamic-binding)). In this case the compile-time type of the expression is `dynamic`, and the resolution described below will take place at run-time using the run-time type of those operands that have the compile-time type `dynamic`.

An operation of the form `x && y` or `x || y` is processed by applying overload resolution ([Binary operator overload resolution](expressions.md#binary-operator-overload-resolution)) as if the operation was written `x & y` or `x | y`. Then,

*  If overload resolution fails to find a single best operator, or if overload resolution selects one of the predefined integer logical operators, a binding-time error occurs.
*  Otherwise, if the selected operator is one of the predefined boolean logical operators ([Boolean logical operators](expressions.md#boolean-logical-operators)) or nullable boolean logical operators ([Nullable boolean logical operators](expressions.md#nullable-boolean-logical-operators)), the operation is processed as described in [Boolean conditional logical operators](expressions.md#boolean-conditional-logical-operators).
*  Otherwise, the selected operator is a user-defined operator, and the operation is processed as described in [User-defined conditional logical operators](expressions.md#user-defined-conditional-logical-operators).

It is not possible to directly overload the conditional logical operators. However, because the conditional logical operators are evaluated in terms of the regular logical operators, overloads of the regular logical operators are, with certain restrictions, also considered overloads of the conditional logical operators. This is described further in [User-defined conditional logical operators](expressions.md#user-defined-conditional-logical-operators).

> __Expression Tree Conversion Translation Steps__
>
> An expression with conditional logical operator `op`
>
> ```csharp
> left op right
> ```
>
> is translated into an expression tree by translating `left` to `leftExpr`, by translating `right` to `rightExpr`, and by optionally constructing an expression `method` of type `MethodInfo` representing the method implementing the corresponding logical `&` or `|` operator if the operator is bound to a user-defined operator.
>
> Expression tree translation then proceeds as follows, where `M` is `AndAlso` if the operator is `&&`, and `OrElse` if the operator is `||`.
>
> When subject to conversion to a query expression tree, if the operation is dynamically bound, an error is produced. Otherwise, it is translated into
>
> ```csharp
> Expression.M(leftExpr, rightExpr, method)
> ```
>
> if the operator is bound to a user-defined operator implementation, or
>
> ```csharp
> Expression.M(leftExpr, rightExpr)
> ```
>
> otherwise.
>
> When subject to conversion to a generalized expression tree it is translated into
>
> ```csharp
> Q.M(info, leftExpr, rightExpr)
> ```
>
> where `info` is
>
> ```csharp
> Q.MInfo(binder)
> ```
>
> if the operation is dynamically bound, where `binder` is an expression of type `Microsoft.CSharp.RuntimeBinder.CallSiteBinder` representing the dynamic operation, and otherwise into
>
> ```csharp
> Q.MInfo(flags, method, truthMethod)
> ```
>
> if the operator is bound to a user-defined operator implementation, where `truthMethod` is an expression of type `MethodInfo` representing the method implementing the corresponding `true` or `false` operator, or
>
> ```csharp
> Q.MInfo(flags)
> ```
>
> otherwise, where `flags` is any of the following:
>
> * `Q.Flags.Lifted` if the operator is [lifted](#lifted-operators), or,
> * `default(Q.Flags)` otherwise.
>
> Note that conversion to generalized expression trees use a `Lifted` flag to convey this information to the expression tree library. This reduces the burden on the expression tree library to infer this information using reflection.
>
> Also note that conversion to generalized expression trees retains information about the bound `true` or `false` method for user-defined conditional logical operators. This reduces the runtime overhead in expression tree libraries to infer this operator from the given `&` or `|` method.
>
> ***TODO***:
> * Evaluate options for the dynamic case:
>   * create a specification for the construction of `binder` (can also be used to specify `dynamic` behavior outside the context of expression trees) and consider making more properties on the `Microsoft.CSharp.RuntimeBinder` library types publicly accessable so expression tree libraries can inspect this info, or,
>   * steer away from reusing `Microsoft.CSharp` and introduce a `MInfo` factory overload for dynamic binding (e.g. a `(Q.DynamicFlags flags, Type context, A leftArgumentInfo, A rightArgumentInfo)` overload, where `A` is built through factory invocations as well).

### Boolean conditional logical operators

When the operands of `&&` or `||` are of type `bool`, or when the operands are of types that do not define an applicable `operator &` or `operator |`, but do define implicit conversions to `bool`, the operation is processed as follows:

*  The operation `x && y` is evaluated as `x ? y : false`. In other words, `x` is first evaluated and converted to type `bool`. Then, if `x` is `true`, `y` is evaluated and converted to type `bool`, and this becomes the result of the operation. Otherwise, the result of the operation is `false`.
*  The operation `x || y` is evaluated as `x ? true : y`. In other words, `x` is first evaluated and converted to type `bool`. Then, if `x` is `true`, the result of the operation is `true`. Otherwise, `y` is evaluated and converted to type `bool`, and this becomes the result of the operation.

### User-defined conditional logical operators

When the operands of `&&` or `||` are of types that declare an applicable user-defined `operator &` or `operator |`, both of the following must be true, where `T` is the type in which the selected operator is declared:

*  The return type and the type of each parameter of the selected operator must be `T`. In other words, the operator must compute the logical `AND` or the logical `OR` of two operands of type `T`, and must return a result of type `T`.
*  `T` must contain declarations of `operator true` and `operator false`.

A binding-time error occurs if either of these requirements is not satisfied. Otherwise, the `&&` or `||` operation is evaluated by combining the user-defined `operator true` or `operator false` with the selected user-defined operator:

*  The operation `x && y` is evaluated as `T.false(x) ? x : T.&(x, y)`, where `T.false(x)` is an invocation of the `operator false` declared in `T`, and `T.&(x, y)` is an invocation of the selected `operator &`. In other words, `x` is first evaluated and `operator false` is invoked on the result to determine if `x` is definitely false. Then, if `x` is definitely false, the result of the operation is the value previously computed for `x`. Otherwise, `y` is evaluated, and the selected `operator &` is invoked on the value previously computed for `x` and the value computed for `y` to produce the result of the operation.
*  The operation `x || y` is evaluated as `T.true(x) ? x : T.|(x, y)`, where `T.true(x)` is an invocation of the `operator true` declared in `T`, and `T.|(x,y)` is an invocation of the selected `operator|`. In other words, `x` is first evaluated and `operator true` is invoked on the result to determine if `x` is definitely true. Then, if `x` is definitely true, the result of the operation is the value previously computed for `x`. Otherwise, `y` is evaluated, and the selected `operator |` is invoked on the value previously computed for `x` and the value computed for `y` to produce the result of the operation.

In either of these operations, the expression given by `x` is only evaluated once, and the expression given by `y` is either not evaluated or evaluated exactly once.

For an example of a type that implements `operator true` and `operator false`, see [Database boolean type](structs.md#database-boolean-type).

## The null coalescing operator

The `??` operator is called the null coalescing operator.

```antlr
null_coalescing_expression
    : conditional_or_expression
    | conditional_or_expression '??' null_coalescing_expression
    ;
```

A null coalescing expression of the form `a ?? b` requires `a` to be of a nullable type or reference type. If `a` is non-null, the result of `a ?? b` is `a`; otherwise, the result is `b`. The operation evaluates `b` only if `a` is null.

The null coalescing operator is right-associative, meaning that operations are grouped from right to left. For example, an expression of the form `a ?? b ?? c` is evaluated as `a ?? (b ?? c)`. In general terms, an expression of the form `E1 ?? E2 ?? ... ?? En` returns the first of the operands that is non-null, or null if all operands are null.

The type of the expression `a ?? b` depends on which implicit conversions are available on the operands. In order of preference, the type of `a ?? b` is `A0`, `A`, or `B`, where `A` is the type of `a` (provided that `a` has a type), `B` is the type of `b` (provided that `b` has a type), and `A0` is the underlying type of `A` if `A` is a nullable type, or `A` otherwise. Specifically, `a ?? b` is processed as follows:

*  If `A` exists and is not a nullable type or a reference type, a compile-time error occurs.
*  If `b` is a dynamic expression, the result type is `dynamic`. At run-time, `a` is first evaluated. If `a` is not null, `a` is converted to dynamic, and this becomes the result. Otherwise, `b` is evaluated, and this becomes the result.
*  Otherwise, if `A` exists and is a nullable type and an implicit conversion exists from `b` to `A0`, the result type is `A0`. At run-time, `a` is first evaluated. If `a` is not null, `a` is unwrapped to type `A0`, and this becomes the result. Otherwise, `b` is evaluated and converted to type `A0`, and this becomes the result.
*  Otherwise, if `A` exists and an implicit conversion exists from `b` to `A`, the result type is `A`. At run-time, `a` is first evaluated. If `a` is not null, `a` becomes the result. Otherwise, `b` is evaluated and converted to type `A`, and this becomes the result.
*  Otherwise, if `b` has a type `B` and an implicit conversion exists from `a` to `B`, the result type is `B`. At run-time, `a` is first evaluated. If `a` is not null, `a` is unwrapped to type `A0` (if `A` exists and is nullable) and converted to type `B`, and this becomes the result. Otherwise, `b` is evaluated and becomes the result.
*  Otherwise, `a` and `b` are incompatible, and a compile-time error occurs.

> __Expression Tree Conversion Translation Steps__
>
> A null coalescing `??` expression
>
> ```csharp
> a ?? b
> ```
>
> is translated into an expression tree by first pre-processing the expression into
>
> ```csharp
> a ?? b0
> ```
>
> where `b0` is `(T)(b)` if evaluation involves an implicit conversion exists from `b` to `T` where `T` and either be `A` or `A0`, and `b` otherwise. Note that this pre-processing step may introduce a cast expression which will be classified as compiler-generated.
>
> Expression tree translation then proceeds as follows, by first translating `a` to `aExpr` and by translating `b0` to `bExpr`.
>
> When subject to conversion to a query expression tree, it is translated into
>
> ```csharp
> Expression.Coalesce(aExpr, bExpr)
> ```
>
> unless evaluation involves a conversion of `a` from `A0` to `B`, resulting in translation into
>
> ```csharp
> Expression.Coalesce(aExpr, bExpr, leftConversion)
> ```
>
> where `leftConversion` is constructed as follows
>
> ```csharp
> Expression.Lambda(Expression.Convert(p, typeof(B)), p)
> ```
>
> where `p` is a compiler-generated local variable defined as
>
> ```csharp
> var p = Expression.Parameter(typeof(A0), "p");
> ```
>
> Note that these translation steps do not rely on the translation of a lambda expression `(A0 p) => (B)p` which would be simpler. The current implementation of the C# compiler generates different `Expression` factory invocations for translation of arbitrary lambda expressions versus translation of the specific lambda expression used for a coalesce expression.
>
> When subject to conversion to a generalized expression tree, it is translated into
>
> ```csharp
> Q.Coalesce(info, aExpr, bExpr)
> ```
>
> where `bExpr` may be a `Convert` node with the `Q.Flags.CompilerGenerated` flag set, and where `info` is
>
> ```csharp
> Q.CoalesceInfo(default(Q.Flags), type)
> ```
>
> unless evaluation involves a conversion from `a` from `A0` to `B`, causing `info` to be
>
> ```csharp
> Q.CoalesceInfo(
>     default(Q.Flags),
>     type,
>     Q.ConvertInfo(Q.Flags.CompilerGenerated, typeof(B))
> )
> ```
>
> where the first flags parameter is reserved for future use, and `type` is an expression of type `Type` representing the type of the coalesce expression.
>
> Note that generalized expression trees leverage the `ConvertInfo` factory to avoid having to construct a `Lambda` expression tree to represent the conversion of `a`.
>
> ***TODO*** Decide on when to pass a `type` to an `Info` factory. We could pass it always after the `flags` (even for nodes such as `Constant`) to reduce the need for type inferencing logic in expression tree libraries. We could also do it only when absolutely needed (i.e. no way to infer within a library, or if ambiguous), at the expense of expression tree libraries having to do more work. Either way, consistency seems desirable. If we specify a type, we could also consider whether we want to offer a way to differentiate `object` from `dynamic`, e.g. using `Q.TypeInfo(type, dynamicInfo)` where `DynamicInfo` follows the convention of `DynamicAttribute.TransformFlags`.

## Logical operators

The `&`, `^`, and `|` operators are called the logical operators.

```antlr
and_expression
    : equality_expression
    | and_expression '&' equality_expression
    ;

exclusive_or_expression
    : and_expression
    | exclusive_or_expression '^' and_expression
    ;

inclusive_or_expression
    : exclusive_or_expression
    | inclusive_or_expression '|' exclusive_or_expression
    ;
```

If an operand of a logical operator has the compile-time type `dynamic`, then the expression is dynamically bound ([Dynamic binding](expressions.md#dynamic-binding)). In this case the compile-time type of the expression is `dynamic`, and the resolution described below will take place at run-time using the run-time type of those operands that have the compile-time type `dynamic`.

For an operation of the form `x op y`, where `op` is one of the logical operators, overload resolution ([Binary operator overload resolution](expressions.md#binary-operator-overload-resolution)) is applied to select a specific operator implementation. The operands are converted to the parameter types of the selected operator, and the type of the result is the return type of the operator.

> __Expression Tree Conversion Translation Steps__
>
> An expression with logical operator `op`
>
> ```csharp
> left op right
> ```
>
> is translated into an expression tree by translating `left` to `leftExpr`, by translating `right` to `rightExpr`, and by optionally constructing an expression `method` of type `MethodInfo` representing the method implementing the operator if the operator is bound to a user-defined operator.
>
> Expression tree translation then proceeds as follows, where `M` is `And` if the operator is `+`, `Or` if the operator is `|`, and `ExclusiveOr` if the operator is `^`.
>
> When subject to conversion to a query expression tree, if the operation is dynamically bound, an error is produced. Otherwise, it is translated into
>
> ```csharp
> Expression.M(leftExpr, rightExpr, method)
> ```
>
> if the operator is bound to a user-defined operator implementation, or
>
> ```csharp
> Expression.M(leftExpr, rightExpr)
> ```
>
> otherwise.
>
> When subject to conversion to a generalized expression tree it is translated into
>
> ```csharp
> Q.M(info, leftExpr, rightExpr)
> ```
>
> where `info` is
>
> ```csharp
> Q.MInfo(binder)
> ```
>
> if the operation is dynamically bound, where `binder` is an expression of type `Microsoft.CSharp.RuntimeBinder.CallSiteBinder` representing the dynamic operation, and otherwise into
>
> ```csharp
> Q.MInfo(flags, method)
> ```
>
> if the operator is bound to a user-defined operator implementation, or
>
> ```csharp
> Q.MInfo(flags)
> ```
>
> otherwise, where `flags` is any of the following:
>
> * `Q.Flags.Lifted` if the operator is [lifted](#lifted-operators), or,
> * `default(Q.Flags)` otherwise.
>
> Note that conversion to generalized expression trees use a `Lifted` flag to convey this information to the expression tree library. This reduces the burden on the expression tree library to infer this information using reflection.
>
> ***TODO***:
> * Evaluate options for the dynamic case:
>   * create a specification for the construction of `binder` (can also be used to specify `dynamic` behavior outside the context of expression trees) and consider making more properties on the `Microsoft.CSharp.RuntimeBinder` library types publicly accessable so expression tree libraries can inspect this info, or,
>   * steer away from reusing `Microsoft.CSharp` and introduce a `Pre*Info` factory overload for dynamic binding (e.g. a `(Q.DynamicFlags flags, Type context, A leftArgumentInfo, A rightArgumentInfo)` overload, where `A` is built through factory invocations as well).

## Conditional operator

The `?:` operator is called the conditional operator. It is at times also called the ternary operator.

```antlr
conditional_expression
    : null_coalescing_expression
    | null_coalescing_expression '?' expression ':' expression
    ;
```

A conditional expression of the form `b ? x : y` first evaluates the condition `b`. Then, if `b` is `true`, `x` is evaluated and becomes the result of the operation. Otherwise, `y` is evaluated and becomes the result of the operation. A conditional expression never evaluates both `x` and `y`.

The conditional operator is right-associative, meaning that operations are grouped from right to left. For example, an expression of the form `a ? b : c ? d : e` is evaluated as `a ? b : (c ? d : e)`.

The first operand of the `?:` operator must be an expression that can be implicitly converted to `bool`, or an expression of a type that implements `operator true`. If neither of these requirements is satisfied, a compile-time error occurs.

The second and third operands, `x` and `y`, of the `?:` operator control the type of the conditional expression.

*  If `x` has type `X` and `y` has type `Y` then
   * If an implicit conversion ([Implicit conversions](conversions.md#implicit-conversions)) exists from `X` to `Y`, but not from `Y` to `X`, then `Y` is the type of the conditional expression.
   * If an implicit conversion ([Implicit conversions](conversions.md#implicit-conversions)) exists from `Y` to `X`, but not from `X` to `Y`, then `X` is the type of the conditional expression.
   * Otherwise, no expression type can be determined, and a compile-time error occurs.
*  If only one of `x` and `y` has a type, and both `x` and `y`, of are implicitly convertible to that type, then that is the type of the conditional expression.
*  Otherwise, no expression type can be determined, and a compile-time error occurs.

The run-time processing of a conditional expression of the form `b ? x : y` consists of the following steps:

*  First, `b` is evaluated, and the `bool` value of `b` is determined:
   * If an implicit conversion from the type of `b` to `bool` exists, then this implicit conversion is performed to produce a `bool` value.
   * Otherwise, the `operator true` defined by the type of `b` is invoked to produce a `bool` value.
*  If the `bool` value produced by the step above is `true`, then `x` is evaluated and converted to the type of the conditional expression, and this becomes the result of the conditional expression.
*  Otherwise, `y` is evaluated and converted to the type of the conditional expression, and this becomes the result of the conditional expression.

> __Expression Tree Conversion Translation Steps__
>
> A conditional expression
>
> ```csharp
> b ? x : y
> ```
>
> is translated into an expression tree by first performing the following pre-processing steps given the types `X` and `Y` as defined earlier:
>
> * if an implicit conversion of `b` to `bool` is required, translate `b` to `(bool)b`,
> * if an implicit conversion exists from `X` to `Y`, but not from `Y` to `X`, translate `x` to `(Y)x`, and,
> * if an implicit conversion exists from `Y` to `X`, but not from `X` to `Y`, translate `y` to `(X)y`.
>
> This pre-processing step effectively makes implicit conversions explicit. Introduced casts are classified as ***compiler-generated*** and their translation to generalized expression trees will include a `Q.Flags.CompilerGenerated` flag.
>
> After applying this pre-processing step, which may zero or more [cast expressions](#cast-expression), translation to expression trees proceeds by
>
> * translating the result of pre-processing `b` to an expression tree `bExpr`,
> * translating the result of pre-processing `x` to an expression tree `xExpr`,
> * translating the result of pre-processing `y` to an expression tree `yExpr`, and
>
> by proceeding as follows.
>
> When subject to conversion to a query expression tree, continue by translating into
>
> ```csharp
> Expression.Condition(bExpr, xExpr, yExpr)
> ```
>
> When subject to conversion to a generalized expression tree, continue by translating into
>
> ```csharp
> Q.Condition(info, bExpr, xExpr, yExpr)
> ```
>
> where `info` is
>
> ```csharp
> Q.ConditionInfo(default(Q.Flags))
> ```
>
> where the flags parameter is reserved for future use.
>
> Note that the expression tree library can infer the type of the expression from either `xExpr` or `yExpr` due to insertion of implicit conversions around `x` or `y`. Also note that any such conversions can be determined to be compiler-generated by checking the `Q.Flags.CompilerGenerated` flag that was passed to the `Q.Convert` factory method invocations.

## Anonymous function expressions

An ***anonymous function*** is an expression that represents an "in-line" method definition. An anonymous function does not have a value or type in and of itself, but is convertible to a compatible delegate or expression tree type. The evaluation of an anonymous function conversion depends on the target type of the conversion: If it is a delegate type, the conversion evaluates to a delegate value referencing the method which the anonymous function defines. If it is an expression tree type, the conversion evaluates to an expression tree which represents the structure of the method as an object structure.

For historical reasons there are two syntactic flavors of anonymous functions, namely *lambda_expression*s and *anonymous_method_expression*s. For almost all purposes, *lambda_expression*s are more concise and expressive than *anonymous_method_expression*s, which remain in the language for backwards compatibility.

```antlr
lambda_expression
    : anonymous_function_signature '=>' anonymous_function_body
    ;

anonymous_method_expression
    : 'delegate' explicit_anonymous_function_signature? block
    ;

anonymous_function_signature
    : explicit_anonymous_function_signature
    | implicit_anonymous_function_signature
    ;

explicit_anonymous_function_signature
    : '(' explicit_anonymous_function_parameter_list? ')'
    ;

explicit_anonymous_function_parameter_list
    : explicit_anonymous_function_parameter (',' explicit_anonymous_function_parameter)*
    ;

explicit_anonymous_function_parameter
    : anonymous_function_parameter_modifier? type identifier
    ;

anonymous_function_parameter_modifier
    : 'ref'
    | 'out'
    ;

implicit_anonymous_function_signature
    : '(' implicit_anonymous_function_parameter_list? ')'
    | implicit_anonymous_function_parameter
    ;

implicit_anonymous_function_parameter_list
    : implicit_anonymous_function_parameter (',' implicit_anonymous_function_parameter)*
    ;

implicit_anonymous_function_parameter
    : identifier
    ;

anonymous_function_body
    : expression
    | block
    ;
```

The `=>` operator has the same precedence as assignment (`=`) and is right-associative.

An anonymous function with the `async` modifier is an async function and follows the rules described in [Iterators](classes.md#iterators).

The parameters of an anonymous function in the form of a *lambda_expression* can be explicitly or implicitly typed. In an explicitly typed parameter list, the type of each parameter is explicitly stated. In an implicitly typed parameter list, the types of the parameters are inferred from the context in which the anonymous function occurs—specifically, when the anonymous function is converted to a compatible delegate type or expression tree type, that type provides the parameter types ([Anonymous function conversions](conversions.md#anonymous-function-conversions)).

In an anonymous function with a single, implicitly typed parameter, the parentheses may be omitted from the parameter list. In other words, an anonymous function of the form
```csharp
( param ) => expr
```
can be abbreviated to
```csharp
param => expr
```

The parameter list of an anonymous function in the form of an *anonymous_method_expression* is optional. If given, the parameters must be explicitly typed. If not, the anonymous function is convertible to a delegate with any parameter list not containing `out` parameters.

A *block* body of an anonymous function is reachable ([End points and reachability](statements.md#end-points-and-reachability)) unless the anonymous function occurs inside an unreachable statement.

Some examples of anonymous functions follow below:

```csharp
x => x + 1                              // Implicitly typed, expression body
x => { return x + 1; }                  // Implicitly typed, statement body
(int x) => x + 1                        // Explicitly typed, expression body
(int x) => { return x + 1; }            // Explicitly typed, statement body
(x, y) => x * y                         // Multiple parameters
() => Console.WriteLine()               // No parameters
async (t1,t2) => await t1 + await t2    // Async
delegate (int x) { return x + 1; }      // Anonymous method expression
delegate { return 1 + 1; }              // Parameter list omitted
```

The behavior of *lambda_expression*s and *anonymous_method_expression*s is the same except for the following points:

*  *anonymous_method_expression*s permit the parameter list to be omitted entirely, yielding convertibility to delegate types of any list of value parameters.
*  *lambda_expression*s permit parameter types to be omitted and inferred whereas *anonymous_method_expression*s require parameter types to be explicitly stated.
*  The body of a *lambda_expression* can be an expression or a statement block whereas the body of an *anonymous_method_expression* must be a statement block.
*  Only *lambda_expression*s have conversions to compatible query expression tree types; *anonymous_method_expression*s can only be converted to generalized expression tree types ([Expression tree types](types.md#expression-tree-types)).

> __Expression Tree Conversion Translation Steps__
>
> Conversion of anonymous function expressions to expression trees first infers the compatible delegate type `D` or expression type `E<D>` from the context in which the anonymous function occurs using the rules of anonymous function conversion ([Anonymous function conversions](conversions.md#anonymous-function-conversions)). If the resulting type is an expression type `E<D>`, the anonymous function is ***quoted***.
>
> When converting to a generalized expression tree, `Q` is used to refer to the expression tree builder type of expression type `E<D>`, if the anonymous function expression is converted to `E<D>`. Otherwise, `Q` refers to the expression tree builder type associated with the expression tree conversion of the closest parent anonymous function expression.
>
> Conversion of an *anonymous_method_expression* to a query expression tree is not supported and results in a compile-time error.
>
> Conversion of an *anonymous_method_expression* to a generalized expression tree is based on rewriting the *anonymous_method_expression* to a *lambda_expression* using the following rules. Given an expression of the form:
>
> ```antlr
> anonymous_method_expression
>     : 'delegate' explicit_anonymous_function_signature? block
>     ;
> ```
>
> If the *explicit_anonymous_function_signature* is missing, construct an *explicit_anonymous_function_signature* by inferring parameter types `P_1` to `P_N`, and optional *anonymous_function_parameter_modifier*s `M_1` to `M_N` from the parameter types on delegate type `D`, where `N` is the number of parameters. Then construct an *explicit_anonymous_function_signature* of the form:
>
> ```csharp
> (M_1 P_1 p1, ..., M_N P_N pN)
> ```
>
> where `p1` to `pN` are compiler-generated identifiers that do not conflict with any variables referenced in the *block* of the anonymous method expression. Each of the introduced parameters is classified as compiler-generated.
>
> Using the *explicit_anonymous_function_signature*, possibly constructed using the steps above, construct a *lambda_expression* as follows:
>
> ```antlr
> explicit_anonymous_function_signature '=>' block
> ```
>
> An example will make this rewrite clear. Consider the following anonymous method expression
>
> ```csharp
> delegate { return 42; }
> ```
>
> being converted to a `Func<string, bool, int>` delegate type. The rewrite rule transforms this to a lambda expression of the following form:
>
> ```csharp
> (string p0, bool p1) => { return 42; }
> ```
>
> Expression tree conversion then proceeds by converting the resulting *lambda_expression*.
>
> Conversion of a *lambda_expression* to an expression tree first rewrites an implicitly typed parameter list of the form
>
> ```antlr
> implicit_anonymous_function_signature
>     : '(' implicit_anonymous_function_parameter_list? ')'
>     | implicit_anonymous_function_parameter
>     ;
> ```
>
> by first eliminating the case of a single *implicit_anonymous_function_parameter* without surrounding `(` and `)` tokens by rewriting it to an *implicit_anonymous_function_parameter_list* of length one, containing the single parameter.
>
> If an *implicit_anonymous_function_parameter_list* is present, rewrite it to an *explicit_anonymous_function_parameter_list* by inferring the parameter type `P_i` for each *implicit_anonymous_function_parameter* from the type of the `i`th parameter on delegate type `D` in order to construct an *explicit_anonymous_function_parameter* of the form:
>
> ```csharp
> P_i identifier
> ```
>
> where `identifier` is obtained from the original *implicit_anonymous_function_parameter*. An *explicit_anonymous_function_parameter_list* is then constructed from the sequence of *explicit_anonymous_function_parameter*s, and the *implicit_anonymous_function_parameter_list* in the *lambda_expression* is replaced by the newly constructed parameter list.
>
> An example will make this rewrite clear. Consider the following lambda expression
>
> ```csharp
> (x, y) => x + y
> ```
>
> being converted to a `Func<long, int, long>` delegate type. The rewrite rule transforms this to a lambda expression of the following form:
>
> ```csharp
> (long x, int x) => x + y
> ```
>
> After applying all the rewrite steps above, conversion to expression trees is reduced to *lambda_expression*s of the following form:
>
> ```antlr
> lambda_expression
>     : '(' explicit_anonymous_function_parameter_list? ')' '=>' anonymous_function_body
>     ;
> 
> explicit_anonymous_function_parameter_list
>     : explicit_anonymous_function_parameter (',' explicit_anonymous_function_parameter)*
>     ;
> 
> explicit_anonymous_function_parameter
>     : anonymous_function_parameter_modifier? type identifier
>     ;
> 
> anonymous_function_parameter_modifier
>     : 'ref'
>     | 'out'
>     ;
> 
> anonymous_function_body
>     : expression
>     | block
>     ;
> ```
>
> Expression tree conversion proceeds by applying the expression tree conversion steps in [Value parameters](variables.md#value-parameters) or [Reference parameters](variables.md#reference-parameters) to each *explicit_anonymous_function_parameter*. This results in the introduction of a local variable `t` for each parameter converted to an expression tree. We subsequently refer to these variables as `parameterVar_i` (where `1 < i < N`), each of these corresponding to the parameter with index `i`.
>
> Next, an expression `parameterList` is constructed from the parameter expressions `parameterVar_i` (where `1 < i < N`). When converted to a query expression tree, an array is constructed:
>
> ```csharp
> new ParameterExpression[] { parameterVar_1, ..., parameterVar_N }
> ```
>
> If `N` is `0`, implementations have the freedom to use an expression other than `new ParameterExpression[] {}` to construct an empty array instance that may not be be unique. An example is using `Array.Empty<ParameterExpression>()` to use a shared empty array instance.
>
> When converted to a generalized expression tree, `parameterList` is constructed as follows:
>
> ```csharp
> Q.ParameterList(info, parameterVar_1, ..., parameterVar_N)
> ```
>
> where `info` is
>
> ```csharp
> Q.ParameterListInfo(default(Q.Flags))
> ```
>
> where the first parameter is reserved for future use. Note that the `ParameterList` factory method can have optimized overloads for low values of `N` in addition to providing a `params` overload.
>
> Next, the *anonymous_function_body* is converted to an expression tree `bodyExpr`, where each use site of the `i`th parameter is translated using the compiler-generated local variable `parameterVar_i` (see [Value parameters](variables.md#value-parameters) and [Reference parameters](variables.md#reference-parameters)).
>
> Note that query expression trees do not support statement blocks for anonymous function bodies. That is, no expression tree conversion rules exist for statements.
>
> Next, an expression tree node `lambdaExpr` is constructed for the anonymous function expression using the `parameterList` and `bodyExpr` expressions, and using the delegate type `D`.
>
> When converted to a query expression tree, and an `async` modifier is specified on the lambda expression, a compile-time error occurs. Otherwise, it is translated into
>
> ```csharp
> Expression.Lambda<D>(bodyExpr, parameterList)
> ```
>
> where `D` is the delegate type.
>
> When converted to a generalized query expression, it is translated into
>
> ```csharp
> Q.Lambda<D>(info, parameterList, bodyExpr)
> ```
>
> where `D` is the delegate type, and where `info` is
>
> ```csharp
> Q.AsyncLambdaInfo(flags)
> ```
>
> if the lambda expression has an `async` modifier and has a `void`, `Task`, or `Task<T>` return type, or
>
> ```csharp
> Q.AsyncLambdaInfo(flags, builderType)
> ```
>
> if the lambda expression has an `async` modifier and has a return type different from `void`, `Task`, or `Task<T>`, where `builderType` is an expression of type `Type` representing the builder type associated with the return type, or
>
> ```csharp
> Q.LambdaInfo(flags)
> ```
>
> otherwise, where `flags` is `Q.Flags.CompilerGenerated` if the anonymous function expression is compiler-generated or `default(Q.Flags)` otherwise.
>
> Note that the order of parameters on `Q.Lambda` differs from the order on `Expression.Lambda`. Generalized query expression trees retain the original lexical order and use separate nodes to describe variable-length lists such as parameter lists.
>
> Finally, if the anonymous function expression is ***quoted*** (as described at the beginning of this section) and the anonymous function expression occurs in an expression or statement that itself is converted to an expression tree, a final translation ***quotation*** step is performed. Otherwise, the result of the expression tree conversion is `lambdaExpr`, as constructed in the steps above.
>
> For purposes of ***quotation***, define `PQ` as the expression tree builder type of the closest parent anonymous function expression to the current anonymous function expression, if it exists. Four distinct cases exist:
>
> 1. `P` does not exist, and `Q` does not exist.
>
>    The parent anonymous function expression is converted to a query expression tree type `Expression<PD>`, where `PD` is some delegate type, and the current anonymous function expression is converted to a query expression tree type `Expression<D>`.
>
>    Expression tree conversion results in
>
>    ```csharp
>    Expression.Quote(lambdaExpr)
>    ```
>
> 2. `P` exists, and `Q` exists.
>
>    The parent anonymous function expression is converted to a generalized expression tree type `PQ<PD>` (with builder type `P`), where `PD` is some delegate type, and the current anonymous function expression is converted to a generalized expression tree type `CQ<D>` (with builder type `Q`).
>
>    Expression tree conversion results in
>
>    ```csharp
>    P.Quote(info, lambdaExpr)
>    ```
>
>    where `info` is
>
>    ```csharp
>    P.QuoteInfo(default(P.Flags))
>    ```
>
> 3. `P` exists, and `Q` does not exist.
>
>    The parent anonymous function expression is converted to a generalized expression tree type `PQ<PD>` (with builder type `P`), where `PD` is some delegate type, and the current anonymous function expression is converted to a query expression tree type `Expression<D>`.
>
>    Expression tree conversion results in
>
>    ```csharp
>    P.Quote(info, lambdaExpr)
>    ```
>
>    where `info` is
>
>    ```csharp
>    P.QuoteInfo(default(P.Flags))
>    ```
>
>    The expression tree library defining `P.Quote` can decide not to support this conversion by omitting an overload that is compatible with a `Expression<D>` argument.
>
> 4. `P` does not exist, and `Q` exists.
>
>    The parent anonymous function expression is converted to a query expression tree type `Expression<PD>`, where `PD` is some delegate type, and the current anonymous function expression is converted to a generalized expression tree type `CQ<D>` (with builder type `Q`).
>
>    This case results in a compile-time error.
>
> ***TODO***
> * There's an issue with cases 2, 3, and 4 due to the way variables are referenced in expression trees.
>    * The current conversion rules in the specification will result in `ParameterExpression` nodes being passed to `Q.*` factory methods, and `Q.Parameter` nodes being passed to `Expression.*` factory methods. For case 2, `P.Parameter` nodes will be passed to `Q.*` factory methods. In summary, any heterogeneous composition of expression tree types results in this type of cross-referencing.
>    * Similar to the rules above, we likely need to augment the section on variables to introduce some form of conversions around nodes representing variables when they occur in a use site with an incompatible expression tree type. Given analogous numbering of cases, and using variable `v` and its corresponding expression tree node stored in variable `t`:
>      2. For the heterogeneous case of `P` and `Q` being different, a use site reference to variable `v` (constructed using `P.Parameter`) is translated to `Q.Variable(t)`. The expression tree library defining `Q` has to support an overload of `Variable` compatible with an argument of the type returned by `P.Parameter` or `P.Variable`. Otherwise, for the homogeneous case, it is translated to just `t`.
>      3. A use site reference to variable `v` (constructed using `Expression.Parameter`) is translated to `Q.Variable(t)`. The expression tree library defining `Q` has to support an overload of `Variable` compatible with a `ParameterExpression` argument.
>      4.  A use site reference to variable `v` (constructed using `P.Parameter`) is translated to `P.ToExpression(t)`. The expression tree library defining `P` has to support a method `ToExpression` with a return type that is implicitly convertible to `Expression`.
>   * For all of the cases described above, cross-referencing node types constructed from `Variable` and `ToExpression` factories can just wrap the given "foreign" node. Various mechanisms can be used to "bind" these nodes via top-down rewrites where a binding environment is passed from parent expression trees to child expression trees. This is explored below.
> * Review if we can make case 4 above work, and more generally binding across heterogeneous compositions of expression tree types.
>   * The general idea is that this could be made to work if there is a context-aware means for the nested expression tree to get converted to a parent expression tree type. The relevant context consists of the variables referenced in the nested expression tree, but declared by the parent tree.
>   * For case 4, the closest construct we have available in `System.Linq.Expressions` is the `RuntimeVariables` node type which can be used to introduce storage locations for variables which are made accessible through `IRuntimeVariables`. This is combined with `StrongBox<T>` entries for mutable value types, so a strongly typed storage location of type `T` (in the form of a field variable) can be obtained, thus retaining the ability to pass a value by `ref` (e.g. for an instance call, a `ref` or `out` parameter binding, etc.).
>     * The mechanism underlying `RuntimeVariables` in the expression tree lambda compiler ensures proper hoisting for the referenced variables, and proper binding of references to these variables using indexing into `IRuntimeVariables` and a proper cast (possibly involving `StrongBox<T>` and a member access to its `Value` field).
>     * The idea could be to emit an `Expression.Call` to a static method `Q.ToExpression` given `lambdaExpr` and a `RuntimeVariables` node representing the environment, where this method on the builder type is supposed to return an `Expression`.
>       * This is awkward to use, because of the `int`-based indexing into `IRuntimeVariables`, so we need an auxiliary map to associate indexes with the corresponding `Q.Parameter` local. That way, the implementation of `Q.ToExpression` can find all unbound parameters in `lambdaExpr` and map them to indexes in the `IRuntimeVariables` data structure, e.g. to inline access expressions if the goal is to support evaluation of the expression tree.
>    * The general heterogenous case seems to call for a similar mechanism which boils down to:
>      * Expression tree conversion steps keep track of variables that are referenced across expression trees of incompatible types (`P` to `Q`, `Expression` to `Q`, or `Q` to `Expression`).
>      * If any such variables exist, the `Quote` is augmented by associating it with a binding environment.
>    * An example for case 4 is sketched below. 
>
>> 4. `P` does not exist, and `Q` exists.
>>
>>    The parent anonymous function expression is converted to a query expression tree type `Expression<PD>`, where `PD` is some delegate type, and the current anonymous function expression is converted to a generalized expression tree type `CQ<D>` (with builder type `Q`).
>>
>>    ~~This case results in a compile-time error.~~
>>
>>    Expression tree conversion proceeds by determining the variables referenced in `lambdaExpr` which are declared in the parent expression tree. Let this set of variables be `var_j` for `1 <= j <= M`. From this, we construct a `bindings` expression of type `ParameterExpression[]`
>>
>>    ```csharp
>>    new ParameterExpression[] { var_1, ..., var_M }
>>    ```
>>
>>    and convert it to an expression tree `bindingsExpr`
>>
>>    ```csharp
>>    Expression.Constant(bindings, typeof(ParameterExpression[]))
>>    ```
>>
>>    Next, we construct an expression `environmentExpr`
>>
>>    ```csharp
>>    Expression.RuntimeVariables(bindings)
>>    ```
>>
>>    Finally, we translate the expression tree to
>>
>>    ```csharp
>>    Expression.Call(
>>        null,
>>        method,
>>        Expression.Constant(lambdaExpr),
>>        bindingsExpr,
>>        environmentExpr
>>    )
>>    ```
>>
>>    where `method` is an expression of type `MethodInfo` representing an overload of `Q.ToExpression` with a compatible argument list, if it exists. Otherwise, a compile-time error occurs.
>>    
>>    An example of a `ToExpression` method is shown below:
>>
>>    ```csharp
>>    [ExpressionBuilder(typeof(Quote))]
>>    class Quote<T> : LambdaQuote { ... }
>>    
>>    class LambdaQuote : Quote { ... }
>>
>>    class OuterVariableQuote : Quote
>>    {
>>        public OuterVariableQuote(ParameterExpression p) =>
>>            Variable = p;
>>    
>>        public ParameterExpression Variable { get; }
>>    }
>>    
>>    class Quote
>>    {
>>        public static Quote<T> Lambda<T>(...) { ... }
>>
>>        public static OuterVariableQuote Variable(ParameterExpression p) { ... }
>>    
>>        public static Expression ToExpression(
>>            LambdaQuote quote,
>>            ParameterExpression[] outerVariables,
>>            IRuntimeVariables bindings) { ... }
>>    }
>>    ```
>>    
>>    The `ToExpression` method can translate the lambda `quote` to an expression tree while binding all occurrences of `OuterVariableQuote` by determining the index of `OuterVariableQuote.Variable` in `outerVariables` and returning an `Expression` constructed from
>>
>>    ```csharp
>>    Expression.Field(
>>        Expression.Convert(
>>            Expression.MakeIndex(
>>                Expression.Constant(bindings),
>>                typeof(IRuntimeVariables).GetMethod("get_Item"),
>>                new[] {
>>                    Expression.Constant(Array.IndexOf(outerVariables, variable))
>>                }
>>            ),
>>            typeof(StrongBox<>).MakeGenericType(variable.Type)
>>        ),
>>        nameof(StrongBox<object>.Value)
>>    )
>>    ```
>>
>>    This translation can be deferred by returning a reducible extension node inheriting from `Expression` whose `Reduce` method performs the steps shown above.

### Anonymous function signatures

The optional *anonymous_function_signature* of an anonymous function defines the names and optionally the types of the formal parameters for the anonymous function. The scope of the parameters of the anonymous function is the *anonymous_function_body*. ([Scopes](basic-concepts.md#scopes)) Together with the parameter list (if given) the anonymous-method-body constitutes a declaration space ([Declarations](basic-concepts.md#declarations)). It is thus a compile-time error for the name of a parameter of the anonymous function to match the name of a local variable, local constant or parameter whose scope includes the *anonymous_method_expression* or *lambda_expression*.

If an anonymous function has an *explicit_anonymous_function_signature*, then the set of compatible delegate types and expression tree types is restricted to those that have the same parameter types and modifiers in the same order. In contrast to method group conversions ([Method group conversions](conversions.md#method-group-conversions)), contra-variance of anonymous function parameter types is not supported. If an anonymous function does not have an *anonymous_function_signature*, then the set of compatible delegate types and expression tree types is restricted to those that have no `out` parameters.

Note that an *anonymous_function_signature* cannot include attributes or a parameter array. Nevertheless, an *anonymous_function_signature* may be compatible with a delegate type whose parameter list contains a parameter array.

Note also that conversion to an expression tree type, even if compatible, may still fail at compile-time ([Expression tree types](types.md#expression-tree-types)).

### Anonymous function bodies

The body (*expression* or *block*) of an anonymous function is subject to the following rules:

*  If the anonymous function includes a signature, the parameters specified in the signature are available in the body. If the anonymous function has no signature it can be converted to a delegate type or expression type having parameters ([Anonymous function conversions](conversions.md#anonymous-function-conversions)), but the parameters cannot be accessed in the body.
*  Except for `ref` or `out` parameters specified in the signature (if any) of the nearest enclosing anonymous function, it is a compile-time error for the body to access a `ref` or `out` parameter.
*  When the type of `this` is a struct type, it is a compile-time error for the body to access `this`. This is true whether the access is explicit (as in `this.x`) or implicit (as in `x` where `x` is an instance member of the struct). This rule simply prohibits such access and does not affect whether member lookup results in a member of the struct.
*  The body has access to the outer variables ([Outer variables](expressions.md#outer-variables)) of the anonymous function. Access of an outer variable will reference the instance of the variable that is active at the time the *lambda_expression* or *anonymous_method_expression* is evaluated ([Evaluation of anonymous function expressions](expressions.md#evaluation-of-anonymous-function-expressions)).
*  It is a compile-time error for the body to contain a `goto` statement, `break` statement, or `continue` statement whose target is outside the body or within the body of a contained anonymous function.
*  A `return` statement in the body returns control from an invocation of the nearest enclosing anonymous function, not from the enclosing function member. An expression specified in a `return` statement must be implicitly convertible to the return type of the delegate type or expression tree type to which the nearest enclosing *lambda_expression* or *anonymous_method_expression* is converted ([Anonymous function conversions](conversions.md#anonymous-function-conversions)).

It is explicitly unspecified whether there is any way to execute the block of an anonymous function other than through evaluation and invocation of the *lambda_expression* or *anonymous_method_expression*. In particular, the compiler may choose to implement an anonymous function by synthesizing one or more named methods or types. The names of any such synthesized elements must be of a form reserved for compiler use.

### Overload resolution and anonymous functions

Anonymous functions in an argument list participate in type inference and overload resolution. Please refer to [Type inference](expressions.md#type-inference) and [Overload resolution](expressions.md#overload-resolution) for the exact rules.

The following example illustrates the effect of anonymous functions on overload resolution.

```csharp
class ItemList<T>: List<T>
{
    public int Sum(Func<T,int> selector) {
        int sum = 0;
        foreach (T item in this) sum += selector(item);
        return sum;
    }

    public double Sum(Func<T,double> selector) {
        double sum = 0;
        foreach (T item in this) sum += selector(item);
        return sum;
    }
}
```

The `ItemList<T>` class has two `Sum` methods. Each takes a `selector` argument, which extracts the value to sum over from a list item. The extracted value can be either an `int` or a `double` and the resulting sum is likewise either an `int` or a `double`.

The `Sum` methods could for example be used to compute sums from a list of detail lines in an order.

```csharp
class Detail
{
    public int UnitCount;
    public double UnitPrice;
    ...
}

void ComputeSums() {
    ItemList<Detail> orderDetails = GetOrderDetails(...);
    int totalUnits = orderDetails.Sum(d => d.UnitCount);
    double orderTotal = orderDetails.Sum(d => d.UnitPrice * d.UnitCount);
    ...
}
```

In the first invocation of `orderDetails.Sum`, both `Sum` methods are applicable because the anonymous function `d => d. UnitCount` is compatible with both `Func<Detail,int>` and `Func<Detail,double>`. However, overload resolution picks the first `Sum` method because the conversion to `Func<Detail,int>` is better than the conversion to `Func<Detail,double>`.

In the second invocation of `orderDetails.Sum`, only the second `Sum` method is applicable because the anonymous function `d => d.UnitPrice * d.UnitCount` produces a value of type `double`. Thus, overload resolution picks the second `Sum` method for that invocation.

### Anonymous functions and dynamic binding

An anonymous function cannot be a receiver, argument or operand of a dynamically bound operation.

### Outer variables

Any local variable, value parameter, or parameter array whose scope includes the *lambda_expression* or *anonymous_method_expression* is called an ***outer variable*** of the anonymous function. In an instance function member of a class, the `this` value is considered a value parameter and is an outer variable of any anonymous function contained within the function member.

#### Captured outer variables

When an outer variable is referenced by an anonymous function, the outer variable is said to have been ***captured*** by the anonymous function. Ordinarily, the lifetime of a local variable is limited to execution of the block or statement with which it is associated ([Local variables](variables.md#local-variables)). However, the lifetime of a captured outer variable is extended at least until the delegate or expression tree created from the anonymous function becomes eligible for garbage collection.

In the example
```csharp
using System;

delegate int D();

class Test
{
    static D F() {
        int x = 0;
        D result = () => ++x;
        return result;
    }

    static void Main() {
        D d = F();
        Console.WriteLine(d());
        Console.WriteLine(d());
        Console.WriteLine(d());
    }
}
```
the local variable `x` is captured by the anonymous function, and the lifetime of `x` is extended at least until the delegate returned from `F` becomes eligible for garbage collection (which doesn't happen until the very end of the program). Since each invocation of the anonymous function operates on the same instance of `x`, the output of the example is:
```
1
2
3
```

When a local variable or a value parameter is captured by an anonymous function, the local variable or parameter is no longer considered to be a fixed variable ([Fixed and moveable variables](unsafe-code.md#fixed-and-moveable-variables)), but is instead considered to be a moveable variable. Thus any `unsafe` code that takes the address of a captured outer variable must first use the `fixed` statement to fix the variable.

Note that unlike an uncaptured variable, a captured local variable can be simultaneously exposed to multiple threads of execution.

> __Expression Tree Conversion Translation Steps__
>
> Conversion of use sites of ***captured*** outer variables to expression trees involves a transformation of
>
> ```csharp
> v
> ```
>
> where `v` is a captured outer variable not equal to `this`, to
>
> ```csharp
> closure.v
> ```
>
> where `closure` is an instance of compiler-generated class, and `v` is a field that is used as the storage location of captured outer variable `v`, thus extending its lifetime.
>
> Expression tree conversion proceeds by converting `closure.v` as a member access expression ([Member access](#member-access)), where `closure` is treated as a a *literal* for purposes of expression tree conversion ([Literals](#literals)). In particular, `closure` will be represented in an expression tree by using the `Constant` expression tree factory method.
>
> The following example illustrates these conversion rules.
>
> ```csharp
> int x = 42;
> Expression<Func<int>> f = () => x;
> ```
>
> is translated to
>
> ```csharp
> var t = new C();
> t.x = 42;
> Expression<Func<int>> f =
>     Expression.Lambda(
>         typeof(Func<int>),
>         Expression.Field(
>             Expression.Constant(t, typeof(C)),
>             field
>         ),
>         new ParameterExpression[0]
>     );
> ```
>
> where `field` is an expression of type `FieldInfo` referring to the instance field `x` declared on the compiler-generated type `C`. The identifier `t` is compiler-generated and otherwise invisible.
>
> If the captured outer variable is `this`, the rules for [This access](#this-access) apply.
>
> Conversion to generalized expression trees proceeds completely analogous using the transformation steps outlined above.
>
> ***TODO***
> * The mechanism of accessing a closure is very visible in expression trees and may warrant discussion in the context of generalized expression trees.
>   * Drawbacks of the current approach include:
>     * Variables that happen to be captured are obfuscated in `Member(Constant(closure), field)` constructions. They no longer are represented as variable expression nodes, thus taking away the WYSIWYG nature of expression trees.
>     * The mechanism by which closures are represented is not formally described in the language specification, yet it leaks deep into expression trees (in leaf nodes in fact). This exposure and lack of formal specification of the closure mechanism results in brittleness because expression tree libraries tend to rely on ugly hacks to detect the pattern. Examples include checking for `CompilerGeneratedAttribute` or even checking type names for occurrence of `DisplayClass`.
>     * Analysis of variables in expression trees performed by expression tree consuming libraries often fails to detect outer variables or classify them as variables. For example, this can lead to pretty printing of expressions that involves the `ToString` of some closure object.
>     * It is not trivial to re-bind an expression tree to another environment because it involves chasing down `Constant` nodes that contain closures, often by pattern matching for `Member(Constant(...), ...)` constructions. Many implementations of expression tree consuming libraries have resorted to techniques that involve eliminating closures by virtue of partial evaluation or deferred forms thereof (sometimes referred to as funcletization).
>   * An alternative could be to:
>     * Let variables be variables, including for outer variables.
>     * Augment an expression tree that accesses outer variables with an enviroment that binds variables. This could be part of the outermost conversion to an expression tree, for example by having `Q.LambdaInfo` take an `environment` parameter which itself is constructed from `Q.Environment(bindings)` where each `binding` in `bindings` is a `Q.VariableBinding(variable, expr)`. A `variable` that occurs in such a binding can be used within the expression tree, and `expr` can be defined to be opaque and implementation-specific.
>     * This approach has a number of benefits:
>       * Variables can easily be discovered by a visitor, not missing out on variables that come from the environment.
>       * Outer variables are explicit immediately. One can simply create a set of all the variables that occur in the environment.
>       * Outer variables can be rebound easily. For example, one could evaluate all of them and substitute them for `Constant` nodes, or one could wrap all of them with a funclet expression that lazily reduces them to constants every time the expression is re-evaluated (e.g. query providers often perform such a technique).
>       * All `Member(Constant(...), ...)` nodes are gone from the corpus of the tree, and thus can't trip up analysis passes using visitors and whatnot that otherwise may have to special case these contraptions.
>       * Multiple references to the same outer variable do not duplicate the `Member(Constant(...), ...)` pattern in all these use sites. A proper variable node is used, which is bound in the environment.
>       * Users can choose to inline these bindings if they don't care about analyzing expression trees, e.g. if they just want to evaluate them.
>       * Outer variables are a well-described construct in the language specification, and a one-to-one mapping onto a `Q.Environment` factory (which could be named `OuterVariables` or whatever makes most sense) is easy to describe.

#### Instantiation of local variables

A local variable is considered to be ***instantiated*** when execution enters the scope of the variable. For example, when the following method is invoked, the local variable `x` is instantiated and initialized three times—once for each iteration of the loop.

```csharp
static void F() {
    for (int i = 0; i < 3; i++) {
        int x = i * 2 + 1;
        ...
    }
}
```

However, moving the declaration of `x` outside the loop results in a single instantiation of `x`:
```csharp
static void F() {
    int x;
    for (int i = 0; i < 3; i++) {
        x = i * 2 + 1;
        ...
    }
}
```

When not captured, there is no way to observe exactly how often a local variable is instantiated—because the lifetimes of the instantiations are disjoint, it is possible for each instantiation to simply use the same storage location. However, when an anonymous function captures a local variable, the effects of instantiation become apparent.

The example
```csharp
using System;

delegate void D();

class Test
{
    static D[] F() {
        D[] result = new D[3];
        for (int i = 0; i < 3; i++) {
            int x = i * 2 + 1;
            result[i] = () => { Console.WriteLine(x); };
        }
        return result;
    }

    static void Main() {
        foreach (D d in F()) d();
    }
}
```
produces the output:
```
1
3
5
```

However, when the declaration of `x` is moved outside the loop:
```csharp
static D[] F() {
    D[] result = new D[3];
    int x;
    for (int i = 0; i < 3; i++) {
        x = i * 2 + 1;
        result[i] = () => { Console.WriteLine(x); };
    }
    return result;
}
```
the output is:
```
5
5
5
```

If a for-loop declares an iteration variable, that variable itself is considered to be declared outside of the loop. Thus, if the example is changed to capture the iteration variable itself:

```csharp
static D[] F() {
    D[] result = new D[3];
    for (int i = 0; i < 3; i++) {
        result[i] = () => { Console.WriteLine(i); };
    }
    return result;
}
```
only one instance of the iteration variable is captured, which produces the output:
```
3
3
3
```

It is possible for anonymous function delegates to share some captured variables yet have separate instances of others. For example, if `F` is changed to
```csharp
static D[] F() {
    D[] result = new D[3];
    int x = 0;
    for (int i = 0; i < 3; i++) {
        int y = 0;
        result[i] = () => { Console.WriteLine("{0} {1}", ++x, ++y); };
    }
    return result;
}
```
the three delegates capture the same instance of `x` but separate instances of `y`, and the output is:
```
1 1
2 1
3 1
```

Separate anonymous functions can capture the same instance of an outer variable. In the example:
```csharp
using System;

delegate void Setter(int value);

delegate int Getter();

class Test
{
    static void Main() {
        int x = 0;
        Setter s = (int value) => { x = value; };
        Getter g = () => { return x; };
        s(5);
        Console.WriteLine(g());
        s(10);
        Console.WriteLine(g());
    }
}
```
the two anonymous functions capture the same instance of the local variable `x`, and they can thus "communicate" through that variable. The output of the example is:
```
5
10
```

### Evaluation of anonymous function expressions

An anonymous function `F` must always be converted to a delegate type `D` or an expression tree type `E`, either directly or through the execution of a delegate creation expression `new D(F)`. This conversion determines the result of the anonymous function, as described in [Anonymous function conversions](conversions.md#anonymous-function-conversions).

## Query expressions

***Query expressions*** provide a language integrated syntax for queries that is similar to relational and hierarchical query languages such as SQL and XQuery.

```antlr
query_expression
    : from_clause query_body
    ;

from_clause
    : 'from' type? identifier 'in' expression
    ;

query_body
    : query_body_clauses? select_or_group_clause query_continuation?
    ;

query_body_clauses
    : query_body_clause
    | query_body_clauses query_body_clause
    ;

query_body_clause
    : from_clause
    | let_clause
    | where_clause
    | join_clause
    | join_into_clause
    | orderby_clause
    ;

let_clause
    : 'let' identifier '=' expression
    ;

where_clause
    : 'where' boolean_expression
    ;

join_clause
    : 'join' type? identifier 'in' expression 'on' expression 'equals' expression
    ;

join_into_clause
    : 'join' type? identifier 'in' expression 'on' expression 'equals' expression 'into' identifier
    ;

orderby_clause
    : 'orderby' orderings
    ;

orderings
    : ordering (',' ordering)*
    ;

ordering
    : expression ordering_direction?
    ;

ordering_direction
    : 'ascending'
    | 'descending'
    ;

select_or_group_clause
    : select_clause
    | group_clause
    ;

select_clause
    : 'select' expression
    ;

group_clause
    : 'group' expression 'by' expression
    ;

query_continuation
    : 'into' identifier query_body
    ;
```

A query expression begins with a `from` clause and ends with either a `select` or `group` clause. The initial `from` clause can be followed by zero or more `from`, `let`, `where`, `join` or `orderby` clauses. Each `from` clause is a generator introducing a ***range variable*** which ranges over the elements of a ***sequence***. Each `let` clause introduces a range variable representing a value computed by means of previous range variables. Each `where` clause is a filter that excludes items from the result. Each `join` clause compares specified keys of the source sequence with keys of another sequence, yielding matching pairs. Each `orderby` clause reorders items according to specified criteria.The final `select` or `group` clause specifies the shape of the result in terms of the range variables. Finally, an `into` clause can be used to "splice" queries by treating the results of one query as a generator in a subsequent query.

> __Expression Tree Conversion Translation Steps__
>
> For purposes of conversion to an expression tree, query expressions are first translated using the rules described in [Query expression translation](#query-expression-translation). The expression that results from this rewrite is then converted to an expression tree.
>
> ***TODO***
> * Should generalized expression trees model query expressions prior to conversion to method invocations?
>   * On the one hand, this would be consistent with our stance on avoiding lowering in order to get WYSIWYG expression trees. On the other hand, query expression translation *is* well-defined in the language specification and is not an implementation-specific lowering step.
>   * For practical reasons, having a high-fidelity representation of the original intent can be useful. For starters, it gets rid of having to derive scopes of ranges variables by inspecting `New` or `NewAnonymous` expression nodes which map properties onto constructor parameters. This is very typical plumbing in query providers to support `let` clauses or other patterns that involve the use of *transparent identifiers* (see [Transparent identifiers](#transparent-identifiers)).
>     * The specification on transparent identifiers combined with the rules for query expression translation actually reveals leakage of an implementation detail into expression trees due to the following remark:
>       > An implementation of C# is permitted to use a different mechanism than anonymous types to group together multiple range variables.
>     * Retaining the original intent in the form of query clauses would eliminate this concern altogether.
>       * Alternatively, the mechanism could be constrained to state that *transparency* is required by the mechanism provided, and it should always be modeled as some `New` expression that offers a way to map construction arguments to expressions that be used to get these arguments back (currently a `Members` list, but that rules out the use of e.g. a dictionary and use indexer access, if anyone would choose to implement the mechanism that way). This would really just drive towards a sufficiently generalized `New` variant such as `NewTransparentIdentifier` of which `NewAnonymous` is merely a special case. If anything, it should work naturally with C# 7 tuples, which would now be a better, more efficient choice to implement transparent identifiers (and we could even consider altering query expression translation steps in generalized expression trees to use this).
>   * Information about bound methods during query expression translation could likely still end up on the nodes representing query clauses. For example, a `let` clause node could have a reference to the `MethodInfo` for the bound `Select` method. However, pieces of the query expression translation lowering steps would sneak in no matter what, e.g. type arguments on `Select<T, R>` may reveal anonymous types originating from *transparent identifiers*. The mechanism to retain this binding information would require some bi-directional association between clauses and the corresponding translated form.
>   * There is an obvious concern around equivalence between an expression tree containing a query expression versus an expression tree constructed from fluent patterns (such as `Queryable` extension methods).
>     * In the former case, one would see the fine-grained nodes for clauses, while the latter case would stitch an expression tree using `Call` factory invocations from the bound methods.
>     * Arguably, the way `Queryable` works is a very handy trick to compensate for the lack of quotation expressions (e.g. `@(from x in xs select x + 1)`) besides conversion of lambda expressions, and to dispatch to the query provider starting from the source. It's a no-starter to suggest that this natural way of formulating queryables would regress in the world of generalized expression trees. This said, generalized quotation expressions (e.g. `@(...)`) may still be worth thinking about (akin to `<@ ... @>` in F#).
>     * It's conceivable that capturing a query expression in a WYSIWYG fashion would have to be paired with a runtime library that performs query expression translation so query providers can normalize all such expressions to method call invocations if that's the way they prefer things to be. This step would be relatively straightforward if bound methods are retained in the expression tree, likely just aggregating the list of clauses by turning clauses into method invocations, for example in a `VisitQuery` method in a visitor.

### Ambiguities in query expressions

Query expressions contain a number of "contextual keywords", i.e., identifiers that have special meaning in a given context. Specifically these are `from`, `where`, `join`, `on`, `equals`, `into`, `let`, `orderby`, `ascending`, `descending`, `select`, `group` and `by`. In order to avoid ambiguities in query expressions caused by mixed use of these identifiers as keywords or simple names, these identifiers are considered keywords when occurring anywhere within a query expression.

For this purpose, a query expression is any expression that starts with "`from identifier`" followed by any token except "`;`", "`=`" or "`,`".

In order to use these words as identifiers within a query expression, they can be prefixed with "`@`" ([Identifiers](lexical-structure.md#identifiers)).

### Query expression translation

The C# language does not specify the execution semantics of query expressions. Rather, query expressions are translated into invocations of methods that adhere to the *query expression pattern* ([The query expression pattern](expressions.md#the-query-expression-pattern)). Specifically, query expressions are translated into invocations of methods named `Where`, `Select`, `SelectMany`, `Join`, `GroupJoin`, `OrderBy`, `OrderByDescending`, `ThenBy`, `ThenByDescending`, `GroupBy`, and `Cast`.These methods are expected to have particular signatures and result types, as described in [The query expression pattern](expressions.md#the-query-expression-pattern). These methods can be instance methods of the object being queried or extension methods that are external to the object, and they implement the actual execution of the query.

The translation from query expressions to method invocations is a syntactic mapping that occurs before any type binding or overload resolution has been performed. The translation is guaranteed to be syntactically correct, but it is not guaranteed to produce semantically correct C# code. Following translation of query expressions, the resulting method invocations are processed as regular method invocations, and this may in turn uncover errors, for example if the methods do not exist, if arguments have wrong types, or if the methods are generic and type inference fails.

A query expression is processed by repeatedly applying the following translations until no further reductions are possible. The translations are listed in order of application: each section assumes that the translations in the preceding sections have been performed exhaustively, and once exhausted, a section will not later be revisited in the processing of the same query expression.

Assignment to range variables is not allowed in query expressions. However a C# implementation is permitted to not always enforce this restriction, since this may sometimes not be possible with the syntactic translation scheme presented here.

Certain translations inject range variables with transparent identifiers denoted by `*`. The special properties of transparent identifiers are discussed further in [Transparent identifiers](expressions.md#transparent-identifiers).

#### Select and groupby clauses with continuations

A query expression with a continuation
```csharp
from ... into x ...
```
is translated into
```csharp
from x in ( from ... ) ...
```

The translations in the following sections assume that queries have no `into` continuations.

The example
```csharp
from c in customers
group c by c.Country into g
select new { Country = g.Key, CustCount = g.Count() }
```
is translated into
```csharp
from g in
    from c in customers
    group c by c.Country
select new { Country = g.Key, CustCount = g.Count() }
```
the final translation of which is
```csharp
customers.
GroupBy(c => c.Country).
Select(g => new { Country = g.Key, CustCount = g.Count() })
```

#### Explicit range variable types

A `from` clause that explicitly specifies a range variable type
```csharp
from T x in e
```
is translated into
```csharp
from x in ( e ) . Cast < T > ( )
```

A `join` clause that explicitly specifies a range variable type
```
join T x in e on k1 equals k2
```
is translated into
```
join x in ( e ) . Cast < T > ( ) on k1 equals k2
```

The translations in the following sections assume that queries have no explicit range variable types.

The example
```csharp
from Customer c in customers
where c.City == "London"
select c
```
is translated into
```csharp
from c in customers.Cast<Customer>()
where c.City == "London"
select c
```
the final translation of which is
```csharp
customers.
Cast<Customer>().
Where(c => c.City == "London")
```

Explicit range variable types are useful for querying collections that implement the non-generic `IEnumerable` interface, but not the generic `IEnumerable<T>` interface. In the example above, this would be the case if `customers` were of type `ArrayList`.

#### Degenerate query expressions

A query expression of the form
```csharp
from x in e select x
```
is translated into
```csharp
( e ) . Select ( x => x )
```

The example
```csharp
from c in customers
select c
```
is translated into
```csharp
customers.Select(c => c)
```

A degenerate query expression is one that trivially selects the elements of the source. A later phase of the translation removes degenerate queries introduced by other translation steps by replacing them with their source. It is important however to ensure that the result of a query expression is never the source object itself, as that would reveal the type and identity of the source to the client of the query. Therefore this step protects degenerate queries written directly in source code by explicitly calling `Select` on the source. It is then up to the implementers of `Select` and other query operators to ensure that these methods never return the source object itself.

#### From, let, where, join and orderby clauses

A query expression with a second `from` clause followed by a `select` clause
```csharp
from x1 in e1
from x2 in e2
select v
```
is translated into
```csharp
( e1 ) . SelectMany( x1 => e2 , ( x1 , x2 ) => v )
```

A query expression with a second `from` clause followed by something other than a `select` clause:

```csharp
from x1 in e1
from x2 in e2
...
```
is translated into
```csharp
from * in ( e1 ) . SelectMany( x1 => e2 , ( x1 , x2 ) => new { x1 , x2 } )
...
```

A query expression with a `let` clause
```csharp
from x in e
let y = f
...
```
is translated into
```csharp
from * in ( e ) . Select ( x => new { x , y = f } )
...
```

A query expression with a `where` clause
```csharp
from x in e
where f
...
```
is translated into
```csharp
from x in ( e ) . Where ( x => f )
...
```

A query expression with a `join` clause without an `into` followed by a `select` clause
```csharp
from x1 in e1
join x2 in e2 on k1 equals k2
select v
```
is translated into
```csharp
( e1 ) . Join( e2 , x1 => k1 , x2 => k2 , ( x1 , x2 ) => v )
```

A query expression with a `join` clause without an `into` followed by something other than a `select` clause
```csharp
from x1 in e1
join x2 in e2 on k1 equals k2
...
```
is translated into
```csharp
from * in ( e1 ) . Join( e2 , x1 => k1 , x2 => k2 , ( x1 , x2 ) => new { x1 , x2 })
...
```

A query expression with a `join` clause with an `into` followed by a `select` clause
```csharp
from x1 in e1
join x2 in e2 on k1 equals k2 into g
select v
```
is translated into
```csharp
( e1 ) . GroupJoin( e2 , x1 => k1 , x2 => k2 , ( x1 , g ) => v )
```

A query expression with a `join` clause with an `into` followed by something other than a `select` clause
```csharp
from x1 in e1
join x2 in e2 on k1 equals k2 into g
...
```
is translated into
```csharp
from * in ( e1 ) . GroupJoin( e2 , x1 => k1 , x2 => k2 , ( x1 , g ) => new { x1 , g })
...
```

A query expression with an `orderby` clause
```csharp
from x in e
orderby k1 , k2 , ..., kn
...
```
is translated into
```csharp
from x in ( e ) . 
OrderBy ( x => k1 ) . 
ThenBy ( x => k2 ) .
... .
ThenBy ( x => kn )
...
```

If an ordering clause specifies a `descending` direction indicator, an invocation of `OrderByDescending` or `ThenByDescending` is produced instead.

The following translations assume that there are no `let`, `where`, `join` or `orderby` clauses, and no more than the one initial `from` clause in each query expression.

The example
```csharp
from c in customers
from o in c.Orders
select new { c.Name, o.OrderID, o.Total }
```
is translated into
```csharp
customers.
SelectMany(c => c.Orders,
     (c,o) => new { c.Name, o.OrderID, o.Total }
)
```

The example
```csharp
from c in customers
from o in c.Orders
orderby o.Total descending
select new { c.Name, o.OrderID, o.Total }
```
is translated into
```csharp
from * in customers.
    SelectMany(c => c.Orders, (c,o) => new { c, o })
orderby o.Total descending
select new { c.Name, o.OrderID, o.Total }
```
the final translation of which is
```csharp
customers.
SelectMany(c => c.Orders, (c,o) => new { c, o }).
OrderByDescending(x => x.o.Total).
Select(x => new { x.c.Name, x.o.OrderID, x.o.Total })
```
where `x` is a compiler generated identifier that is otherwise invisible and inaccessible.

The example
```csharp
from o in orders
let t = o.Details.Sum(d => d.UnitPrice * d.Quantity)
where t >= 1000
select new { o.OrderID, Total = t }
```
is translated into
```csharp
from * in orders.
    Select(o => new { o, t = o.Details.Sum(d => d.UnitPrice * d.Quantity) })
where t >= 1000 
select new { o.OrderID, Total = t }
```
the final translation of which is
```csharp
orders.
Select(o => new { o, t = o.Details.Sum(d => d.UnitPrice * d.Quantity) }).
Where(x => x.t >= 1000).
Select(x => new { x.o.OrderID, Total = x.t })
```
where `x` is a compiler generated identifier that is otherwise invisible and inaccessible.

> __Expression Tree Conversion Translation Steps__
>
> See remarks on [Transparent identifiers](#transparent-identifiers).

The example
```csharp
from c in customers
join o in orders on c.CustomerID equals o.CustomerID
select new { c.Name, o.OrderDate, o.Total }
```
is translated into
```csharp
customers.Join(orders, c => c.CustomerID, o => o.CustomerID,
    (c, o) => new { c.Name, o.OrderDate, o.Total })
```

The example
```csharp
from c in customers
join o in orders on c.CustomerID equals o.CustomerID into co
let n = co.Count()
where n >= 10
select new { c.Name, OrderCount = n }
```
is translated into
```csharp
from * in customers.
    GroupJoin(orders, c => c.CustomerID, o => o.CustomerID,
        (c, co) => new { c, co })
let n = co.Count()
where n >= 10 
select new { c.Name, OrderCount = n }
```
the final translation of which is
```csharp
customers.
GroupJoin(orders, c => c.CustomerID, o => o.CustomerID,
    (c, co) => new { c, co }).
Select(x => new { x, n = x.co.Count() }).
Where(y => y.n >= 10).
Select(y => new { y.x.c.Name, OrderCount = y.n)
```
where `x` and `y` are compiler generated identifiers that are otherwise invisible and inaccessible.

The example
```csharp
from o in orders
orderby o.Customer.Name, o.Total descending
select o
```
has the final translation
```csharp
orders.
OrderBy(o => o.Customer.Name).
ThenByDescending(o => o.Total)
```

#### Select clauses

A query expression of the form
```csharp
from x in e select v
```
is translated into
```csharp
( e ) . Select ( x => v )
```
except when v is the identifier x, the translation is simply
```csharp
( e )
```

For example
```csharp
from c in customers.Where(c => c.City == "London")
select c
```
is simply translated into
```csharp
customers.Where(c => c.City == "London")
```

#### Groupby clauses

A query expression of the form
```csharp
from x in e group v by k
```
is translated into
```csharp
( e ) . GroupBy ( x => k , x => v )
```
except when v is the identifier x, the translation is
```csharp
( e ) . GroupBy ( x => k )
```

The example
```csharp
from c in customers
group c.Name by c.Country
```
is translated into
```csharp
customers.
GroupBy(c => c.Country, c => c.Name)
```

#### Transparent identifiers

Certain translations inject range variables with ***transparent identifiers*** denoted by `*`. Transparent identifiers are not a proper language feature; they exist only as an intermediate step in the query expression translation process.

When a query translation injects a transparent identifier, further translation steps propagate the transparent identifier into anonymous functions and anonymous object initializers. In those contexts, transparent identifiers have the following behavior:

*  When a transparent identifier occurs as a parameter in an anonymous function, the members of the associated anonymous type are automatically in scope in the body of the anonymous function.
*  When a member with a transparent identifier is in scope, the members of that member are in scope as well.
*  When a transparent identifier occurs as a member declarator in an anonymous object initializer, it introduces a member with a transparent identifier.
*  In the translation steps described above, transparent identifiers are always introduced together with anonymous types, with the intent of capturing multiple range variables as members of a single object. An implementation of C# is permitted to use a different mechanism than anonymous types to group together multiple range variables. The following translation examples assume that anonymous types are used, and show how transparent identifiers can be translated away.

> __Expression Tree Conversion Translation Steps__
>
> When translated to an expression tree, the compiler-generated identifier `x` becomes visible as the `string` passed to the `Parameter` factory. This identifier is implementation-specific. For example, the Microsoft C# compiler implementation generates identifiers of the form `<>h__TransparentIdentifier` followed by an integer. Expression tree libraries should not rely on the exact form of this name.
>
> ***TODO***
> * For generalized expression trees:
>   * Should we consider calling the `ParameterInfo` factory with no `string` parameter (i.e. only with a `Type` parameter), rather than leaking a compiler-generated name?
>   * Can we rely on the `Q.Flags.CompilerGenerated` flag passed to the `ParameterInfo` factory to convey that the identifier (and the parameter itself) is compiler-generated?
>   * Should we model a transparent identifier as a first-class parameter kind? After all, the concept of a transparent identifier is well-described in the language specification.
>     * For example, by having a `TransparentIdentifier` factory or by using a flag that classifies a parameter/variable as such?
>     * In fact, we already have `Variable` versus `Parameter` factories. It'd be up to the expression library to decide whether they implement all of these methods by calling a common helper, or by having a distinct implementation that preserves the classification of variables.
> * In general, it's worrisome that the specification states the following, because this implementation detail currently leaks into expression trees (see remarks on [Query expressions](#query-expressions)):
>   > An implementation of C# is permitted to use a different mechanism than anonymous types to group together multiple range variables.

The example
```csharp
from c in customers
from o in c.Orders
orderby o.Total descending
select new { c.Name, o.Total }
```
is translated into
```csharp
from * in customers.
    SelectMany(c => c.Orders, (c,o) => new { c, o })
orderby o.Total descending
select new { c.Name, o.Total }
```

which is further translated into
```csharp
customers.
SelectMany(c => c.Orders, (c,o) => new { c, o }).
OrderByDescending(* => o.Total).
Select(* => new { c.Name, o.Total })
```
which, when transparent identifiers are erased, is equivalent to
```csharp
customers.
SelectMany(c => c.Orders, (c,o) => new { c, o }).
OrderByDescending(x => x.o.Total).
Select(x => new { x.c.Name, x.o.Total })
```
where `x` is a compiler generated identifier that is otherwise invisible and inaccessible.

The example
```csharp
from c in customers
join o in orders on c.CustomerID equals o.CustomerID
join d in details on o.OrderID equals d.OrderID
join p in products on d.ProductID equals p.ProductID
select new { c.Name, o.OrderDate, p.ProductName }
```
is translated into
```csharp
from * in customers.
    Join(orders, c => c.CustomerID, o => o.CustomerID, 
        (c, o) => new { c, o })
join d in details on o.OrderID equals d.OrderID
join p in products on d.ProductID equals p.ProductID
select new { c.Name, o.OrderDate, p.ProductName }
```
which is further reduced to
```csharp
customers.
Join(orders, c => c.CustomerID, o => o.CustomerID, (c, o) => new { c, o }).
Join(details, * => o.OrderID, d => d.OrderID, (*, d) => new { *, d }).
Join(products, * => d.ProductID, p => p.ProductID, (*, p) => new { *, p }).
Select(* => new { c.Name, o.OrderDate, p.ProductName })
```
the final translation of which is
```csharp
customers.
Join(orders, c => c.CustomerID, o => o.CustomerID,
    (c, o) => new { c, o }).
Join(details, x => x.o.OrderID, d => d.OrderID,
    (x, d) => new { x, d }).
Join(products, y => y.d.ProductID, p => p.ProductID,
    (y, p) => new { y, p }).
Select(z => new { z.y.x.c.Name, z.y.x.o.OrderDate, z.p.ProductName })
```
where `x`, `y`, and `z` are compiler generated identifiers that are otherwise invisible and inaccessible.

### The query expression pattern

The ***Query expression pattern*** establishes a pattern of methods that types can implement to support query expressions. Because query expressions are translated to method invocations by means of a syntactic mapping, types have considerable flexibility in how they implement the query expression pattern. For example, the methods of the pattern can be implemented as instance methods or as extension methods because the two have the same invocation syntax, and the methods can request delegates or expression trees because anonymous functions are convertible to both.

The recommended shape of a generic type `C<T>` that supports the query expression pattern is shown below. A generic type is used in order to illustrate the proper relationships between parameter and result types, but it is possible to implement the pattern for non-generic types as well.

```csharp
delegate R Func<T1,R>(T1 arg1);

delegate R Func<T1,T2,R>(T1 arg1, T2 arg2);

class C
{
    public C<T> Cast<T>();
}

class C<T> : C
{
    public C<T> Where(Func<T,bool> predicate);

    public C<U> Select<U>(Func<T,U> selector);

    public C<V> SelectMany<U,V>(Func<T,C<U>> selector,
        Func<T,U,V> resultSelector);

    public C<V> Join<U,K,V>(C<U> inner, Func<T,K> outerKeySelector,
        Func<U,K> innerKeySelector, Func<T,U,V> resultSelector);

    public C<V> GroupJoin<U,K,V>(C<U> inner, Func<T,K> outerKeySelector,
        Func<U,K> innerKeySelector, Func<T,C<U>,V> resultSelector);

    public O<T> OrderBy<K>(Func<T,K> keySelector);

    public O<T> OrderByDescending<K>(Func<T,K> keySelector);

    public C<G<K,T>> GroupBy<K>(Func<T,K> keySelector);

    public C<G<K,E>> GroupBy<K,E>(Func<T,K> keySelector,
        Func<T,E> elementSelector);
}

class O<T> : C<T>
{
    public O<T> ThenBy<K>(Func<T,K> keySelector);

    public O<T> ThenByDescending<K>(Func<T,K> keySelector);
}

class G<K,T> : C<T>
{
    public K Key { get; }
}
```

The methods above use the generic delegate types `Func<T1,R>` and `Func<T1,T2,R>`, but they could equally well have used other delegate or expression tree types with the same relationships in parameter and result types.

Notice the recommended relationship between `C<T>` and `O<T>` which ensures that the `ThenBy` and `ThenByDescending` methods are available only on the result of an `OrderBy` or `OrderByDescending`. Also notice the recommended shape of the result of `GroupBy` -- a sequence of sequences, where each inner sequence has an additional `Key` property.

The `System.Linq` namespace provides an implementation of the query operator pattern for any type that implements the `System.Collections.Generic.IEnumerable<T>` interface.

## Assignment operators

The assignment operators assign a new value to a variable, a property, an event, or an indexer element.

```antlr
assignment
    : unary_expression assignment_operator expression
    ;

assignment_operator
    : '='
    | '+='
    | '-='
    | '*='
    | '/='
    | '%='
    | '&='
    | '|='
    | '^='
    | '<<='
    | right_shift_assignment
    ;
```

The left operand of an assignment must be an expression classified as a variable, a property access, an indexer access, or an event access.

The `=` operator is called the ***simple assignment operator***. It assigns the value of the right operand to the variable, property, or indexer element given by the left operand. The left operand of the simple assignment operator may not be an event access (except as described in [Field-like events](classes.md#field-like-events)). The simple assignment operator is described in [Simple assignment](expressions.md#simple-assignment).

The assignment operators other than the `=` operator are called the ***compound assignment operators***. These operators perform the indicated operation on the two operands, and then assign the resulting value to the variable, property, or indexer element given by the left operand. The compound assignment operators are described in [Compound assignment](expressions.md#compound-assignment).

The `+=` and `-=` operators with an event access expression as the left operand are called the *event assignment operators*. No other assignment operator is valid with an event access as the left operand. The event assignment operators are described in [Event assignment](expressions.md#event-assignment).

The assignment operators are right-associative, meaning that operations are grouped from right to left. For example, an expression of the form `a = b = c` is evaluated as `a = (b = c)`.

### Simple assignment

The `=` operator is called the simple assignment operator.

If the left operand of a simple assignment is of the form `E.P` or `E[Ei]` where `E` has the compile-time type `dynamic`, then the assignment is dynamically bound ([Dynamic binding](expressions.md#dynamic-binding)). In this case the compile-time type of the assignment expression is `dynamic`, and the resolution described below will take place at run-time based on the run-time type of `E`.

In a simple assignment, the right operand must be an expression that is implicitly convertible to the type of the left operand. The operation assigns the value of the right operand to the variable, property, or indexer element given by the left operand.

The result of a simple assignment expression is the value assigned to the left operand. The result has the same type as the left operand and is always classified as a value.

If the left operand is a property or indexer access, the property or indexer must have a `set` accessor. If this is not the case, a binding-time error occurs.

The run-time processing of a simple assignment of the form `x = y` consists of the following steps:

*  If `x` is classified as a variable:
   * `x` is evaluated to produce the variable.
   * `y` is evaluated and, if required, converted to the type of `x` through an implicit conversion ([Implicit conversions](conversions.md#implicit-conversions)).
   * If the variable given by `x` is an array element of a *reference_type*, a run-time check is performed to ensure that the value computed for `y` is compatible with the array instance of which `x` is an element. The check succeeds if `y` is `null`, or if an implicit reference conversion ([Implicit reference conversions](conversions.md#implicit-reference-conversions)) exists from the actual type of the instance referenced by `y` to the actual element type of the array instance containing `x`. Otherwise, a `System.ArrayTypeMismatchException` is thrown.
   * The value resulting from the evaluation and conversion of `y` is stored into the location given by the evaluation of `x`.
*  If `x` is classified as a property or indexer access:
   * The instance expression (if `x` is not `static`) and the argument list (if `x` is an indexer access) associated with `x` are evaluated, and the results are used in the subsequent `set` accessor invocation.
   * `y` is evaluated and, if required, converted to the type of `x` through an implicit conversion ([Implicit conversions](conversions.md#implicit-conversions)).
   * The `set` accessor of `x` is invoked with the value computed for `y` as its `value` argument.

The array co-variance rules ([Array covariance](arrays.md#array-covariance)) permit a value of an array type `A[]` to be a reference to an instance of an array type `B[]`, provided an implicit reference conversion exists from `B` to `A`. Because of these rules, assignment to an array element of a *reference_type* requires a run-time check to ensure that the value being assigned is compatible with the array instance. In the example
```csharp
string[] sa = new string[10];
object[] oa = sa;

oa[0] = null;               // Ok
oa[1] = "Hello";            // Ok
oa[2] = new ArrayList();    // ArrayTypeMismatchException
```
the last assignment causes a `System.ArrayTypeMismatchException` to be thrown because an instance of `ArrayList` cannot be stored in an element of a `string[]`.

When a property or indexer declared in a *struct_type* is the target of an assignment, the instance expression associated with the property or indexer access must be classified as a variable. If the instance expression is classified as a value, a binding-time error occurs. Because of [Member access](expressions.md#member-access), the same rule also applies to fields.

Given the declarations:
```csharp
struct Point
{
    int x, y;

    public Point(int x, int y) {
        this.x = x;
        this.y = y;
    }

    public int X {
        get { return x; }
        set { x = value; }
    }

    public int Y {
        get { return y; }
        set { y = value; }
    }
}

struct Rectangle
{
    Point a, b;

    public Rectangle(Point a, Point b) {
        this.a = a;
        this.b = b;
    }

    public Point A {
        get { return a; }
        set { a = value; }
    }

    public Point B {
        get { return b; }
        set { b = value; }
    }
}
```
in the example
```csharp
Point p = new Point();
p.X = 100;
p.Y = 100;
Rectangle r = new Rectangle();
r.A = new Point(10, 10);
r.B = p;
```
the assignments to `p.X`, `p.Y`, `r.A`, and `r.B` are permitted because `p` and `r` are variables. However, in the example
```csharp
Rectangle r = new Rectangle();
r.A.X = 10;
r.A.Y = 10;
r.B.X = 100;
r.B.Y = 100;
```
the assignments are all invalid, since `r.A` and `r.B` are not variables.

> __Expression Tree Conversion Translation Steps__
>
> Conversion of simple assignment expressions to query expression trees produces a compile-time error.
>
> When converted to a generalized expression tree, a simple assignment expression of the form
>
> ```csharp
> a = b
> ```
>
> proceeds by optionally rewriting the expression to
>
> ```csharp
> a = (T)b
> ```
>
> if the right-hand side expression `b` requires an implicit conversion to type `T` as described in the section above. The introduced cast expression is classified as compiler-generated.
>
> Expression tree conversion proceeds by converting the left-hand side `a` to an expression tree `aExpr`, and by converting the right-hand side `b` or `(T)b` to an expression tree `bExpr`. The resulting expressions are then combined into
>
> ```csharp
> Q.Assign(info, aExpr, bExpr)
> ```
>
> where `info` is
>
> ```csharp
> Q.AssignInfo(flags)
> ```
>
> where `flags` is
>
> * `Q.Flags.ResultDiscarded` if the expression occurs in an *expression_statement* ([Expression statements](statements.md#expression-statements)), or,
> * `default(Q.Flags)` otherwise.
>
> ***TODO***
> * Review the need for `info` nodes when no additional info is needed, with an eye on future extensions (or do we evolve by adding new factory methods only?).
> * An alternative option could be to avoid rewriting `b` to `(T)b` but parameterize `AssignInfo` on a `ConvertInfo` describing the conversion applied to the right-hand side. This gets more tricky for compound assignment which may involve a conversion applied to the left-hand side as well (but we could have two such converts if needed).

### Compound assignment

If the left operand of a compound assignment is of the form `E.P` or `E[Ei]` where `E` has the compile-time type `dynamic`, then the assignment is dynamically bound ([Dynamic binding](expressions.md#dynamic-binding)). In this case the compile-time type of the assignment expression is `dynamic`, and the resolution described below will take place at run-time based on the run-time type of `E`.

An operation of the form `x op= y` is processed by applying binary operator overload resolution ([Binary operator overload resolution](expressions.md#binary-operator-overload-resolution)) as if the operation was written `x op y`. Then,

*  If the return type of the selected operator is implicitly convertible to the type of `x`, the operation is evaluated as `x = x op y`, except that `x` is evaluated only once.
*  Otherwise, if the selected operator is a predefined operator, if the return type of the selected operator is explicitly convertible to the type of `x`, and if `y` is implicitly convertible to the type of `x` or the operator is a shift operator, then the operation is evaluated as `x = (T)(x op y)`, where `T` is the type of `x`, except that `x` is evaluated only once.
*  Otherwise, the compound assignment is invalid, and a binding-time error occurs.

The term "evaluated only once" means that in the evaluation of `x op y`, the results of any constituent expressions of `x` are temporarily saved and then reused when performing the assignment to `x`. For example, in the assignment `A()[B()] += C()`, where `A` is a method returning `int[]`, and `B` and `C` are methods returning `int`, the methods are invoked only once, in the order `A`, `B`, `C`.

When the left operand of a compound assignment is a property access or indexer access, the property or indexer must have both a `get` accessor and a `set` accessor. If this is not the case, a binding-time error occurs.

The second rule above permits `x op= y` to be evaluated as `x = (T)(x op y)` in certain contexts. The rule exists such that the predefined operators can be used as compound operators when the left operand is of type `sbyte`, `byte`, `short`, `ushort`, or `char`. Even when both arguments are of one of those types, the predefined operators produce a result of type `int`, as described in [Binary numeric promotions](expressions.md#binary-numeric-promotions). Thus, without a cast it would not be possible to assign the result to the left operand.

The intuitive effect of the rule for predefined operators is simply that `x op= y` is permitted if both of `x op y` and `x = y` are permitted. In the example
```csharp
byte b = 0;
char ch = '\0';
int i = 0;

b += 1;             // Ok
b += 1000;          // Error, b = 1000 not permitted
b += i;             // Error, b = i not permitted
b += (byte)i;       // Ok

ch += 1;            // Error, ch = 1 not permitted
ch += (char)1;      // Ok
```
the intuitive reason for each error is that a corresponding simple assignment would also have been an error.

This also means that compound assignment operations support lifted operations. In the example
```csharp
int? i = 0;
i += 1;             // Ok
```
the lifted operator `+(int?,int?)` is used.

> __Expression Tree Conversion Translation Steps__
>
> Conversion of compound assignment expressions to query expression trees produces a compile-time error.
>
> When converted to a generalized expression tree, a compound assignment expression of the form
>
> ```csharp
> x op= y
> ```
>
> proceeds by first defining an identifier `M` with value
>
> * `Add` if `op` is `+`, or,
> * `Subtract` if `op` is `-`, or,
> * `Multiply` if `op` is `*`, or,
> * `Divide` if `op` is `/`, or,
> * `Modulo` if `op` is `%`, or,
> * `And` if `op` is `&`, or,
> * `Or` if `op` is `|`, or,
> * `ExclusiveOr` if `op` is `^`, or,
> * `LeftShift` if `op` is `<<`, or,
> * `RightShift` if `op` is `>>`,
>
> and by converting `x` to an expression tree `xExpr`. Conversion of the right-hand side of the expression to an expression tree `yExpr` is done by converting
>
> ```csharp
> (T)y
> ```
>
> to an expression tree, if `y` needs an implicit conversion to the type `T` of `x`, where the cast expression is classified as compiler-generated, or by converting
>
> ```csharp
> y
> ```
>
> to an expression tree, otherwise.
>
> Next, the steps for converting the expression `x op y` to an expression tree (see various sections of this specification) are performed to construct a method invocation expression `info` that invokes `Q.MInfo` which describes the underlying operation.
>
> Next, an expression `leftConversion` is constructed by checking whether the evaluation of `x + y` requires an implicit conversion of `x` to some type `T`. If such a conversion is needed, `leftConversion` is defined as the result of calling `Q.ConvertInfo` using the expression tree conversion rules for `(T)x` to an expression tree. Otherwise, `leftConversion` is the `default` literal.
>
> Next, an expression `finalConversion` is constructed by checking whether the evaluation of `x + y` requires an implicit conversion to some type `T` prior to assignment to `x`. If such a conversion is needed, `finalConversion` is defined as the result of calling `Q.ConvertInfo` using the expression tree conversion rules for `(T)(x op y)` to an expression tree. Otherwise, `finalConversion` is the `default` literal.
>
> Note that the resulting `info`, `leftConversion`, and `finalConversion` expressions can describe any of the following behaviors:
>
> * The use of a user-defined operator referenced using a `MethodInfo`.
> * The use of checked arithmetic indicated by a `Q.Flags.CheckedContext` flag.
> * The use of nullable lifting indicated by a `Q.Flags.IsLifted` flag.
> * The use of dynamic binding specified through a `binder` object.
>
> Finally, the resulting expressions are then combined into
>
> ```csharp
> Q.MAssign(info, xExpr, yExpr)
> ```
>
> where `info` is
>
> ```csharp
> Q.MAssignInfo(flags, leftConversion, finalConversion, isEventBinder)
> ```
>
> if the `op` is `+` or `-` and the compound assignment is dynamically bound, where `isEventBinder` is an object representing the dynamic operation required to check whether `x` represents an event access, or
>
> ```csharp
> Q.MAssignInfo(flags, leftConversion, finalConversion)
> ```
>
> otherwise, where `flags` is the bitwise `|` combination of
>
> * `Q.Flags.ResultDiscarded` if the expression occurs in an *expression_statement* ([Expression statements](statements.md#expression-statements)), and,
> * `Q.Flags.CheckedContext` if the expression occurs in a checked context, or,
> * `default(Q.Flags)` if none of the flags apply.
>
> Note that `leftConversion` and `finalConversion` may be `default` literals, so the expression tree library should define `MAssignInfo` methods such that target typing results in no ambiguity during overload resolution.

### Event assignment

If the left operand of a `+=` or `-=` operator is classified as an event access, then the expression is evaluated as follows:

*  The instance expression, if any, of the event access is evaluated.
*  The right operand of the `+=` or `-=` operator is evaluated, and, if required, converted to the type of the left operand through an implicit conversion ([Implicit conversions](conversions.md#implicit-conversions)).
*  An event accessor of the event is invoked, with argument list consisting of the right operand, after evaluation and, if necessary, conversion. If the operator was `+=`, the `add` accessor is invoked; if the operator was `-=`, the `remove` accessor is invoked.

An event assignment expression does not yield a value. Thus, an event assignment expression is valid only in the context of a *statement_expression* ([Expression statements](statements.md#expression-statements)).

> __Expression Tree Conversion Translation Steps__
>
> Conversion of event assignment expressions to query expression trees produces a compile-time error.
>
> When converted to a generalized expression tree, an event assignment expression of the form
>
> ```csharp
> x op= y
> ```
>
> proceeds by first defining an identifier `M` with value
>
> * `Add` if `op` is `+`, or,
> * `Subtract` if `op` is `-`,
>
> and by converting `instance` to an expression tree `instanceExpr` if `x` is an event access `instance.E` with an instance expression. Conversion of the right-hand side of the expression to an expression tree `yExpr` is done by converting
>
> ```csharp
> (T)y
> ```
>
> to an expression tree, if `y` needs an implicit conversion to the type `T` of `x`, where the cast expression is classified as compiler-generated, or by converting
>
> ```csharp
> y
> ```
>
> to an expression tree, otherwise.
>
> The expression is then translated into
>
> ```csharp
> Q.EventM(info, instanceExpr, yExpr)
> ```
>
> if `instanceExpr` is defined above, or into
>
> ```csharp
> Q.EventM(info, yExpr)
> ```
>
> otherwise, where `info` is
>
> ```csharp
> Q.EventMInfo(default(Q.Flags), method)
> ```
>
> where the flags parameter is reserved for future use, and `method` is an expression of type `MethodInfo` representing the `add` or `remove` accessor of the target event.
>
> Note that dynamically bound event assignment operations are covered by the rules for [Compound assignment](#compound-assignment).

## Expression

An *expression* is either a *non_assignment_expression* or an *assignment*.

```antlr
expression
    : non_assignment_expression
    | assignment
    ;

non_assignment_expression
    : conditional_expression
    | lambda_expression
    | query_expression
    ;
```

## Constant expressions

A *constant_expression* is an expression that can be fully evaluated at compile-time.

```antlr
constant_expression
    : expression
    ;
```

A constant expression must be the `null` literal or a value with one of the following types: `sbyte`, `byte`, `short`, `ushort`, `int`, `uint`, `long`, `ulong`, `char`, `float`, `double`, `decimal`, `bool`, `object`, `string`, or any enumeration type. Only the following constructs are permitted in constant expressions:

*  Literals (including the `null` literal).
*  References to `const` members of class and struct types.
*  References to members of enumeration types.
*  References to `const` parameters or local variables
*  Parenthesized sub-expressions, which are themselves constant expressions.
*  Cast expressions, provided the target type is one of the types listed above.
*  `checked` and `unchecked` expressions
*  Default value expressions
*  Nameof expressions
*  The predefined `+`, `-`, `!`, and `~` unary operators.
*  The predefined `+`, `-`, `*`, `/`, `%`, `<<`, `>>`, `&`, `|`, `^`, `&&`, `||`, `==`, `!=`, `<`, `>`, `<=`, and `>=` binary operators, provided each operand is of a type listed above.
*  The `?:` conditional operator.

The following conversions are permitted in constant expressions:

*  Identity conversions
*  Numeric conversions
*  Enumeration conversions
*  Constant expression conversions
*  Implicit and explicit reference conversions, provided that the source of the conversions is a constant expression that evaluates to the null value.

Other conversions including boxing, unboxing and implicit reference conversions of non-null values are not permitted in constant expressions. For example:
```csharp
class C {
    const object i = 5;         // error: boxing conversion not permitted
    const object str = "hello"; // error: implicit reference conversion
}
```
the initialization of i is an error because a boxing conversion is required. The initialization of str is an error because an implicit reference conversion from a non-null value is required.

Whenever an expression fulfills the requirements listed above, the expression is evaluated at compile-time. This is true even if the expression is a sub-expression of a larger expression that contains non-constant constructs.

The compile-time evaluation of constant expressions uses the same rules as run-time evaluation of non-constant expressions, except that where run-time evaluation would have thrown an exception, compile-time evaluation causes a compile-time error to occur.

Unless a constant expression is explicitly placed in an `unchecked` context, overflows that occur in integral-type arithmetic operations and conversions during the compile-time evaluation of the expression always cause compile-time errors ([Constant expressions](expressions.md#constant-expressions)).

Constant expressions occur in the contexts listed below. In these contexts, a compile-time error occurs if an expression cannot be fully evaluated at compile-time.

*  Constant declarations ([Constants](classes.md#constants)).
*  Enumeration member declarations ([Enum members](enums.md#enum-members)).
*  Default arguments of formal parameter lists ([Method parameters](classes.md#method-parameters))
*  `case` labels of a `switch` statement ([The switch statement](statements.md#the-switch-statement)).
*  `goto case` statements ([The goto statement](statements.md#the-goto-statement)).
*  Dimension lengths in an array creation expression ([Array creation expressions](expressions.md#array-creation-expressions)) that includes an initializer.
*  Attributes ([Attributes](attributes.md)).

An implicit constant expression conversion ([Implicit constant expression conversions](conversions.md#implicit-constant-expression-conversions)) permits a constant expression of type `int` to be converted to `sbyte`, `byte`, `short`, `ushort`, `uint`, or `ulong`, provided the value of the constant expression is within the range of the destination type.

> __Expression Tree Conversion Translation Steps__
>
> Conversion of constant expressions to query expression trees proceeds as follows:
>
> ```csharp
> Expression.Constant(expr, type)
> ```
>
> where `expr` is the original constant expression, and `type` is an expression of type `Type` representing the type of the constant expression. By applying the rules described in this section, this results in compile-time evaluation of `expr` to a constant value, or a compile-time exception.
>
> Conversion of constant expressions to generalized expression trees suppresses compile-time evaluation and constructs an expression tree out of the underlying expression.
>
> ***TODO***
> * Review the compile-time evaluation suppression for generalized expression trees, which may be contentious.
>   * There is prior art of this, e.g. in F#.
>   * This approach results in higher fidelity WYSIWYG expression trees.
>   * Reduced runtime performance can be a concern:
>     * More expression tree nodes get constructed.
>     * Naive expression tree evaluation may skip constant folding steps.
>     * One option would be for an expression type to be configurable, e.g. support a flags enum value on the `ExpressionFactoryAttribute` attribute.
>       * One such flag could then control compile-time evaluation of constant expressions.
>       * We should proceed with care and avoid a 747 cockpit of flags, but not prematurely rule out use of flags and think of other cases that may warrant such knobs. The distinction of typed versus untyped quotes in # is a good example, though it's not a declaration site thing (on the quote type) but a use site choice (`<@...@>` versus `<@@..@@>`).

## Boolean expressions

A *boolean_expression* is an expression that yields a result of type `bool`; either directly or through application of `operator true` in certain contexts as specified in the following.

```antlr
boolean_expression
    : expression
    ;
```

The controlling conditional expression of an *if_statement* ([The if statement](statements.md#the-if-statement)), *while_statement* ([The while statement](statements.md#the-while-statement)), *do_statement* ([The do statement](statements.md#the-do-statement)), or *for_statement* ([The for statement](statements.md#the-for-statement)) is a *boolean_expression*. The controlling conditional expression of the `?:` operator ([Conditional operator](expressions.md#conditional-operator)) follows the same rules as a *boolean_expression*, but for reasons of operator precedence is classified as a *conditional_or_expression*.

A *boolean_expression* `E` is required to be able to produce a value of type `bool`, as follows:

*  If `E` is implicitly convertible to `bool` then at runtime that implicit conversion is applied.
*  Otherwise, unary operator overload resolution ([Unary operator overload resolution](expressions.md#unary-operator-overload-resolution)) is used to find a unique best implementation of operator `true` on `E`, and that implementation is applied at runtime.
*  If no such operator is found, a binding-time error occurs.

The `DBBool` struct type in [Database boolean type](structs.md#database-boolean-type) provides an example of a type that implements `operator true` and `operator false`.

> __Expression Tree Conversion Translation Steps__
>
> A Boolean expression
>
> ```csharp
> b
> ```
>
> is translated into an expression tree by first pre-processing the expression to
>
> ```csharp
> (bool)b
> ```
>
> if `b` is implicitly convertible to `bool`. Note that this pre-processing step may introduce a cast expression which will be classified as compiler-generated.
>
> The expression is then translated into an expression tree by first translating the resulting expression to `bExpr`. Expression tree translation then proceeds as follows.
>
> When subject to conversion to a query expression tree, the expression is translated into
>
> ```csharp
> Expression.Call(null, method, bExpr)
> ```
>
> if the operator is bound to a user-defined `operator true`, where `method` is an expression of type `MethodInfo` representing the method implementing `operator true`, or into
>
> ```csharp
> bExpr
> ```
>
> otherwise.
>
> Note that query expression trees do not bind to the `Expression.IsTrue` factory method for reasons of compatibility. An example of a case where an Boolean expression can occur in a query expression tree is use of the conditional `?:` operator with a condition of type `DBBool` (see [Database boolean type](structs.md#database-boolean-type)).
>
> When subject to conversion to a generalized expression tree it is translated into
>
> ```csharp
> Q.IsTrue(info, bExpr)
> ```
>
> where `info` is
>
> ```csharp
> Q.IsTrueInfo(Q.Flags.CompilerGenerated, method)
> ```
>
> if the operator is bound to a user-defined `operator true`, where `method` is an expression of type `MethodInfo` representing the method implementing `operator true`, or into
>
> ```csharp
> bExpr
> ```
>
> otherwise, which may include a `Convert` node marked as `Q.Flags.CompilerGenerated`.