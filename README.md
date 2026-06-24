# Rustacean

Rustacean is an Agent Skill for strict Rust engineering work. It guides AI coding agents to prefer complete mathematical models, explicit error types, documented public APIs, functional core design, careful ownership semantics, rich tests, and panic-free production code.

The repository follows the common `SKILL.md` package shape:

```text
Rustacean/
├── SKILL.md
├── agents/
│   └── openai.yaml
└── README.md
```

Use symlinks during development so every runtime sees the same working copy.

## Claude Code

Claude Code discovers personal skills from `~/.claude/skills/<skill-name>/SKILL.md` and project skills from `.claude/skills/<skill-name>/SKILL.md`.

Install for all Claude Code projects:

```bash
cd /path/to/Rustacean
mkdir -p ~/.claude/skills
ln -s "$(pwd)" ~/.claude/skills/rustacean
```

Install for one project:

```bash
cd /path/to/Rustacean
mkdir -p /path/to/repo/.claude/skills
ln -s "$(pwd)" /path/to/repo/.claude/skills/rustacean
```

Invoke explicitly in Claude Code:

```text
/rustacean
```

Claude Code can also invoke the skill automatically when the request matches the `description` in `SKILL.md`. If the top-level skills directory did not exist when Claude Code started, restart Claude Code.

## Codex

Codex discovers user-level skills from `~/.agents/skills` and repository-scoped skills from `.agents/skills`.

Install for all Codex projects:

```bash
cd /path/to/Rustacean
mkdir -p ~/.agents/skills
ln -s "$(pwd)" ~/.agents/skills/rustacean
```

Install for one repository:

```bash
cd /path/to/Rustacean
mkdir -p /path/to/repo/.agents/skills
ln -s "$(pwd)" /path/to/repo/.agents/skills/rustacean
```

Invoke explicitly in Codex:

```text
$rustacean
```

Codex can also invoke the skill implicitly when a Rust writing, review, or refactoring task matches the `description` in `SKILL.md`. Restart Codex or open a new session if the skill does not appear.

## OpenClaw

OpenClaw skill discovery varies by build and deployment. The most portable locations are the shared Agent Skills path `~/.agents/skills` and workspace-scoped `.agents/skills` directories. Some legacy or shared OpenClaw installs may also scan `~/.openclaw/skills`; use that path only when your OpenClaw build is configured to read it.

Install in the shared Agent Skills path:

```bash
cd /path/to/Rustacean
mkdir -p ~/.agents/skills
ln -s "$(pwd)" ~/.agents/skills/rustacean
```

Install for one OpenClaw workspace:

```bash
cd /path/to/Rustacean
mkdir -p /path/to/openclaw-workspace/.agents/skills
ln -s "$(pwd)" /path/to/openclaw-workspace/.agents/skills/rustacean
```

Install in the legacy shared OpenClaw path only if your build scans it:

```bash
cd /path/to/Rustacean
mkdir -p ~/.openclaw/skills
ln -s "$(pwd)" ~/.openclaw/skills/rustacean
```

Start a new OpenClaw session after installation. Invoke it by asking OpenClaw to use the `rustacean` skill for Rust code. If your OpenClaw build exposes skill slash commands, use:

```text
/rustacean
```

## Hermes

Hermes stores active skills under `~/.hermes/skills`. Hermes can also load additional skill directories through `skills.external_dirs` in `~/.hermes/config.yaml`.

Install directly into the active Hermes skills directory:

```bash
cd /path/to/Rustacean
mkdir -p ~/.hermes/skills
ln -s "$(pwd)" ~/.hermes/skills/rustacean
```

Alternatively, keep the skill in a shared skills directory and add that parent directory to `skills.external_dirs`:

```bash
cd /path/to/Rustacean
mkdir -p /path/to/shared-skills
ln -s "$(pwd)" /path/to/shared-skills/rustacean
```

```yaml
skills:
  external_dirs:
    - /path/to/shared-skills
```

With this layout, the skill lives at `/path/to/shared-skills/rustacean/SKILL.md`.

Start a new Hermes session after installation. Invoke it from the CLI by preloading the skill:

```bash
hermes --skills rustacean
```

For a one-shot check:

```bash
hermes --toolsets skills -q "Use the rustacean skill to review this Rust code."
```

## Verify

Check that the installed path points to this repository:

```bash
ls -l ~/.claude/skills/rustacean
ls -l ~/.agents/skills/rustacean
ls -l ~/.openclaw/skills/rustacean
ls -l ~/.hermes/skills/rustacean
```

Only the paths for runtimes you installed need to exist. The skill root must contain:

```text
SKILL.md
agents/openai.yaml
README.md
```

## Update

If the skill was installed with a symlink, update this repository and restart the agent runtime if changes are not picked up:

```bash
cd /path/to/Rustacean
git pull
```

If the skill was copied instead of symlinked, copy the updated repository contents into each installed skill directory again.

## Uninstall

Remove the installed symlink or copied directory for each runtime:

```bash
rm ~/.claude/skills/rustacean
rm ~/.agents/skills/rustacean
rm ~/.openclaw/skills/rustacean
rm ~/.hermes/skills/rustacean
```

For project or workspace scoped installs, remove the corresponding `rustacean` path under that project's `.claude/skills`, `.agents/skills`, or OpenClaw workspace skills directory.
