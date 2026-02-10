# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

SourceGit is a cross-platform Git GUI client built with .NET 10 and Avalonia UI 11. Single-project solution (`src/SourceGit.csproj`).

## Build Commands

```bash
# Restore and build
dotnet restore
dotnet build -c Release

# Run the application
dotnet run --project src/SourceGit.csproj

# Check formatting (used in CI)
dotnet format --verify-no-changes

# Publish for a specific platform (win-x64, linux-x64, osx-x64, osx-arm64, etc.)
dotnet publish -c Release -r <RUNTIME_ID> -o <OUTPUT_DIR> src/SourceGit.csproj
```

AOT compilation is enabled by default in Release. Pass `-p:DisableAOT=true` to disable it for faster builds during development.

There are no automated tests in this repository.

## Architecture

**Pattern**: MVVM using CommunityToolkit.Mvvm (`ObservableObject` base class).

**Source layout** (`src/`):
- `Commands/` — Git CLI wrappers. Each class sets `Args` and calls `Exec()` or `ExecAsync()` on the `Command` base class, which spawns a `git` process.
- `Models/` — Domain data structures (commits, branches, diffs, etc.) and interfaces.
- `ViewModels/` — Application logic. ViewModels drive all UI state and orchestrate commands.
- `Views/` — Avalonia AXAML files paired with code-behind. Data-bound to ViewModels.
- `Converters/` — XAML value converters for data binding.
- `Native/` — Platform abstraction (`IBackend` interface) with implementations for Windows, macOS, and Linux. Handles OS-specific operations like finding Git, opening terminals, and shell integration.
- `Resources/` — Fonts, icons, TextMate grammars for syntax highlighting, themes, and locale files.

**Data flow**: View ↔ ViewModel → Command → git process → parsed into Models → ViewModel notifies View.

**Key entry points**: `App.axaml.cs` (application startup), `App.Commands.cs` (global commands), `App.JsonCodeGen.cs` (source-generated JSON serialization context).

## Code Style

Defined in `.editorconfig`:
- C#: 4 spaces, Allman braces, prefer `var`
- Private fields: `_camelCase`, private static: `s_camelCase`
- AXAML/XML: 2 spaces
- Expression bodies for properties/indexers, not for methods

## Contributing

- PRs target the `develop` branch (not `master`)
- `master` is the release branch
- CI runs `dotnet format --verify-no-changes` — run `dotnet format` before committing

## Localization

Locale files are in `src/Resources/Locales/`. English (`en_US.axaml`) is the base. A helper script validates translations:
```bash
python translate_helper.py <locale> [--check]
```
