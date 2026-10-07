# Etch Studio + ACSS starter skill

A standalone AI agent skill for building and editing pages in **Etch Studio** through **Etch Connector**, using the project's existing **Automatic.css (ACSS)** framework.

The skill guides an agent through connecting to Studio, inspecting the active page, resolving generated ACSS tokens, building native editable blocks, applying class styles, saving changes, and checking the rendered result. It requires no other local skills or private project tools.

## Requirements

- An Etch Studio project with ACSS available.
- An AI coding agent with terminal access and support for `SKILL.md` skills.
- Node.js and npm to run Etch Connector.
- An open Studio builder tab connected through Etch's AI Connector.

This skill targets Etch Studio. It does not provide a WordPress workflow. Etch Studio, Automatic.css, and Etch Connector are separate products and are not bundled with this repository.

## Install

Clone this repository into your agent's skills directory as `etch-studio-acss`. Common user-level locations are:

| Agent | Skill directory |
| --- | --- |
| Codex | `~/.codex/skills/etch-studio-acss/` |
| Claude Code | `~/.claude/skills/etch-studio-acss/` |

For example, for Codex:

```bash
git clone https://github.com/gdcrosbie/etch-studio-acss.git ~/.codex/skills/etch-studio-acss
```

For Claude Code:

```bash
git clone https://github.com/gdcrosbie/etch-studio-acss.git ~/.claude/skills/etch-studio-acss
```

If your skills directory is managed by another repository or installer, follow that setup's installation process instead. For other agents, use their supported skill location. The installed folder must contain [SKILL.md](SKILL.md). Start a new agent session after installation if needed for discovery.

You can also give an agent the contents of `SKILL.md` directly if it does not support skill discovery.

## Connect and use

1. Open the intended page in Etch Studio.
2. Ask your agent to use the `etch-studio-acss` skill. The agent checks the local Connector and starts it if needed:

   ```bash
   npx @digital-gravy/etch-connector serve
   ```

   The first run may download the Connector package. Keep its terminal process running.

3. In Studio, enable **Settings → AI → AI Connector**, then use the **AI sparkles** toolbar button to connect the builder tab. If the interface differs, consult the current [Connector usage guide](https://docs.etchwp.com/etch-connector/usage).
4. Give the agent your page brief, copy, assets, and design direction. Keep the builder tab open while it works.

Example prompt:

> Use the etch-studio-acss skill and Etch Connector to build the open Etch Studio page with ACSS. Create a hero, three service cards, and a contact CTA using the supplied copy and images. Follow the project's existing styling, keep the blocks editable, and check desktop and mobile. Save the page for review.

The agent reads the live Studio guides and generated ACSS tokens before editing. Missing design choices should be raised with you. Shared components, global settings, external services, and publication are surfaced separately; saving a page does not authorise publishing it.

## Scope and verification

The skill covers semantic page structure, responsive styling, existing assets, reusable content, and simple interactions. A working form still needs an agreed submission destination.

Visual verification needs an available browser inspection tool, Studio preview, or a compatible Connector CDP session. CDP is optional. The agent should report any checks it could not perform and any remaining placeholders.

The Etch Public API is experimental. The skill directs the agent to check live capabilities and schemas rather than assume that examples match every version.

## Repository contents

- [SKILL.md](SKILL.md) — the complete standalone skill.
- [LICENSE](LICENSE) — MIT licence.

## References

- [Etch Connector usage](https://docs.etchwp.com/etch-connector/usage)
- [Etch Connector AI agent guide](https://docs.etchwp.com/etch-connector/ai-agent-guide)
- [Etch Public API](https://docs.etchwp.com/public-api)
- [Automatic.css documentation](https://docs.automaticcss.com/)

## Licence

Copyright © 2026 Graham Crosbie. Released under the [MIT License](LICENSE).

The licence covers this repository's skill and documentation; it does not license the separate products used by the workflow. This is an independent community skill, not an official Etch or Automatic.css distribution.
