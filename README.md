# Open Source Contributions

List of all my open source contributions.

**Total merged: 4**

---

## openrewrite/rewrite-static-analysis (4 merged)

| PR | Title | Date |
|----|-------|------|
| [#1035](https://github.com/openrewrite/rewrite-static-analysis/pull/1035) | `NoDoubleBraceInitialization`: don't retarget invocations inside nested anonymous classes | 2026-09-17 |
| [#1008](https://github.com/openrewrite/rewrite-static-analysis/pull/1008) | `FallThrough`: don't add a `break` after a guarded early exit | 2026-09-17 |
| [#1022](https://github.com/openrewrite/rewrite-static-analysis/pull/1022) | `NoValueOfOnStringType`: keep `String.valueOf` when the argument can be null | 2026-08-30 |
| [#998](https://github.com/openrewrite/rewrite-static-analysis/pull/998) | `UnnecessaryExplicitTypeArguments`: retain witness when enclosing method has dependent type parameters | 2026-08-14 |

Fixed [#352](https://github.com/openrewrite/rewrite-static-analysis/issues/352), where method calls inside a nested double-brace initializer were retargeted onto the outer variable, producing code that no longer compiled.

Fixed [#460](https://github.com/openrewrite/rewrite-static-analysis/issues/460), where `FallThrough` inserted a `break` after an `if` whose then-branch already exited, turning an intentional fall-through into changed behavior. Also resolved the compile failure reported in [#229](https://github.com/openrewrite/rewrite-static-analysis/issues/229).

Fixed [#1009](https://github.com/openrewrite/rewrite-static-analysis/issues/1009), where the recipe stripped `String.valueOf` from nullable arguments, turning `"null"` into a `NullPointerException` at runtime.

Fixed [#783](https://github.com/openrewrite/rewrite-static-analysis/issues/783), a 9-month-old bug where the recipe removed a load-bearing explicit type witness from an argument to a method with interdependent type parameters (`<T, S extends T>`), producing code that no longer compiled.
