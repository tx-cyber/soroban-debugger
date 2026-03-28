# Repo Health Snapshot (2026-03-28, post-pull)

## Pull + Audit Summary

- Pulled latest `origin/main` to `307faeb`.
- Reviewed changed files in range `55272aa..307faeb` (41 files changed).
- Ran validation gates and spot checks.

## Validation Results

- `git pull --rebase --autostash`: pull succeeded after stashing local WIP; local tree is now clean.
- `./run_local_ci.sh`: failed at `cargo fmt --check`.
- `cargo check --workspace --all-features`: failed with Rust type error in `src/analyzer/security.rs`.
- `cargo test --workspace --all-features --no-run`: failed with same compile error.
- `make check-man`: passed.
- `make test-vscode`: blocked in this sandbox (`listen EPERM 127.0.0.1`), compile phase passed.

## 30 New Issues to Open

1. **Fix compile break in reentrancy helper (`&usize` vs `usize`)**  
Problem: `find_writes_seen_by_frame` returns `count` by reference, causing `E0308`.  
Files to touch: `src/analyzer/security.rs`, `tests/security_import_name_tests.rs`  
Acceptance criteria: `cargo check --workspace --all-features` passes this module with no type errors.

2. **Restore rustfmt compliance for newly merged Rust edits**  
Problem: `cargo fmt --check` fails in multiple files.  
Files to touch: `src/analyzer/security.rs`, `src/repl/commands.rs`, `tests/security_import_name_tests.rs`  
Acceptance criteria: `cargo fmt --all -- --check` passes cleanly.

3. **Remove duplicate field initialization in `InspectArgs` construction**  
Problem: `source_map_diagnostics` and `source_map_limit` are initialized twice in one literal.  
Files to touch: `src/main.rs`  
Acceptance criteria: only one assignment per field; compile remains clean after issue #1 fix.

4. **Normalize `src/cli/commands.rs` line endings to LF**  
Problem: file is checked in with CRLF and causes noisy diffs/churn.  
Files to touch: `src/cli/commands.rs`  
Acceptance criteria: `git ls-files --eol src/cli/commands.rs` reports `i/lf`.

5. **Remove UTF-8 BOM from runtime executor source**  
Problem: `src/runtime/executor.rs` starts with BOM bytes (`EF BB BF`).  
Files to touch: `src/runtime/executor.rs`  
Acceptance criteria: file starts directly with `//!` and no BOM byte prefix remains.

6. **Fix mojibake/corrupted comment text in runtime docs/comments**  
Problem: text like `â€”`, `â†’`, `faÃ§ade` appears in source comments.  
Files to touch: `src/runtime/executor.rs`, `src/runtime/result.rs`  
Acceptance criteria: comments render proper ASCII or valid UTF-8 intended symbols, no mojibake sequences.

7. **Reinstate clear pass/fail markers in scenario output**  
Problem: scenario messages now show `?` where pass/fail markers were expected.  
Files to touch: `src/scenario.rs`  
Acceptance criteria: output clearly distinguishes pass vs fail (text or symbols) with no ambiguous markers.

8. **Add regression tests for scenario include recursion and cycle detection**  
Problem: include support was added without explicit cycle-focused integration tests.  
Files to touch: `src/scenario.rs`, `tests/fixture_tests.rs`  
Acceptance criteria: tests cover direct cycle, indirect cycle, and valid include graph behavior.

9. **Define and enforce include default semantics in scenario runner**  
Problem: included files may have `defaults`, but runtime currently applies root defaults only.  
Files to touch: `src/scenario.rs`, `docs/batch-execution.md`  
Acceptance criteria: documented and tested behavior for which defaults apply to included steps.

10. **Document scenario `include` feature and edge cases**  
Problem: include behavior is not clearly documented for users.  
Files to touch: `README.md`, `docs/getting-started.md`, `docs/faq.md`  
Acceptance criteria: docs include include syntax, path resolution, and cycle/error examples.

11. **Add recursion depth guard for scenario includes**  
Problem: recursion is cycle-checked but has no explicit depth limit for pathological graphs.  
Files to touch: `src/scenario.rs`  
Acceptance criteria: configurable max include depth with deterministic error on overflow.

12. **Avoid reparsing WASM index per function in inspect output**  
Problem: `function_has_source_mapped` reparses WASM index per function call.  
Files to touch: `src/debugger/source_map.rs`, `src/commands/inspect.rs`  
Acceptance criteria: WASM index parsed once per inspect command, with unchanged output semantics.

13. **Add tests for inspect JSON `has_source_debug` field**  
Problem: new field added but no dedicated JSON contract test.  
Files to touch: `src/commands/inspect.rs`, `tests/json_output.rs`  
Acceptance criteria: tests verify field presence/values for both mapped and unmapped functions.

14. **Add pretty output snapshots for inspect function table formatting**  
Problem: new 3-column pretty table lacks regression snapshots.  
Files to touch: `src/commands/inspect.rs`, `tests/cli/run_tests.rs`  
Acceptance criteria: stable snapshots for short and long signatures with source/debug column.

