# Open Source Contributions

List of all my open source contributions.

**Total merged: 20**

---

## rust-lang/cargo (2 merged)

### [#17702](https://github.com/rust-lang/cargo/pull/17702) — `fix(doc)`: do not suggest cargo fix for rustdoc warnings
*Merged: 2026-11-09*

When `cargo doc` runs `rustdoc` on a crate that emits compiler or lint warnings with machine-applicable suggestions (such as `rustdoc::bare_urls`), Cargo's warning summary in `DrainState::report_warning_count` previously prompted users with `(run cargo fix --lib -p <pkg> to apply <n> suggestion)`. However, `cargo fix` only invokes `rustc` in check mode and does not execute `rustdoc` (tracked in [#13025](https://github.com/rust-lang/cargo/issues/13025)), meaning running the suggested command produced no effect and confused developers. This PR updated `DrainState::report_warning_count` in `src/compiler/job_queue/mod.rs` to exclude `rustdoc` compilation units (`!unit.mode.is_doc() && !unit.mode.is_doc_scrape()`), matching how Cargo distinguishes `rustdoc` from `rustc` invocations across the compiler pipeline. This prevents misleading `cargo fix` prompts when generating documentation while continuing to offer automated fix suggestions for standard `rustc` compiler units.

---

### [#17698](https://github.com/rust-lang/cargo/pull/17698) — `fix(lockfile)`: sort unused patches in Cargo.lock
*Merged: 2026-11-07*

