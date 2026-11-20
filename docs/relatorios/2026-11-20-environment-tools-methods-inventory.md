# Relatório — 2026-11-20 — Levantamento: plugins, skills, métodos e ferramentas usados nesta sessão

Delivery: DSH lead session `a69b320fbfab` (owner request: «quero levantamento dos plugins, skills, metodos
tudo que estavamos usar aqui»). Reference inventory of everything used to deliver the Core S-round
(2026-11-19/20). Snapshot `main @ 59275d2` (pushed to `origin/main`). Companion to the ESTADO handover
(«Ronda S de 2026-11-20») and `ci/review-env/README.md`.

## 1. DSH plugins / bundles

### Installed custom (profile) — the two added for this work
| Bundle | Version | Source | Rows | Purpose |
| --- | --- | --- | --- | --- |
| **`@local/dsh-cpp-lsp`** | 1.0.0 | `file:…\default-workspace\dsh-cpp-lsp` | `lsp` (`@deepseek-ai/dsh-lsp`), `lsp-stdio` (`dsh-lsp-stdio`), `tool-lsp` (`dsh-tool-lsp`) | C++ code navigation via clangd: mounts the LSP seam + clangd stdio provider + the model-facing `lsp` tool. Wired to the Core Debug compile DB (`D:/Core-build/core-review/debug`) with `--query-driver=…\cl.exe` (the app does not inherit vcvars). Proven: hpp→cpp definition, cross-TU, findReferences, hover. |
| **`@local/agent-task-factory`** | 1.0.0 | `link:…\default-workspace\agent-task-factory` | `agent-task-factory` | Convenience tools: `agent_create` (subagent from role templates researcher/coder/reviewer/writer/analyst), `task_create` (scheduled task with plain timing), `todo_add` (incremental todo merge). No external imports (pure JSON-Schema tool defs). |

### Active base bundles (enabled, the running surface)
- **`@deepseek-ai/dsh-base`** 0.2.2-rc.5 — the shared core (≈100 rows): agent loop, sessions, LLM adapters,
  tools (pwsh/fs/search/jobs/web/skill/goal/todo/subagent/agent-team/workflow), sandbox+permissions,
  compaction, telemetry, credentials. The foundation everything runs on.
- **`@deepseek-ai/dsh-web-app`** 0.2.2-rc.5 — the browser GUI surface (this session's UI at
  `http://127.0.0.1:<port>`): workspace, file references, deliverables, plugin inventory UI, schedule,
  office-kit, agent presets (standard/ptc/minimal/cordis), settings.
- **`@deepseek-ai/dsh-experimental-plugin-market-bundle`** (enabled, optional) — Discover tab / plugin indexes.
- **`@deepseek-ai/dsh-experimental-forge-bundle`** (enabled, optional) — worktree + change-request panel
  (`create_worktree` tool available).

### Available but DISABLED (optional, not used this session)
acp-app, headless, sdk-app, sdk-minimal, experimental-inspector, voice-input (SenseVoice), auto-review,
cot-translation, badge-skill, ralph (repeated work), session-titles-all-prompts, terminal-bundle,
liveshare, session-search, tool-worktree (standalone), cua (macOS, unsupported-platform here),
personalized-recommendations, deepseek-chat-import. **Missing** (not installed): subagent-claude-code,
subagent-codex (alternative subagent backends).

## 2. DSH tools used (model-facing)

| Tool | Use in this session |
| --- | --- |
| `pwsh` | Windows shell — builds, gates, git, ctest, probes, file ops (the primary executor; bash disabled on Windows) |
| `read` / `write` / `edit` | file inspection / creation / targeted edits (reports, tests, CMake, ESTADO) |
| `glob` / `grep` | file discovery / content search (ripgrep) |
| `lsp` | clangd navigation (goToDefinition/findReferences/hover) via dsh-cpp-lsp |
| `subagent` / `subagent_fork` / `agent_create` | spawned 28 task subagents (disjoint scopes, own builds) |
| `send_message` / `list_agents` / `wait_agent` | subagent steering / tree inspection |
| `team_task_*` (create/get/list/update) | shared task board (22 tasks, dependency chain, write scopes) |
| `create_goal` / `update_goal` / `get_goal` | goal-driven continuation across rounds (the S-round objective) |
| `todo_write` / `todo_add` | progress tracking |
| `job_output` / `job_list` / `job_kill` | background build/gate job management |
| `plugin_manager` | list_bundles/list_plugins (this inventory); install/enable of dsh-cpp-lsp earlier |
| `working_directory` | cd to D:\CORE\CORE |
| `load_workspace_dependencies` | bundled Python/Node paths (probe builds, WAV fixtures) |
| `ask_user_question` | owner decisions (plan-clone path) |
| `web_search` / `web_fetch` | available (not needed — all local) |
| `schedule_create` / `task_create` | available (not used — no recurring tasks requested) |
| `present` | available (deliverables shown inline instead) |
| `skill` | available (no bailian/office skill matched this C++/DSH work — see §3) |

