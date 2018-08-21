### This document lists known breaking changes in Roslyn between the last VS2017 update (15.*) and C# 8.0 (16.0)

*Breaks are formatted with a monotonically increasing numbered list to allow them to referenced via shorthand (i.e., "known break #1").
Each entry should include a short description of the break, followed by either a link to the issue describing the full details of the break or the full details of the break inline.*

1. It is no longer permitted to use a constant named `_` as a constant pattern. This change is made to permit the syntax `_` to be used for a *discard pattern*. See https://github.com/dotnet/csharplang/blob/master/meetings/2017/LDM-2017-11-20.md
  ``` c#
      const int _ = 1;
      switch (1) { case _: break; } // error: A constant named '_' cannot be used as a pattern.
  ```

2. A warning is issued when an *is-type* expression tests an expression against a type named `_`. This syntax could be confused with a use of the *discard pattern*.
  ``` c#
      class _ { }

      if (o is _) // warning: The name '_' refers to the type '_', not the discard pattern. Use '@_' for the type, or 'var _' to discard.
  ```

3. In C# 8.0, the parentheses of a switch statement are optional when the expression being switched on is a tuple expression, because the tuple expression has its own parentheses:
  ``` c#
      switch (a, b)
  ```
   Due to this the `OpenParenToken` and `CloseParenToken` fields of a `SwitchStatementSyntax` node may now sometimes be empty.

4. In an *is-pattern-expression*, a warning is now issued when a constant expression does not match the provided pattern because of its value. Such code was previously accepted but gave no warning. For example
  ``` c#
      if (3 is 4) // warning: the given expression never matches the provided pattern.
  ```
  We also issue a warning when a constant expression *always* matches a constant pattern in an *is-pattern-expression*. For example
  ``` c#
      if (3 is 3) // warning: the given expression always matches the provided constant.
  ```
  Other cases of the pattern always matching (e.g. `e is var t`) do not trigger a warning, even when they are known by the compiler to produce an invariant result.

5. https://github.com/dotnet/roslyn/issues/26098 In C# 8, we give a warning when an is-type expression is always `false` because the input type is an open class type and the type it is tested against is a value type:
  ``` c#
    class C<T> { }
    void M<T>(C<T> x)
    {
        if (x is int) { } // warning: the given expression is never of the provided ('int') type.
    }
  ```
  previously, we gave the warning only in the reverse case
  ``` c#
    void M<T>(int x)
    {
        if (x is C<T>) { } // warning: the given expression is never of the provided ('C<T>') type.
    }
  ```
