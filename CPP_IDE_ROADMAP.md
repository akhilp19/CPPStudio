# C/C++ IDE/Editor Capability Roadmap

> A living document that tracks the biggest gaps in modern C and C++ tooling and a phased plan for closing them. It is written from the perspective of a C/C++ product team that wants to improve the daily development experience for professional C and C++ engineers.

---

## 1. Vision

Make C and C++ development as productive, discoverable, and collaborative as modern managed-language stacks, **without hiding the power and control that systems-language developers expect**.

### Scope

This roadmap treats **C** and **C++ as first-class peers**. Features must work for:
- C89/C90, C99, C11, C17, and the upcoming C23/C2x standards.
- C++98/03, C++11, C++14, C++17, C++20, and C++23/26.
- Mixed C/C++ codebases, including projects where C is compiled as C++ (or vice versa) and projects that expose C APIs from C++ internals.
- C-compatible dialects and extensions consumed by the same compiler drivers (e.g., Objective-C++ with Clang, C++/CLI, C++/WinRT, C++/CX, CUDA C++, OpenCL C++, MASM/assembly, HLSL).

---

## 2. Guiding Principles

1. **Build-system agnostic, but convention-aware.** Support CMake, Bazel, Meson, Make, MSBuild, QMake, and custom tools through a single abstraction layer.
2. **Source of truth is the build graph.** Indexing, IntelliSense, debugging, and refactoring must derive from the real compiler invocation, not heuristics.
3. **Progressive disclosure.** Beginners get one-click project onboarding; experts keep full control over toolchains, flags, and diagnostics.
4. **Batteries included, but swappable.** Provide default integrations for dependencies, testing, static analysis, and CI, but allow replacement with enterprise or open-source alternatives.
5. **Cross-platform by default.** macOS, Linux, Windows, remote hosts, containers, and embedded targets must share the same core workflows.

---

## 2.5 Base Platform Decision

> This section records the starting technology bet and the plan to validate it against alternatives over time.

### 2.5.1 Selected base: Eclipse Theia

The first implementation will be built on **Eclipse Theia** (`eclipse-theia/theia`, EPL-2.0).

**Rationale**

| Factor | Why Theia Fits |
|--------|----------------|
| **Purpose-built for IDEs** | Theia is an IDE platform, not just an extensible editor. |
| **VS Code extension compatibility** | Lets us reuse Microsoft C/C++, CMake Tools, Remote-SSH, Docker, and devcontainer extensions while building differentiated features. |
| **LSP/DAP native** | Aligns with the compiler-driven indexer and debugger strategy in this roadmap. |
| **Cloud + desktop** | Runs as Electron desktop app or browser/cloud IDE, matching the remote/WSL/container workflows. |
| **Modular architecture** | Dependency-injection design allows replacing the file system, terminal, search, build adapter, and language services. |
| **License** | EPL-2.0 is business-friendly for a commercial or proprietary product compared to GPL-based C++ IDEs. |

### 2.5.2 Initial Theia-based architecture

```
┌─────────────────────────────────────────────┐
│  Theia Shell (UI, workbench, extensions)    │
├─────────────────────────────────────────────┤
│  C/C++ Extension Pack                       │
│  - Microsoft C/C++ extension (interim)      │
│  - CMake Tools                              │
│  - Remote Development + Dev Containers      │
│  - Podman/Docker integration                │
├─────────────────────────────────────────────┤
│  Custom C/C++ Services (differentiators)    │
│  - Compiler-driven indexer                  │
│  - Build-system adapter                     │
│  - Memory layout inspector                  │
│  - Template/concept debugger                │
│  - Static-analysis hub                      │
│  - Cross-platform launch orchestrator       │
├─────────────────────────────────────────────┤
│  LSP / DAP / Debug Adapter Host             │
├─────────────────────────────────────────────┤
│  Compilers & Runtimes                       │
│  - clangd / clang / lldb / clang-tidy       │
│  - MSVC / cl.exe / CDB                      │
│  - GCC / gdb                                │
│  - Cross-compilers & embedded toolchains    │
└─────────────────────────────────────────────┘
```

### 2.5.3 Alternative platforms for future cross-checking

The Theia bet will be revisited at major milestones. The following alternatives will be evaluated in parallel tracks to avoid lock-in and to borrow proven ideas:

| Platform | License | Cross-Check Purpose |
|----------|---------|---------------------|
| **VS Code (microsoft/vscode)** | MIT | Validate UX expectations, extension-marketplace compatibility, and remote-dev behavior. |
| **Qt Creator** | GPL-3.0 / Commercial Qt | Benchmark native C++ indexing, debugging, and qmake/CMake performance. |
| **KDevelop** | GPL-2.0+/LGPL | Compare C++ parser accuracy, refactoring coverage, and Linux desktop integration. |
| **Code::Blocks** | GPL-3.0 | Reference for lightweight plugin architecture and beginner onboarding. |
| **Lapce** | Apache-2.0 | Track modern GPU-rendered editor architectures and Rust-based implementations. |

### 2.5.4 Upstream-sync strategy

This project will **never push code back to the upstream Theia repository** as pull requests. The fork exists only to:

1. Track upstream Theia improvements.
2. Apply custom C/C++ IDE features on top of a stable base.
3. Rebase or merge upstream fixes on a predictable cadence.

**Fork configuration**

| Setting | Value | Rationale |
|---------|-------|-----------|
| Repository visibility | Public | Forks of public repos must remain public on GitHub. |
| Issues | Disabled | Avoid duplicate issue tracking; project issues live in `CPPStudio`. |
| Discussions | Disabled | Not used for a downstream sync fork. |
| Wiki | Disabled | Documentation lives in `CPPStudio`. |
| Projects | Disabled | Roadmap is managed in `CPPStudio`. |
| Actions | Enabled | Required for automated upstream sync. |
| Allow forking | Disabled | Prevents accidental downstream forks of the fork. |

**Automated sync workflow**

The file `.github/workflows/sync-upstream.yml` in `akhilp19/theia` syncs `upstream/master` into `origin/master` every Monday at 06:00 UTC and supports manual triggering.

> **Note:** All upstream Theia workflows (CI/CD, Playwright Tests, Production Build Smoke Test, etc.) have been disabled in this fork to avoid unnecessary failing runs. Only `Sync upstream Theia` remains active.

Required setup:
1. Create a GitHub **Personal Access Token (PAT)** with `repo` scope.
2. Add it to `akhilp19/theia` under **Settings > Secrets and variables > Actions** as `PAT`.

Manual fallback:

```bash
git fetch upstream
git checkout master
git merge upstream/master --no-edit
git push origin master
```

### 2.5.5 When to revisit the platform choice

- **End of Phase 1** — if Theia cannot meet the 95% "open without manual config" target for sample repos.
- **End of Phase 2** — if replacing generic LSP services with custom C/C++ services is harder in Theia than in a fork of VS Code or Qt Creator.
- **Before Phase 4** — if performance, extension-marketplace limitations, or licensing block enterprise adoption.

### 2.5.6 Current status

- **Fork created:** `https://github.com/akhilp19/theia`
- **Upstream sync workflow:** `.github/workflows/sync-upstream.yml` pushed; requires a `PAT` secret with `repo` scope to be enabled.
- **Scaffold extension:** `packages/cpp-build/` created with:
  - Common RPC protocol, preference schema, and build-system model
  - Frontend commands, service proxy, and status-bar contribution
  - Backend server, build-system registry, and adapter interface
  - Placeholder adapters for CMake, Bazel, Meson, and Make
  - Registered in `examples/browser/package.json`

**Next implementation target:** Phase F — route build-system detection, configure/build, and `compile_commands.json` reading through Theia's remote abstractions so the IDE works with WSL, containers, and SSH hosts.

### 2.5.7 Theia extension implementation plan

These milestones focus on building `packages/cpp-build/` inside the Theia fork. They are intentionally narrower than the product-level phases in Section 4 and should fit within Phase 1 of the main roadmap.

