# LORE

## Overview

- TraceForge: a lightweight code-reading environment on Windows-native GNU Emacs GUI for browsing repositories, tracing symbols through LSP, reviewing Git history, and reading rendered Markdown with Mermaid.
- MUST read [README.md](README.md) before implementing; it defines the product scope, capability boundaries, roadmap, and acceptance criteria.

## Document Rules

- AI agent documents MUST stay bilingual with equivalent content (the file without a suffix is the English version), for example AGENTS.md = AGENTS_ch.md.
- Links in AI agent documents MUST match the document's language: an English file links to the English target, and a Traditional Chinese file links to the _ch target when one exists.
- [README.md](README.md) is for human readers and is written in Traditional Chinese.

## Scope and Platform

- TraceForge MUST run on Windows-native GNU Emacs GUI and MUST be designed only for it.
- MVP MUST NOT support WSL / WSL2, Linux, macOS, terminal-only Emacs, or Docker-based Emacs.
- WSL MUST NOT be used as an alternative implementation of a core feature.
- MVP MUST NOT implement: a debugger, build and deployment management, a full Java refactoring engine, Spring / Hibernate-specific IDE features, a database client, an AI coding agent, Email / Calendar / Org-mode, or a complex GUI framework.

## Emacs Configuration

- Features SHOULD be implemented as independent modules that avoid coupling to each other.
- Prefer built-in Emacs APIs.
- Prefer built-in Emacs packages; manage non-built-in packages with `package.el` and `use-package`.
- MUST NOT rewrite mature Emacs packages; prefer mature packages over custom implementations, and MUST NOT implement a new search engine.
- MUST NOT introduce unnecessary frameworks.
- Keep native Emacs key bindings; expose other features preferably through named commands and Magit, and SHOULD NOT heavily modify the native keymap.
- SHOULD NOT require users to know many Emacs commands to complete basic operations.
- Status display (project, language, major mode, LSP state, Git branch) may use the mode line; a custom status bar framework is not needed.
- MVP does not need a full visual project tree.
- Use `early-init.el` only when the actual startup mechanism needs it; MUST NOT assume that loading a bootstrap under `-Q` processes it automatically.
- The directory layout may be trimmed to actual needs; avoid creating empty modules with no real use.

## Language Servers

- Eglot is the only LSP client; MVP MUST NOT introduce `lsp-mode` alongside it.
- Prefer Eglot's default language server mapping; modify `eglot-server-programs` only when the defaults are insufficient.
- Adding a language SHOULD, in principle, only add the major mode and language server configuration, and SHOULD NOT modify the core navigation API.
- Lifecycle:
  - Lazy initialization is the default; the user can start a server with `M-x eglot` on first use.
  - Buffers of the same project and language mode SHOULD share an LSP session where possible; different languages use their own sessions.
  - A failing language server MUST NOT affect other languages or plain file reading.
  - The auto-start option `traceforge-lsp-auto-start` defaults to `nil` and MUST NOT load every language server at Emacs startup.
- Different language servers SHOULD coexist in the same Emacs GUI session.
- A missing language server MUST NOT break navigation for other languages; language servers MUST be installable separately.
- If a language server does not support a feature, SHOULD show a reasonable hint instead of letting the whole navigation feature fail.
- Workspace root:
  - MUST NOT assume the Git repository root equals the LSP workspace root.
  - SHOULD NOT force every file onto the top-level Git root.
  - Prefer the language server's own workspace detection and the Emacs project API; add an overridable root resolver only when necessary.
  - MUST avoid indexing a large monorepo more than once because of wrong root detection.
- Java (Eclipse JDT LS):
  - MUST account for the JDK runtime version, jdtls launch arguments, Windows-specific configuration, Maven / Gradle project import, the workspace cache, isolation between Java projects, and extra dependencies such as Lombok.
  - MUST NOT assume an executable `jdtls.exe` exists on every Windows environment; when needed, create a dedicated PowerShell launcher or Emacs Lisp startup configuration.
  - The JDT LS workspace cache SHOULD live in the TraceForge runtime directory, separated per project.
