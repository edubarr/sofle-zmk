# AGENTS.md
Guide for agentic coding tools operating in `sofle-zmk`.

## Scope
- ZMK user-config repository for a Sofle keyboard.
- Editable sources are mainly in `config/` and `boards/shields/sofle_dongle/`.
- CI builds from `build.yaml` via `.github/workflows/build.yml`.
- `firmware/*.uf2` are generated artifacts; do not hand-edit.

## Checked agent rule files
- `.cursorrules`: not found.
- `.cursor/rules/`: not found.
- `.github/copilot-instructions.md`: not found.
- If these are added later, treat them as higher-priority instructions.

## Key paths
- `build.yaml` - build matrix and artifact names.
- `.github/workflows/build.yml` - reusable ZMK build workflow entry.
- `config/west.yml` - West manifest (pins ZMK `v0.3`).
- `config/sofle.keymap` - standard split keymap.
- `config/sofle_dongle.keymap` - dongle keymap.
- `config/sofle.conf` - shared Kconfig values.
- `boards/shields/sofle_dongle/Kconfig.shield` - shield declaration.
- `boards/shields/sofle_dongle/Kconfig.defconfig` - shield defaults.
- `boards/shields/sofle_dongle/sofle_dongle.conf` - shield Kconfig settings.
- `boards/shields/sofle_dongle/sofle_dongle.overlay` - shield devicetree overlay.

## Environment setup
Run from repo root:

```bash
west init -l config
west update
west zephyr-export
west --version
```

Notes:
- This is the canonical bootstrap for this repo layout.
- Install `west` first if missing (for example `pip install --user west`).

## Build commands
Use pristine builds (`-p always`) after changing `.keymap`, `.overlay`, `.conf`, or `Kconfig*` files.

### Build all matrix targets (CI-equivalent)

```bash
west build -p always -d build/sofle_dongle -s zmk/app -b nice_nano -- -DSHIELD=sofle_dongle
west build -p always -d build/sofle_left_dongle -s zmk/app -b nice_nano@2.0.0 -- -DSHIELD=sofle_left -DCONFIG_ZMK_SPLIT=y -DCONFIG_ZMK_SPLIT_ROLE_CENTRAL=n
west build -p always -d build/sofle_right_dongle -s zmk/app -b nice_nano@2.0.0 -- -DSHIELD=sofle_right -DCONFIG_ZMK_SPLIT=y -DCONFIG_ZMK_SPLIT_ROLE_CENTRAL=n
west build -p always -d build/sofle_left_standard -s zmk/app -b nice_nano@2.0.0 -- -DSHIELD=sofle_left
west build -p always -d build/sofle_right_standard -s zmk/app -b nice_nano@2.0.0 -- -DSHIELD=sofle_right
west build -p always -d build/settings_reset -s zmk/app -b nice_nano -- -DSHIELD=settings_reset
```

### Build a single target (quick verification)

```bash
west build -p always -d build/sofle_left_standard -s zmk/app -b nice_nano@2.0.0 -- -DSHIELD=sofle_left
```

### Rebuild without reconfigure

```bash
west build -d build/sofle_left_standard
```

### Artifact output
- Typical output: `build/<target>/zephyr/zmk.uf2`.
- Keep copied file names aligned with `build.yaml` `artifact-name` values.

## Lint and formatting
This repo has no dedicated lint script or formatter configuration.

- Primary validation is successful `west build` for impacted targets.
- Keep YAML syntax valid in `build.yaml`, `config/west.yml`, and workflow YAML.
- Keep devicetree syntax valid in `.keymap` and `.overlay` files.
- Keep Kconfig syntax valid in `.conf` and `Kconfig*` files.

Recommended local checks:

```bash
git diff --check
```

Optional (if installed):

```bash
yamllint build.yaml config/west.yml .github/workflows/build.yml
```

## Test commands
There is no repository-local unit test suite.

- Treat successful firmware builds as the required test signal.
- Build every target affected by your change.
- If impact is unclear, run all matrix targets.

### Running a single test scenario (upstream workspace only)
If your workspace includes upstream ZMK/Zephyr tests and you touched upstream code:

```bash
west twister -T zmk/app/tests -s <test_scenario_name>
```

Use this only when that test tree exists locally.

## Code style

### General
- Make minimal, targeted edits.
- Preserve existing structure and naming patterns.
- Avoid unrelated refactors.
- Add comments only when they clarify non-obvious behavior.

### Includes/imports
- Keep `#include` directives at the top of keymap/overlay files.
- Use angle-bracket includes (`<...>`) as in current files.
- Keep include order stable and grouped logically.

### Formatting
- `.keymap` and `.overlay`: 4-space indentation inside nodes/properties.
- YAML: 2-space indentation; never tabs.
- Keep matrix-style bindings readable; preserve alignment where practical.
- Avoid trailing whitespace.

### Types/config values
- Use canonical ZMK behaviors (`&kp`, `&mo`, `&bt`, `&rgb_ug`, etc.).
- Use explicit Kconfig values (`y`, `n`, numeric literals).
- Keep split-role intent explicit in config/build args.
- Use valid upstream-compatible devicetree `compatible` strings.

### Naming conventions
- Layer constants are uppercase (`BASE`, `LOWER`, `RAISE`, `ADJUST`).
- Node labels/names are lowercase snake_case (for example `left_encoder`).
- Keep artifact names in `sofle_<side>_<mode>` format.
- Keep display names short and human-readable.

### Error handling and validation
- Prefer fail-fast validation through builds.
- Do not depend on silent defaults for required options.
- Keep `build.yaml` matrix entries synchronized with shield/config files.
- Rebuild at least one directly impacted target before finishing.

## Agent workflow checklist
1. Inspect `build.yaml` and identify impacted targets.
2. Edit only relevant files under `config/` and `boards/shields/`.
3. Run `west build` for affected targets.
4. Report exactly which targets passed or failed.
5. Do not commit generated `*.uf2` files unless explicitly requested.

## Project-specific guardrails
- Keep both standard split and dongle split modes working.
- Keep `settings_reset` in the matrix unless a human asks to remove it.
- Place dongle-specific behavior in `boards/shields/sofle_dongle/`.

## If unsure
- Match style and patterns from adjacent lines in the same file.
- Treat `build.yaml` as source of truth for supported build combos.
- If a direct human instruction conflicts with this file, follow the human instruction.
