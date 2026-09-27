# UI Toolchain

UI quality toolchain for AI coding agents (ZCode / Claude Code compatible plugin): design spec with a 7-dimension review rubric, Anthropic's aesthetic direction guidance, 129 authoritative UX audit checklists, a screenshot review command and an independent reviewer agent.

[中文说明（README.zh-CN.md）](README.zh-CN.md)

## Install

**ZCode**: Settings → Plugin Marketplace → Add → Add Plugin Marketplace → paste this repo URL (`https://github.com/Bryce-Peng/ui-toolchain-marketplace`) → Install **UI Toolchain**.

**Claude Code**: `/plugin marketplace add <you>/<repo>` → `/plugin install ui-toolchain`.


## Components

| Component | Type | What it does |
|---|---|---|
| `design-spec` | skill | Design spec: token discipline, negative list, state completeness, 7-dimension rubric, immediate-mode GUI alignment recipes, per-stack observation channels |
| `frontend-design` | skill | Anthropic's aesthetic direction guidance (anti-slop), vendored |
| `checklist-design` | skill | Checklist Design's 129 authoritative UX checklists (audit/critique modes, works offline), vendored, MIT |
| `/ui-review` | command | Run the 7-dimension rubric review on a screenshot / URL / local page; outputs a scored report + structured fix instructions |
| `ui-reviewer` | agent | Independent reviewer subagent (implementation/review separation) |
| `references/` | docs | vercel web-interface-guidelines, two SOPs (greenfield / legacy retrofit), GUI observation & alignment recipes |

## Usage

- Design rules auto-apply to any UI task; say "self-review against the rubric" before delivery.
- Review a page: `/ui-review <screenshot-path-or-URL>`, or dispatch the ui-reviewer subagent.
- Legacy retrofit: point the agent at `references/sop-legacy.md` (three-step strangler process).

## Language of generated UI copy

Follows your conversation / project requirements — no mode switch. Hard rule: if copy contains CJK on an immediate-mode GUI stack (egui/Slint/Fyne), CJK font injection is an acceptance item (recipes in `references/gui-recipes.md`).

## License

- Plugin structure: MIT
- `frontend-design`: Apache-2.0 (Anthropic), vendored
- `checklist-design`: MIT (Checklist Design)
