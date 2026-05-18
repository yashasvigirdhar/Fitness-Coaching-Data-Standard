# Standard Seed Data

**Fixture:** `examples/standard-seed-package.v1.json`  
**Schema:** `open-coaching-data.v1.schema.json`  
**Purpose:** provide one stable, realistic dataset that every competitor migration pilot tries to seed, extract, normalize, and compare.

## Why This Exists

The seed fixture is our control sample. If we enter the same coaching data into multiple coaching platforms, then each extractor can be judged against the same expected output.

For each platform run, the goal is:

1. Seed as much of this fixture as the platform supports.
2. Record any field or domain the platform cannot represent.
3. Extract the account back into `open-coaching-data.v1`.
4. Compare the extracted package against this seed fixture.
5. Explain all mismatches as either platform limitations, seeding limitations, extraction limitations, normalization differences, or schema gaps.

This makes migration quality measurable instead of vibes-based.

## Source Of Truth

The canonical seed data for this public standard repo lives in:

```text
examples/standard-seed-package.v1.json
```

Do not create platform-specific seed datasets unless the platform forces a transformation. Platform-specific seed docs should reference this fixture and maintain a run log of what was entered, skipped, transformed, or blocked.

Older platform-specific seed documents can be useful during early pilots, but the V1 fixture should be the primary seed source going forward.

## Coverage

The fixture intentionally covers every tight V1 domain:

| V1 domain | Seed coverage |
|---|---|
| Manifest | Package metadata, record counts, source identity, validation status |
| Coach | Coach profile, business name, contact, timezone, units, brand/settings |
| Clients | Five realistic clients with different schedules, goals, and constraints |
| Client assignments | Assignments from clients to forms, macro-only meal plans, and workout plans |
| Forms | Intake form, weekly check-in form, field definitions, submissions |
| Check-ins | First-class check-in records linked to check-in form submissions |
| Nutrition | Macro-only meal plans with assigned plan names, status, and targets |
| Training | Workout plans, workout sessions, and embedded prescribed exercise details |

## Client Personas

The five seed clients are intentionally different from each other so coaches can recognize practical migration scenarios instead of toy data:

| Client | Practical scenario | Why it matters for migration testing |
|---|---|---|
| Jordan Lee | Barbell trainee pursuing fat loss while protecting strength | Tests gym-based strength programming, weekly check-ins, body measurements, and macro targets. |
| Avery Patel | Beginner/recomposition client training mostly at home | Tests simpler equipment, beginner-friendly workout plans, and adherence/support check-ins. |
| Marcus Reed | Hybrid endurance client balancing running/cycling and strength | Tests cardio-style prescriptions, higher calorie targets, and fatigue/context notes. |
| Sofia Martinez | Postpartum return-to-training client | Tests conservative training constraints, health-context sensitivity, and lower-volume programming. |
| Noah Kim | Frequent-travel professional with high stress | Tests flexible programming, realistic lower adherence, and travel-friendly nutrition targets. |

When seeding a competitor platform, use these personas as the canonical target. If the platform cannot represent part of a persona, record the limitation in that platform's run log instead of weakening the shared fixture.

### Persona Detail Matrix

Each client should stay recognizable as a real coaching case when seeded into a competitor platform. The data should not read like five copies of the same client with different names.

| Client | Nutrition target | Training target | Check-in/intake emphasis |
|---|---|---|---|
| Jordan Lee | Moderate calorie deficit, high protein, strength-preserving macros | Gym-based strength plan with barbell lifts | Body measurements, strength consistency, knee-irritation context, weekly support needs |
| Avery Patel | Recomposition target with beginner-friendly calories and protein | Short home sessions with dumbbells/bands/bodyweight | Habit consistency, confidence, equipment limits, beginner adherence |
| Marcus Reed | Higher carbohydrate performance target for endurance training | Hybrid 10K support with strength maintenance | Fatigue, recovery, run/bike balance, training load context |
| Sofia Martinez | Conservative return-to-training target with flexible nutrition | Low-volume postpartum-friendly full-body work | Core control, sleep disruption, health-context sensitivity, gradual progression |
| Noah Kim | Sustainable fat-loss target for travel and inconsistent schedule | Hotel-gym/travel-proof sessions | Stress, schedule variability, low-friction nutrition, realistic adherence |

When a platform supports only part of a persona, keep the supported values exact and record the missing pieces in that platform's run log. Do not simplify the shared fixture just to make one platform look more complete.

Current expected record counts:

| Record type | Count |
|---|---:|
| Coach profiles | 1 |
| Coach settings | 1 |
| Clients | 5 |
| Client assignments | 15 |
| Form templates | 2 |
| Form submissions | 7 |
| Check-ins | 5 |
| Meal plans | 5 |
| Workout plans | 5 |
| Workout sessions | 5 |
| Workout session exercises | 6 |

## Seeding Rules

- Use the exact fixture values wherever the platform allows.
- Prefer realistic synthetic emails and phone numbers; never seed real client data.
- Do not send client invitations, payment requests, purchase links, SMS messages, app invites, or emails unless the run explicitly approves that action.
- If a platform requires a routable email to create a client, pause and record the blocker instead of improvising.
- If a platform transforms a value, record both the seed value and the displayed/exported value.
- If the platform lacks a field, skip it and record the skip reason.
- If the platform has a richer field than V1, record it in private run diagnostics, not in the V1 seed fixture.

## Skip Reason Codes

Use these codes in platform run logs:

| Code | Meaning |
|---|---|
| `unsupported_by_platform` | The platform has no equivalent concept or field. |
| `requires_client_invite` | The data requires inviting/logging in as a client. |
| `requires_payment_or_billing_action` | The data requires payment setup, purchase, or billing action. |
| `requires_mobile_app` | The data can only be entered from the platform's mobile app. |
| `requires_wearable_or_integration` | The data requires connecting an external account/device. |
| `read_only_sample_data` | The field is visible in sample/demo data but cannot be edited for the seed account. |
| `not_seeded_by_choice` | The platform supports it, but we intentionally skipped it for this run. |
| `blocked_by_validation` | The platform rejected the canonical value and no safe substitute was chosen. |
| `unknown` | We could not determine support during this run. |

## Run Log Requirements

Each platform should keep a seed run section with:

- Run ID and date.
- Platform account used.
- Fixture version/path.
- Domains attempted.
- Domains seeded successfully.
- Domains skipped with reason codes.
- Field transformations.
- Counts entered into the platform.
- Counts extracted back out.
- Comparison result against the fixture.

The comparison result should never require private source IDs or internal platform URLs in the open standard. Those details belong in local diagnostics only.

## Comparison Policy

A platform extraction passes the seed test when:

- The extracted package validates against `open-coaching-data.v1`.
- Every seeded value that the platform accepted appears in the extracted package or has a documented omission.
- Counts match for every seeded domain the platform supports.
- Transformations are explainable, repeatable, and documented.
- Unsupported platform areas are recorded separately from extractor failures.

This distinction matters: if a source platform cannot represent a V1 field, that is platform coverage. If the source platform can represent the field but an extractor misses it, that is extractor coverage.
