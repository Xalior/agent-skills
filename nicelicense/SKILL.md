---
name: nicelicense
description: "Add or identify open-source licenses in projects using the nicelicense CLI. Use this skill when the user asks to add a license, change a license, check what license a project uses, or manage LICENSE files."
license: MIT
---

This skill provides complete knowledge of the `nicelicense` CLI (`npx nicelicense`) so you can add, validate, and manage open-source LICENSE files without rediscovering its interface.

## Invocation

```bash
npx nicelicense [options]
```

Always run via `npx` -- there is no global install required.

## Core Operations

### Add a license to a project

```bash
# Interactive (prompts for license choice)
npx nicelicense

# Non-interactive (specify everything)
npx nicelicense --license MIT --name "Jane Doe" --years 2024 --yes
```

`--yes` accepts all prompts (overwrite existing LICENSE, update `package.json` license field).

### Identify an existing license

```bash
npx nicelicense --validate
```

Reads the existing LICENSE file and reports the detected SPDX ID with a confidence score.

### List supported licenses

```bash
npx nicelicense --list              # SPDX IDs only
npx nicelicense --list --verbose    # IDs with URLs and metadata
```

## Supported SPDX IDs

`MIT`, `Apache-2.0`, `BSD-3-Clause`, `BSD-2-Clause`, `ISC`, `MPL-2.0`, `LGPL-3.0-only`, `GPL-3.0-only`, `GPL-3.0-or-later`, `Unlicense`

## All Options

| Flag | Description |
|------|-------------|
| `--license <SPDX>` | Select license without prompting |
| `--name <name>` | License holder name |
| `--email <email>` | License holder email |
| `--years <years>` | Copyright years (e.g. `2024` or `2020-2024`) |
| `--software <name>` | Software/project name |
| `--description <text>` | Project description |
| `--organization <org>` | Organization name |
| `--path <path>` | Write LICENSE to a specific path (default: cwd) |
| `--data <path>` | Load licenses from a custom JSON file |
| `--list` | Print supported SPDX IDs |
| `--validate` | Identify existing LICENSE file |
| `--dry-run` | Do not write files or update package.json |
| `--stdout` | Emit license text to stdout (no files written) |
| `--verbose` | Include URLs and metadata with `--list` |
| `--json` | Emit machine-readable JSON output |
| `--yes` | Accept all prompts automatically |

## Template Variables

Different licenses require different template variables. The main ones:

- **years, name**: MIT, Apache-2.0, BSD-3-Clause, BSD-2-Clause
- No template variables: ISC, MPL-2.0, LGPL-3.0-only, GPL-3.0-only, GPL-3.0-or-later, Unlicense

If a license requires a template variable and it isn't supplied, nicelicense will prompt interactively. Use `--name` and `--years` to avoid prompts for the common licenses.

## Common Patterns

### Fully non-interactive license creation
```bash
npx nicelicense --license MIT --name "Acme Corp" --years 2024 --yes
```

### Preview license text without writing files
```bash
npx nicelicense --license Apache-2.0 --name "Acme Corp" --years 2024 --stdout
```

### Check before writing
```bash
npx nicelicense --license MIT --name "Acme Corp" --years 2024 --dry-run
```

### Machine-readable output
```bash
npx nicelicense --validate --json
npx nicelicense --list --json
```

## Tips

- Always pass `--yes` in automated/agent workflows to avoid blocking on interactive prompts.
- Use `--stdout` or `--dry-run` when you just want to inspect output without side effects.
- Use `--json` when you need to parse the output programmatically.
- `--validate` is useful for auditing existing projects before modifying their license.
- When the user doesn't specify a license, check if one already exists with `--validate` first, then ask what they want.