## 3. Skills (catalog available vs used)

Available in this session (loaded on demand via the `skill` tool):
- **Bailian family** (Aliyun DashScope): `bailian-cli`, `bailian-docs-llm-wiki`, `bailian-finetune`,
  `bailian-gen`, `bailian-knowledge`, `bailian-managed-agent`, `bailian-memory`, `bailian-model-recommend`,
  `bailian-protocol`, `bailian-sandbox`, `bailian-train-deploy`, `bailian-web-search`.
- **Office**: `office`, `office-docx`, `office-pptx`, `office-xlsx`.
- **DSH/meta**: `dsh-doc` (DSH facts/plugin design), `diagnose-windows-sandbox-acl` (Windows ACL repair).

**Used this session: NONE invoked via the `skill` tool** — the work was C++/JUCE engine + DSH tooling,
which matched no bailian/office skill. `dsh-doc` is the relevant reference for the plugin work
(dsh-cpp-lsp/agent-task-factory); `diagnose-windows-sandbox-acl` was available for sandbox denials
(none occurred — danger-full-access policy). The bailian/office skills remain available for future tasks.

## 4. Methods / processes (the development methodology)

- **Red-first regression** (AGENTS.md): every fix measured failing BEFORE (own build of the unfixed tree),
  then minimal change, then green AFTER + full merge suite; counts per suite, no coverage shrink.
- **Registered gate**: `ci/gate.ps1 -Build <dir> [-Desktop]` — Release, Ninja, vcvars64 + VSLANG=1033 +
  chcp 65001, unique guid `.bat` (D114 anti-collision), Ninja header-deps check, `--no-tests=error`.
  merge label = integration scope; desktop = JUCE self-test (null device). **version==sha** proven on a
  clean tree (other-session rows stashed/restored).
- **SHA-bound marker validation** (ci/review-env): `.done` markers bound to
  `fp1 sha+local(build-affecting diff)+juce(patch state)+script+cfg`; reuse only on full match; WSL hard
  checkout gate (fail-closed). Proofs of reuse/invalidation in fresh folders (Windows 7/7, WSL 6/6).
- **Sanitizers**: ASan MSVC (`CORE_SANITIZE=address`, app OFF); WSL ASan+UBSan (`address,undefined`) and
  TSan (`thread`, separate tree, `setarch $(uname -m) -R` for ASLR). Canaries must trip before a green
  suite is trusted (D92/D93). Instrumented scope = our targets only (JUCE/miniaudio/plugins out, D92).
- **clang-tidy scope gate**: `ci/tidy_scope.py` asserts the analysis scope (negative control must fail);
  WSL clang-tidy 18, checks `bugprone-*/concurrency-*/performance-*/clang-analyzer-*`.
- **Debugger-symbol proof**: `probe/stack_probe.cpp` — StackWalk64 + SymFromAddr + SymGetLineFromAddr64
  against build PDBs, resolves Core frames to file:line. ENV-03 used live remote-thread stack capture.
- **RT measurement**: S7 probe (new/delete overrides + IAT hooks for heap/fileIO/CS/SRWLOCK/waits +
  `core::rtProbe` seam) with negative controls that MUST fire; real-device baseline (WASAPI, 64/128,
  load ramp, dropouts/max/p99, breaking point).
- **Bit-exact proofs**: render subtraction / memcmp / SHA-256 for deterministic cases (serial≡M5, PDC,
  block-independence, save/reopen).
- **Subagent orchestration**: disjoint write scopes + own build folders + acceptance test per agent;
  short-turn discipline + incremental notes (long turns killed several agents); coordinator integrates
  (reviews diffs, runs the gate). Mechanical micro-agents for builds; analysis agents for proofs.
- **Task board + goal**: shared tasks with dependency chain (0→3A→{1,4,2}→6→5→7 + S-round), write-scope
  advisories; goal-driven continuation across automatic rounds.
- **Evidence archiving**: verbatim logs/CSVs/probes/WAVs/stacks/transcripts in
  `docs/relatorios/evidencia/<date>-<subject>/`; historical evidence archived not overwritten.
- **Append-only ledger**: ESTADO.md decisions D### with verbatim owner quotes + source; reports propose
  lines, scribe writes (D72/D80) — waived for this handover by explicit owner instruction (D116).
- **Governance**: owner verbatim authorization for decisions (D48/D50, silence≠approval); architecture
  needs numbered decision; data > RT > performance > compatibility (D54); no new agent tokens / CI.

## 5. Core toolchain & dependencies

