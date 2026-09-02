# Solana Finance Plugin & Skill

![Quicknode Solana Finance Claude Skill](assets/banner.png)

A Claude Code plugin for creating and editing Solana projects, specifically Rust projects in Anchor and Quasar, with a focus on:

- Accuracy, consistency and security for financial software
- Maintainability, readability, and minimal code
- Current best practices including Rust/LiteSVM for testing

This plugin has the **most stars of any ecosystem Solana Claude plugin** and is based on production-code used by some of the largest programs on Solana. If you're new to Solana: Solana programs are called 'smart contracts' on older blockchains, that's what this plugin builds.

> [!TIP]
> If you find this plugin useful, please add a GitHub star above! 🙏

## What is this?

**Solana Finance** is a Claude Code plugin that bundles the `solana` skill - a reusable instruction set Claude automatically applies when working on Solana code, or that you can invoke manually. Skills are triggered automatically based on context (e.g. opening an Anchor program or a Solana Kit client).

## Installation

### As a plugin (recommended)

Add this repository as a marketplace, then install the plugin from it:

```
/plugin marketplace add quicknode/solana-finance-claude-plugin
/plugin install solana-finance@quicknode
```

### As a standalone skill

You can also install just the skill directly:

```bash
npx skills add https://github.com/quicknode/solana-finance-claude-plugin
```

This installs the skill into the current project's `.claude/skills/` directory. Add `-g` to install it for every project, under `~/.claude/skills/`, and `--agent claude-code` if the CLI detects other agents you do not want it installed for.

## Usage

Once installed, the skill automatically applies when Claude Code works on Solana/Anchor/Quasar projects. You can also invoke it manually:

```
/solana-finance:solana
```

(When installed as a standalone skill rather than via the plugin, invoke it with `/solana`.)

## Repository layout

```
.claude-plugin/plugin.json   # Plugin manifest (name, author, version)
.claude-plugin/marketplace.json  # Marketplace manifest, so /plugin marketplace add accepts this repo
skills/solana/               # The skill
  SKILL.md                   # Skill entry point + general guidelines
  ANCHOR.md  RUST.md  QUASAR.md  TYPESCRIPT.md   # Language/framework references
  ANCHOR-V2.md               # Porting a program to Anchor 2.0.0-rc.1
  ENVIRONMENT.md             # Toolchain setup for CI and remote containers
  SUMMARIZING-PROGRAMS.md    # How to explain and summarize programs
  DIAGRAMS.md                # Account diagram (SVG figure) conventions
```
