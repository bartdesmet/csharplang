# Statements

C# provides a variety of statements. Most of these statements will be familiar to developers who have programmed in C and C++.

```antlr
statement
    : labeled_statement
    | declaration_statement
    | embedded_statement
    ;

embedded_statement
    : block
    | empty_statement
    | expression_statement
    | selection_statement
    | iteration_statement
    | jump_statement
    | try_statement
    | checked_statement
    | unchecked_statement
    | lock_statement
    | using_statement
    | yield_statement
    | embedded_statement_unsafe
    ;
```

The *embedded_statement* nonterminal is used for statements that appear within other statements. The use of *embedded_statement* rather than *statement* excludes the use of declaration statements and labeled statements in these contexts. The example
```csharp
void F(bool b) {
    if (b)
        int i = 44;
}
```
results in a compile-time error because an `if` statement requires an *embedded_statement* rather than a *statement* for its if branch. If this code were permitted, then the variable `i` would be declared, but it could never be used. Note, however, that by placing `i`'s declaration in a block, the example is valid.

## End points and reachability

Every statement has an ***end point***. In intuitive terms, the end point of a statement is the location that immediately follows the statement. The execution rules for composite statements (statements that contain embedded statements) specify the action that is taken when control reaches the end point of an embedded statement. For example, when control reaches the end point of a statement in a block, control is transferred to the next statement in the block.

If a statement can possibly be reached by execution, the statement is said to be ***reachable***. Conversely, if there is no possibility that a statement will be executed, the statement is said to be ***unreachable***.

In the example
```csharp
void F() {
    Console.WriteLine("reachable");
    goto Label;
    Console.WriteLine("unreachable");
    Label:
    Console.WriteLine("reachable");
}
```
the second invocation of `Console.WriteLine` is unreachable because there is no possibility that the statement will be executed.

A warning is reported if the compiler determines that a statement is unreachable. It is specifically not an error for a statement to be unreachable.

