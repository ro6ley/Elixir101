# Pattern Matching
In Elixir the `=` sign is a match operator used in pattern matching.

It works by determining if the left-hand side can be made equal to the right-hand side.

It succeeds if Elixir can find a way of making the left-hand side equal to the right-hand side.

ELixir will only change the value of the variable on the left hand-side of the `=` sign.

A pattern (the left-side) is matched if the values (the right side) have the same structure and if each term in the pattern can be matched to the corresponding term in the values.

Once a variable has been bound to a value in the matching process, it keeps that value for the remainder of the match.

However, a variable can be bound to a new value in a subsequent match and its current value does not participate in the new match.

Prefix a variable with the `^` sign if you want Elixir to use the exisiting value of a variable in the pattern. This also works if the variable is a component of a pattern.

> Look at the `=` sign the same way it's used in algebra. When you say `x = a + 1`, you are not assigning `a + 1` to `x`, instead you are simply asserting that the two expressions have the same value.
