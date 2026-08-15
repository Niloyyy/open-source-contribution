# Open Source Contributions

List of all my open source contributions.

**Total merged: 1**

---

## openrewrite/rewrite-static-analysis (1 merged)

| PR | Title | Date |
|----|-------|------|
| [#998](https://github.com/openrewrite/rewrite-static-analysis/pull/998) | `UnnecessaryExplicitTypeArguments`: retain witness when enclosing method has dependent type parameters | 2026-08-14 |

Fixed [#783](https://github.com/openrewrite/rewrite-static-analysis/issues/783), a 9-month-old bug where the recipe removed a load-bearing explicit type witness from an argument to a method with interdependent type parameters (`<T, S extends T>`), producing code that no longer compiled.