To determine whether a particular statement or end point is reachable, the compiler performs flow analysis according to the reachability rules defined for each statement. The flow analysis takes into account the values of constant expressions ([Constant expressions](expressions.md#constant-expressions)) that control the behavior of statements, but the possible values of non-constant expressions are not considered. In other words, for purposes of control flow analysis, a non-constant expression of a given type is considered to have any possible value of that type.

In the example
```csharp
void F() {
    const int i = 1;
    if (i == 2) Console.WriteLine("unreachable");
}
```
the boolean expression of the `if` statement is a constant expression because both operands of the `==` operator are constants. As the constant expression is evaluated at compile-time, producing the value `false`, the `Console.WriteLine` invocation is considered unreachable. However, if `i` is changed to be a local variable
```csharp
void F() {
    int i = 1;
    if (i == 2) Console.WriteLine("reachable");
}
```
the `Console.WriteLine` invocation is considered reachable, even though, in reality, it will never be executed.

The *block* of a function member is always considered reachable. By successively evaluating the reachability rules of each statement in a block, the reachability of any given statement can be determined.

In the example
```csharp
void F(int x) {
    Console.WriteLine("start");
    if (x < 0) Console.WriteLine("negative");
}
```
the reachability of the second `Console.WriteLine` is determined as follows:

*  The first `Console.WriteLine` expression statement is reachable because the block of the `F` method is reachable.
*  The end point of the first `Console.WriteLine` expression statement is reachable because that statement is reachable.
*  The `if` statement is reachable because the end point of the first `Console.WriteLine` expression statement is reachable.
*  The second `Console.WriteLine` expression statement is reachable because the boolean expression of the `if` statement does not have the constant value `false`.

There are two situations in which it is a compile-time error for the end point of a statement to be reachable:

*  Because the `switch` statement does not permit a switch section to "fall through" to the next switch section, it is a compile-time error for the end point of the statement list of a switch section to be reachable. If this error occurs, it is typically an indication that a `break` statement is missing.
*  It is a compile-time error for the end point of the block of a function member that computes a value to be reachable. If this error occurs, it typically is an indication that a `return` statement is missing.

## Blocks

A *block* permits multiple statements to be written in contexts where a single statement is allowed.

```antlr
block
    : '{' statement_list? '}'
    ;
```

A *block* consists of an optional *statement_list* ([Statement lists](statements.md#statement-lists)), enclosed in braces. If the statement list is omitted, the block is said to be empty.

A block may contain declaration statements ([Declaration statements](statements.md#declaration-statements)). The scope of a local variable or constant declared in a block is the block.

A block is executed as follows:

*  If the block is empty, control is transferred to the end point of the block.
*  If the block is not empty, control is transferred to the statement list. When and if control reaches the end point of the statement list, control is transferred to the end point of the block.

The statement list of a block is reachable if the block itself is reachable.

The end point of a block is reachable if the block is empty or if the end point of the statement list is reachable.

A *block* that contains one or more `yield` statements ([The yield statement](statements.md#the-yield-statement)) is called an iterator block. Iterator blocks are used to implement function members as iterators ([Iterators](classes.md#iterators)). Some additional restrictions apply to iterator blocks:

*  It is a compile-time error for a `return` statement to appear in an iterator block (but `yield return` statements are permitted).
*  It is a compile-time error for an iterator block to contain an unsafe context ([Unsafe contexts](unsafe-code.md#unsafe-contexts)). An iterator block always defines a safe context, even when its declaration is nested in an unsafe context.

> __Expression Tree Conversion Translation Steps__
>
> Translation of a *block* to a query expression tree produces a compile-time error. When converted to a generalized expression tree, and the block is an iterator block, a compile-time error occurs. Otherwise, it is translated into:
>
> ```csharp
> Q.Block(info)
> ```
>
> if the block is empty, or,
>
> ```csharp
> Q.Block(info, stmtsExpr)
> ```
>
> otherwise, where `stmtsExpr` is the result of converting the statement list to an expression tree, and `info` is
>
> ```csharp
> Q.BlockInfo(default(Q.Flags))
> ```
>
> if the block does not contain any variable declarations, or
>
> ```csharp
> Q.BlockInfo(default(Q.Flags), scope)
> ```
>
> otherwise, where `scope` is an expression obtained using the rules in [Scopes in expression trees](#scopes-in-expression-trees).
>
> ***TODO***
> * Iterators are currently not supported in anonymous function expressions. If this restriction gets lifted, a `Q.IteratorBlockInfo` could be used for `info`, reusing the same node, or we could resort to using a flag.
> * Refine language on the block containing variable declarations to be more precise. In particular, we don't want to imply any form of recursion into child nodes. We likely need to specifically call out all the nodes that contribute a variable to the block's scope (to be extended in C# 7 due to `out` variables, deconstruction, etc.).

### Statement lists

A ***statement list*** consists of one or more statements written in sequence. Statement lists occur in *block*s ([Blocks](statements.md#blocks)) and in *switch_block*s ([The switch statement](statements.md#the-switch-statement)).

```antlr
statement_list
    : statement+
    ;
```

A statement list is executed by transferring control to the first statement. When and if control reaches the end point of a statement, control is transferred to the next statement. When and if control reaches the end point of the last statement, control is transferred to the end point of the statement list.

A statement in a statement list is reachable if at least one of the following is true:

*  The statement is the first statement and the statement list itself is reachable.
*  The end point of the preceding statement is reachable.
*  The statement is a labeled statement and the label is referenced by a reachable `goto` statement.

The end point of a statement list is reachable if the end point of the last statement in the list is reachable.

> __Expression Tree Conversion Translation Steps__
>
> Translation of a *statement_list* to a query expression tree produces a compile-time error. When converted to a generalized expression tree, it is translated into:
>
> ```csharp
> Q.StatementList(info, stmtExpr1, ..., stmtExprN)
> ```
>
> where `stmtExpr1` to `stmtExprN` are the result of translating the statements to expression tree, and `info` is
>
> ```csharp
> Q.StatementListInfo(default(Q.Flags))
> ```

## The empty statement

An *empty_statement* does nothing.

```antlr
empty_statement
    : ';'
    ;
```

An empty statement is used when there are no operations to perform in a context where a statement is required.

Execution of an empty statement simply transfers control to the end point of the statement. Thus, the end point of an empty statement is reachable if the empty statement is reachable.

An empty statement can be used when writing a `while` statement with a null body:
```csharp
bool ProcessMessage() {...}

void ProcessMessages() {
    while (ProcessMessage())
        ;
}
```

Also, an empty statement can be used to declare a label just before the closing "`}`" of a block:
```csharp
void F() {
    ...
    if (done) goto exit;
    ...
    exit: ;
}
```

> __Expression Tree Conversion Translation Steps__
>
> Translation of an *empty_statement* to a query expression tree produces a compile-time error. When converted to a generalized expression tree, it is translated into:
>
> ```csharp
> Q.Empty(info)
> ```
>
> where `info` is
>
> ```csharp
> Q.EmptyInfo(default(Q.Flags))
> ```
>
> ***TODO***
> * Review the use of an info node. Consistent on the one hand, verbose on the other. Support for annotations may do away with this.

## Labeled statements

A *labeled_statement* permits a statement to be prefixed by a label. Labeled statements are permitted in blocks, but are not permitted as embedded statements.

```antlr
labeled_statement
    : identifier ':' statement
    ;
```

A labeled statement declares a label with the name given by the *identifier*. The scope of a label is the whole block in which the label is declared, including any nested blocks. It is a compile-time error for two labels with the same name to have overlapping scopes.

A label can be referenced from `goto` statements ([The goto statement](statements.md#the-goto-statement)) within the scope of the label. This means that `goto` statements can transfer control within blocks and out of blocks, but never into blocks.

Labels have their own declaration space and do not interfere with other identifiers. The example
```csharp
int F(int x) {
    if (x >= 0) goto x;
    x = -x;
    x: return x;
}
```
is valid and uses the name `x` as both a parameter and a label.

Execution of a labeled statement corresponds exactly to execution of the statement following the label.

In addition to the reachability provided by normal flow of control, a labeled statement is reachable if the label is referenced by a reachable `goto` statement. (Exception: If a `goto` statement is inside a `try` that includes a `finally` block, and the labeled statement is outside the `try`, and the end point of the `finally` block is unreachable, then the labeled statement is not reachable from that `goto` statement.)

## Declaration statements

A *declaration_statement* declares a local variable or constant. Declaration statements are permitted in blocks, but are not permitted as embedded statements.

```antlr
declaration_statement
    : local_variable_declaration ';'
    | local_constant_declaration ';'
    ;
```

### Local variable declarations

A *local_variable_declaration* declares one or more local variables.

```antlr
local_variable_declaration
    : local_variable_type local_variable_declarators
    ;

local_variable_type
    : type
    | 'var'
    ;

local_variable_declarators
    : local_variable_declarator
    | local_variable_declarators ',' local_variable_declarator
    ;

local_variable_declarator
    : identifier
    | identifier '=' local_variable_initializer
    ;

local_variable_initializer
    : expression
    | array_initializer
    | local_variable_initializer_unsafe
    ;
```

The *local_variable_type* of a *local_variable_declaration* either directly specifies the type of the variables introduced by the declaration, or indicates with the identifier `var` that the type should be inferred based on an initializer. The type is followed by a list of *local_variable_declarator*s, each of which introduces a new variable. A *local_variable_declarator* consists of an *identifier* that names the variable, optionally followed by an "`=`" token and a *local_variable_initializer* that gives the initial value of the variable.

In the context of a local variable declaration, the identifier var acts as a contextual keyword ([Keywords](lexical-structure.md#keywords)).When the *local_variable_type* is specified as `var` and no type named `var` is in scope, the declaration is an ***implicitly typed local variable declaration***, whose type is inferred from the type of the associated initializer expression. Implicitly typed local variable declarations are subject to the following restrictions:

*  The *local_variable_declaration* cannot include multiple *local_variable_declarator*s.
*  The *local_variable_declarator* must include a *local_variable_initializer*.
*  The *local_variable_initializer* must be an *expression*.
*  The initializer *expression* must have a compile-time type.
*  The initializer *expression* cannot refer to the declared variable itself

The following are examples of incorrect implicitly typed local variable declarations:

```csharp
var x;               // Error, no initializer to infer type from
var y = {1, 2, 3};   // Error, array initializer not permitted
var z = null;        // Error, null does not have a type
var u = x => x + 1;  // Error, anonymous functions do not have a type
var v = v++;         // Error, initializer cannot refer to variable itself
```

The value of a local variable is obtained in an expression using a *simple_name* ([Simple names](expressions.md#simple-names)), and the value of a local variable is modified using an *assignment* ([Assignment operators](expressions.md#assignment-operators)). A local variable must be definitely assigned ([Definite assignment](variables.md#definite-assignment)) at each location where its value is obtained.

The scope of a local variable declared in a *local_variable_declaration* is the block in which the declaration occurs. It is an error to refer to a local variable in a textual position that precedes the *local_variable_declarator* of the local variable. Within the scope of a local variable, it is a compile-time error to declare another local variable or constant with the same name.

A local variable declaration that declares multiple variables is equivalent to multiple declarations of single variables with the same type. Furthermore, a variable initializer in a local variable declaration corresponds exactly to an assignment statement that is inserted immediately after the declaration.

The example
```csharp
void F() {
    int x = 1, y, z = x * 2;
}
```
corresponds exactly to
```csharp
void F() {
    int x; x = 1;
    int y;
    int z; z = x * 2;
}
```

In an implicitly typed local variable declaration, the type of the local variable being declared is taken to be the same as the type of the expression used to initialize the variable. For example:
```csharp
var i = 5;
var s = "Hello";
var d = 1.0;
var numbers = new int[] {1, 2, 3};
var orders = new Dictionary<int,Order>();
```

The implicitly typed local variable declarations above are precisely equivalent to the following explicitly typed declarations:
```csharp
int i = 5;
string s = "Hello";
double d = 1.0;
int[] numbers = new int[] {1, 2, 3};
Dictionary<int,Order> orders = new Dictionary<int,Order>();
```

### Local constant declarations

A *local_constant_declaration* declares one or more local constants.

```antlr
local_constant_declaration
    : 'const' type constant_declarators
    ;

constant_declarators
    : constant_declarator (',' constant_declarator)*
    ;

constant_declarator
    : identifier '=' constant_expression
    ;
```

The *type* of a *local_constant_declaration* specifies the type of the constants introduced by the declaration. The type is followed by a list of *constant_declarator*s, each of which introduces a new constant. A *constant_declarator* consists of an *identifier* that names the constant, followed by an "`=`" token, followed by a *constant_expression* ([Constant expressions](expressions.md#constant-expressions)) that gives the value of the constant.

The *type* and *constant_expression* of a local constant declaration must follow the same rules as those of a constant member declaration ([Constants](classes.md#constants)).

The value of a local constant is obtained in an expression using a *simple_name* ([Simple names](expressions.md#simple-names)).

The scope of a local constant is the block in which the declaration occurs. It is an error to refer to a local constant in a textual position that precedes its *constant_declarator*. Within the scope of a local constant, it is a compile-time error to declare another local variable or constant with the same name.

A local constant declaration that declares multiple constants is equivalent to multiple declarations of single constants with the same type.

## Expression statements

An *expression_statement* evaluates a given expression. The value computed by the expression, if any, is discarded.

```antlr
expression_statement
    : statement_expression ';'
    ;

statement_expression
    : invocation_expression
    | null_conditional_invocation_expression
    | object_creation_expression
    | assignment
    | post_increment_expression
    | post_decrement_expression
    | pre_increment_expression
    | pre_decrement_expression
    | await_expression
    ;
```

Not all expressions are permitted as statements. In particular, expressions such as `x + y` and `x == 1` that merely compute a value (which will be discarded), are not permitted as statements.

Execution of an *expression_statement* evaluates the contained expression and then transfers control to the end point of the *expression_statement*. The end point of an *expression_statement* is reachable if that *expression_statement* is reachable.

> __Expression Tree Conversion Translation Steps__
>
> Translation of a *expression_statement* to a query expression tree produces a compile-time error. When converted to a generalized expression tree, it is translated into:
>
> ```csharp
> Q.StatementExpression(info, expr)
> ```
>
> where `expr` is the result of converting the *statement_expression* to an expression tree, and `info` is
>
> ```csharp
> Q.EmptyInfo(default(Q.Flags))
> ```
>
> Note that *statement_expression*s use `Q.Flags.ResultDiscarded` to denote that the result of evalating the expression is discarded. This information can be used for dynamic binding and to improve expression tree evaluation without having to manually track when results are discarded. A good example of this is code generation for `x++` and `x--` where the result of evaluating `x` prior to performing the increment or decrement operation can be discarded. The `StatementExpression` node retains the syntactic isomorphism between the original code and the generated expression tree, and also acts as a conversion to `void`.

## Selection statements

Selection statements select one of a number of possible statements for execution based on the value of some expression.

```antlr
selection_statement
    : if_statement
    | switch_statement
    ;
```

### The if statement

The `if` statement selects a statement for execution based on the value of a boolean expression.

```antlr
if_statement
    : 'if' '(' boolean_expression ')' embedded_statement
    | 'if' '(' boolean_expression ')' embedded_statement 'else' embedded_statement
    ;
```

An `else` part is associated with the lexically nearest preceding `if` that is allowed by the syntax. Thus, an `if` statement of the form
```csharp
if (x) if (y) F(); else G();
```
is equivalent to
```csharp
if (x) {
    if (y) {
        F();
    }
    else {
        G();
    }
}
```

An `if` statement is executed as follows:

*  The *boolean_expression* ([Boolean expressions](expressions.md#boolean-expressions)) is evaluated.
*  If the boolean expression yields `true`, control is transferred to the first embedded statement. When and if control reaches the end point of that statement, control is transferred to the end point of the `if` statement.
*  If the boolean expression yields `false` and if an `else` part is present, control is transferred to the second embedded statement. When and if control reaches the end point of that statement, control is transferred to the end point of the `if` statement.
*  If the boolean expression yields `false` and if an `else` part is not present, control is transferred to the end point of the `if` statement.

The first embedded statement of an `if` statement is reachable if the `if` statement is reachable and the boolean expression does not have the constant value `false`.

The second embedded statement of an `if` statement, if present, is reachable if the `if` statement is reachable and the boolean expression does not have the constant value `true`.

The end point of an `if` statement is reachable if the end point of at least one of its embedded statements is reachable. In addition, the end point of an `if` statement with no `else` part is reachable if the `if` statement is reachable and the boolean expression does not have the constant value `true`.

> __Expression Tree Conversion Translation Steps__
>
> Translation of an *if_statement* to a query expression tree produces a compile-time error. When converted to a generalized expression tree, it is translated into:
>
> ```csharp
> Q.If(info, conditionExpr, thenStmtExpr)
> ```
>
> if the statement does not have an `else` part, or
>
> ```csharp
> Q.If(info, conditionExpr, thenStmtExpr, elseStmtExpr)
> ```
>
> otherwise, where `conditionExpr` is the result if translating the boolean expression condition to an expression tree, `thenStmtExpr` is the result of translating the embedded statement immediately after the condition to an expression tree, `elseStmtExpr` is the result of translating the embedded statement following the `else` part to an expression tree, and `info` is
>
> ```csharp
> Q.IfInfo(default(Q.Flags))
> ```
>
> ***TODO***
> * Decide on the strategy for info nodes. Even though these may be redundant now, it provides extensibility if statements change in the future from a semantic fashion. For example, if they allow binding to APIs and start requiring reflection info objects. A good example is `Lambda` where async lambdas were easily modeled by defining a new info factory, while keeping the syntactic structure of the node the same. Note that expression tree libraries can overload factory methods `X` to vary the return types based on the type returned from `XInfo`. For example, `Q.Lambda<T>(Q.LambdaInfo(...), ...)` may return a `LambdaQuote<T>` while `Q.Lambda<T>(Q.AsyncLambdaInfo(...), ...)` may return an `AsyncLambdaQuote<T>` if the library wishes to do so.

### The switch statement

The switch statement selects for execution a statement list having an associated switch label that corresponds to the value of the switch expression.

```antlr
switch_statement
    : 'switch' '(' expression ')' switch_block
    ;

switch_block
    : '{' switch_section* '}'
    ;

switch_section
    : switch_label+ statement_list
    ;

switch_label
    : 'case' constant_expression ':'
    | 'default' ':'
    ;
```

A *switch_statement* consists of the keyword `switch`, followed by a parenthesized expression (called the switch expression), followed by a *switch_block*. The *switch_block* consists of zero or more *switch_section*s, enclosed in braces. Each *switch_section* consists of one or more *switch_label*s followed by a *statement_list* ([Statement lists](statements.md#statement-lists)).

The ***governing type*** of a `switch` statement is established by the switch expression.

*  If the type of the switch expression is `sbyte`, `byte`, `short`, `ushort`, `int`, `uint`, `long`, `ulong`, `bool`, `char`, `string`, or an *enum_type*, or if it is the nullable type corresponding to one of these types, then that is the governing type of the `switch` statement.
*  Otherwise, exactly one user-defined implicit conversion ([User-defined conversions](conversions.md#user-defined-conversions)) must exist from the type of the switch expression to one of the following possible governing types: `sbyte`, `byte`, `short`, `ushort`, `int`, `uint`, `long`, `ulong`, `char`, `string`, or,  a nullable type corresponding to one of those types.
*  Otherwise, if no such implicit conversion exists, or if more than one such implicit conversion exists, a compile-time error occurs.

The constant expression of each `case` label must denote a value that is implicitly convertible ([Implicit conversions](conversions.md#implicit-conversions)) to the governing type of the `switch` statement. A compile-time error occurs if two or more `case` labels in the same `switch` statement specify the same constant value.

There can be at most one `default` label in a switch statement.

A `switch` statement is executed as follows:

*  The switch expression is evaluated and converted to the governing type.
*  If one of the constants specified in a `case` label in the same `switch` statement is equal to the value of the switch expression, control is transferred to the statement list following the matched `case` label.
*  If none of the constants specified in `case` labels in the same `switch` statement is equal to the value of the switch expression, and if a `default` label is present, control is transferred to the statement list following the `default` label.
*  If none of the constants specified in `case` labels in the same `switch` statement is equal to the value of the switch expression, and if no `default` label is present, control is transferred to the end point of the `switch` statement.

If the end point of the statement list of a switch section is reachable, a compile-time error occurs. This is known as the "no fall through" rule. The example
```csharp
switch (i) {
case 0:
    CaseZero();
    break;
case 1:
    CaseOne();
    break;
default:
    CaseOthers();
    break;
}
```
is valid because no switch section has a reachable end point. Unlike C and C++, execution of a switch section is not permitted to "fall through" to the next switch section, and the example
```csharp
switch (i) {
case 0:
    CaseZero();
case 1:
    CaseZeroOrOne();
default:
    CaseAny();
}
```
results in a compile-time error. When execution of a switch section is to be followed by execution of another switch section, an explicit `goto case` or `goto default` statement must be used:
```csharp
switch (i) {
case 0:
    CaseZero();
    goto case 1;
case 1:
    CaseZeroOrOne();
    goto default;
default:
    CaseAny();
    break;
}
```

Multiple labels are permitted in a *switch_section*. The example
```csharp
switch (i) {
case 0:
    CaseZero();
    break;
case 1:
    CaseOne();
    break;
case 2:
default:
    CaseTwo();
    break;
}
```
is valid. The example does not violate the "no fall through" rule because the labels `case 2:` and `default:` are part of the same *switch_section*.

The "no fall through" rule prevents a common class of bugs that occur in C and C++ when `break` statements are accidentally omitted. In addition, because of this rule, the switch sections of a `switch` statement can be arbitrarily rearranged without affecting the behavior of the statement. For example, the sections of the `switch` statement above can be reversed without affecting the behavior of the statement:
```csharp
switch (i) {
default:
    CaseAny();
    break;
case 1:
    CaseZeroOrOne();
    goto default;
case 0:
    CaseZero();
    goto case 1;
}
```

The statement list of a switch section typically ends in a `break`, `goto case`, or `goto default` statement, but any construct that renders the end point of the statement list unreachable is permitted. For example, a `while` statement controlled by the boolean expression `true` is known to never reach its end point. Likewise, a `throw` or `return` statement always transfers control elsewhere and never reaches its end point. Thus, the following example is valid:
```csharp
switch (i) {
case 0:
    while (true) F();
case 1:
    throw new ArgumentException();
case 2:
    return;
}
```

The governing type of a `switch` statement may be the type `string`. For example:
```csharp
void DoCommand(string command) {
    switch (command.ToLower()) {
    case "run":
        DoRun();
        break;
    case "save":
        DoSave();
        break;
    case "quit":
        DoQuit();
        break;
    default:
        InvalidCommand(command);
        break;
    }
}
```

Like the string equality operators ([String equality operators](expressions.md#string-equality-operators)), the `switch` statement is case sensitive and will execute a given switch section only if the switch expression string exactly matches a `case` label constant.

When the governing type of a `switch` statement is `string`, the value `null` is permitted as a case label constant.

The *statement_list*s of a *switch_block* may contain declaration statements ([Declaration statements](statements.md#declaration-statements)). The scope of a local variable or constant declared in a switch block is the switch block.

The statement list of a given switch section is reachable if the `switch` statement is reachable and at least one of the following is true:

*  The switch expression is a non-constant value.
*  The switch expression is a constant value that matches a `case` label in the switch section.
*  The switch expression is a constant value that doesn't match any `case` label, and the switch section contains the `default` label.
*  A switch label of the switch section is referenced by a reachable `goto case` or `goto default` statement.

The end point of a `switch` statement is reachable if at least one of the following is true:

*  The `switch` statement contains a reachable `break` statement that exits the `switch` statement.
*  The `switch` statement is reachable, the switch expression is a non-constant value, and no `default` label is present.
*  The `switch` statement is reachable, the switch expression is a constant value that doesn't match any `case` label, and no `default` label is present.

> __Expression Tree Conversion Translation Steps__
>
> Translation of a *switch_statement* to a query expression tree produces a compile-time error. When converted to a generalized expression tree, the following steps are taken.
>
> ### The `switch` expression
>
> Given a *switch_statement*
>
> ```antlr
> switch_statement
>     : 'switch' '(' expression ')' switch_block
>     ;
> ```
>
> of the form `switch (e) ...`, construct an expression `switchExpr` by converting the switch expression `e` to an expression tree, if the type of the switch expression `e` matches the governing type, or by converting the expression `(T)(e)` where `T` is the governing type, otherwise. The introduced cast expression, if any, is classified as compiler-generated and represents the user-defined implicit conversion ([User-defined conversions](conversions.md#user-defined-conversions)) to a supported governing type.
>
> ### The `break` label
>
> Construct a variable `breakLabel` holding an expression tree node representing a compiler-generated label according to the rules described in [Labeled statements](#labeled-statements). This compiler-generated label acts as the target for a `break` statement ([The break statement](statements.md#the-break-statement)) that breaks out of the *switch_statement*.
>
> If the *switch_block* does not contain any *switch_section*s or none of the *switch_section*s contains a `break` statement ([The break statement](statements.md#the-break-statement)) that breaks out of the *switch_statement*, `breakLabel` is not constructed.
>
> ### The `switch` block
>
> Construct a `switchBlockExpr` expression for the *switch_block*
>
> ```antlr
> switch_block
>     : '{' switch_section* '}'
>     ;
> ```
>
> by invoking the following factory:
>
> ```csharp
> Q.SwitchBlock(info, switchSectionExpr_1, ..., switchSectionExpr_N)
> ```
>
> where `info` is
>
> ```csharp
> Q.SwitchBlockInfo(default(Q.Flags))
> ```
>
> and each `switchSectionExpr_i` expression (with `1 <= i <= N`) is constructed from the *switch_section*s that occur in the *switch_block*, in lexical order, if any exist, as described in the next section.
>
> ### The `switch` sections
>
> Construct each `switchSectionExpr_i` expression (with `1 <= i <= N`) from the `i`th *switch_section*
>
> ```antlr
> switch_section
>     : switch_label+ statement_list
>     ;
> ```
>
> in the parent *switch_block*, by constructing an expression `stmtExpr` from converting *statement_list* to an expression tree, and by constructing an expression `labelsExpr` as described in the next section. The switch section is then translated into
>
> ```csharp
> Q.SwitchSection(info, labelsExpr, stmtExpr)
> ```
>
> where `info` is
>
> ```csharp
> Q.SwitchSectionInfo(default(Q.Flags))
> ```
>
> ### The `case` and `default` labels
>
> Construct a `labelsExpr` expression for all the *switch_label*s
>
> ```antlr
> switch_label
>     : 'case' constant_expression ':'
>     | 'default' ':'
>     ;
> ```
>
> in a *switch_section* by invoking the following factory:
>
> ```csharp
> Q.SwitchLabels(info, switchLabelExpr_1, ..., switchLabelExpr_M)
> ```
>
> where `info` is
>
> ```csharp
> Q.SwitchLabelsInfo(default(Q.Flags))
> ```
>
> and each `switchLabelExpr_i` expression (with `1 <= i <= M`) is constructed from the *switch_label*s that occur in the *switch_section*, in lexical order, as described below.
>
> A `default` switch label is translated into
>
> ```csharp
> Q.SwitchLabelDefault(info)
> ```
>
> where `info` is
>
> ```csharp
> Q.SwitchLabelDefaultInfo(default(Q.Flags))
> ```
>
> if no `goto default` statement references the *switch_label*, or
>
> ```csharp
> Q.SwitchLabelDefaultInfo(default(Q.Flags), defaultLabel)
> ```
>
> otherwise, where `defaultLabel` is a variable holding an expression tree node representing a compiler-generated label according to the rules described in [Labeled statements](#labeled-statements). This compiler-generated label acts as the target for a `goto default` statement ([The goto statement](statements.md#the-goto-statement)) that jumps to the *switch_label*.
>
> A `case c` switch label is translated into
>
> ```csharp
> Q.SwitchLabelCase(info, constExpr)
> ```
>
> where `constExpr` is an expression constructed by converting the *constant_expression* `c` to an expression tree, if the type of `c` matches the governing type, or by converting the expression `(T)(c)` where `T` is the governing type, otherwise. The introduced cast expression, if any, is classified as compiler-generated and represents the user-defined implicit conversion ([User-defined conversions](conversions.md#user-defined-conversions)) to a supported governing type. 
>
> Furthermore, `info` is
>
> ```csharp
> Q.SwitchLabelCaseInfo(default(Q.Flags))
> ```
>
> if no `goto case` statement references the *switch_label*, or
>
> ```csharp
> Q.SwitchLabelCaseInfo(default(Q.Flags), caseLabel)
> ```
>
> otherwise, where `caseLabel` is a variable holding an expression tree node representing a compiler-generated label according to the rules described in [Labeled statements](#labeled-statements). This compiler-generated label acts as the target for a `goto case` statement ([The goto statement](statements.md#the-goto-statement)) that jumps to the *switch_label*.
>
> Note that the *constant_expression* specified on a `case` label is evaluated at compile time to allow for comparison of its value to any of the `goto case` statements' *constant_expression*. The relationship between a `case` label and a `goto case` statement is established by sharing a label node in the resulting expression tree. However, the expression tree for *constant_expression* is retained as well (both on the `case` label and on any `goto case` statement), either in its compile-time evaluated form, or in its raw form if compile-time evaluation is suppressed in expression trees. This enables expression tree libraries to inspect the original expressions and enables them to perform a variety of analyses and manipulations.
>
> ### Constructing the expression tree
>
> Given the constituents built using the steps described in the previous sections, the expression tree for the *switch_statement* is constructed as follows:
>
> ```csharp
> Q.Switch(info, switchExpr, switchBlockExpr)
> ```
>
> where `info` is
>
> ```csharp
> Q.SwitchInfo(default(Q.Flags), breakLabel)
> ```
>
> if `breakLabel` has been defined, or
>
> ```csharp
> Q.SwitchInfo(default(Q.Flags))
> ```
>
> otherwise.
>
> ***TODO***
> * C# 7 adds scoped locals to the bound node, due to declaration expressions. These would be passed to the `SwitchInfo` factory method as a third argument supplied with the result of the `ScopeInfo` factory method.

## Iteration statements

Iteration statements repeatedly execute an embedded statement.

```antlr
iteration_statement
    : while_statement
    | do_statement
    | for_statement
    | foreach_statement
    ;
```

### The while statement

The `while` statement conditionally executes an embedded statement zero or more times.

```antlr
while_statement
    : 'while' '(' boolean_expression ')' embedded_statement
    ;
```

A `while` statement is executed as follows:

*  The *boolean_expression* ([Boolean expressions](expressions.md#boolean-expressions)) is evaluated.
*  If the boolean expression yields `true`, control is transferred to the embedded statement. When and if control reaches the end point of the embedded statement (possibly from execution of a `continue` statement), control is transferred to the beginning of the `while` statement.
*  If the boolean expression yields `false`, control is transferred to the end point of the `while` statement.

Within the embedded statement of a `while` statement, a `break` statement ([The break statement](statements.md#the-break-statement)) may be used to transfer control to the end point of the `while` statement (thus ending iteration of the embedded statement), and a `continue` statement ([The continue statement](statements.md#the-continue-statement)) may be used to transfer control to the end point of the embedded statement (thus performing another iteration of the `while` statement).

The embedded statement of a `while` statement is reachable if the `while` statement is reachable and the boolean expression does not have the constant value `false`.

The end point of a `while` statement is reachable if at least one of the following is true:

*  The `while` statement contains a reachable `break` statement that exits the `while` statement.
*  The `while` statement is reachable and the boolean expression does not have the constant value `true`.

> __Expression Tree Conversion Translation Steps__
>
> Translation of a *while_statement* to a query expression tree produces a compile-time error. When converted to a generalized expression tree, it is translated into:
>
> ```csharp
> Q.While(info, conditionExpr, stmtExpr)
> ```
>
> where `conditionExpr` is the result if translating the boolean expression condition to an expression tree, `stmtExpr` is the result of translating the embedded statement, and `info` is
>
> ```csharp
> Q.WhileInfo(default(Q.Flags))
> ```
>
> if no `break` or `continue` statements exist within the body of the iteration statement, targeting the current iteration statement, or
>
> ```csharp
> Q.WhileInfo(default(Q.Flags), breakLabel)
> ```
>
> if a `break` statement exists but no `continue` statement exists within the body of the iteration statement, targeting the current iteration statement, or
>
> ```csharp
> Q.WhileInfo(default(Q.Flags), breakLabel, continueLabel)
> ```
>
> otherwise, where `breakLabel` and `continueLabel` refer to variables holding expression tree nodes representing compiler-generated labels according to the rules described in [Labeled statements](#labeled-statements). If no `break` statement exists within the body of the iteration statement, targeting the current iteration statement, `breakLabel` will be the `default` literal instead.
>
> The rationale of these overloads is to allow for efficient expression tree libraries that can detect control flow behavior associated with the loop, without having to perform complex analysis of the iteration statement body. Note that libraries can implement `WhileInfo` using two optional parameters rather than having three distinct overloads. For example:
>
> ```csharp
> static WhileInfo WhileInfo(Flags flags, LabelInfo @break = default, LabelInfo @continue = default) => ...
> ```
>
> ***TODO***
> * C# 7 adds scoped locals to the bound node, due to declaration expressions. These would be passed to the `WhileInfo` factory method as a fourth argument supplied with the result of the `ScopeInfo` factory method.

### The do statement

The `do` statement conditionally executes an embedded statement one or more times.

```antlr
do_statement
    : 'do' embedded_statement 'while' '(' boolean_expression ')' ';'
    ;
```

A `do` statement is executed as follows:

*  Control is transferred to the embedded statement.
*  When and if control reaches the end point of the embedded statement (possibly from execution of a `continue` statement), the *boolean_expression* ([Boolean expressions](expressions.md#boolean-expressions)) is evaluated. If the boolean expression yields `true`, control is transferred to the beginning of the `do` statement. Otherwise, control is transferred to the end point of the `do` statement.

Within the embedded statement of a `do` statement, a `break` statement ([The break statement](statements.md#the-break-statement)) may be used to transfer control to the end point of the `do` statement (thus ending iteration of the embedded statement), and a `continue` statement ([The continue statement](statements.md#the-continue-statement)) may be used to transfer control to the end point of the embedded statement.

The embedded statement of a `do` statement is reachable if the `do` statement is reachable.

The end point of a `do` statement is reachable if at least one of the following is true:

*  The `do` statement contains a reachable `break` statement that exits the `do` statement.
*  The end point of the embedded statement is reachable and the boolean expression does not have the constant value `true`.

> __Expression Tree Conversion Translation Steps__
>
> Translation of a *do_statement* to a query expression tree produces a compile-time error. When converted to a generalized expression tree, it is translated into:
>
> ```csharp
> Q.Do(info, stmtExpr, conditionExpr)
> ```
>
> where `stmtExpr` is the result of translating the embedded statement, `conditionExpr` is the result if translating the boolean expression condition to an expression tree, and `info` is
>
> ```csharp
> Q.DoInfo(default(Q.Flags))
> ```
>
> if no `break` or `continue` statements exist within the body of the iteration statement, targeting the current iteration statement, or
>
> ```csharp
> Q.DoInfo(default(Q.Flags), breakLabel)
> ```
>
> if a `break` statement exists but no `continue` statement exists within the body of the iteration statement, targeting the current iteration statement, or
>
> ```csharp
> Q.DoInfo(default(Q.Flags), breakLabel, continueLabel)
> ```
>
> otherwise, where `breakLabel` and `continueLabel` refer to variables holding expression tree nodes representing compiler-generated labels according to the rules described in [Labeled statements](#labeled-statements). If no `break` statement exists within the body of the iteration statement, targeting the current iteration statement, `breakLabel` will be the `default` literal instead.
>
> The rationale of these overloads is to allow for efficient expression tree libraries that can detect control flow behavior associated with the loop, without having to perform complex analysis of the iteration statement body. Note that libraries can implement `DoInfo` using two optional parameters rather than having three distinct overloads. For example:
>
> ```csharp
> static DoInfo DoInfo(Flags flags, LabelInfo @break = default, LabelInfo @continue = default) => ...
> ```
>
> ***TODO***
> * C# 7 adds scoped locals to the bound node, due to declaration expressions. These would be passed to the `DoInfo` factory method as a fourth argument supplied with the result of the `ScopeInfo` factory method.

### The for statement

The `for` statement evaluates a sequence of initialization expressions and then, while a condition is true, repeatedly executes an embedded statement and evaluates a sequence of iteration expressions.

```antlr
for_statement
    : 'for' '(' for_initializer? ';' for_condition? ';' for_iterator? ')' embedded_statement
    ;

for_initializer
    : local_variable_declaration
    | statement_expression_list
    ;

for_condition
    : boolean_expression
    ;

for_iterator
    : statement_expression_list
    ;

statement_expression_list
    : statement_expression (',' statement_expression)*
    ;
```

The *for_initializer*, if present, consists of either a *local_variable_declaration* ([Local variable declarations](statements.md#local-variable-declarations)) or a list of *statement_expression*s ([Expression statements](statements.md#expression-statements)) separated by commas. The scope of a local variable declared by a *for_initializer* starts at the *local_variable_declarator* for the variable and extends to the end of the embedded statement. The scope includes the *for_condition* and the *for_iterator*.

The *for_condition*, if present, must be a *boolean_expression* ([Boolean expressions](expressions.md#boolean-expressions)).

The *for_iterator*, if present, consists of a list of *statement_expression*s ([Expression statements](statements.md#expression-statements)) separated by commas.

A for statement is executed as follows:

*  If a *for_initializer* is present, the variable initializers or statement expressions are executed in the order they are written. This step is only performed once.
*  If a *for_condition* is present, it is evaluated.
*  If the *for_condition* is not present or if the evaluation yields `true`, control is transferred to the embedded statement. When and if control reaches the end point of the embedded statement (possibly from execution of a `continue` statement), the expressions of the *for_iterator*, if any, are evaluated in sequence, and then another iteration is performed, starting with evaluation of the *for_condition* in the step above.
*  If the *for_condition* is present and the evaluation yields `false`, control is transferred to the end point of the `for` statement.

Within the embedded statement of a `for` statement, a `break` statement ([The break statement](statements.md#the-break-statement)) may be used to transfer control to the end point of the `for` statement (thus ending iteration of the embedded statement), and a `continue` statement ([The continue statement](statements.md#the-continue-statement)) may be used to transfer control to the end point of the embedded statement (thus executing the *for_iterator* and performing another iteration of the `for` statement, starting with the *for_condition*).

The embedded statement of a `for` statement is reachable if one of the following is true:

*  The `for` statement is reachable and no *for_condition* is present.
*  The `for` statement is reachable and a *for_condition* is present and does not have the constant value `false`.

The end point of a `for` statement is reachable if at least one of the following is true:

*  The `for` statement contains a reachable `break` statement that exits the `for` statement.
*  The `for` statement is reachable and a *for_condition* is present and does not have the constant value `true`.

> __Expression Tree Conversion Translation Steps__
>
> Translation of a *for_statement* to a query expression tree produces a compile-time error. When converted to a generalized expression tree, the following steps are taken.
>
> ### The initializer
>
> The *for_initializer*, if it exists,
>
> ```antlr
> for_initializer
>     : local_variable_declaration
>     | statement_expression_list
>     ;
> ```
>
> is converted to an expression tree by constructing an expression `initExpr`
>
> ```csharp
> Q.ForInitializer(info, expr)
> ```
>
> where `expr` is the result of converting the *local_variable_declaration* or the *statement_expression_list* to an expression tree, and where `info` is
>
> ```csharp
> Q.ForInitializerInfo(default(Q.Flags))
> ```
>
> ### The condition expression
>
> The *for_condition*, if it exists,
>
> ```antlr
> for_condition
>     : boolean_expression
>     ;
> ```
>
> is converted to an expression tree by constructing an expression `conditionExpr`
>
> ```csharp
> Q.ForCondition(info, expr)
> ```
>
> where `expr` is the result of converting the *boolean_expression* to an expression tree, and where `info` is
>
> ```csharp
> Q.ForConditionInfo(default(Q.Flags))
> ```
>
> ***TODO***
> * Given that we don't always compile-time evaluate constant expressions to reduce nodes to `Q.Constant` (depending on flags on the builder type), it may be useful to still provide access to the constant value. It seems this is just one of the many annotations that could be stuff on "info" nodes. For this node, this would be particularly useful to analyze the loop condition value.
>
> ### The iterator
>
> The *for_iterator*, if it exists,
>
> ```antlr
> for_iterator
>     : statement_expression_list
>     ;
> ```
>
> is converted to an expression tree by constructing an expression `iterExpr`
>
> ```csharp
> Q.ForIterator(info, expr)
> ```
>
> where `expr` is the result of converting the *statement_expression_list* to an expression tree, and where `info` is
>
> ```csharp
> Q.ForIteratorInfo(default(Q.Flags))
> ```
>
> ### Statement expression lists
>
> The *for_initializer* and *for_iterator* can be a *statement_expression_list*
>
> ```antlr
> statement_expression_list
>     : statement_expression (',' statement_expression)*
>     ;
> ```
>
> which is translated by converting each *statement_expression* to an expression tree `stmtExpr_i` (for `1 <= i <= N`), in lexical order, and combining the resulting expressions into
>
> ```csharp
> Q.StatementExpressionList(info, stmtExpr_1, ..., stmtExpr_N)
> ```
>
> where `info` is
>
> ```csharp
> Q.StatementExpressionListInfo(default(Q.Flags))
> ```
>
> ### Constructing the expression tree
>
> it is translated into
>
> ```csharp
> Q.For(info, initExpr, conditionExpr, iterExpr, stmtExpr)
> ```
>
> where `initExpr` is the result of translating *for_initializer* to an expression tree if it exists, or a `default` literal otheriwse, `conditionExpr` is the result of translating *for_condition* to an expression tree if it exists, or a `default` literal otherwise, `iterExpr` is the result of translating *for_iterator* if it exists, or a `default` literal otherwise, and `stmtExpr` is the result of translating *embedded_statement* to an expression tree, and `info` is
>
> ```csharp
> Q.ForInfo(default(Q.Flags), scope)
> ```
>
> if no `break` or `continue` statements exist within the body of the iteration statement, targeting the current iteration statement, or
>
> ```csharp
> Q.ForInfo(default(Q.Flags), scope, breakLabel)
> ```
>
> if a `break` statement exists but no `continue` statement exists within the body of the iteration statement, targeting the current iteration statement, or
>
> ```csharp
> Q.ForInfo(default(Q.Flags), scope, breakLabel, continueLabel)
> ```
>
> otherwise, where `scope` is an expression representing the local variables declared by a *for_initializer*, obtained using the rules in [Scopes in expression trees](#scopes-in-expression-trees), and `breakLabel` and `continueLabel` refer to variables holding expression tree nodes representing compiler-generated labels according to the rules described in [Labeled statements](#labeled-statements). If no `break` statement exists within the body of the iteration statement, targeting the current iteration statement, `breakLabel` will be the `default` literal instead.
>
> The rationale of these overloads is to allow for efficient expression tree libraries that can detect control flow behavior associated with the loop, without having to perform complex analysis of the iteration statement body. Note that libraries can implement `ForInfo` using two optional parameters rather than having three distinct overloads. For example:
>
> ```csharp
> static ForInfo ForInfo(Flags flags, ScopeInfo scope, LabelInfo @break = default, LabelInfo @continue = default) => ...
> ```
>
> ***TODO***
> * C# 7 adds scoped (inner) locals to the bound node, due to declaration expressions. These would be passed to the `ForInfo` factory method as a fifth argument supplied with the result of the `ScopeInfo` factory method.
>   * We should review the API design for future proofing. The C# 6.0 specification shown above calls for only one scope parameter (for out locals, as defined higher up). This is passed as the second argument, in a way because it's very common to have such variables (e.g. the idiomatic `for (int i = 0; i < n; i++)` construct), while it's less common to have `break` and `continue`. We treat the scope as required, even though it could be empty. Where does that leave us when inner locals are added in C# 7.0, and if we decide we indeed want to keep scope info around (to help expression tree libraries discover these scopes rather than having to reconstruct them). Would we add the additional scope parameter at the end, or insert it (which often leads to worries about overload resolution)? Adding at the end could work, but leads to:
>     * Growing API inconsistency over time, which we may have to live with.
>     * A need to come up with a plan for expressing missing information, e.g. if no `break` label is present. The use of target-typed `default` literals is the most natural choice, but limits overload flexibility. Maybe there's always only one overload for an `Info` method? Variants that are well-knownand differentiable from the specification can also use different names (e.g. `DynamicAddInfo` or `AsyncLambdaInfo`).

### The foreach statement

The `foreach` statement enumerates the elements of a collection, executing an embedded statement for each element of the collection.

```antlr
foreach_statement
    : 'foreach' '(' local_variable_type identifier 'in' expression ')' embedded_statement
    ;
```

The *type* and *identifier* of a `foreach` statement declare the ***iteration variable*** of the statement. If the `var` identifier is given as the *local_variable_type*, and no type named `var` is in scope, the iteration variable is said to be an ***implicitly typed iteration variable***, and its type is taken to be the element type of the `foreach` statement, as specified below. The iteration variable corresponds to a read-only local variable with a scope that extends over the embedded statement. During execution of a `foreach` statement, the iteration variable represents the collection element for which an iteration is currently being performed. A compile-time error occurs if the embedded statement attempts to modify the iteration variable (via assignment or the `++` and `--` operators) or pass the iteration variable as a `ref` or `out` parameter.

In the following, for brevity, `IEnumerable`, `IEnumerator`, `IEnumerable<T>` and `IEnumerator<T>` refer to the corresponding types in the namespaces `System.Collections` and `System.Collections.Generic`.

The compile-time processing of a foreach statement first determines the ***collection type***, ***enumerator type*** and ***element type*** of the expression. This determination proceeds as follows:

*  If the type `X` of *expression* is an array type then there is an implicit reference conversion from `X` to the `IEnumerable` interface (since `System.Array` implements this interface). The ***collection type*** is the `IEnumerable` interface, the ***enumerator type*** is the `IEnumerator` interface and the ***element type*** is the element type of the array type `X`.
*  If the type `X` of *expression* is `dynamic` then there is an implicit conversion from *expression* to the `IEnumerable` interface ([Implicit dynamic conversions](conversions.md#implicit-dynamic-conversions)). The ***collection type*** is the `IEnumerable` interface and the ***enumerator type*** is the `IEnumerator` interface. If the `var` identifier is given as the *local_variable_type* then the ***element type*** is `dynamic`, otherwise it is `object`.
*  Otherwise, determine whether the type `X` has an appropriate `GetEnumerator` method:
   * Perform member lookup on the type `X` with identifier `GetEnumerator` and no type arguments. If the member lookup does not produce a match, or it produces an ambiguity, or produces a match that is not a method group, check for an enumerable interface as described below. It is recommended that a warning be issued if member lookup produces anything except a method group or no match.
   * Perform overload resolution using the resulting method group and an empty argument list. If overload resolution results in no applicable methods, results in an ambiguity, or results in a single best method but that method is either static or not public, check for an enumerable interface as described below. It is recommended that a warning be issued if overload resolution produces anything except an unambiguous public instance method or no applicable methods.
   * If the return type `E` of the `GetEnumerator` method is not a class, struct or interface type, an error is produced and no further steps are taken.
   * Member lookup is performed on `E` with the identifier `Current` and no type arguments. If the member lookup produces no match, the result is an error, or the result is anything except a public instance property that permits reading, an error is produced and no further steps are taken.
   * Member lookup is performed on `E` with the identifier `MoveNext` and no type arguments. If the member lookup produces no match, the result is an error, or the result is anything except a method group, an error is produced and no further steps are taken.
   * Overload resolution is performed on the method group with an empty argument list. If overload resolution results in no applicable methods, results in an ambiguity, or results in a single best method but that method is either static or not public, or its return type is not `bool`, an error is produced and no further steps are taken.
   * The ***collection type*** is `X`, the ***enumerator type*** is `E`, and the ***element type*** is the type of the `Current` property.

*  Otherwise, check for an enumerable interface:
   * If among all the types `Ti` for which there is an implicit conversion from `X` to `IEnumerable<Ti>`, there is a unique type `T` such that `T` is not `dynamic` and for all the other `Ti` there is an implicit conversion from `IEnumerable<T>` to `IEnumerable<Ti>`, then the ***collection type*** is the interface `IEnumerable<T>`, the ***enumerator type*** is the interface `IEnumerator<T>`, and the ***element type*** is `T`.
   * Otherwise, if there is more than one such type `T`, then an error is produced and no further steps are taken.
   * Otherwise, if there is an implicit conversion from `X` to the `System.Collections.IEnumerable` interface, then the ***collection type*** is this interface, the ***enumerator type*** is the interface `System.Collections.IEnumerator`, and the ***element type*** is `object`.
   * Otherwise, an error is produced and no further steps are taken.

The above steps, if successful, unambiguously produce a collection type `C`, enumerator type `E` and element type `T`. A foreach statement of the form
```csharp
foreach (V v in x) embedded_statement
```
is then expanded to:
```csharp
{
    E e = ((C)(x)).GetEnumerator();
    try {
        while (e.MoveNext()) {
            V v = (V)(T)e.Current;
            embedded_statement
        }
    }
    finally {
        ... // Dispose e
    }
}
```

The variable `e` is not visible to or accessible to the expression `x` or the embedded statement or any other source code of the program. The variable `v` is read-only in the embedded statement. If there is not an explicit conversion ([Explicit conversions](conversions.md#explicit-conversions)) from `T` (the element type) to `V` (the *local_variable_type* in the foreach statement), an error is produced and no further steps are taken. If `x` has the value `null`, a `System.NullReferenceException` is thrown at run-time.

An implementation is permitted to implement a given foreach-statement differently, e.g. for performance reasons, as long as the behavior is consistent with the above expansion.

The placement of `v` inside the while loop is important for how it is captured by any anonymous function occurring in the *embedded_statement*.

For example:
```csharp
int[] values = { 7, 9, 13 };
Action f = null;

foreach (var value in values)
{
    if (f == null) f = () => Console.WriteLine("First value: " + value);
}

f();
```
If `v` was declared outside of the while loop, it would be shared among all iterations, and its value after the for loop would be the final value, `13`, which is what the invocation of `f` would print. Instead, because each iteration has its own variable `v`, the one captured by `f` in the first iteration will continue to hold the value `7`, which is what will be printed. (Note: earlier versions of C# declared `v` outside of the while loop.)

The body of the finally block is constructed according to the following steps:

*  If there is an implicit conversion from `E` to the `System.IDisposable` interface, then
   *  If `E` is a non-nullable value type then the finally clause is expanded to the semantic equivalent  of:

      ```csharp
      finally {
          ((System.IDisposable)e).Dispose();
      }
      ```

   *  Otherwise the finally clause is expanded to the semantic equivalent of:

      ```csharp
      finally {
          if (e != null) ((System.IDisposable)e).Dispose();
      }
      ```

   except that if `E` is a value type, or a type parameter instantiated to a value type, then the cast of `e` to `System.IDisposable` will not cause boxing to occur.

*  Otherwise, if `E` is a sealed type, the finally clause is expanded to an empty block:

   ```csharp
   finally {
   }
   ```

*  Otherwise, the finally clause is expanded to:

   ```csharp
   finally {
       System.IDisposable d = e as System.IDisposable;
       if (d != null) d.Dispose();
   }
   ```    

   The local variable `d` is not visible to or accessible to any user code. In particular, it does not conflict with any other variable whose scope includes the finally block.

The order in which `foreach` traverses the elements of an array, is as follows: For single-dimensional arrays elements are traversed in increasing index order, starting with index `0` and ending with index `Length - 1`. For multi-dimensional arrays, elements are traversed such that the indices of the rightmost dimension are increased first, then the next left dimension, and so on to the left.

The following example prints out each value in a two-dimensional array, in element order:
```csharp
using System;

class Test
{
    static void Main() {
        double[,] values = {
            {1.2, 2.3, 3.4, 4.5},
            {5.6, 6.7, 7.8, 8.9}
        };

        foreach (double elementValue in values)
            Console.Write("{0} ", elementValue);

        Console.WriteLine();
    }
}
```
The output produced is as follows:
```csharp
1.2 2.3 3.4 4.5 5.6 6.7 7.8 8.9
```

In the example
```csharp
int[] numbers = { 1, 3, 5, 7, 9 };
foreach (var n in numbers) Console.WriteLine(n);
```
the type of `n` is inferred to be `int`, the element type of `numbers`.

> __Expression Tree Conversion Translation Steps__
>
> Translation of a *foreach_statement*
>
> ```csharp
> foreach (V v in x) embedded_statement
> ```
>
> to a query expression tree produces a compile-time error. When converted to a generalized expression tree, the following steps are taken.
>
> ### The collection expression
>
> The collection *expression* `x` is converted to an expression tree by constructing an expression `collectionExpr` as follows, where `X` is the type of *expression*, and `C` is the collection type as defined earlier.
>
> If `E` is an array type, or the collection type `C` is identical to `X`, then `collectionExpr` is the result of converting `x` to an expression tree.
>
> Otherwise, `collectionExpr` is the result of converting `(C)(x)` to an expression tree, where the introduced cast expression is classified as compiler-generated. Note that this case also covers the case where `E` is `dynamic` and where `x` is converted to `IEnumerable`.
>
> ### The iteration variable
>
> The iteration variable declaration `V v` is converted to an expression tree by constructing an expression `iterationVariable` as follows, where `T` is the element type, as defined earlier.
>
> If `V` is the identifier `var` and no type named `var` is in scope, substitute `T` for `V`. Treat the resulting `V v` as a *local_variable_declaration* and follow the steps in [Local variables](variables.md#local-variables) to construct the expression `iterationVariable`.
>
> ### Element conversion
>
> An expression `elementConversion` is constructed if `T` differs from `V`, where `T` is the element type, as defined earlier, and where `V` is the (possibly inferred) iteration variable type, as defined in the previous section, as follows.
>
> Let `t` represent a local variable with a compiler-generated identifier `t` and type `T`. Then construct a cast expression `(V)t`, classified as compiler-generated, and follow the steps in [Cast expressions](expressions.md#cast-expressions) to construct a `Q.ConvertInfo` object, which becomes `elementConversion`. Note that this conversion supports dynamically bound conversion in case `T` is `dynamic` and `V` is not `dynamic`. Furthermore, note that if the *for_statement* occurs in a checked contet, the resulting `ConvertInfo` node will have the `Q.Flags.CheckedContext` flag specified.
>
> Otherwise, `elementConversion` is defined as the `default` literal.
>
> ### The embedded statement
>
> The *embedded_statement* in the *foreac_statement* is converted to an expression tree `bodyExpr`, where the iteration variable is assumed to be in scope and all use sites get converted to `iterationVariable`.
>
> ### Constructing the expression tree
>
> Given the constituents constructed in the preceding steps, the expression tree for the *foreach_statement* is constructed as follows.
>
> ```csharp
> Q.ForEach(info, iterationVariable, collectionExpr, bodyExpr)
> ```
>
> where `info` is
>
> ```csharp
> Q.ForEachInfo(flags, elementConversion)
> ```
>
> if the `C` is an array type, `IEnumerable`, or `IEnumerable<T>`, or
>
> ```csharp
> Q.ForEachInfo(flags, elementConversion, getEnumeratorMethod, moveNextMethod, getCurrentMethod)
> ```
>
> otherwise, where `getEnumerator` method is an expression of type `MethodInfo` representing the bound `GetEnumerator` method,  `moveNextMethod` method is an expression of type `MethodInfo` representing the bound `MoveNext` method, and  `getCurrentMethod` method is an expression of type `MethodInfo` representing the `get` accessor of the bound `Current` property.
>
> In all cases, `flags` is `Q.Flags.CheckedContext` if the *foreach_statement* occurs in a checked context, and `default(Q.Flags)` otherwise.
>
> ***TODO***
> * Should the collection conversion be applied as a `(T)(x)` rewrite step (similar to what's done for many other nodes), or be modeled as a collection conversion passed to `ForEachInfo`?
> * Should enumerator conversion, which is omitted from the spec but needed in some cases to convert the enumerator to `object` in order to perform a `null`-check in the `finally` block? Likely this can be left to the reduction or compilation stage in the expression tree library where a `ReferenceEqual` node could easily be used. A conversion to `object` is well-understood without passing C# compiler conversion info to the factory.
> * Should information about the bound `Dispose` method be passed as well? E.g. to avoid boxing for value type enumerators (or can we rely on constrained virtual calls being emitted for these cases by a reduction/lowering phase in the library)?
> * Should `elementConversion` be allowed to be `default` or should we emit an identity conversion instead?

## Jump statements

Jump statements unconditionally transfer control.

```antlr
jump_statement
    : break_statement
    | continue_statement
    | goto_statement
    | return_statement
    | throw_statement
    ;
```

The location to which a jump statement transfers control is called the ***target*** of the jump statement.

When a jump statement occurs within a block, and the target of that jump statement is outside that block, the jump statement is said to ***exit*** the block. While a jump statement may transfer control out of a block, it can never transfer control into a block.

Execution of jump statements is complicated by the presence of intervening `try` statements. In the absence of such `try` statements, a jump statement unconditionally transfers control from the jump statement to its target. In the presence of such intervening `try` statements, execution is more complex. If the jump statement exits one or more `try` blocks with associated `finally` blocks, control is initially transferred to the `finally` block of the innermost `try` statement. When and if control reaches the end point of a `finally` block, control is transferred to the `finally` block of the next enclosing `try` statement. This process is repeated until the `finally` blocks of all intervening `try` statements have been executed.

In the example
```csharp
using System;

class Test
{
    static void Main() {
        while (true) {
            try {
                try {
                    Console.WriteLine("Before break");
                    break;
                }
                finally {
                    Console.WriteLine("Innermost finally block");
                }
            }
            finally {
                Console.WriteLine("Outermost finally block");
            }
        }
        Console.WriteLine("After break");
    }
}
```
the `finally` blocks associated with two `try` statements are executed before control is transferred to the target of the jump statement.

The output produced is as follows:
```
Before break
Innermost finally block
Outermost finally block
After break
```

### The break statement

The `break` statement exits the nearest enclosing `switch`, `while`, `do`, `for`, or `foreach` statement.

```antlr
break_statement
    : 'break' ';'
    ;
```

The target of a `break` statement is the end point of the nearest enclosing `switch`, `while`, `do`, `for`, or `foreach` statement. If a `break` statement is not enclosed by a `switch`, `while`, `do`, `for`, or `foreach` statement, a compile-time error occurs.

When multiple `switch`, `while`, `do`, `for`, or `foreach` statements are nested within each other, a `break` statement applies only to the innermost statement. To transfer control across multiple nesting levels, a `goto` statement ([The goto statement](statements.md#the-goto-statement)) must be used.

A `break` statement cannot exit a `finally` block ([The try statement](statements.md#the-try-statement)). When a `break` statement occurs within a `finally` block, the target of the `break` statement must be within the same `finally` block; otherwise, a compile-time error occurs.

A `break` statement is executed as follows:

*  If the `break` statement exits one or more `try` blocks with associated `finally` blocks, control is initially transferred to the `finally` block of the innermost `try` statement. When and if control reaches the end point of a `finally` block, control is transferred to the `finally` block of the next enclosing `try` statement. This process is repeated until the `finally` blocks of all intervening `try` statements have been executed.
*  Control is transferred to the target of the `break` statement.

Because a `break` statement unconditionally transfers control elsewhere, the end point of a `break` statement is never reachable.

> __Expression Tree Conversion Translation Steps__
>
> Translation of a *break_statement* to a query expression tree produces a compile-time error. When converted to a generalized expression tree, it is translated into:
>
> ```csharp
> Q.Break(info)
> ```
>
> where `info` is
>
> ```csharp
> Q.BreakInfo(default(Q.Flags), label)
> ```
>
> where `label` is a variable corresponding to the break label introduced by the closest parent *loop_statement* or the closes parent *switch_statement* using the steps described in [Labeled statements](#labeled-statements). 
>
> Note that this design can enable breaking out of other loops than the closest parent *loop_statement*.

### The continue statement

The `continue` statement starts a new iteration of the nearest enclosing `while`, `do`, `for`, or `foreach` statement.

```antlr
continue_statement
    : 'continue' ';'
    ;
```

The target of a `continue` statement is the end point of the embedded statement of the nearest enclosing `while`, `do`, `for`, or `foreach` statement. If a `continue` statement is not enclosed by a `while`, `do`, `for`, or `foreach` statement, a compile-time error occurs.

When multiple `while`, `do`, `for`, or `foreach` statements are nested within each other, a `continue` statement applies only to the innermost statement. To transfer control across multiple nesting levels, a `goto` statement ([The goto statement](statements.md#the-goto-statement)) must be used.

A `continue` statement cannot exit a `finally` block ([The try statement](statements.md#the-try-statement)). When a `continue` statement occurs within a `finally` block, the target of the `continue` statement must be within the same `finally` block; otherwise a compile-time error occurs.

A `continue` statement is executed as follows:

*  If the `continue` statement exits one or more `try` blocks with associated `finally` blocks, control is initially transferred to the `finally` block of the innermost `try` statement. When and if control reaches the end point of a `finally` block, control is transferred to the `finally` block of the next enclosing `try` statement. This process is repeated until the `finally` blocks of all intervening `try` statements have been executed.
*  Control is transferred to the target of the `continue` statement.

Because a `continue` statement unconditionally transfers control elsewhere, the end point of a `continue` statement is never reachable.

> __Expression Tree Conversion Translation Steps__
>
> Translation of a *continue_statement* to a query expression tree produces a compile-time error. When converted to a generalized expression tree, it is translated into:
>
> ```csharp
> Q.Continue(info)
> ```
>
> where `info` is
>
> ```csharp
> Q.ContinueInfo(default(Q.Flags), label)
> ```
>
> where `label` is a variable corresponding to the break label introduced by the closest parent *loop_statement* using the steps described in [Labeled statements](#labeled-statements).
>
> Note that this design can enable continuing a loop other than the closest parent *loop_statement*.

### The goto statement

The `goto` statement transfers control to a statement that is marked by a label.

```antlr
goto_statement
    : 'goto' identifier ';'
    | 'goto' 'case' constant_expression ';'
    | 'goto' 'default' ';'
    ;
```

The target of a `goto` *identifier* statement is the labeled statement with the given label. If a label with the given name does not exist in the current function member, or if the `goto` statement is not within the scope of the label, a compile-time error occurs. This rule permits the use of a `goto` statement to transfer control out of a nested scope, but not into a nested scope. In the example
```csharp
using System;

class Test
{
    static void Main(string[] args) {
        string[,] table = {
            {"Red", "Blue", "Green"},
            {"Monday", "Wednesday", "Friday"}
        };

        foreach (string str in args) {
            int row, colm;
            for (row = 0; row <= 1; ++row)
                for (colm = 0; colm <= 2; ++colm)
                    if (str == table[row,colm])
                         goto done;

            Console.WriteLine("{0} not found", str);
            continue;
    done:
            Console.WriteLine("Found {0} at [{1}][{2}]", str, row, colm);
        }
    }
}
```
a `goto` statement is used to transfer control out of a nested scope.

The target of a `goto case` statement is the statement list in the immediately enclosing `switch` statement ([The switch statement](statements.md#the-switch-statement)), which contains a `case` label with the given constant value. If the `goto case` statement is not enclosed by a `switch` statement, if the *constant_expression* is not implicitly convertible ([Implicit conversions](conversions.md#implicit-conversions)) to the governing type of the nearest enclosing `switch` statement, or if the nearest enclosing `switch` statement does not contain a `case` label with the given constant value, a compile-time error occurs.

The target of a `goto default` statement is the statement list in the immediately enclosing `switch` statement ([The switch statement](statements.md#the-switch-statement)), which contains a `default` label. If the `goto default` statement is not enclosed by a `switch` statement, or if the nearest enclosing `switch` statement does not contain a `default` label, a compile-time error occurs.

A `goto` statement cannot exit a `finally` block ([The try statement](statements.md#the-try-statement)). When a `goto` statement occurs within a `finally` block, the target of the `goto` statement must be within the same `finally` block, or otherwise a compile-time error occurs.

A `goto` statement is executed as follows:

*  If the `goto` statement exits one or more `try` blocks with associated `finally` blocks, control is initially transferred to the `finally` block of the innermost `try` statement. When and if control reaches the end point of a `finally` block, control is transferred to the `finally` block of the next enclosing `try` statement. This process is repeated until the `finally` blocks of all intervening `try` statements have been executed.
*  Control is transferred to the target of the `goto` statement.

Because a `goto` statement unconditionally transfers control elsewhere, the end point of a `goto` statement is never reachable.

> __Expression Tree Conversion Translation Steps__
>
> Translation of a *goto_statement* to a query expression tree produces a compile-time error. When converted to a generalized expression tree, it is translated into:
>
> ```csharp
> Q.Goto(info)
> ```
>
> if the *goto_statement* is of the form
>
> ```antlr
> 'goto' identifier ';'
> ```
>
> where `info` is
>
> ```csharp
> Q.GotoInfo(default(Q.Flags), label)
> ```
>
> where `label` is a variable representing the label introduced by the steps described in [Labeled statements](#labeled-statements), or into
>
> ```csharp
> Q.GotoCase(info, expr)
> ```
>
> if the *goto_statement* is of the form `goto case c`
>
> ```antlr
> 'goto' 'case' constant_expression ';'
> ```
>
> where `info` is
>
> ```csharp
> Q.GotoCaseInfo(default(Q.Flags), label)
> ```
>
> where `constExpr` is an expression constructed by converting the *constant_expression* `c` to an expression tree, if the type of `c` matches the governing type of the parent switch statement, or by converting the expression `(T)(c)` where `T` is the governing type, otherwise. The introduced cast expression, if any, is classified as compiler-generated and represents the user-defined implicit conversion ([User-defined conversions](conversions.md#user-defined-conversions)) to a supported governing type.
>
> Furthermore, `label` is a variable representing the label associated with the *switch_label* containing the *constant_expression*, introduced by the steps described in [Labeled statements](#labeled-statements).
>
> Otherwise, it is translated into
>
> ```csharp
> Q.GotoDefault(info)
> ```
>
> if the *goto_statement* is of the form
>
> ```antlr
> 'goto' 'default' ';'
> ```
>
> where `info` is
>
> ```csharp
> Q.GotoDefaultInfo(default(Q.Flags), label)
> ```
>
> where `label` is a variable representing the label associated with the *switch_section* containing the `default` *switch_label*, introduced by the steps described in [Labeled statements](#labeled-statements).

### The return statement

The `return` statement returns control to the current caller of the function in which the `return` statement appears.

```antlr
return_statement
    : 'return' expression? ';'
    ;
```

A `return` statement with no expression can be used only in a function member that does not compute a value, that is, a method with the result type ([Method body](classes.md#method-body)) `void`, the `set` accessor of a property or indexer, the `add` and `remove` accessors of an event, an instance constructor, a static constructor, or a destructor.

A `return` statement with an expression can only be used in a function member that computes a value, that is, a method with a non-void result type, the `get` accessor of a property or indexer, or a user-defined operator. An implicit conversion ([Implicit conversions](conversions.md#implicit-conversions)) must exist from the type of the expression to the return type of the containing function member.

Return statements can also be used in the body of anonymous function expressions ([Anonymous function expressions](expressions.md#anonymous-function-expressions)), and participate in determining which conversions exist for those functions.

It is a compile-time error for a `return` statement to appear in a `finally` block ([The try statement](statements.md#the-try-statement)).

A `return` statement is executed as follows:

*  If the `return` statement specifies an expression, the expression is evaluated and the resulting value is converted to the return type of the containing function by an implicit conversion. The result of the conversion becomes the result value produced by the function.
*  If the `return` statement is enclosed by one or more `try` or `catch` blocks with associated `finally` blocks, control is initially transferred to the `finally` block of the innermost `try` statement. When and if control reaches the end point of a `finally` block, control is transferred to the `finally` block of the next enclosing `try` statement. This process is repeated until the `finally` blocks of all enclosing `try` statements have been executed.
*  If the containing function is not an async function, control is returned to the caller of the containing function along with the result value, if any.
*  If the containing function is an async function, control is returned to the current caller, and the result value, if any, is recorded in the return task as described in ([Enumerator interfaces](classes.md#enumerator-interfaces)).

Because a `return` statement unconditionally transfers control elsewhere, the end point of a `return` statement is never reachable.

> __Expression Tree Conversion Translation Steps__
>
> Translation of a *return_statement* to a query expression tree produces a compile-time error. When converted to a generalized expression tree, it is translated into
>
> ```csharp
> Q.Return(info, expr)
> ```
>
> if the *return_statement* has an *expression*, where `expr` is an expression constructed from converting *expression* to an expression tree, or
>
> ```csharp
> Q.Return(info)
> ```
>
> otherwise, where `info` is
>
> ```csharp
> Q.ThrowInfo(default(Q.Flags))
> ```
>
> ***TODO***
> * Should this node also include a reference to a label introduced by the containing anonymous function expression?

### The throw statement

The `throw` statement throws an exception.

```antlr
throw_statement
    : 'throw' expression? ';'
    ;
```

A `throw` statement with an expression throws the value produced by evaluating the expression. The expression must denote a value of the class type `System.Exception`, of a class type that derives from `System.Exception` or of a type parameter type that has `System.Exception` (or a subclass thereof) as its effective base class. If evaluation of the expression produces `null`, a `System.NullReferenceException` is thrown instead.

A `throw` statement with no expression can be used only in a `catch` block, in which case that statement re-throws the exception that is currently being handled by that `catch` block.

Because a `throw` statement unconditionally transfers control elsewhere, the end point of a `throw` statement is never reachable.

When an exception is thrown, control is transferred to the first `catch` clause in an enclosing `try` statement that can handle the exception. The process that takes place from the point of the exception being thrown to the point of transferring control to a suitable exception handler is known as ***exception propagation***. Propagation of an exception consists of repeatedly evaluating the following steps until a `catch` clause that matches the exception is found. In this description, the ***throw point*** is initially the location at which the exception is thrown.

*  In the current function member, each `try` statement that encloses the throw point is examined. For each statement `S`, starting with the innermost `try` statement and ending with the outermost `try` statement, the following steps are evaluated:

   * If the `try` block of `S` encloses the throw point and if S has one or more `catch` clauses, the `catch` clauses are examined in order of appearance to locate a suitable handler for the exception, according to the rules specified in Section [The try statement](statements.md#the-try-statement). If a matching `catch` clause is located, the exception propagation is completed by transferring control to the block of that `catch` clause.

   * Otherwise, if the `try` block or a `catch` block of `S` encloses the throw point and if `S` has a `finally` block, control is transferred to the `finally` block. If the `finally` block throws another exception, processing of the current exception is terminated. Otherwise, when control reaches the end point of the `finally` block, processing of the current exception is continued.

*  If an exception handler was not located in the current function invocation, the function invocation is terminated, and one of the following occurs:

   * If the current function is non-async, the steps above are repeated for the caller of the function with a throw point corresponding to the statement from which the function member was invoked.

   * If the current function is async and task-returning, the exception is recorded in the return task, which is put into a faulted or cancelled state as described in [Enumerator interfaces](classes.md#enumerator-interfaces).

   * If the current function is async and void-returning, the synchronization context of the current thread is notified as described in [Enumerable interfaces](classes.md#enumerable-interfaces).

*  If the exception processing terminates all function member invocations in the current thread, indicating that the thread has no handler for the exception, then the thread is itself terminated. The impact of such termination is implementation-defined.

> __Expression Tree Conversion Translation Steps__
>
> Translation of a *throw_statement* to a query expression tree produces a compile-time error. When converted to a generalized expression tree, it is translated into
>
> ```csharp
> Q.Throw(info, expr)
> ```
>
> if the *throw_statement* has an *expression*, where `expr` is an expression constructed from converting *expression* to an expression tree, or
>
> ```csharp
> Q.Throw(info)
> ```
>
> otherwise, where `info` is
>
> ```csharp
> Q.ThrowInfo(default(Q.Flags))
> ```
>
> ***TODO***
> * Note that adding the C# 7 *throw_expression* should be easy by introducing a new `ThrowInfo` overload that has a `Type` parameter. Expression tree libraries can make that overload of `ThrowInfo` return a different type, which influences overload resolution for the parent `Throw` invocation, which on its turn can return a different type to distinguish the expression variant from the statement variant (if that is desirable for some expression tree library).

## The try statement

The `try` statement provides a mechanism for catching exceptions that occur during execution of a block. Furthermore, the `try` statement provides the ability to specify a block of code that is always executed when control leaves the `try` statement.

```antlr
try_statement
    : 'try' block catch_clause+
    | 'try' block finally_clause
    | 'try' block catch_clause+ finally_clause
    ;

catch_clause
    : 'catch' exception_specifier? exception_filter?  block
    ;

exception_specifier
    : '(' type identifier? ')'
    ;

exception_filter
    : 'when' '(' expression ')'
    ;

finally_clause
    : 'finally' block
    ;
```

There are three possible forms of `try` statements:

*  A `try` block followed by one or more `catch` blocks.
*  A `try` block followed by a `finally` block.
*  A `try` block followed by one or more `catch` blocks followed by a `finally` block.

When a `catch` clause specifies an *exception_specifier*, the type must be `System.Exception`, a type that derives from `System.Exception` or a type parameter type that has `System.Exception` (or a subclass thereof) as its effective base class.

When a `catch` clause specifies both an *exception_specifier* with an *identifier*, an ***exception variable*** of the given name and type is declared. The exception variable corresponds to a local variable with a scope that extends over the `catch` clause. During execution of the *exception_filter* and *block*, the exception variable represents the exception currently being handled. For purposes of definite assignment checking, the exception variable is considered definitely assigned in its entire scope.

Unless a `catch` clause includes an exception variable name, it is impossible to access the exception object in the filter and `catch` block.

A `catch` clause that does not specify an *exception_specifier* is called a general `catch` clause.

Some programming languages may support exceptions that are not representable as an object derived from `System.Exception`, although such exceptions could never be generated by C# code. A general `catch` clause may be used to catch such exceptions. Thus, a general `catch` clause is semantically different from one that specifies the type `System.Exception`, in that the former may also catch exceptions from other languages.

In order to locate a handler for an exception, `catch` clauses are examined in lexical order. If a `catch` clause specifies a type but no exception filter, it is a compile-time error for a later `catch` clause in the same `try` statement to specify a type that is the same as, or is derived from, that type. If a `catch` clause specifies no type and no filter, it must be the last `catch` clause for that `try` statement.

Within a `catch` block, a `throw` statement ([The throw statement](statements.md#the-throw-statement)) with no expression can be used to re-throw the exception that was caught by the `catch` block. Assignments to an exception variable do not alter the exception that is re-thrown.

In the example
```csharp
using System;

class Test
{
    static void F() {
        try {
            G();
        }
        catch (Exception e) {
            Console.WriteLine("Exception in F: " + e.Message);
            e = new Exception("F");
            throw;                // re-throw
        }
    }

    static void G() {
        throw new Exception("G");
    }

    static void Main() {
        try {
            F();
        }
        catch (Exception e) {
            Console.WriteLine("Exception in Main: " + e.Message);
        }
    }
}
```
the method `F` catches an exception, writes some diagnostic information to the console, alters the exception variable, and re-throws the exception. The exception that is re-thrown is the original exception, so the output produced is:
```
Exception in F: G
Exception in Main: G
```

If the first catch block had thrown `e` instead of rethrowing the current exception, the output produced would be as follows:
```csharp
Exception in F: G
Exception in Main: F
```

It is a compile-time error for a `break`, `continue`, or `goto` statement to transfer control out of a `finally` block. When a `break`, `continue`, or `goto` statement occurs in a `finally` block, the target of the statement must be within the same `finally` block, or otherwise a compile-time error occurs.

It is a compile-time error for a `return` statement to occur in a `finally` block.

A `try` statement is executed as follows:

*  Control is transferred to the `try` block.
*  When and if control reaches the end point of the `try` block:
   *  If the `try` statement has a `finally` block, the `finally` block is executed.
   *  Control is transferred to the end point of the `try` statement.

*  If an exception is propagated to the `try` statement during execution of the `try` block:
   *  The `catch` clauses, if any, are examined in order of appearance to locate a suitable handler for the exception. If a `catch` clause does not specify a type, or specifies the exception type or a base type of the exception type:
      *  If the `catch` clause declares an exception variable, the exception object is assigned to the exception variable.
      *  If the `catch` clause declares an exception filter, the filter is evaluated. If it evaluates to `false`, the catch clause is not a match, and the search continues through any subsequent `catch` clauses for a suitable handler.
      *  Otherwise, the `catch` clause is considered a match, and control is transferred to the matching `catch` block.
      *  When and if control reaches the end point of the `catch` block:
         * If the `try` statement has a `finally` block, the `finally` block is executed.
         * Control is transferred to the end point of the `try` statement.
      *  If an exception is propagated to the `try` statement during execution of the `catch` block:
         *  If the `try` statement has a `finally` block, the `finally` block is executed.
         *  The exception is propagated to the next enclosing `try` statement.
   *  If the `try` statement has no `catch` clauses or if no `catch` clause matches the exception:
      *  If the `try` statement has a `finally` block, the `finally` block is executed.
      *  The exception is propagated to the next enclosing `try` statement.

The statements of a `finally` block are always executed when control leaves a `try` statement. This is true whether the control transfer occurs as a result of normal execution, as a result of executing a `break`, `continue`, `goto`, or `return` statement, or as a result of propagating an exception out of the `try` statement.

If an exception is thrown during execution of a `finally` block, and is not caught within the same finally block, the exception is propagated to the next enclosing `try` statement. If another exception was in the process of being propagated, that exception is lost. The process of propagating an exception is discussed further in the description of the `throw` statement ([The throw statement](statements.md#the-throw-statement)).

The `try` block of a `try` statement is reachable if the `try` statement is reachable.

A `catch` block of a `try` statement is reachable if the `try` statement is reachable.

The `finally` block of a `try` statement is reachable if the `try` statement is reachable.

The end point of a `try` statement is reachable if both of the following are true:

*  The end point of the `try` block is reachable or the end point of at least one `catch` block is reachable.
*  If a `finally` block is present, the end point of the `finally` block is reachable.

> __Expression Tree Conversion Translation Steps__
>
> Translation of a *try_statement* to a query expression tree produces a compile-time error. When converted to a generalized expression tree, the following steps are taken.
>
> ### Convert `try` and `finally` blocks
>
> Construct an expression `bodyExpr` by converting the *block* immediately following the `try` keyword to an expression tree. Next, construct an expression `finallyExpr` by converting the *block* immediately following the `finally` keyword to an expression tree, if it exists.
>
> ### Convert `catch` clauses
>
> Then, construct expressions `catchExpr_i` (where `1 <= i <= N`, where `N` is the number of *catch_clause*s) by translating each *catch_clause* as follows.
>
> ```antlr
> catch_clause
>     : 'catch' exception_specifier? exception_filter?  block
>     ;
>
> exception_specifier
>     : '(' type identifier? ')'
>     ;
> 
> exception_filter
>     : 'when' '(' expression ')'
>     ;
> ```
>
> If an *exception_specifier* is present, construct an expression `type` of type `Type` representing the *type* in the *exception_specifier*. Furthermore, if an *identifier* is specified in the *exception_specifier*, obtain a variable `caughtExceptionVar` by following the translation steps for local variables ([Local variables](variables.md#local-variables)). For example, applying these rules to
>
> ```csharp
> catch (Exception ex)
> ```
>
> will result in the construction of the following statement
>
> ```csharp
> var t = Q.Variable(Q.VariableInfo(default(Q.Flags), typeof(Exeption), "ex"));
> ```
>
> where `t` is a compiler-generated identifier that is otherwise invisible and is referred to as `caughtExceptionVar` in the remainder of this section.
>
> The variable declared in the *exception_specifier* remains in scope for the expression tree conversion steps of the *block* and the *exception_filter*, if it exists. That is, any use site of the variable will be translated to `caughtExceptionVar`, referring to the variable introduces here.
>
> Next, construct an expression `catchBlockExpr` by converting the *block* to an expression tree, and construct an expression `filterExpr` by converting the *exception_filter* to an expression tree, if it exists.
>
> An expression representing the catch clause is then constructed using the following decision tree
>
> * If an *exception_specifier* is present *and* has an *identifier*
>   * If an *exception_filter* is present, translate to
>     ```csharp
>     Q.CatchClause(info, caughtExceptionVar, filterExpr, catchBlockExpr)
>     ```
>   * else, translate to
>     ```csharp
>     Q.CatchClause(info, caughtExceptionVar, catchBlockExpr)
>     ```
> * else
>   * If an *exception_filter* is present, translate to
>     ```csharp
>     Q.CatchClause(info, filterExpr, catchBlockExpr)
>     ```
>   * else, translate to
>     ```csharp
>     Q.CatchClause(info, catchBlockExpr)
>     ```
>
> where `info` is
>
> ```csharp
> Q.CatchClauseInfo(default(Q.Flags), type)
> ```
>
> Note that we retain syntactic lexical order in expression tree factory method invocations.
>
> Finally, if `N > 0`, combine all `catchExpr_i` expression tree fragments by constructing an expression `catchClauses` from
>
> ```csharp
> Q.CatchClauses(info, catchExpr_1, ..., catchExpr_N)
> ```
>
> where `info` is
>
> ```csharp
> Q.CatchClausesInfo(default(Q.Flags))
> ```
>
> ***TODO***
> * Determine a design stance on API evolution. While we can start with lexical order, it may break down if a node evolves to have new child nodes and an ambiguity arises.
> * Determine whether `type` belongs on `CatchClause` or on `CatchClauseInfo`. Note that reflection members are often on info nodes, e.g. for `Add`, `New`, `Call`, etc.
>    * On stance is that if it shows up in syntax, it should be on the node itself and not the "info". Then `Call`, rather than `CallInfo`, should have a `MethodInfo` (or a dynamic binder object). But for nodes like `Add` the bound method is not visible in syntax.
>    * Another stance is that the nodes modulo "info" objects capture the shape of the tree modulo any bound symbols. That is, "info" objects carry semantic info (which never refer to tree nodes, but may refer to other "info" nodes; `ScopeInfo` violates this layering right now), which in practice is reflection.
>      * An expression tree library could easily perform type erasure by rewriting "info" nodes while leaving the rest of the tree shape the same (e.g. by changing them for `string`-based nodes to refer to types or members by name).
>      * Discovering all binding information would also be very easy if an expression tree library has a visitor that visits all "info" nodes.
>    * Or we could flatten the hierarchy and do away with "info" nodes, though often nodes are structurally the same (reflective of the grammar of the language) but are semantically bound in a different manner. By having "info" nodes, variations in syntactic shape and variations in semantic binding info never clash and don't lead to a cartesian product of overloads. It also allows for reuse of "info" nodes (e.g. `ConvertInfo`).
>
> ### Construct the expression tree
>
> Conversion to an expression tree proceeds as follows, using the expressions constructed in the sections above.
>
> If *try_statement* contains a *finally_clause* but no *catch_clause*s, it is translated into
>
> ```csharp
> Q.Try(info, bodyExpr, finallyExpr)
> ```
>
> or, if the *try_statement* has one or more *catch_clause*s but no *finally_clause*, it is translated into
>
> ```csharp
> Q.Try(info, bodyExpr, catchClauses)
> ```
>
> or, if the *try_statement* has one or more *catch_clause*s and a *finally_clause*, it is translated into
>
> ```csharp
> Q.Try(info, bodyExpr, catchClauses, finallyExpr)
> ```
>
> where `info` is
>
> ```csharp
> Q.TryInfo(default(Q.Flags))
> ```
>
> ***TODO***
> * Note that the presence of a `TryInfo` node with `Q.Flags` can also come in handy for expression tree libraries when they perform their own lowering steps. For example, translating a `using` statement into a `try` statement could use `Q.Flags.CompilerGenerated` passed to `TryInfo`.

## The checked and unchecked statements

The `checked` and `unchecked` statements are used to control the ***overflow checking context*** for integral-type arithmetic operations and conversions.

```antlr
checked_statement
    : 'checked' block
    ;

unchecked_statement
    : 'unchecked' block
    ;
```

The `checked` statement causes all expressions in the *block* to be evaluated in a checked context, and the `unchecked` statement causes all expressions in the *block* to be evaluated in an unchecked context.

The `checked` and `unchecked` statements are precisely equivalent to the `checked` and `unchecked` operators ([The checked and unchecked operators](expressions.md#the-checked-and-unchecked-operators)), except that they operate on blocks instead of expressions.

> __Expression Tree Conversion Translation Steps__
>
> Translation of a *checked_statement* or an *unchecked_statement* to a query expression tree produces a compile-time error.
>
> For purposes of conversion to a generalized expression tree, `Q.Flags.CheckedContext` flags are passed to generalized expression tree factory invocations for expressions occurring within a `checked` context, and a node is constructed to retain the syntactic structure of the original code.
>
> First, construct a `blockExpr` expression representing the translation of the block to an expression tree. Conversion to an expression tree then proceeds as follows.
>
> A *checked_statement* is translated into
>
> ```csharp
> Q.Checked(info, blockExpr)
> ```
>
> where `info` is
>
> ```csharp
> Q.CheckedInfo(default(Q.Flags))
> ```
>
> An *unchecked_statement* is translated into
>
> ```csharp
> Q.Unchecked(info, blockExpr)
> ```
>
> where `info` is
>
> ```csharp
> Q.UncheckedInfo(default(Q.Flags))
> ```

## The lock statement

The `lock` statement obtains the mutual-exclusion lock for a given object, executes a statement, and then releases the lock.

```antlr
lock_statement
    : 'lock' '(' expression ')' embedded_statement
    ;
```

The expression of a `lock` statement must denote a value of a type known to be a *reference_type*. No implicit boxing conversion ([Boxing conversions](conversions.md#boxing-conversions)) is ever performed for the expression of a `lock` statement, and thus it is a compile-time error for the expression to denote a value of a *value_type*.

A `lock` statement of the form
```csharp
lock (x) ...
```
where `x` is an expression of a *reference_type*, is precisely equivalent to
```csharp
bool __lockWasTaken = false;
try {
    System.Threading.Monitor.Enter(x, ref __lockWasTaken);
    ...
}
finally {
    if (__lockWasTaken) System.Threading.Monitor.Exit(x);
}
```
except that `x` is only evaluated once.

While a mutual-exclusion lock is held, code executing in the same execution thread can also obtain and release the lock. However, code executing in other threads is blocked from obtaining the lock until the lock is released.

Locking `System.Type` objects in order to synchronize access to static data is not recommended. Other code might lock on the same type, which can result in deadlock. A better approach is to synchronize access to static data by locking a private static object. For example:
```csharp
class Cache
{
    private static readonly object synchronizationObject = new object();

    public static void Add(object x) {
        lock (Cache.synchronizationObject) {
            ...
        }
    }

    public static void Remove(object x) {
        lock (Cache.synchronizationObject) {
            ...
        }
    }
}
```

> __Expression Tree Conversion Translation Steps__
>
> Translation of a *lock_statement* to a query expression tree produces a compile-time error. When converted to a generalized expression tree, conversion proceeds as follows.
>
> First, construct an expression `expr` representing the conversion of the lock object expression to an expression tree, and construct an expression `body` representing the conversion of the embedded statement to an expression tree. The lock statement is then translated into
>
> ```csharp
> Q.Lock(info, expr, body)
> ```
>
> where `info` is
>
> ```csharp
> Q.LockInfo(default(Q.Flags), enter, exit)
> ```
>
> where `enter` and `exit` are expressions of type `MethodInfo` representing the `System.Threading.Monitor.Enter` and `System.Threading.Monitor.Exit` methods, respectively.
>
> Note that information about the bound methods is passed to the `LockInfo` factory method. This reduces the burden on the expression tree library to locate these methods using reflection. Furthermore, the use of a `LockInfo` factory method enables alternative bindings should the specification of lock statements be relaxed in the future, at which point overloads could be added.
>
> ***TODO***
> * Specifying `enter` and `exit` is subject to debate. Does expression tree library convenience (of having these methods passed) outweigh information about lowering steps getting into expression nodes?
>   * Many nodes carry information about binding, e.g. method calls. In these places, the information is required to capture the bound target unambiguously.
>   * `Enter` and `Exit` are assumed to be built-in here, but strictly speaking the translation described in the specification does not specify the method signature of the target methods, nor whether it can bind to a `System.Threading.Monitor` type declared in a place other than the BCL.
>     * The binding target is not unambiguous according to the letter of the specification.
>     * If `Enter` and `Exit` were to have many overloads, the translation steps here imply that overload resolution is used to find the best one. This is already the case, because `Enter` has two overloads.
>       * Do we want the burden to be put on the expression tree libraries to pick the right one (using brittle reflection logic)?
>       * Is it possible that future versions of the BCL have better overloads for these methods, in particular `Enter`? For example, an addition of `void Enter<T>(T, ref bool)` would just bind fine for any `T` that is not equal to `object`.
>   * My bias is towards keeping these methods here. Less work for the expression library, more flexibility going forward, the specification is clear about the binding to these methods, and the way things are described in the specification implies there's regular overload resolution and binding for method invocation expressions targeting these methods.

## The using statement

The `using` statement obtains one or more resources, executes a statement, and then disposes of the resource.

```antlr
using_statement
    : 'using' '(' resource_acquisition ')' embedded_statement
    ;

resource_acquisition
    : local_variable_declaration
    | expression
    ;
```

A ***resource*** is a class or struct that implements `System.IDisposable`, which includes a single parameterless method named `Dispose`. Code that is using a resource can call `Dispose` to indicate that the resource is no longer needed. If `Dispose` is not called, then automatic disposal eventually occurs as a consequence of garbage collection.

If the form of *resource_acquisition* is *local_variable_declaration* then the type of the *local_variable_declaration* must be either `dynamic` or a type that can be implicitly converted to `System.IDisposable`. If the form of *resource_acquisition* is *expression* then this expression must be implicitly convertible to `System.IDisposable`.

Local variables declared in a *resource_acquisition* are read-only, and must include an initializer. A compile-time error occurs if the embedded statement attempts to modify these local variables (via assignment or the `++` and `--` operators) , take the address of them, or pass them as `ref` or `out` parameters.

A `using` statement is translated into three parts: acquisition, usage, and disposal. Usage of the resource is implicitly enclosed in a `try` statement that includes a `finally` clause. This `finally` clause disposes of the resource. If a `null` resource is acquired, then no call to `Dispose` is made, and no exception is thrown. If the resource is of type `dynamic` it is dynamically converted through an implicit dynamic conversion ([Implicit dynamic conversions](conversions.md#implicit-dynamic-conversions)) to `IDisposable` during acquisition in order to ensure that the conversion is successful before the usage and disposal.

A `using` statement of the form
```csharp
using (ResourceType resource = expression) statement
```
corresponds to one of three possible expansions. When `ResourceType` is a non-nullable value type, the expansion is
```csharp
{
    ResourceType resource = expression;
    try {
        statement;
    }
    finally {
        ((IDisposable)resource).Dispose();
    }
}
```

Otherwise, when `ResourceType` is a nullable value type or a reference type other than `dynamic`, the expansion is
```csharp
{
    ResourceType resource = expression;
    try {
        statement;
    }
    finally {
        if (resource != null) ((IDisposable)resource).Dispose();
    }
}
```

Otherwise, when `ResourceType` is `dynamic`, the expansion is
```csharp
{
    ResourceType resource = expression;
    IDisposable d = (IDisposable)resource;
    try {
        statement;
    }
    finally {
        if (d != null) d.Dispose();
    }
}
```

In either expansion, the `resource` variable is read-only in the embedded statement, and the `d` variable is inaccessible in, and invisible to, the embedded statement.

An implementation is permitted to implement a given using-statement differently, e.g. for performance reasons, as long as the behavior is consistent with the above expansion.

A `using` statement of the form
```csharp
using (expression) statement
```
has the same three possible expansions. In this case `ResourceType` is implicitly the compile-time type of the `expression`, if it has one. Otherwise the interface `IDisposable` itself is used as the `ResourceType`. The `resource` variable is inaccessible in, and invisible to, the embedded statement.

When a *resource_acquisition* takes the form of a *local_variable_declaration*, it is possible to acquire multiple resources of a given type. A `using` statement of the form
```csharp
using (ResourceType r1 = e1, r2 = e2, ..., rN = eN) statement
```
is precisely equivalent to a sequence of nested `using` statements:
```csharp
using (ResourceType r1 = e1)
    using (ResourceType r2 = e2)
        ...
            using (ResourceType rN = eN)
                statement
```

The example below creates a file named `log.txt` and writes two lines of text to the file. The example then opens that same file for reading and copies the contained lines of text to the console.
```csharp
using System;
using System.IO;

class Test
{
    static void Main() {
        using (TextWriter w = File.CreateText("log.txt")) {
            w.WriteLine("This is line one");
            w.WriteLine("This is line two");
        }

        using (TextReader r = File.OpenText("log.txt")) {
            string s;
            while ((s = r.ReadLine()) != null) {
                Console.WriteLine(s);
            }

        }
    }
}
```

Since the `TextWriter` and `TextReader` classes implement the `IDisposable` interface, the example can use `using` statements to ensure that the underlying file is properly closed following the write or read operations.

> __Expression Tree Conversion Translation Steps__
>
> Translation of a *using_statement* to a query expression tree produces a compile-time error. When converted to a generalized expression tree, it is translated into:
>
> ```csharp
> Q.Using(info, resourceExpr, stmtExpr)
> ```
>
> where `resourceExpr` is an expression representing the translation of *resource_acquisition* to an expression tree, `stmtExpr` is an expression representing the translation of *embedded_statement* to an expression tree, and `info` is
>
> ```csharp
> Q.UsingInfo(default(Q.Flags), convert)
> ```
>
> where `convert` is an expression representing the conversion of the resource(s) to `IDisposable`, as described above, constructed using the `Q.ConvertInfo` factory as described in [Cast expressions](expression.md#cast-expressions). Note that this conversion may describe a dynamic conversion.
>
> ***TODO***
> * C# 7 adds scoped locals to the bound node, due to declaration expressions. These would be passed to the `UsingInfo` factory method as a third argument supplied with the result of the `ScopeInfo` factory method.
> * Lowering steps are left to the expression tree library, e.g. for purposes of expression tree evaluation.
>   * The C# compiler performs some non-trivial optimizations for structs using `constrained` virtual calls which are covered by the specification stating "*An implementation is permitted to implement a given using-statement differently, e.g. for performance reasons, as long as the behavior is consistent with the above expansion.*".
>   * Should any information about the suggested lowering steps make it into the expression tree nodes? Likely not, given that the spec is non-prescriptive about any such details beyond the expansions to `try..finally..` constructs.

## The yield statement

The `yield` statement is used in an iterator block ([Blocks](statements.md#blocks)) to yield a value to the enumerator object ([Enumerator objects](classes.md#enumerator-objects)) or enumerable object ([Enumerable objects](classes.md#enumerable-objects)) of an iterator or to signal the end of the iteration.

```antlr
yield_statement
    : 'yield' 'return' expression ';'
    | 'yield' 'break' ';'
    ;
```

`yield` is not a reserved word; it has special meaning only when used immediately before a `return` or `break` keyword. In other contexts, `yield` can be used as an identifier.

There are several restrictions on where a `yield` statement can appear, as described in the following.

*  It is a compile-time error for a `yield` statement (of either form) to appear outside a *method_body*, *operator_body* or *accessor_body*
*  It is a compile-time error for a `yield` statement (of either form) to appear inside an anonymous function.
*  It is a compile-time error for a `yield` statement (of either form) to appear in the `finally` clause of a `try` statement.
*  It is a compile-time error for a `yield return` statement to appear anywhere in a `try` statement that contains any `catch` clauses.

The following example shows some valid and invalid uses of `yield` statements.

```csharp
delegate IEnumerable<int> D();

IEnumerator<int> GetEnumerator() {
    try {
        yield return 1;        // Ok
        yield break;           // Ok
    }
    finally {
        yield return 2;        // Error, yield in finally
        yield break;           // Error, yield in finally
    }

    try {
        yield return 3;        // Error, yield return in try...catch
        yield break;           // Ok
    }
    catch {
        yield return 4;        // Error, yield return in try...catch
        yield break;           // Ok
    }

    D d = delegate { 
        yield return 5;        // Error, yield in an anonymous function
    }; 
}

int MyMethod() {
    yield return 1;            // Error, wrong return type for an iterator block
}
```

An implicit conversion ([Implicit conversions](conversions.md#implicit-conversions)) must exist from the type of the expression in the `yield return` statement to the yield type ([Yield type](classes.md#yield-type)) of the iterator.

A `yield return` statement is executed as follows:

*  The expression given in the statement is evaluated, implicitly converted to the yield type, and assigned to the `Current` property of the enumerator object.
*  Execution of the iterator block is suspended. If the `yield return` statement is within one or more `try` blocks, the associated `finally` blocks are not executed at this time.
*  The `MoveNext` method of the enumerator object returns `true` to its caller, indicating that the enumerator object successfully advanced to the next item.

The next call to the enumerator object's `MoveNext` method resumes execution of the iterator block from where it was last suspended.

A `yield break` statement is executed as follows:

*  If the `yield break` statement is enclosed by one or more `try` blocks with associated `finally` blocks, control is initially transferred to the `finally` block of the innermost `try` statement. When and if control reaches the end point of a `finally` block, control is transferred to the `finally` block of the next enclosing `try` statement. This process is repeated until the `finally` blocks of all enclosing `try` statements have been executed.
*  Control is returned to the caller of the iterator block. This is either the `MoveNext` method or `Dispose` method of the enumerator object.

Because a `yield break` statement unconditionally transfers control elsewhere, the end point of a `yield break` statement is never reachable.

> __Expression Tree Conversion Translation Steps__
>
> Translation of a *yield_statement* to a query expression tree produces a compile-time error.
>
> When a `yield return e;` statement is converted to a generalized expression tree, it is translated into:
>
> ```csharp
> Q.YieldReturn(info, expr)
> ```
>
> where `expr` is an expression representing the translation of `(T)(e)` to an expression tree if a non-trivial implicit conversion of `e` to yield type `T` ([Yield type](classes.md#yield-type)) is required, where the cast expression is classified as compiler-generated, or representing the translation of `e` to an expression tree otherwise, and `info` is
>
> ```csharp
> Q.YieldReturnInfo(default(Q.Flags))
> ```
>
> When a `yield break;` statement is converted to a generalized expression tree, it is translated into:
>
> ```csharp
> Q.YieldBreak(info)
> ```
>
> where `info` is
>
> ```csharp
> Q.YieldBreakInfo(default(Q.Flags))
> ```
>
> ***TOD***:
> * Anonymous function expressions do not support iterator blocks today; this whole section is predicated on a lift of this restriction.
> * If iterators are supported, we have a couple more things to evaluate:
>   * Would builders exists, similar to `async` methods with task-like types? If so, the `Lambda` factory would likely accept an `IteratorLambdaInfo` for its `info` parameter, which refers to the builder type.
>   * Should `info` nodes on `yield` statements carry any information about the enumerator (or enumerator-like type) being constructed? It doesn't seem like it; all this info would likely go on the lambda parent.
>   * Should the implicit conversion of a `yield return` expression operand be represented as a `ConvertInfo` child to `YieldReturnInfo`? Likely not; we don't do that either for the rhs of an assignment (and only for things such as "left conversions").
>   * Because `yield` statements perform control flow, should their info objects be parameterized on some type of "anchor" that can be used to traverse to the parent iterator block? This is similar to the use of label target nodes on `break` and `continue`.
>     * Note that use of references between `info` nodes makes the final expression "tree" really an object graph. This is no different from the the existing query expression trees where `Parameter` occurs both in declaration and use sites. Either way, we should classify leaf nodes to make them explicit (variables, labels, others?).

## Scopes in expression trees

Statements that introduce a scope for local variables are converted to expression trees using a ***scope info*** expression tree node.

When a statement establishes a scope for one or more local variables, each variable (in lexical order based on the declarations of these variables) is translated into an expression tree as described in [Local variables](variables.md#local-variables). This results in the introduction of local variables `varExpr_1` to `varExpr_N` that have been assigned from `Q.Variable` invocations.

A scope info expression tree node is then constructed from

```csharp
Q.ScopeInfo(varExpr_1, ..., varExpr_N)
```

For example, consider the following block:

```csharp
{
    Console.WriteLine(1);
    int x;
    Console.WriteLine(2);
    x = 3;
    Console.WriteLine(x);
    int y = 4;
    Console.WriteLine(y);
}
```

This block declares variables `x` and `y`, in that order. These variables are translated to expression trees using two local variables:

```csharp
var t0 = Q.Variable(Q.VariableInfo(default(Q.Flags), typeof(int), "x"));
var t1 = Q.Variable(Q.VariableInfo(default(Q.Flags), typeof(int), "y"));
```

where `t0` and `t1` are compiler-generated identifiers that are otherwise invisible. A scope info for the block is then constructed:

```csharp
var t2 = Q.ScopeInfo(t0, t1);
```

where `t2` is a compiler-generated identifier that is otherwise invisible. Conversion of the block to an expression tree then proceeds as follows:

```csharp
Q.Block(
    Q.BlockInfo(default(Q.Flags), t2),
    Q.StatementList(
        Q.Call(...),
        Q.VariableDeclaration(t0),
        Q.Call(...),
        Q.Assign(...),
        Q.Call(...),
        Q.VariableDeclaration(t1, ...)
        Q.Call(...)
    )
)
```

This translation is similar to C-style variable declarations at the top of a block, but retains the lexical structure of the program.

Expression tree nodes representing variables can be referenced in various places, including scope info nodes, lambda expression parameter lists, variable declaration nodes, and expressions.

It is left to expression tree libraries to ensure that some form of identity exists to associate the various use sites of `Q.Variable` and `Q.Parameter` nodes, if the expression tree library has such a requirement. The most straightforward choice is the use of reference equality, though the language specification does not rely on this. For example, variable nodes could be implemented as structs encapsulating some notion of identity.