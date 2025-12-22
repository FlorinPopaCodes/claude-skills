# Claude Code Plugin Marketplace

A personal collection of plugins for [Claude Code](https://claude.com/claude-code).

## Plugins

### always-works-testing

Default testing standard for all implementation work - ensures code actually works through mandatory execution validation before confirming to user.

**Category:** Testing
**Keywords:** testing, validation, quality, verification, TDD

---

### simplicity-first

Cornerstone plugin embedding the "simplest thing that could possibly work" philosophy into ALL development tasks. YAGNI as supreme principle - design for now, not imagined futures.

**Category:** Architecture
**Keywords:** simplicity, YAGNI, architecture, design, cornerstone, philosophy
**Inspired by:** [The Simplest Thing That Could Possibly Work](https://www.seangoedecke.com/the-simplest-thing-that-could-possibly-work/) by Sean Goedecke

---

### cognitive-load

Cornerstone plugin applying cognitive load principles to reduce mental burden in code. Humans hold ~4 chunks in working memory - this skill helps minimize extraneous cognitive load.

**Category:** Code Quality
**Keywords:** cognitive-load, complexity, readability, code-review, mental-model
**Inspired by:** [Cognitive Load in Software Development](https://github.com/zakirullin/cognitive-load) by Artem Zakirullin

---

### rails-codegen

Rails code generation standards based on Evil Martians' AGENTS.md. Ensures AI-generated Rails code follows conventions and remains maintainable.

**Category:** Frameworks
**Keywords:** rails, ruby, codegen, conventions, best-practices
**Based on:** [AGENTS.md](https://gist.github.com/palkan/482010bd6ec434685f106779e863d0ef) by @palkan (Evil Martians)
**License:** MIT

## Installation

Add this marketplace to Claude Code:

```bash
/plugin marketplace FlorinPopaCodes/claude-marketplace
```

Then install individual plugins as needed.

## License

This marketplace and its included plugins are released under the [Unlicense](./LICENSE) (public domain), except where noted otherwise (rails-codegen uses MIT license).
