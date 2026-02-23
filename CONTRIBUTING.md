# Contributing to ColectorHub

First off, thanks for taking the time to contribute! Every contribution helps make ColectorHub better for game collectors everywhere.

## Table of Contents

- [Code of Conduct](#code-of-conduct)
- [How Can I Contribute?](#how-can-i-contribute)
- [Development Setup](#development-setup)
- [Branch Naming](#branch-naming)
- [Commit Messages](#commit-messages)
- [Pull Request Process](#pull-request-process)
- [Code Style](#code-style)
- [Project Architecture](#project-architecture)

---

## Code of Conduct

This project follows a simple rule: **be respectful and constructive**. We're all here to build something cool for the game collecting community.

---

## How Can I Contribute?

### Reporting Bugs

- Use the [GitHub Issues](https://github.com/FalvesDev/ColectorHub/issues) tab
- Include steps to reproduce the bug
- Include your device model, OS version, and Flutter version
- Screenshots or screen recordings are very helpful

### Suggesting Features

- Open an issue with the `feature-request` label
- Describe the use case and why it would benefit collectors
- Check existing issues to avoid duplicates

### Code Contributions

1. Fork the repository
2. Create a feature branch
3. Make your changes
4. Submit a pull request

---

## Development Setup

### Prerequisites

- Flutter SDK >= 3.22
- Dart >= 3.4
- A Supabase project (free tier works)
- Android Studio or VS Code with Flutter extensions

### Steps

```bash
# 1. Fork and clone
git clone https://github.com/YOUR_USERNAME/ColectorHub.git
cd ColectorHub

# 2. Install dependencies
flutter pub get

# 3. Set up environment variables
cp .env.example .env
# Edit .env with your Supabase and API credentials

# 4. Run the app
flutter run
```

---

## Branch Naming

Use descriptive branch names with prefixes:

| Prefix | Use Case | Example |
|---|---|---|
| `feature/` | New functionality | `feature/wishlist-screen` |
| `fix/` | Bug fixes | `fix/scan-crash-on-dark-images` |
| `refactor/` | Code improvements | `refactor/collection-repository` |
| `docs/` | Documentation | `docs/update-readme` |
| `chore/` | Maintenance tasks | `chore/update-dependencies` |

---

## Commit Messages

Follow the [Conventional Commits](https://www.conventionalcommits.org/) specification:

```
<type>(<scope>): <description>

[optional body]
```

### Types

| Type | Description |
|---|---|
| `feat` | A new feature |
| `fix` | A bug fix |
| `docs` | Documentation changes |
| `style` | Code style (formatting, no logic change) |
| `refactor` | Code refactoring |
| `test` | Adding or updating tests |
| `chore` | Maintenance (dependencies, config) |

### Examples

```
feat(scanner): add Gemini Vision fallback for OCR failures
fix(collection): prevent duplicate game entries
docs(readme): add screenshots section
refactor(auth): extract email validation to util
```

---

## Pull Request Process

1. **Update your branch** with the latest `main` before submitting
2. **Fill out the PR template** with a clear description of changes
3. **Link related issues** (e.g., "Closes #12")
4. **Ensure the app builds** — run `flutter build apk --debug` before submitting
5. **One feature per PR** — keep pull requests focused and reviewable

---

## Code Style

### General Rules

- Follow the official [Dart Style Guide](https://dart.dev/effective-dart/style)
- Use `flutter analyze` to catch issues before committing
- Maximum line length: **80 characters**
- Use trailing commas for better formatting

### Architecture Rules

This project follows **Feature-First + Clean Architecture**:

```
features/
  feature_name/
    data/           # Implementations (Supabase, APIs, Hive)
    domain/         # Models, interfaces, use cases
    presentation/   # Screens, controllers, widgets
```

- **Never import `data/` from `domain/`** — domain should be independent
- **Use cases** should have a single `call()` method
- **Repository interfaces** live in `domain/`, implementations in `data/`
- **Riverpod controllers** handle state, not business logic

### Naming Conventions

| Element | Convention | Example |
|---|---|---|
| Files | `snake_case` | `game_detail_screen.dart` |
| Classes | `PascalCase` | `CollectionRepository` |
| Variables | `camelCase` | `gameTitle` |
| Constants | `camelCase` | `maxImageWidth` |
| Private | `_prefixed` | `_isLoading` |

---

## Project Architecture

Before contributing, please read the [PLANEJAMENTO.md](PLANEJAMENTO.md) file for a comprehensive understanding of:

- System architecture and data flow
- Database schema and relationships
- Scan pipeline (OCR + AI)
- Feature specifications

This document is in Portuguese (PT-BR) and serves as the complete technical specification.

---

Thank you for contributing!
