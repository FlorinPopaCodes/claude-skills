# Rails Codegen Plugin

Rails code generation standards for Claude Code, based on [Evil Martians' AGENTS.md](https://gist.github.com/palkan/482010bd6ec434685f106779e863d0ef) by [@palkan](https://github.com/palkan).

## Purpose

Ensures AI-generated Rails code follows conventions and remains maintainable. As the original guidelines state:

> "Generated code should be so simple and clear that reading it feels like reading well-written documentation."

## Features

- Comprehensive Rails conventions organized by building blocks
- Clear constraints for code generation
- Anti-patterns to avoid with recommended alternatives
- Technology stack guidance (required/forbidden gems)

## Skill Activation

This skill activates automatically when you:
- Generate Rails models, controllers, views, or jobs
- Ask about Rails conventions or best practices
- Create database migrations or schema changes
- Set up background jobs or configuration
- Write RSpec tests for Rails applications

## Reference Topics

| Reference | Content |
|-----------|---------|
| `stack.md` | Required/forbidden gems, file structure |
| `models.md` | Model patterns, organization, enums, validations |
| `controllers.md` | Thin controllers, guard clauses, namespacing |
| `database.md` | Schema design, constraints, indexes, migrations |
| `jobs.md` | Background work, ActiveJob::Continuable, workflows |
| `views.md` | Hotwire, ViewComponent, frontend patterns |
| `forms-queries.md` | Form objects, query objects |
| `testing.md` | RSpec conventions, test organization |
| `configuration.md` | anyway_config patterns |
| `anti-patterns.md` | Common mistakes, alternatives, checklist |

## Credits

Based on practical standards from [Evil Martians](https://evilmartians.com), produced through iterative refactoring of real-world Rails applications.