| Phase | Focus | Deliverables | Duration | Status |
|-------|-------|--------------|----------|--------|
| **A — Bootstrap** | Package wiring and extension registration | `packages/cpp-build/` created, registered in `examples/browser`, commands appear in palette, frontend/backend modules load. | 1–2 days | ✅ Complete |
| **B — Detection service** | Detect the correct build system per workspace root | `BuildSystemRegistry` selects best `BuildSystemAdapter`; CMake adapter reads `CMakePresets.json`, resolves build directory and `compile_commands.json`; quick-pick preset selection; status bar shows active system. | 1 week | ✅ Complete |
| **C — Build execution & output** | Run configure/build and surface output | `runStreamingCommand` invokes `cmake`; output streams to a dedicated **C/C++ Build** output channel; build events notify frontend. | 1 week | ✅ Complete |
| **D — clangd wiring** | Feed compiler invocations to the language server | Adapter writes `.clangd` config pointing at `compile_commands.json`; VS Code clangd extension picks it up automatically. | Few days | ✅ Complete |
| **E — Debug integration** | Launch debugger from build graph | `DebugAdapterContribution` for `cppdbg`; resolves `program`, `args`, `cwd`, and debugger path from selected build target. | 1 week | ✅ Complete |
| **F — Remote builds** | Cross-platform and containerized builds | Detect remote workspace URI; route discovery/build commands through `RemoteConnection.exec()`; read generated `compile_commands.json` via `FileService`. | 1–2 weeks | 🚧 In progress |

**Current progress:** Phase E complete; Phase F in progress.

---

## 3. Current Pain Points (Problem Catalog)

The table below maps each issue to its user-facing impact and a priority tier. Items marked **P0** are blockers for mainstream adoption; **P1** are strong productivity wins; **P2** are differentiators for advanced users.

| # | Issue | User Impact | Priority |
|---|-------|-------------|----------|
| 1 | Build-system fragmentation | Every repo needs a custom IDE setup. | P0 |
| 2 | C++20 modules support gaps | Modules cannot be used confidently in production. | P0 |
| 2b | C standard support gaps | C23 features, `_Generic`, `typeof`, `constexpr` semantics, and C/C++ compatibility checks are inconsistently supported. | P1 |
| 3 | Dependency/package management | Third-party libraries are painful to acquire and keep in sync. | P0 |
| 4 | Refactoring quality | Rename/move/extract often miss symbols due to macros/templates. | P1 |
| 5 | Compile/indexing speed | Large codebases leave developers waiting. | P1 |
| 6 | Cross-compilation & embedded | Embedded workflows require manual toolchain/debugger wiring. | P1 |
| 7 | Template/concept debugging | Template instantiation errors remain opaque. | P1 |
| 8 | Memory layout & low-level visualization | Object layout, padding, cache effects are invisible. | P1 |
| 9 | Project onboarding friction | Clone-and-build is rare; setup can take hours or days. | P0 |
| 10 | Native AI-assisted coding | Code generation is weaker and less trustworthy than for other languages. | P2 |
| 11 | Real-time collaboration | Pair programming and remote cloud IDE sessions are immature. | P2 |
| 12 | Binary/reverse-engineering integration | Inspecting compiled artifacts requires leaving the IDE. | P2 |
| 13 | Test integration & coverage | Running tests, viewing coverage, and bisecting failures are disconnected. | P1 |
| 14 | Static analysis integration | Clang-Tidy, PVS-Studio, Cppcheck, Sonar results are siloed. | P1 |
| 15 | Documentation generation | Keeping Doxygen/XML comments in sync with code is manual. | P2 |
| 16 | API breaking-change detection | Upgrading dependencies often surfaces link/runtime failures late. | P1 |
| 17 | Sanitizer integration | ASan/TSan/UBSan/MSan findings are not surfaced as first-class diagnostics. | P1 |
| 18 | CI/CD & reproducibility | Local builds and CI builds diverge; reproducing failures is hard. | P1 |
| 19 | Post-mortem/crash-dump debugging | Core dumps and minidumps require external tooling. | P2 |
| 20 | Multi-language interop | C++/Rust, C++/Python, and C++/C# boundaries lack smooth debugging. | P2 |
| 21 | Code formatting consistency | clang-format configuration is fragmented across teams. | P1 |
| 22 | SBOM & security scanning | Vulnerable dependencies are discovered late or not at all. | P1 |
| 23 | Lifetime & ownership hints | No borrow-checker-like assistance for raw pointers and references. | P2 |
| 24 | C/C++ interop tooling | Hard to maintain C APIs exported from C++ or consume C headers in C++ safely. | P1 |
| 25 | Performance regression tracking | Historical build/test/profile data is not correlated with commits. | P2 |

