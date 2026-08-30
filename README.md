# Open Source Contributions

List of all my open source contributions.

**Total merged: 2**

---

## openrewrite/rewrite-static-analysis (2 merged)

| PR | Title | Date |
|----|-------|------|
| [#1022](https://github.com/openrewrite/rewrite-static-analysis/pull/1022) | `NoValueOfOnStringType`: keep `String.valueOf` when the argument can be null | 2026-08-30 |
| [#998](https://github.com/openrewrite/rewrite-static-analysis/pull/998) | `UnnecessaryExplicitTypeArguments`: retain witness when enclosing method has dependent type parameters | 2026-08-14 |

Fixed [#1009](https://github.com/openrewrite/rewrite-static-analysis/issues/1009), where the recipe stripped `String.valueOf` from nullable arguments, turning `"null"` into a `NullPointerException` at runtime.

Fixed [#783](https://github.com/openrewrite/rewrite-static-analysis/issues/783), a 9-month-old bug where the recipe removed a load-bearing explicit type witness from an argument to a method with interdependent type parameters (`<T, S extends T>`), producing code that no longer compiled.