15. **Integrate `RuntimeError` enum into runtime execution path**  
Problem: `RuntimeError` exists but is not used in runtime control flow.  
Files to touch: `src/runtime/result.rs`, `src/runtime/executor.rs`, `src/server/debug_server.rs`  
Acceptance criteria: timeout/cancellation paths emit structured runtime errors end-to-end.

16. **Use structured runtime error formatting in DAP adapter**  
Problem: `parseRuntimeError` and `formatDapError` are present but unused.  
Files to touch: `extensions/vscode/src/dap/adapter.ts`  
Acceptance criteria: error catch paths call the structured parser/formatter and emit user-facing diagnostics.

17. **Add adapter tests for `storage.search` evaluate command**  
Problem: new storage search feature lacks test coverage.  
Files to touch: `extensions/vscode/src/test/suites.ts`, `extensions/vscode/src/dap/variableStore.ts`  
Acceptance criteria: tests verify result count, truncation messaging, and variable expansion.

18. **Add adapter tests for `storage.page` evaluate command**  
Problem: paging path added without dedicated e2e assertions.  
Files to touch: `extensions/vscode/src/test/suites.ts`  
Acceptance criteria: tests validate page numbering, bounds behavior, and page metadata text.

19. **Add adapter tests for `storage.count` evaluate command**  
Problem: count command has no regression test.  
Files to touch: `extensions/vscode/src/test/suites.ts`  
Acceptance criteria: tests verify count output and no variables reference on scalar result.

20. **Harden `pagedStorage` for invalid page-size configurations**  
Problem: page-size edge cases can produce unstable paging behavior.  
Files to touch: `extensions/vscode/src/dap/variableStore.ts`  
Acceptance criteria: page size is clamped to `>= 1`; tests cover zero/negative values.

21. **Provide ASCII fallback for pager labels in variable views**  
Problem: labels use Unicode arrows, which can degrade in limited terminals/fonts.  
Files to touch: `extensions/vscode/src/dap/variableStore.ts`, `extensions/vscode/README.md`  
Acceptance criteria: fallback label mode available or labels changed to ASCII-safe wording.

22. **Skip VS Code smoke tests gracefully when loopback sockets are blocked**  
Problem: smoke suite fails hard in restricted environments with `EPERM`.  
Files to touch: `extensions/vscode/src/test/suites.ts`, `extensions/vscode/src/test/runSmokeTest.ts`  
Acceptance criteria: loopback restriction yields explicit skip output rather than hard failure.

23. **Add VS Code sandbox-friendly test script**  
Problem: current scripts do not separate socket-dependent and non-socket tests.  
Files to touch: `extensions/vscode/package.json`, `extensions/vscode/README.md`  
Acceptance criteria: `npm run test:sandbox` executes only sandbox-safe checks.

24. **Use ephemeral server port in `remote_run_tests`**  
Problem: fixed port `9245` is collision-prone and flaky.  
Files to touch: `tests/remote_run_tests.rs`  
Acceptance criteria: tests reserve and use an ephemeral free port dynamically.

25. **Replace fixed startup sleep with readiness probe in remote tests**  
Problem: static `sleep(1500ms)` is timing-sensitive and flaky across hosts.  
Files to touch: `tests/remote_run_tests.rs`  
Acceptance criteria: polling/ping readiness gate replaces fixed delay.

26. **Strengthen loopback capability probe to cover connect path**  
Problem: current helper only checks `bind`, not client connect viability.  
Files to touch: `tests/network/mod.rs`, `tests/parity_tests.rs`, `tests/remote_run_tests.rs`  
Acceptance criteria: helper verifies both bind and connect, with explicit failure reason text.

27. **Make `ci-sandbox` deterministic by excluding network-dependent test subsets**  
Problem: script states sandbox-safe intent but still runs full Rust tests.  
Files to touch: `run_local_ci.sh`, `Makefile`  
Acceptance criteria: sandbox mode runs documented non-network subset and reports skipped suites explicitly.

28. **Stop swallowing manpage generation build failures**  
Problem: `scripts/check_manpages.sh` currently uses `cargo build --quiet ... || true`.  
Files to touch: `scripts/check_manpages.sh`  
Acceptance criteria: build/generation failures fail fast with actionable error output.

29. **Fail fixture build script on package metadata parsing errors**  
Problem: fixture build script now `continue`s on missing package name, masking failures.  
Files to touch: `tests/fixtures/build_fixtures.sh`  
Acceptance criteria: script exits non-zero with clear summary when any fixture cannot be built.

30. **Add repository-wide line-ending policy with `.gitattributes`**  
Problem: mixed line endings already entered the tree; no policy file exists.  
Files to touch: `.gitattributes`, `CONTRIBUTING.md`  
Acceptance criteria: source files are enforced as LF and contribution docs describe the policy.