---

## 4. Roadmap Phases

### Phase 1 — Foundation (0–6 months)
**Goal:** Remove the biggest blockers to opening, building, and navigating any C or C++ project.

| Initiative | Deliverables | Success Metric |
|------------|--------------|----------------|
| Unified build-system adapter | A single `BuildModel` abstraction that imports CMake, Bazel, Meson, Make, MSBuild, QMake, and compile_commands.json for C and C++. | 95% of sample repos open without manual configuration. |
| True compiler-driven indexing | Indexer consumes real compiler invocations (clang, GCC, MSVC) instead of guessing include paths; per-file language mode (C vs. C++) detected from compile command. | IntelliSense correctness matches build output for 98% of symbols. |
| One-click onboarding wizard | Detects compiler, toolchain, vcpkg/Conan, presets, and missing system deps; generates a working project config for C and C++. | Median time from clone to first successful build < 5 min for common repos. |
| C++20 modules — phase 1 | Basic parsing of `import`, `export module`, and module partitions; module dependency graph visualization. | Modules compile and navigate in sample projects. |
| C standard support — phase 1 | Recognize C99/C11/C17/C23 keywords (`_Generic`, `typeof`, `static_assert`, `noreturn`, etc.); flag C/C++ compatibility issues at edit time. | C projects index and build with the same ease as C++ projects. |
| Dependency manager integration | Native vcpkg and Conan workflows; manifest editing, version pinning, lock-file sync. | Adding a dependency is a 2-click or 1-command operation. |
| Test runner unification | Discover CTest, GoogleTest, Catch2, doctest, Boost.Test, Unity, CMocka; run, filter, debug, and show results inline. | All major C/C++ test frameworks work out of the box. |
| Cross-platform IDE evaluation harness | Automated benchmark suite that exercises Theia, VS Code, Qt Creator, and KDevelop against the same sample projects and metrics. | Objective comparison data available before Phase 2 platform revisit. |

### Phase 2 — Productivity (6–12 months)
**Goal:** Make everyday editing, refactoring, and debugging feel modern and fast.

| Initiative | Deliverables | Success Metric |
|------------|--------------|----------------|
| Refactoring 2.0 | Rename, move, extract function, change signature aware of macros, templates, `using` aliases, and C-style struct/function declarations. | 95% accuracy on representative open-source codebases. |
| Incremental & background indexing | Parallel index shards, cache invalidation per translation unit, low-priority background work. | Initial index of 1 MLOC in < 2 min; edits reflected in < 2 s. |
| Template & concept debugger | Visualize template instantiation stacks, concept substitution failures, and SFINAE branches (C++ only). | Mean time to resolve template errors reduced by 50%. |
| C standard conformance mode | Per-file standard selection, diagnostics for mixing C and C++ headers, warnings for C99/C11/C17/C23 feature availability. | Zero false positives when a project contains both `.c` and `.cpp` files. |
| Sanitizer-as-diagnostic | ASan/TSan/UBSan/MSan output mapped to source lines, gutter icons, and fix suggestions. | Sanitizer reports are actionable without reading raw logs. |
| Static-analysis hub | Aggregate clang-tidy, PVS-Studio, Cppcheck, clang-static-analyzer, custom checkers, and C-specific MISRA/CERT checkers into a unified panel. | Zero false-positive noise through suppressions and baselines. |
| clang-format governance | Workspace-shared format config, format-on-save, CI gate, format-diff preview; separate C and C++ style rules supported. | 100% of committed code matches team style. |
| Cross-platform build environments | CMakePresets.json-first support; native WSL2, remote SSH, container (Docker/Podman), and cross-compilation toolchain workflows; per-target IntelliSense. | New platform target setup time < 15 min; same project builds on Windows, Linux, and macOS hosts without code changes. |

### Phase 3 — Insight (12–18 months)
**Goal:** Give developers visibility into runtime behavior, architecture, and quality trends.