- MUST NOT assume standard LSP traces calls across languages; a cross-language call graph, API relationships, and distributed service traces are out of MVP scope.

## Git

- Git MUST run through native Git for Windows; MUST NOT depend on WSL or Git Bash as the launch environment.

## Markdown and Mermaid Preview

- Prefer rendering through the Windows default browser; MUST NOT require a special Emacs build or WebKit.
- SHOULD avoid copying repository documents to a public server.
- MUST NOT assume EWW supports full browser JavaScript rendering; MUST clearly distinguish basic in-buffer rendering from full HTML / CSS / JavaScript rendering in a browser.
- MVP uses Mermaid.js; rendering assets MUST work offline from installed resources, with pinned dependency versions.
- Ondine MUST be an optional renderer backend; MVP MUST NOT require Ondine. Before integrating it, SHOULD verify its Windows availability, Mermaid syntax compatibility, and required runtime.

## Installation and Isolation

- Installation SHOULD NOT directly overwrite the user's existing Emacs configuration.
- MUST avoid overwriting `~/.emacs` and `~/.emacs.d/`.
- Startup SHOULD isolate Emacs, for example `runemacs.exe -Q` with a dedicated bootstrap that explicitly loads the TraceForge configuration.
- Initialization MUST set `user-emacs-directory`, `package-user-dir`, `custom-file`, the cache directory, and the runtime directory correctly.
- MUST NOT assume `--init-directory` avoids every existing Emacs initialization file.
- Every external dependency MUST have documented installation instructions in [README.md](README.md).

## Runtime Data

- Mutable data is recommended to live under `%LOCALAPPDATA%\TraceForge\`.
- The Git repository MUST NOT store: language server workspace caches, temporary files automatically generated by Emacs, temporary Markdown preview output, absolute local paths of the user, or tokens, credentials, and other secrets.
- Every external tool SHOULD support Windows Unicode paths.

## Diagnostics

- Provide `scripts/doctor.ps1` and `M-x traceforge-doctor` to check core dependencies and LSP state.
- Diagnostics SHOULD distinguish: Critical Error, Optional Dependency Missing, Language Server Missing, LSP Connection Failed, Workspace Initialization Failed.
- When a language server is missing, diagnostics MUST give readable installation advice.

## Security and Compatibility

- Paths MUST be tested for: spaces, Unicode / Traditional Chinese, Windows path separators, case handling, executables and `.cmd` launchers, and PowerShell argument quoting.
- MUST NOT concatenate unsanitized user input directly into a shell command; prefer the Emacs process API or PowerShell safe argument passing.
- When previewing an untrusted repository, MUST avoid: arbitrary JavaScript execution, unnecessary external resource loading, arbitrary local file access, and uploading repository documents to external services.
- If a local preview server is created, it MUST: listen only on `127.0.0.1`, use a dynamically available port, restrict file access scope, prevent path traversal, provide a stop mechanism, and not expose `.git` or user secret files.

## Development Workflow

- Implement Phase 1 through Phase 6 in order; Phase 7 MUST NOT block the MVP.
- Every phase SHOULD have an executable way to verify acceptance.
- When a phase is complete, MUST:
  1. Verify that the Emacs configuration loads.
  2. Run the applicable automated tests.
  3. Update [README.md](README.md) or related documents.
  4. Record unfinished and unverified items in the phase status of the [README roadmap](README.md#roadmap).
  5. Commit each acceptable feature separately.
- Acceptance MUST be verified in a real Windows-native environment. Emacs batch mode may serve as automated testing but MUST NOT replace the GUI smoke test.
- MUST NOT claim full acceptance for an item when the Windows GUI test could not run in the current environment.
- Commit messages use a Conventional Commits prefix (`feat`, `fix`, `test`, `docs`, …) followed by a Traditional Chinese description.
