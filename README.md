# Fitness Coaching Data Standard

The Fitness Coaching Data Standard is an open draft JSON standard for portable fitness coaching business data.

The goal is to give fitness coaches a clean, product-neutral way to move essential data between coaching platforms. This repository starts with the V1 standard, design principles, and a realistic seed/example package. Public extraction helpers for specific fitness coaching platforms can be added later as separate tools that output this standard.

## Repository Structure

```text
schemas/
  v1/
    open-coaching-data.v1.schema.json
examples/
  standard-seed-package.v1.json
docs/
  design-principles.md
  standard-seed-data.md
```

## V1 Scope

Tight V1 focuses on structured records needed for migration continuity:

- Coach profile and settings
- Clients
- Client assignments
- Form templates and form submissions
- Check-ins linked to form submissions
- Macro-only meal plans
- Workout plans, sessions, and prescribed exercise details

V1 intentionally excludes platform internals, payment data, private source URLs, screenshots, generic extension bags, standalone exercise libraries, detailed meal foods, media files, messages, notes, goals, and workout logs.

## Status

Draft. The schema and example package are intended for review before this repository is published publicly.