When Cargo serializes resolution state into `Cargo.lock`, it explicitly sorts `[[package]]` entries and individual package dependencies to ensure deterministic lockfile output. However, in issue [#17375](https://github.com/rust-lang/cargo/issues/17375), `[[patch.unused]]` entries were collected directly from `Resolve::unused_patches()` without sorting. While patch entries normally appeared in alphabetical order when reading from a warm cache, asynchronous network and git queries polled via `FuturesUnordered` yielded results in unpredictable completion order on cold `~/.cargo` caches, or when combining configuration and manifest patches. Running `cargo update` on a cold cache produced out-of-order `[[patch.unused]]` entries, only for a second run on a warm cache to reorder them back, creating non-deterministic lockfile output and noisy git diff churn across different developer machines and CI runners. This PR sorted `unused` entries during lockfile serialization in `src/resolver/encode.rs` using `TomlLockfileDependency`'s `Ord` implementation before constructing the patch table. This guarantees deterministic, reproducible `Cargo.lock` serialization across all cache states and network conditions without altering Cargo's internal resolution storage.

---

## openrewrite/rewrite (5 merged)

### [#9536](https://github.com/openrewrite/rewrite/pull/9536) — Preserve outer class type arguments when changing inner type in ChangeType
*Merged: 2026-11-06*

The `ChangeType` recipe automates renaming classes and updating their imports and type references across codebases. However, in issue [#9291](https://github.com/openrewrite/rewrite/issues/9291), when an inner class was referenced through a parameterized outer class (such as `OldOuter<String>.Registry` $\to$ `NewOuter<String>.Registry2`), the recipe matched the inner class reference and replaced the entire expression with a raw target type string. This wiped out the enclosing class's generic type arguments (`<String>`) and imported the inner class directly, stripping outer class qualification and generating invalid code that failed Java generic type inference. This PR introduced `updateParameterizedFieldAccess` to update enclosing parameterized types in place, preserve their type arguments while checking for nested type changes, and properly update outer class imports across multiple levels of nesting. This guarantees that automated type migration recipes cleanly preserve generic parameters on enclosing classes, preventing type-inference breakage and compilation errors in heavily parameterized enterprise code.

---

### [#9689](https://github.com/openrewrite/rewrite/pull/9689) — Support single-element package names in ChangePackage
*Merged: 2026-11-06*

The `ChangePackage` recipe renames packages across a codebase, updating both package declarations and all referencing imports, fully qualified types, and Javadoc tags. However, in issue [#4142](https://github.com/openrewrite/rewrite/issues/4142), when renaming a single-segment package (e.g., `somepkg` without dots), the recipe only updated the `package` declaration itself while leaving referencing files untouched. This happened because multi-segment packages are represented in the AST as nested field accesses, whereas single-segment package prefixes are root identifiers where `isFullyQualifiedClassReference` returned `false`. This PR updated `visitFieldAccess` to recognize and rewrite single-segment package identifiers in imports, star imports, fully qualified references, and Javadoc `@link` tags, while safely ignoring local variables or fields that share the package name and preserving non-recursive subpackage boundaries. This ensures that renaming packages with single-segment names updates all dependent references across the codebase, preventing broken imports and missing-symbol compilation errors.

---

### [#8964](https://github.com/openrewrite/rewrite/pull/8964) — Preserve Java import comments through ordering and folding
*Merged: 2026-09-29*

In OpenRewrite's syntax tree, comments and whitespace do not exist on standalone lines; they are attached as a prefix to whatever code appears immediately after them. Because of this design, whenever the `OrderImports` recipe sorted Java imports alphabetically, it accidentally dragged comments to the wrong lines. An inline comment next to an import would get attached to a completely unrelated import, and top-of-file copyright or license headers would get pulled right into the middle of the import list ([#6143](https://github.com/openrewrite/rewrite/issues/6143), an issue open for nearly a year). This PR decoupled top-level license headers before sorting, mapped trailing inline comments to their unique import IDs in memory, and used temporary cursor messages (following review feedback from creator Jonathan Schneider to avoid altering object identity) to restore all comments to their exact proper places after reordering. Also ensured comments are preserved above surviving wildcard (`*`) imports during folding. Because import ordering runs during almost every major automated recipe (such as Spring Boot upgrades and Java 17/21 migrations), this fix prevents automated refactoring from corrupting legal headers and developer notes across thousands of enterprise repositories, eliminating tedious manual review and cleanup in CI pull requests.

---

### [#8904](https://github.com/openrewrite/rewrite/pull/8904) — Fix merging duplicate nested YAML sections after unfolding
*Merged: 2026-09-22*

When automated recipes move or rename configuration properties in YAML files, OpenRewrite unfolds them into structured trees. However, if multiple properties shared the same parent section (such as `security:`), the visitor that was supposed to merge them mistook indentation spaces for comments and skipped the merge ([#8838](https://github.com/openrewrite/rewrite/issues/8838)), leaving duplicate identical keys sitting side-by-side in the same YAML block. This PR updated the comment-checking logic to use `StringUtils.isBlank` so the parser recognizes plain indentation whitespace and does not treat it as a comment, allowing duplicate nested sections to combine into one clean mapping. Strict YAML parsers (like SnakeYAML, used by Spring Boot) immediately crash at application startup if a configuration file contains duplicate keys. This fix guarantees that automated YAML migrations produce valid configuration files, preventing fatal runtime application crashes in production.

---

### [#8664](https://github.com/openrewrite/rewrite/pull/8664) — Gradle: disable the no-repositories metadata test only when a mirror is injected
*Merged: 2026-09-21*

The test `cannotDownloadMetaDataWhenNoRepositoriesAreDefined` verifies that OpenRewrite gracefully handles Gradle build files that have no repositories configured, expecting a clean "Unable to download metadata" warning. However, maintainers had placed a blanket `@Disabled` annotation on this test back in [#7566](https://github.com/openrewrite/rewrite/pull/7566) because the team's internal Artifactory mirror automatically injected repositories during CI, causing the test to fail. Because of this blanket disable, the test never ran anywhere—not in CI, and not for any local contributor. This PR replaced the blanket `@Disabled` annotation with a conditional `@DisabledIfEnvironmentVariable` guard that only skips the test when the internal mirror environment variables are active, matching how similar tests in the repository are handled. This restores active test coverage for open-source contributors and local builds without a mirror, ensuring that OpenRewrite's dependency metadata error-handling path remains thoroughly tested and verified against regressions.

---

## openrewrite/rewrite-static-analysis (13 merged)

### [#1179](https://github.com/openrewrite/rewrite-static-analysis/pull/1179) — `CollectionToArrayShouldHaveProperType`: handle implicit `this` receivers
*Merged: 2026-11-06*

The `CollectionToArrayShouldHaveProperType` recipe modernizes legacy `collection.toArray()` calls by passing a typed, zero-length array (e.g., `toArray(new String[0])`) to avoid costly array allocations and unsafe type casting. However, in issue [#1116](https://github.com/openrewrite/rewrite-static-analysis/issues/1116) (item C21), when `(String[]) toArray()` was called without an explicit receiver inside a `Collection` or `List` subclass (such as `class MyList extends ArrayList<String>`), the method call's target receiver was `null`. Passing this `null` receiver into `requireNonNull(methodInvocation.getSelect())` triggered an immediate `NullPointerException`, halting recipe execution. This PR updated the visitor to synthesize a temporary `this` receiver using the declaring class type so `JavaTemplate` can accurately compile and attribute the replacement call, and then cleared the select on the result to preserve the original implicit receiver syntax. This eliminates recipe crashes on collection subclasses and ensures type-safe array conversions work reliably across custom collection implementations.

---

### [#1178](https://github.com/openrewrite/rewrite-static-analysis/pull/1178) — `SimplifyRedundantLogicalExpression`: keep parentheses needed for precedence
*Merged: 2026-11-06*

The `SimplifyRedundantLogicalExpression` recipe cleans up duplicate boolean or bitwise subexpressions (e.g., simplifying `a && b && a` to `a && b`). However, in issue [#1116](https://github.com/openrewrite/rewrite-static-analysis/issues/1116) (item E4), whenever the recipe replaced duplicate compound expressions, it called `.unwrap()` directly on the replacement node. Because `.unwrap()` unconditionally strips surrounding parentheses, compound expressions with lower operator precedence than the outer expression lost their grouping—for instance, turning `return (x || y) && (x || y) && z;` into `return x || y && z;`, which evaluates as `x || (y && z)` and silently alters the program's runtime boolean logic. This PR replaced unconditional unwrapping with `ParenthesizeVisitor.maybeParenthesize`, which evaluates the surrounding AST context and operator precedence to keep parentheses whenever higher-precedence outer operators require them, while safely unwrapping only truly redundant grouping. This prevents automated logical simplification from introducing dangerous runtime logic bugs and guarantees semantic equivalence across Java, Kotlin, Groovy, and TypeScript codebases.

---

### [#1180](https://github.com/openrewrite/rewrite-static-analysis/pull/1180) — `InlineVariable`: avoid changing semantics when inlining before return or throw
*Merged: 2026-11-06*

The `InlineVariable` recipe simplifies code by inlining temporary variable declarations and assignments immediately preceding a `return` or `throw` statement. However, in issue [#1116](https://github.com/openrewrite/rewrite-static-analysis/issues/1116) (item C20), the recipe performed inlining without checking for subtle semantic changes: it inlined variables whose declared types differed from their initializers (such as `long v = 1; return v;` in an `Object`-returning method inlining to `return 1;`, which boxed into an `Integer` instead of `Long`), stripped target typing for lambdas and method references (turning `Runnable r = () -> {}; return r;` into an uncompilable `return () -> {};`), and inlined reassignments that were still read in enclosing `finally` or `catch` blocks. This PR introduced `canInline` to prevent inlining when expressions unwrap to lambdas/method references or perform implicit type conversions, and added `isReadInEnclosingCatchOrFinally` to ensure variables read during error handling or cleanup retain their assignments. This guarantees that automated variable inlining preserves exact runtime behavior, prevent silent boxing/widening bugs, and avoids introducing compilation errors.

---

### [#1161](https://github.com/openrewrite/rewrite-static-analysis/pull/1161) — `UseDiamondOperator`: keep type arguments of an anonymous class assigned to a wildcard type
*Merged: 2026-10-28*

The `UseDiamondOperator` recipe modernizes Java generic instantiations by replacing explicit type arguments with the diamond operator (`new ArrayList<>()`). In Java 9+, the diamond operator is also supported on anonymous class instantiations, but only when Java can cleanly infer concrete types. In issue [#1116](https://github.com/openrewrite/rewrite-static-analysis/issues/1116) (item C10), when an anonymous class was assigned to a variable with wildcard type bounds (such as `Consumer<?> c = new Consumer<String>() { ... };` or `Consumer<? extends CharSequence>`), the recipe stripped the explicit type into `new Consumer<>() { ... }`. Because the diamond infers `Consumer<Object>`, the Java compiler rejects the code with compilation errors stating that the anonymous class fails to implement the required method signatures. This PR added a check in `maybeRemoveParams` that skips adding diamond operators to anonymous classes whenever the target type has wildcard type arguments. This ensures that cleaning up generic instantiations never generates invalid wildcard inferences, eliminating compile-time method-override errors on Java 9+ codebases.

---

### [#1164](https://github.com/openrewrite/rewrite-static-analysis/pull/1164) — `ReplaceClassIsInstanceWithInstanceof`: skip incompatible types or primitives
*Merged: 2026-10-28*

The `ReplaceClassIsInstanceWithInstanceof` recipe modernizes reflection-based checks like `String.class.isInstance(obj)` into standard Java `obj instanceof String` expressions. However, while `Class.isInstance(obj)` returns `false` safely at runtime when types cannot be cast or when checking primitive values, Java language specification rules for `instanceof` require both types to be cast-convertible reference types. In issue [#1116](https://github.com/openrewrite/rewrite-static-analysis/issues/1116) (item C12), the recipe blindly transformed checks involving completely incompatible types (such as `Integer.class.isInstance(str)`, `String.class.isInstance(num)`, array types, or primitives like `hashcode()`), generating code that failed to compile with `incompatible types` or `required reference, found int` errors. This PR introduced comprehensive type-compatibility verification (`cannotBeInstanceOf`) that checks primitive types, array compatibility, class inheritance, interface implementations, and generic bounds before rewriting. This guarantees that reflection checks are only converted to `instanceof` when types are genuinely cast-compatible, preventing automated static analysis refactorings from introducing fatal compilation errors.

---

### [#1153](https://github.com/openrewrite/rewrite-static-analysis/pull/1153) — `ReplaceLambdaWithMethodReference`: fix qualified `this` resolution
*Merged: 2026-10-28*

The `ReplaceLambdaWithMethodReference` recipe converts single-statement lambdas into clean method references (e.g., `s -> log(s)` to `this::log`). When an instance method is called on an enclosing outer class, the recipe must qualify `this` with the enclosing class name (such as `Outer.this::log`). In issue [#1116](https://github.com/openrewrite/rewrite-static-analysis/issues/1116) (item C13), the cursor lookup incorrectly stopped at intermediate `J.Lambda` expressions and emitted invalid qualifications like `Base.this::log` using the superclass name instead of the actual enclosing class name (`A.this::log`), producing compilation errors like `error: not an enclosing class: Base`. This PR corrected the cursor resolution so it ignores lambda scopes and walks outward to find the nearest enclosing class or anonymous class assignable to the method's declaring type, correctly prefixing `this` with the enclosing class. This fixes method reference conversions inside nested lambdas and inner classes, ensuring that modernizing functional code produces valid Java scoping without breaking compilation.

---

### [#1132](https://github.com/openrewrite/rewrite-static-analysis/pull/1132) — `UseJavaStyleArrayDeclarations`: only rewrite declarations with matching dimensions
*Merged: 2026-10-26*

The `UseJavaStyleArrayDeclarations` recipe converts C-style array brackets placed after a variable name (`int a[];`) into standard Java-style brackets on the type (`int[] a;`). However, in issue [#1116](https://github.com/openrewrite/rewrite-static-analysis/issues/1116), when a single statement declared multiple variables with different array dimensions—such as `int a, b[];` or `for (int i = 0, arr[] = {1, 2}; ...)`—the recipe stripped the brackets from each variable and applied the last array variable's dimensions to the shared type, corrupting the code into `int[] a, b;` or `int[] i = 0, arr = {1, 2};`. This PR updated `visitVariableDeclarations` to verify that all variables in a declaration share the same post-name dimension count before moving the brackets onto the shared type, while removing the error-prone cursor message previously used between visitors. This prevents the recipe from turning non-array variables (like loop counters) into arrays or mismatching multi-dimensional arrays, eliminating severe type-mismatch compiler errors when running `UseJavaStyleArrayDeclarations` or `CommonStaticAnalysis`.

---

### [#1064](https://github.com/openrewrite/rewrite-static-analysis/pull/1064) — `UnwrapRepeatableAnnotations`: only unwrap container annotations
*Merged: 2026-09-17*

In Java, repeatable annotations are often grouped inside an outer container annotation. The `UnwrapRepeatableAnnotations` recipe is meant to clean up single annotations by removing the redundant outer wrapper. However, the recipe had a major flaw ([#389](https://github.com/openrewrite/rewrite-static-analysis/issues/389)): it aggressively replaced *any* annotation that contained a repeatable annotation attribute with just the inner annotation, discarding the outer container and all its other configured parameters. This PR added validation to ensure the recipe only unwraps annotations that are genuinely designated as repeatable container annotations, leaving unrelated outer annotations and their other parameters completely untouched. This protects complex enterprise frameworks—such as Jakarta Persistence / JPA `@NamedEntityGraph` mappings—from having critical configuration parameters wiped out, preventing silent data-loading bugs and runtime database errors.

---

### [#1036](https://github.com/openrewrite/rewrite-static-analysis/pull/1036) — `DeclarationSiteTypeVariance`: skip `@Override` methods when the supertype is unresolved
*Merged: 2026-09-17*

The `DeclarationSiteTypeVariance` recipe modernizes Java generic signatures by adding wildcard variance (`? extends` or `? super`). However, in issue [#277](https://github.com/openrewrite/rewrite-static-analysis/issues/277) (a 2.5-year-old bug), when a class method had an `@Override` annotation but its parent class came from an external library outside the local source set, the recipe still modified the method signature with wildcards. Because Java requires `@Override` methods to match their parent declaration exactly, this made the method incompatible with its parent interface. This PR updated the recipe to check if the superclass or interface is resolvable; if the parent type cannot be inspected, the recipe safely skips modifying the `@Override` method. This prevents the recipe from generating invalid Java code that fails to compile against external third-party libraries, allowing teams to safely run type-variance cleanups on large projects without breaking interface contracts.

---

### [#1035](https://github.com/openrewrite/rewrite-static-analysis/pull/1035) — `NoDoubleBraceInitialization`: don't retarget invocations inside nested anonymous classes
*Merged: 2026-09-17*

Double-brace initialization (`new ArrayList<>() {{ add("item"); }}`) is an old Java anti-pattern that creates unnecessary anonymous classes. The `NoDoubleBraceInitialization` recipe replaces this with cleaner setup code by retargeting method calls onto an outer variable. However, in issue [#352](https://github.com/openrewrite/rewrite-static-analysis/issues/352), when a double-brace block contained another nested anonymous class (like an inline event listener or comparator), the recipe mistakenly retargeted the inner class's methods onto the outer variable as well. This PR improved scope tracking to distinguish between calls belonging to the double-brace block itself and calls belonging to inner nested classes, ensuring inner method invocations are left alone. This stops the recipe from producing invalid method calls on the wrong objects, preventing compiler errors when cleaning up legacy double-brace initializers in complex enterprise code.

---

### [#1008](https://github.com/openrewrite/rewrite-static-analysis/pull/1008) — `FallThrough`: don't add a `break` after a guarded early exit
*Merged: 2026-09-17*

The `FallThrough` recipe ensures switch statement cases end with a `break` to prevent accidental fall-through into the next case. But in issues [#460](https://github.com/openrewrite/rewrite-static-analysis/issues/460) and [#229](https://github.com/openrewrite/rewrite-static-analysis/issues/229), if a switch case contained an `if` block that exited early (using `return` or `throw`) while the rest of the case was meant to fall through intentionally, the recipe blindly inserted a `break` right after the `if`. This PR analyzed the execution flow of guarded branches to ensure a `break` is only added when the code actually needs one, preventing breaks from being placed after conditional exits. This preserves intentional switch case fall-through logic and avoids introducing unreachable code, ensuring that automated switch refactoring doesn't silently alter business logic or cause compiler failures.

---

### [#1022](https://github.com/openrewrite/rewrite-static-analysis/pull/1022) — `NoValueOfOnStringType`: keep `String.valueOf` when the argument can be null
*Merged: 2026-08-30*

The `NoValueOfOnStringType` recipe cleans up redundant calls like `"" + String.valueOf(str)` by simplifying them to `"" + str`. However, in issue [#1009](https://github.com/openrewrite/rewrite-static-analysis/issues/1009), if `str` was `null`, `String.valueOf(null)` returns the safe text `"null"`. But when the recipe stripped `String.valueOf` in methods like `append(str)` or string operations, passing `null` directly led to a runtime `NullPointerException`. This PR added a nullability check to ensure `String.valueOf` is only removed when the expression is guaranteed to be non-null, retaining it whenever the value could potentially be null. This prevents subtle and dangerous production bugs by ensuring that cleaning up string conversions never turns a safe `"null"` string into an unexpected `NullPointerException` crash at runtime.

---

### [#998](https://github.com/openrewrite/rewrite-static-analysis/pull/998) — `UnnecessaryExplicitTypeArguments`: retain witness when enclosing method has dependent type parameters
*Merged: 2026-08-14*

The `UnnecessaryExplicitTypeArguments` recipe removes verbose generic type arguments when the Java compiler can infer them automatically (e.g., `Collections.<String>emptyList()` $\to$ `Collections.emptyList()`). However, in issue [#783](https://github.com/openrewrite/rewrite-static-analysis/issues/783) (a 9-month-old bug), when a method has interdependent generic bounds (like `<T, S extends T>`), the Java compiler cannot always figure out the types on its own without the explicit hint (called a type witness). Removing it caused the code to fail to compile. This PR checked for interdependent generic type parameters on enclosing methods and preserved the explicit type arguments whenever removing them would cause compiler ambiguity. This ensures that cleaning up redundant generic syntax never breaks the build, maintaining full code compilation safety across complex generic codebases.