- **Windows**: VS 18 Build Tools, MSVC 19.51.36260 (toolset 14.51.36231), vcvars64; CMake 4.4.4; Ninja
  1.13.2; Windows PowerShell 5.1 (scripts) + pwsh 7 (executor). `/fp:fast` unconditional for the engine
  (engine/CMakeLists.txt:9, Debug+Release).
- **WSL2**: Ubuntu 24.04.5 (kernel 6.18.58.1), GCC 13.3.0, clang/clang-tidy 18.1.3, CMake 3.28.3,
  Ninja 1.11.1, python3 3.12.3. **No gdb, no apt network** (ENV-04). TSan needs ASLR off (setarch -R).
- **JUCE 9.0.5** at `D:\Core-deps\JUCE-9.0.5` (extracted, no .git) + vendored D113 crash patch
  (`ci/patches/juce-failed-plugin-restart.patch`, 14 hunks). Since S11 CMake applies/verifies it.
- **Bundled runtimes** (DSH): Python (numpy/pandas/openpyxl/python-docx/pptx/Pillow/lxml/XlsxWriter),
  Node.js + pnpm. Used for probes/WAV fixtures.
- **clangd 23.1.0** (`…\clangd_23.1.0\bin\clangd.exe`); **gh CLI** (GitHub auth: d-aicore/diogo-o/dmo-app).
- **Model route**: `bailian-tpp/qwen4-max-preview` (the only authorized subagent route).

## 6. Core plugins (VST3)

- **Test plugins** (app/selftest-plugins, built with the app, never shipped): CoreTestSynth, CoreTestGain,
  CoreTestCrash (CORE_TEST_CRASH compile-time variant), CoreTestCrashRestart (CORE_TEST_CRASH_RESTART),
  CoreTestFault (runtime CORE_TEST_FAULT_* seams), CoreTestThrow. Plus out-of-repo probes:
  CoreTestDelayProbe (S9, fixed 480 delay), CoreTimingDelay/CoreTimingClick (timing proof).
- **Real plugins tested** (VST3 front): Soundtoys **EchoBoy 5.5.5.19885** + **Decapitator 5.5.5.19885**
  (23 Soundtoys VST3 bundles scanned at `C:\Program Files\Common Files\VST3\Soundtoys`). VST2 DLLs present
  (18) but the engine refuses format vst2 (project.cpp:387) — S10 pending.
- **Second hosts available** (not used for comparison yet): Bitwig Studio 6.1, Ableton Live 12 Trial,
  Cakewalk Next.

## 7. Scripts & key artifacts

- `ci/review-env/` (in repo): `run-all.ps1`, `run-wsl.sh`, `verify.ps1`, `proof-windows.ps1`,
  `proof-wsl.sh`, `prep-wsl-repull.sh`, `probe/{stack_probe.cpp,lsp-smoke.mjs}`, `README.md`.
- `ci/gate.ps1` (registered gate), `ci/compilar.bat` (GUI build launcher), `ci/tidy_scope.py` (tidy scope).
- Out-of-tree builds: `D:\Core-build\core-review\{debug,asan}` (canonical env), `gate-final` (Release gate),
  per-agent dirs (`s1-fix`, `s2s3`, `s6`, `s7`, `s9`, `timing` on C:, `chain-series`, `vst3-host`, …).
- Reports: `docs/relatorios/2026-11-20-*.md` (17) + `evidencia/` (verbatim). Coordination:
  `2026-11-20-round1-coordination.md`. Ledger: ESTADO.md «Ronda S de 2026-11-20».

## 8. Subagents deployed this session (28)

csv-triage, csv-triage-2 (CSV confirmation); resume-check (fresh-session resume proof); plan-hunt
(original-plan search); s11-proof/s11-clean-build/s11-negctrl/s11-release-proof/s11-fix (S11);
s1-build/s1-red/s1-repro/s1-repro-2/s1-fix (S1); s5-red/s5-fix (S5); s4-red/s4-fix (S4);
env03-debug (3A); s2s3-fix (S2/S3); s6-proof (S6); s7-rt (S7); s8-undo (S8); s9-mute (S9);
vst3-host (VST3 front); timing-proof (timing); chain-series (Chain). All inactive now (work delivered).
Pattern learned: long turns kill agents → mechanical micro-agents for builds + short-turn analysis
agents + incremental notes; coordinator runs final measurements/reports when an agent dies.

## 9. What was NOT used / limitations

- No bailian/office skill invoked (work was C++/DSH). No web search needed (all local). No scheduled
  tasks. No workflow-tool orchestration (plain subagents sufficed). No MCP servers connected.
- gdb absent in WSL (Windows stack probe covered symbols). No real-device ASan/VST3-under-sanitizer.
  M5 8h soak not done (D51). UI click-through of fixes not done (headless + code). Second-host VST3
  comparison pending. Linux≠Windows proof (AGENTS.md).
