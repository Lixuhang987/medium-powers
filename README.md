# Medium Powers

## Quick Start

Medium Powers is not in the official Codex plugin marketplace yet. Install it as a local plugin.

Clone the repository into the current directory:

```bash
mkdir medium-powers
cd medium-powers
git clone https://github.com/Lixuhang987/medium-powers.git .
```

Copy the plugin into Codex's local plugin directory:

```bash
mkdir -p ~/plugins/medium-powers
rsync -a --delete --exclude '.git' ./ ~/plugins/medium-powers/
```

Register the local plugin marketplace entry:

```bash
python3 - <<'PY'
import json
from pathlib import Path

marketplace = Path.home() / '.agents/plugins/marketplace.json'
marketplace.parent.mkdir(parents=True, exist_ok=True)

if marketplace.exists():
    data = json.loads(marketplace.read_text())
else:
    data = {
        'name': 'local',
        'interface': {'displayName': 'Local Plugins'},
        'plugins': [],
    }

entry = {
    'name': 'medium-powers',
    'source': {
        'source': 'local',
        'path': './plugins/medium-powers',
    },
    'policy': {
        'installation': 'AVAILABLE',
        'authentication': 'ON_INSTALL',
    },
    'category': 'Developer Tools',
}

plugins = data.setdefault('plugins', [])
plugins[:] = [plugin for plugin in plugins if plugin.get('name') != 'medium-powers']
plugins.append(entry)

marketplace.write_text(json.dumps(data, indent=2) + '\n')
print(f'Updated {marketplace}')
PY
```

Restart Codex, open the plugin picker, and install `Medium Powers` from the local marketplace.

The local marketplace path `./plugins/medium-powers` resolves from your home directory, so the actual plugin files live at `~/plugins/medium-powers`.

## What It Is

Medium Powers is an experimental Codex plugin for use-case-driven development.

It started as a response to a pattern I kept seeing in agent workflows: as models get stronger, strict TDD templates can become procedural latency instead of engineering leverage. They often push the agent toward scattered, function-level tests, fragmented implementation steps, and too little meaningful review of whether the real user flow works.

Medium Powers keeps the useful discipline of planning, tests, verification, and review, but changes the center of gravity:

> Define the use case, control the core data structures and interface contracts, write a use-case-level integration test, then implement the smallest amount of code needed to make that flow work.

This is not anti-testing. It is a narrower testing strategy for coding agents: test the real flow first, avoid over-mocking business logic, and review the work against the approved use case instead of celebrating isolated green tests.

## How It Works

Medium Powers is built as a set of Codex skills. The main development loop is:

1. **Brainstorming** - clarify the intent and produce a reviewed design.
2. **Writing plans** - map existing flows, define use cases, data structures, interfaces, and the integration test target.
3. **Use-case-driven development** - start from the approved plan, write the use-case-level integration test first, then implement.
4. **Subagent-driven development** or **executing plans** - execute the plan with review checkpoints.
5. **Requesting code review** - review against the plan and use case, not just against local implementation details.
6. **Verification before completion** - verify the observable behavior before claiming the task is done.

## Why Not Just Use Classic TDD?

Classic red/green/refactor TDD can still be valuable, especially for small pure functions, reused infrastructure, and hard-to-reproduce edge cases.

For coding agents, though, a rigid TDD template can create the wrong default behavior:

- Tests become distributed across many tiny internal functions.
- Mocks prove that calls happened, not that the product flow works.
- Agents spend time satisfying ceremony instead of clarifying contracts.
- Review becomes rare or shallow because each step appears locally green.
- The final system can pass tests while the use case remains incoherent.

Medium Powers tries a different default: one integration test group per use case, with explicit data structures, interface boundaries, and review gates.

## What's Inside

### Core workflow

- `using-medium-powers` - entry skill that tells the agent how to use the workflow.
- `brainstorming` - turns rough intent into a reviewed design.
- `writing-plans` - writes implementation plans around use-case flows and integration tests.
- `use-case-driven-development` - plan-gated integration-test-first implementation.
- `executing-plans` - executes approved plans in batches with checkpoints.
- `subagent-driven-development` - dispatches implementation work with review stages.

### Review and verification

- `requesting-code-review` - asks for focused review before issues compound.
- `receiving-code-review` - handles review feedback with technical judgment.
- `verification-before-completion` - requires evidence before completion claims.
- `systematic-debugging` - traces failures to root causes before fixing.

### Supporting skills

- `dispatching-parallel-agents` - coordinates independent parallel work.
- `using-git-worktrees` - isolates feature work.
- `writing-skills` - helps create and revise skills.

## Installation Details

The Codex local marketplace convention uses this file:

```text
~/.agents/plugins/marketplace.json
```

A local marketplace entry for Medium Powers should look like this:

```json
{
  "name": "medium-powers",
  "source": {
    "source": "local",
    "path": "./plugins/medium-powers"
  },
  "policy": {
    "installation": "AVAILABLE",
    "authentication": "ON_INSTALL"
  },
  "category": "Developer Tools"
}
```

If you update the Git checkout later, copy it again into the local plugin directory:

```bash
rsync -a --delete --exclude '.git' ./ ~/plugins/medium-powers/
```

## Codex Official Marketplace

Once Medium Powers is accepted into the official Codex plugin marketplace, installation should use Codex's normal plugin UI instead of the local-copy workflow:

1. Open **Plugins** in the Codex app sidebar, or run `/plugins` in Codex CLI.
2. Search for `Medium Powers`.
3. Select **Install Plugin**.

## Publishing As A Codex Plugin

This repository already includes the required Codex plugin manifest:

```text
.codex-plugin/plugin.json
```

Before publishing publicly:

1. Confirm `.codex-plugin/plugin.json` still points to `https://github.com/Lixuhang987/medium-powers` for `homepage`, `repository`, and `websiteURL`.
2. Review the plugin display metadata: `displayName`, `shortDescription`, `longDescription`, `developerName`, and `brandColor`.
3. If you add icons or screenshots, put them under `assets/` and reference them from `.codex-plugin/plugin.json`.
4. Tag a release after the metadata is correct.

To publish through the official Codex marketplace, follow the structure used by [openai/plugins](https://github.com/openai/plugins): each plugin lives under `plugins/<name>/` and contains its own `.codex-plugin/plugin.json`. A marketplace PR would add Medium Powers as `plugins/medium-powers/` with this manifest and the skill directories included.

## Design Principles

- Use cases are the unit of delivery.
- Integration tests should prove real flow coherence.
- Core data structures and interface contracts should be explicit before implementation.
- Mocks belong at external system boundaries, not around the business flow itself.
- Review should happen before local choices compound into architecture.
- Verification should produce evidence, not confidence theater.

## Status

Medium Powers is experimental. The workflow is intended to be used, criticized, and revised against real agent development sessions.

## Upstream Attribution

Medium Powers is based on and adapted from [Superpowers](https://github.com/obra/superpowers), created by Jesse Vincent and licensed under the MIT License.

The original Superpowers copyright and MIT license notice are preserved in [NOTICE](NOTICE).

## License

Medium Powers is licensed under the [Apache License 2.0](LICENSE). Portions adapted from Superpowers remain attributed under the original MIT license notice in [NOTICE](NOTICE).
