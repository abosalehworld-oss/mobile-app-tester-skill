# Contributing to Mobile App Tester Skill

Thank you for your interest in contributing! This skill gets better with real-world experience.

## How to Contribute

### Adding Bug Patterns to the Database
The `11-common-bugs-database.md` file is the main place for community contributions.

**Format for new bug entries:**
```
BUG-X###: Short descriptive title
  Frameworks: [Which frameworks this affects]
  Pattern: [Code pattern showing the problem]
  Impact: [What goes wrong if not fixed]
  Fix: [Correct code or approach]
```

**Severity prefixes:**
- `BUG-C###` — Critical (security breach, app crash)
- `BUG-H###` — High (major functionality broken)
- `BUG-M###` — Medium (UX degradation)
- `BUG-L###` — Low (code quality)

### Adding Framework-Specific Checks
If you have deep expertise in a specific framework, consider adding checks to the relevant phase files:
- Add checks under the appropriate `CHECK XX:` section
- Follow the existing format (WHAT TO CHECK, RED FLAGS, COMMON BUGS, CITATION REQUIRED)
- Keep checks static-analysis-friendly (no emulator required)

### Improving Phase Gates
Phase gate checklists should cover everything needed to consider a phase complete.
If you find gaps, open a PR adding the missing items.

## Guidelines

1. **Keep it emulator-free** — All checks must be doable through static code analysis
2. **Be specific** — Vague checks like "check security" are not useful
3. **Include examples** — Real code patterns for both the problem and the fix
4. **Framework-accurate** — Test your checks against real projects
5. **Security first** — Never suggest patterns that weaken security

## Opening Issues

- 🐛 **Bug Report**: Wrong information in the skill
- 💡 **Feature Request**: New phase, new check, new framework support
- 📖 **Documentation**: Unclear instructions or missing examples

## Pull Request Process

1. Fork the repository
2. Create a branch: `git checkout -b add-kotlin-coroutine-checks`
3. Make your changes following the existing format
4. Test against at least one real project
5. Submit PR with description of what was added and why

## Code of Conduct

Be respectful, be specific, and always cite your sources.
