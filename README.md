# Open Source Contributions

List of all my open source contributions.

**Total merged: 9**

---

## openrewrite/rewrite (3 merged)

| PR | Title | Date |
|----|-------|------|
| [#8964](https://github.com/openrewrite/rewrite/pull/8964) | Preserve Java import comments through ordering and folding | 2026-09-29 |
| [#8904](https://github.com/openrewrite/rewrite/pull/8904) | Fix merging duplicate nested YAML sections after unfolding | 2026-09-22 |
| [#8664](https://github.com/openrewrite/rewrite/pull/8664) | Gradle: disable the no-repositories metadata test only when a mirror is injected | 2026-09-21 |

**[#8964](https://github.com/openrewrite/rewrite/pull/8964)**: Fixed [#6143](https://github.com/openrewrite/rewrite/issues/6143), an 11-month-old bug where `OrderImports` scrambled inline comments and dragged top-of-file copyright headers into the middle of the import list during alphabetical sorting. Because OpenRewrite's syntax tree stores whitespace and comments on the following line, sorting imports inadvertently displaced any comments attached underneath them. This PR decoupled section headers, tracked trailing comments by import ID using cursor-scoped transient messaging, and safely restored them in reverse order without altering AST referential identity. Since import reordering runs across virtually every major automated recipe (including Spring Boot upgrades and Java 17/21 migrations), this fix prevents automated refactorings from silently corrupting legal headers and developer comments across thousands of enterprise repositories while eliminating manual review churn in CI.

**[#8904](https://github.com/openrewrite/rewrite/pull/8904)**: Fixed [#8838](https://github.com/openrewrite/rewrite/issues/8838), where unfolding nested YAML properties left duplicate sibling mapping keys because `MergeDuplicateSectionsVisitor` mistook indentation whitespace for comments and skipped the merge pass. By using `StringUtils.isBlank` to properly recognize whitespace-only prefixes, duplicate YAML blocks now merge cleanly into a single unified section. This prevents invalid duplicate keys in configuration files, eliminating fatal runtime parsing crashes in strict loaders like SnakeYAML when teams apply automated configuration refactoring recipes.

**[#8664](https://github.com/openrewrite/rewrite/pull/8664)**: Replaced a blanket `@Disabled` annotation on `UpgradeDependencyVersionTest.cannotDownloadMetaDataWhenNoRepositoriesAreDefined` (carried since [#7566](https://github.com/openrewrite/rewrite/pull/7566)) with a conditional `@DisabledIfEnvironmentVariable`. The test asserts that build files without repositories produce a clean "Unable to download metadata" marker, but it failed under CI environments where internal Artifactory mirrors prepend repositories into Gradle builds. By replacing the blanket disable with an environment guard, this change re-enabled critical regression coverage for local contributors and downstream builds without a mirror, ensuring this failure path remains actively verified instead of dead across the test suite.

---

## openrewrite/rewrite-static-analysis (6 merged)

| PR | Title | Date |
|----|-------|------|
| [#1064](https://github.com/openrewrite/rewrite-static-analysis/pull/1064) | `UnwrapRepeatableAnnotations`: only unwrap container annotations | 2026-09-17 |
| [#1036](https://github.com/openrewrite/rewrite-static-analysis/pull/1036) | `DeclarationSiteTypeVariance`: skip `@Override` methods when the supertype is unresolved | 2026-09-17 |
| [#1035](https://github.com/openrewrite/rewrite-static-analysis/pull/1035) | `NoDoubleBraceInitialization`: don't retarget invocations inside nested anonymous classes | 2026-09-17 |
| [#1008](https://github.com/openrewrite/rewrite-static-analysis/pull/1008) | `FallThrough`: don't add a `break` after a guarded early exit | 2026-09-17 |
| [#1022](https://github.com/openrewrite/rewrite-static-analysis/pull/1022) | `NoValueOfOnStringType`: keep `String.valueOf` when the argument can be null | 2026-08-30 |
| [#998](https://github.com/openrewrite/rewrite-static-analysis/pull/998) | `UnnecessaryExplicitTypeArguments`: retain witness when enclosing method has dependent type parameters | 2026-08-14 |

**[#1064](https://github.com/openrewrite/rewrite-static-analysis/pull/1064)**: Fixed [#389](https://github.com/openrewrite/rewrite-static-analysis/issues/389) in `UnwrapRepeatableAnnotations`, where the recipe aggressively unwrapped any annotation containing a repeatable annotation attribute, discarding the outer container and all its other elements. This PR restricted the recipe to only unwrap true repeatable container annotations, ensuring critical enterprise metadata—such as Jakarta Persistence / JPA `@NamedEntityGraph` mappings—remains fully intact and preventing silent runtime entity mapping failures in production applications.

**[#1036](https://github.com/openrewrite/rewrite-static-analysis/pull/1036)**: Fixed [#277](https://github.com/openrewrite/rewrite-static-analysis/issues/277), a 2.5-year-old bug in `DeclarationSiteTypeVariance` where the recipe blindly added `? super` or `? extends` wildcards to `@Override` methods even when the supertype lived in external compiled bytecode outside the project's source set. By skipping overrides whose supertypes are unresolved, this fix prevents the recipe from generating incompatible method signatures that violate Java inheritance rules, ensuring automated type variance migrations no longer produce uncompilable code against external libraries.

**[#1035](https://github.com/openrewrite/rewrite-static-analysis/pull/1035)**: Fixed [#352](https://github.com/openrewrite/rewrite-static-analysis/issues/352) in `NoDoubleBraceInitialization`, where refactoring nested double-brace initializers improperly resolved scoping and retargeted method invocations inside inner anonymous classes onto the outer variable. By checking invocation scopes, this change ensures that methods belonging to nested anonymous classes are preserved, preventing the anti-pattern cleanup recipe from generating illegal method calls that cause compilation failures.

**[#1008](https://github.com/openrewrite/rewrite-static-analysis/pull/1008)**: Fixed [#460](https://github.com/openrewrite/rewrite-static-analysis/issues/460) and compile failures reported in [#229](https://github.com/openrewrite/rewrite-static-analysis/issues/229) in `FallThrough`, where the recipe inserted an unnecessary `break` statement after an `if` block whose then-branch already exited (via `return` or `throw`). The fix inspects the control flow of guarded early exits before inserting breaks, preventing the recipe from altering intentional fall-through logic into changed runtime behavior or generating unreachable code compilation errors.

**[#1022](https://github.com/openrewrite/rewrite-static-analysis/pull/1022)**: Fixed [#1009](https://github.com/openrewrite/rewrite-static-analysis/issues/1009) in `NoValueOfOnStringType`, where the recipe stripped `String.valueOf()` from string concatenation arguments without checking whether the argument was nullable. By retaining `String.valueOf` whenever an expression cannot be proven non-null, this change prevents automated cleanup recipes from turning safe `"null"` string conversions into catastrophic runtime `NullPointerException`s in production.

**[#998](https://github.com/openrewrite/rewrite-static-analysis/pull/998)**: Fixed [#783](https://github.com/openrewrite/rewrite-static-analysis/issues/783), a 9-month-old bug in `UnnecessaryExplicitTypeArguments` where the recipe stripped explicit generic type witnesses from method invocations with interdependent type bounds (`<T, S extends T>`). The fix retains the explicit type witness when enclosing methods require it to resolve dependent parameters, preventing javac type-inference ambiguity failures and ensuring that removing redundant type arguments never breaks compilation.