| Initiative | Deliverables | Success Metric |
|------------|--------------|----------------|
| Memory layout inspector | Visual struct/union layout: fields, offsets, sizes, padding, alignment; suggest packing or reordering. Works for C `struct`/`union` and C++ `class`. | Developers identify padding waste in < 30 s. |
| Assembly/source correlation | Mixed source/asm stepping, hot-loop annotation, inline assembly support. | Function assembly reachable in 2 clicks. |
| Test coverage & mutation | Line/branch coverage overlay, coverage diff, mutation testing integration. | Coverage data visible per file/function/commit. |
| API compatibility & upgrade assistant | Detect breaking changes when upgrading dependencies; generate migration patches. | Breaking changes caught before merge. |
| Performance regression dashboard | Track compile times, binary size, test duration, benchmark results per commit. | Regressions flagged within the PR workflow. |
| Post-mortem debugging | Import core dumps/minidumps, map symbols, inspect stacks and memory. | Crash triage possible without leaving the IDE. |
| Multi-language debugging | Mixed C/C++, C++/Rust, C++/Python, and C# call-stack navigation and breakpoints. | Step across language boundaries seamlessly. |

### Phase 4 — Differentiation (18–24 months)
**Goal:** Leap ahead of today's tooling with AI, collaboration, and security features.

| Initiative | Deliverables | Success Metric |
|------------|--------------|----------------|
| Trustworthy AI assistant | Fine-tuned model trained on C++ codebases; respects project conventions and build graph; explains template errors. | 80% acceptance rate for generated refactorings. |
| Real-time collaboration | Live shared sessions with synchronized cursors, shared debugging, and cloud-based build agents. | Pair-programming latency < 100 ms. |
| Binary explorer | Inspect object files, disassembly, symbols, dependencies, and call graphs inside the IDE; support both x86/x64 and ARM/embedded object formats. | Binary analysis no longer requires external reverse-engineering tools. |
| SBOM & vulnerability scanning | Generate SPDX/CycloneDX SBOMs, scan CVEs, surface fixes, gate CI. | Critical CVEs blocked at PR time. |
| Lifetime & ownership hints | Static heuristics and annotations that flag potential use-after-free, dangling references, and ownership bugs. | Detect common lifetime bugs at edit time. |
| Documentation intelligence | Auto-generate Doxygen from signatures, detect stale docs, render Markdown API docs. | 90% of public APIs have up-to-date docs. |

---

## 5. Cross-Cutting Concerns

These topics must be addressed continuously across every phase:

| Concern | Approach |
|---------|----------|
| **Open standards** | Prefer LSP, DAP, CMakePresets, compile_commands.json, SBOM standards over proprietary formats. |
| **Extensibility** | Expose extension points so teams can add custom build systems, linters, and target platforms. |
| **Telemetry & privacy** | Collect opt-in metrics for indexing time, error rates, and feature usage; never exfiltrate source code. |
| **Accessibility** | Full keyboard navigation, screen-reader support, high-contrast themes, and configurable fonts. |
| **Reproducibility** | Docker/Podman/devcontainer templates, Nix/Guix shells, lock files, hermetic toolchain packages, CI parity checks. |
| **Platform portability** | Core services must be abstracted so they can be lifted into VS Code, Qt Creator, or a custom IDE later; keep Theia-specific code isolated in an adapter layer. |
| **Cross-IDE validation** | Run the same benchmark suite against Theia, VS Code, Qt Creator, and KDevelop at the end of each phase to learn from alternatives and detect lock-in risk. |
| **Documentation** | Every feature ships with docs, examples, and a troubleshooting guide. |

---

## 6. Definition of Done for Each Phase

A phase is considered complete when:

1. All P0 items for that phase are shipped and covered by automated tests.
2. At least one representative open-source C++ project of > 500 KLOC works end-to-end without manual hacks.
3. Public documentation and sample projects are updated.
4. Telemetry shows a measurable improvement in the success metrics listed above.
5. A community or internal feedback review has been completed and action items captured.

---

## 7. Appendix: Suggested Sample Validation Projects

Use these projects as acceptance-test candidates because they exercise different build systems, scales, and language features:

| Project | Build System | Why It Matters |
|---------|--------------|----------------|
| LLVM/Clang | CMake + Ninja | Large scale, templates, modules, cross-platform |
| Chromium | GN + Ninja | Massive code base, multiple languages, strict builds |
| Linux Kernel | Kbuild | C at massive scale, headers-only workflows, cross-arch |
| Redis | Make | Pure C, performance-critical, embedded-oriented |
| Qt Base | CMake + qmake | Cross-platform, moc, QML interop |
| Abseil | CMake + Bazel | Modern C++, GoogleTest, Bazel support |
| fmt / spdlog | CMake | Header-only and compiled library patterns |
| OpenCV | CMake | Heavy dependencies, optional modules, cross-compilation |
| Catch2 / doctest | CMake | Test framework discovery edge cases |

---

## 8. Cross-Platform Compilation & Reproducible Build Environments

Yes — this IDE must support cross-platform compilation as a first-class workflow. A developer on Windows should be able to build, test, and debug code targeting Linux, macOS, embedded RTOS, or bare-metal with the same project configuration used on any other host.

### 8.1 Recommended mechanisms (beyond plain Docker)

| Mechanism | Best For | Host → Target Example | Notes |
|-----------|----------|----------------------|-------|
| **WSL2** | Windows developers building Linux binaries | Windows → Linux x64/ARM64 | Native Linux kernel on Windows; fastest file-system performance for source on ext4; gdb/lldb work out of the box. |
| **Remote SSH build host** | Corporate build farms, powerful workstations, CI runners | Any → Linux/macOS | IDE runs locally, compiler/debugger run on the remote machine via SSH; lowest local resource usage. |
| **Podman** | Daemonless, rootless containers; RHEL/CentOS/Fedora shops; security-sensitive environments | Any → Linux | Drop-in replacement for Docker; supports Dockerfiles and Compose files; rootless by default. |
| **Docker Desktop** | Teams already standardized on Docker; local Linux containers on Windows/macOS | Any → Linux | Mature ecosystem; devcontainer support; licensing considerations for enterprise. |
| **Dev Containers / devcontainer.json** | Reproducible, IDE-integrated environments | Any → Linux | Works with Docker, Podman, WSL2, and remote hosts; share exact environment across team and CI. |
| **Cross-compilation toolchains** | Embedded, mobile, console SDKs | x64 Windows → ARM Linux / iOS / Android | CMake toolchain files + sysroots; no VM/container overhead; requires correct sysroot and libraries. |
| **Nix / Guix** | Hermetic, reproducible dependency environments | Any → same host or cross target | Declarative toolchains and libraries; excellent for CI parity. |
| **MSYS2 / MinGW-w64** | Windows-native POSIX-like builds | Windows → Windows | Build Windows binaries using GCC/Clang; useful for open-source projects without MSVC support. |
| **QEMU user-mode emulation** | Running foreign-architecture tests on the build host | x64 Linux → ARM64 Linux | Slower than native but useful for ARM/embedded unit-test verification. |
| **Cloud build agents / GitHub Codespaces** | Elastic capacity, zero local setup | Any → configurable | Centralized environments; integrate with source control and CI. |

### 8.2 How Podman fits in

Podman should be treated as a **first-class peer to Docker**, not an afterthought:

1. **Compatibility layer**
   - Accept the same `Dockerfile` and `Containerfile` syntax.
   - Support `podman-compose` for multi-service dev environments.
   - Use the `docker.io`/`quay.io` image registries transparently.

2. **Rootless operation**
   - Default to rootless containers on Linux hosts.
   - Map host UIDs/GIDs correctly so file ownership in bind mounts stays sane.

3. **Podman Desktop integration**
   - On Windows and macOS, Podman Desktop manages the VM lifecycle.
   - IDE detects Podman Desktop and offers the same "Open in Container" workflow as Docker Desktop.

4. **Build command abstraction**
   - IDE uses an internal `ContainerRuntime` interface.
   - Commands adapt to whichever runtime is installed:
     - Docker: `docker build`, `docker run`, `docker compose`
     - Podman: `podman build`, `podman run`, `podman-compose`

5. **Security advantages**
   - No persistent root daemon reduces attack surface.
   - Good fit for air-gapped or regulated environments where Docker is restricted.

### 8.3 Windows → Linux / macOS build strategies

| Scenario | Recommended Approach |
|----------|---------------------|
| Windows dev, Linux target, local iteration | WSL2 + CMakePresets |
| Windows dev, macOS target | Remote SSH to a Mac build host (Apple Silicon or Intel) |
| Windows dev, Linux target, team parity | Podman or Docker devcontainer |
| Windows dev, embedded target | Cross-compiler toolchain file + optional QEMU for tests |
| Reproducible builds across all hosts | Nix shell or locked devcontainer image |

### 8.4 IDE responsibilities for cross-platform builds

- **CMakePresets.json first**: presets should encode the host/target/os/compiler triple so the IDE can switch targets in one click.
- **Per-target IntelliSense**: include paths, defines, and compiler macros must match the selected target, not the host.
- **Toolchain detection**: auto-discover installed WSL distros, Podman/Docker runtimes, remote SSH hosts, and cross-compilers.
- **Unified launch/debug**: the debugger attach command must adapt to local, WSL, container, SSH, or emulator targets transparently.
- **Artifact parity**: produced binaries, packages, and SBOMs should be identical regardless of the developer's host OS (within target-architecture limits).

---

## 9. Languages & Dialects That Compile With the MSVC / C++ Toolchain

Many languages and language extensions are compiled by the same driver (MSVC, Clang, or a host compiler wrapper). The IDE should treat them as related workflows, not isolated silos.

### 9.1 First-class C and C++

| Language | Compiler / Driver | Notes |
|----------|-------------------|-------|
| **C** | MSVC (`cl.exe /std:c11`, `/std:c17`), Clang, GCC | Often mixed with C++ in embedded, systems, and game code. |
| **C++** | MSVC, Clang, GCC | Primary target of this roadmap. |

### 9.2 Microsoft-specific C++ dialects

| Language | Compiler / Driver | Notes |
|----------|-------------------|-------|
| **C++/CLI** | MSVC (`/clr`) | Managed interop for .NET; uses `gcnew`, handles (`^`), and `ref class`. |
| **C++/CX** | MSVC (`/ZW`) | Older Windows Runtime component extensions; largely superseded by C++/WinRT. |
| **C++/WinRT** | MSVC + `cppwinrt.exe` | Modern C++ projection for the Windows Runtime; plain C++ headers. |

### 9.3 Other C-family dialects compiled by Clang/LLVM family

| Language | Compiler / Driver | Notes |
|----------|-------------------|-------|
| **Objective-C++** | Clang (`clang++ -x objective-c++`) | macOS/iOS C++ interop with Objective-C. |
| **OpenCL C / C++** | Clang, vendor ICDs | GPU kernel language derived from C/C++. |
| **CUDA C++** | `nvcc` with MSVC or Clang host compiler | GPU kernel development; host code is C++. |
| **SYCL** | `icpx`, `clang++`, ` hipsycl` | Standard C++-based heterogeneous parallel programming. |

### 9.4 Assembly and shader languages in the same toolchain

| Language | Tool | Notes |
|----------|------|-------|
| **MASM / x64 assembly** | `ml.exe`, `ml64.exe` | Microsoft's macro assembler, ships with MSVC. |
| **HLSL** | `dxc.exe`, `fxc.exe` | DirectX shading language; compiled by the DirectX Shader Compiler. |

### 9.5 Languages that interop heavily with C/C++ but do NOT compile with MSVC

These should still be supported for mixed debugging and FFI navigation, even though they use their own compilers:

| Language | Typical Compiler | Interop mechanism |
|----------|------------------|-------------------|
| **Rust** | `rustc` / Cargo | `extern "C"` FFI, bindgen, cxx |
| **D** | `dmd`, `ldc2`, `gdc` | `extern(C)` and `extern(C++)` |
| **Swift** | `swiftc` | C++ interop mode (experimental) |
| **Python** | CPython runtime | Python C/API, pybind11, nanobind |
| **Zig** | `zig` | `zig cc` can compile C/C++; links natively |
| **Fortran** | `ifx`, `gfortran` | ISO_C_BINDING for C interop |

---

## 10. How to Contribute

1. Open an issue referencing the pain-point number from Section 3.
2. Propose a design doc or prototype for the relevant phase.
3. Ensure changes include tests, docs, and a note in the changelog.
4. Tag the issue with the appropriate phase label (`phase-1`, `phase-2`, etc.).

---

*Last updated: 2026-07-25 — added Eclipse Theia base platform decision and cross-IDE validation plan.*
