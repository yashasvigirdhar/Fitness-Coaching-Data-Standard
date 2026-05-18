# Fitness Coaching Data Standard Design Principles

**Last updated:** 2026-05-18
**Applies to:** `open-coaching-data.v1`

## Purpose

This document records the design principles behind the Fitness Coaching Data Standard JSON format.

The standard needs to survive unknown future data because competitor platforms will expose domains, fields, and quirks we cannot fully predict. The goal is not to create a perfect model on the first attempt. The goal is to create a stable, understandable, extensible package that can preserve coaching data faithfully, import the supported parts safely, and evolve without breaking every extractor or importer.

## Research Basis

These principles are based on established interoperability patterns from JSON, schema, API, and data-exchange standards:

- [RFC 8259 JSON](https://www.rfc-editor.org/rfc/rfc8259.html): JSON interoperability guidance, including unique object member names and avoiding reliance on object member order.
- [RFC 7493 I-JSON](https://www.rfc-editor.org/rfc/rfc7493): stricter JSON profile for interoperable internet messages, including UTF-8, numeric safety, duplicate-key rejection, must-ignore extension handling, RFC3339 timestamps, and binary handling recommendations.
- [JSON Schema](https://json-schema.org/understanding-json-schema/reference/object): schema tools for object validation and controlled extensibility, including `additionalProperties` and `unevaluatedProperties`.
- [Semantic Versioning](https://semantic-versioning.org/): compatibility semantics for major, minor, and patch changes.
- [JSON:API extensions and profiles](https://jsonapi.org/extensions/): distinction between extensions that must be understood and profiles that can be ignored.
- [W3C extensibility notes](https://www.w3.org/wiki/ExtensionSpeclite): explicit planning for extension behavior, including must-ignore and must-understand approaches.
- [HL7 FHIR extensibility](https://www.hl7.org/fhir/extensibility.html): structured extension mechanisms, modifier extensions, retaining unknown extensions where possible, and using published definitions for extension meaning.

## Principles

### 1. Separate The Standard From Any Product Database

The JSON format should describe coaching concepts, not any one product's tables or pages.

Applied in the current draft:

- Top-level domains use product-neutral names such as `clients`, `forms`, `check_ins`, `nutrition`, and `training`.
- Product import/export code should be adapters, not the standard itself.
- Platform extractors should normalize into `open-coaching-data.v1`, not directly into destination database rows.

Why:

- A standard tied to our database will inherit our current product gaps.
- A neutral standard lets future platforms and tools produce the same package regardless of source or destination.

### 2. Prefer Additive Evolution

New fields, new optional domains, and new optional enum values should usually be minor-version changes. Removing fields, changing meanings, or adding new required fields should require a major version.

Applied in the current draft:

- `schema_version` uses semantic version shape, for example `1.0.0`.
- Most sections are optional.
- Most object fields are optional except for stable identity and minimum interpretability fields.

Rules:

- Adding an optional field is compatible.
- Adding a new optional domain is compatible.
- Adding a new required field is breaking.
- Removing a field is breaking.
- Changing a field type is breaking.
- Changing the meaning of an existing field is breaking.

### 3. Keep Core Fields Strict

The standard should validate known core fields strictly. Tight V1 should not include a generic extension bag until there is a concrete, well-defined interoperability use case.

Applied in the current draft:

- Core objects generally use `additionalProperties: false`.
- Source-platform data should be modeled as neutral standard domains or kept in extractor-private artifacts until the standard can represent it.

Rules:

- Unknown fields should not be sprinkled randomly into core objects.
- Data that cannot be safely modeled in product-neutral terms should not be forced into the open package.
- If a future version needs extensions, define the extension mechanism explicitly and version it.

This keeps V1 strict, readable, and easier to validate.

### 4. Distinguish Must-Ignore From Must-Understand Data

Some unknown data can be safely ignored. Some unknown data changes the meaning of a record and must not be ignored.

Applied in the current draft:

- Unknown source data is kept in extractor-private diagnostics or import/export reports unless it can be represented as a neutral V1 field.
- Importers should produce warnings when they cannot understand or import data from an otherwise valid package.

Future improvement:

- Add a first-class extension or `requires_understanding` mechanism if real migrations reveal portable data that cannot fit the strict V1 fields.

### 5. Keep Public Provenance Minimal

The open package should identify the source platform and extraction timing, but it should not include competitor-internal object IDs, exact private app URLs, selectors, screenshots, or other extractor/debug artifacts.

Applied in the current draft:

- `manifest.source_platform` identifies the source platform at package level.
- `manifest.created_at` identifies when the portable package was created.
- Exact platform object IDs, internal URLs, selectors, screenshots, and detailed evidence belong in extractor-private logs or runbooks, not in the open standard package.
- Platform guides should document where to find each type of data without embedding private source URLs into the portable package.

Why:

- The open format should not leak implementation details of competitor systems.
- Coaches may share the package with another importer, consultant, or open-source validator.
- Debugging evidence can be useful during extraction, but it is not necessarily portable coaching data.

### 6. Prefer Structured Source Data During Extraction

When an extractor runs inside a coach-authorized browser session, it should prefer structured data already available to that session before falling back to rendered DOM or screenshots.

Applied in the current draft:

- The open package records normalized coaching data, not the extraction channel.
- Extractor-private diagnostics and migration reports should record whether each domain came from a source API, server-rendered HTML, rendered DOM, a download, or screenshot-assisted extraction.
- Structured source APIs are preferred when they expose the same coach-visible data without requiring additional permissions, secrets, or unsupported access.
- Rendered DOM remains a valid fallback for data that is only present in the UI, and a useful verification layer even when structured APIs are available.

Rules:

- Use only data the coach is authorized to access through their own account/session.
- Do not store auth tokens, cookies, API credentials, internal object IDs, or private endpoint URLs in the open package.
- Prefer structured source data because it is usually less lossy than screenshots or DOM text.
- Verify critical structured extracts against visible UI where practical, especially during early platform pilots.
- If a structured source endpoint and the visible UI disagree, record the mismatch in private diagnostics and report it rather than silently choosing one.

Why:

- Many modern platforms render UI from internal JSON APIs.
- Using those existing structured responses can preserve relationships, units, ordering, and nested records more reliably than screen text.
- Keeping the extraction channel outside the public standard preserves portability while still making extractor runs auditable.

### 7. Use Package-Scoped IDs

Records should reference other records using stable IDs inside the package, not source-platform IDs and not destination database IDs.

Applied in the current draft:

- Objects use IDs like `client_001`, `form_001`, `checkin_001`, `exercise_001`, and `workoutplan_001`.
- Source platform IDs are not stored in the public package. Extractors can keep them in private run logs if needed for debugging.
- Destination systems assign their own IDs during import.
- Relationship fields such as `assigned_record_id`, `form_template_id`, `meal_id`, and `workout_session_id` must point to records that are present in the same package.

Why:

- The same package can be validated before import.
- Importers can resolve relationships without relying on source-platform IDs or destination database IDs.
- Re-import, deduplication, and dry-run behavior become easier to reason about.
- Packages avoid phantom assignments, where a client appears to have a form or workout plan that was not actually exported.

### 8. Model Relationships Explicitly

Do not rely on folder placement, array order, or duplicated text to infer relationships.

Applied in the current draft:

- Client assignments live in `clients.client_assignments`.
- Client assignments should only be emitted when the assigned form template, meal plan, or workout plan is also included in the package.
- Form submissions reference `client_id` and `form_template_id`.
- Check-ins reference `client_id` and optionally `form_submission_id`.
- Workout session exercises reference `workout_session_id` and embed the prescribed exercise details.

Why:

- Many coaching objects are reusable.
- Coaching platforms often separate reusable content from client assignments.
- Explicit references make dry-run reports and conflict handling possible.

### 9. Preserve Original Values When Normalizing

When converting units, ratings, dates, labels, or source-specific values, keep the original value if conversion could lose meaning.

Applied in the current draft:

- Ratings support `source_display`.
- Form answers support `original`.

Rules:

- Store normalized values only when unit and semantics are known.
- Keep source labels when UI labels explain meaning.
- Never silently convert a value if the conversion is uncertain.
- If a value is uncertain, prefer preserving the original value and warning during dry-run rather than adding extractor-debug metadata to the public package.

### 10. Use Explicit Units And Timezones

All measurements should carry units. All timestamps should include timezone.

Applied in the current draft:

- Measurements use `{ "value": number, "unit": string }`.
- Durations use `{ "value": number, "unit": "seconds" | "minutes" | "hours" }`.
- Timestamps use JSON Schema `date-time`.
- Date-only fields use JSON Schema `date`.

Rules:

- Avoid unitless numbers except for true counts or scale values.
- For source-local dates with unclear timezone, preserve the original value and record uncertainty in extractor-private diagnostics or the dry-run report.
- Prefer RFC3339-compatible timestamp strings.
- Put a timestamp on the event it directly describes. Linked records should reference that event instead of repeating the same timestamp.

### 11. Treat JSON Objects As Unordered

The meaning of the package must not depend on object member order.

Applied in the current draft:

- Ordered content uses arrays plus `order`, `day_index`, or `set_number`.
- Object keys are not used to imply display order.

Rules:

- Use arrays for ordered form fields, meals, workouts, prescribed exercises, and sets.
- Include explicit order fields where order matters.
- Do not define semantics based on object property ordering.

### 11. Keep JSON Interoperable

The package should follow I-JSON-style interoperability constraints.

Rules:

- Encode JSON as UTF-8.
- Do not use duplicate object member names.
- Do not use `NaN`, `Infinity`, or other non-JSON number values.
- Avoid numeric values that exceed interoperable JSON number precision. Use strings for exact large identifiers.
- Do not rely on member order.
- Prefer object or array at the top level.

Applied in the current draft:

- Source platform identifiers, when used in private diagnostics, should be treated as strings.
- Package IDs are strings.
- Money, measurements, and durations use typed objects rather than ambiguous raw numbers.

### 12. Do Not Invent Data To Satisfy A Schema

Unknown data should remain unknown. Required fields should be minimal enough that extractors do not need to fabricate values.

Applied in the current draft:

- Most fields are optional.
- Extractor-private diagnostics can preserve raw data that cannot be normalized into the open standard.
- Extractor-private diagnostics and dry-run reports can mark uncertain extracted values.

Rules:

- Do not guess a client's age from an ambiguous date.
- Do not create fake units.
- Do not convert "not visible" into false.
- Use warnings, `original`, and extractor-private diagnostics instead.

### 13. Defer Media Until It Can Be Designed Carefully

Media can be huge, private, missing, temporary, or legally sensitive. Tight V1 intentionally excludes media files, photos, videos, and media manifests.

Applied in the current draft:

- V1 keeps structured coaching data independent from media packaging.
- Form and check-in schemas do not require image, video, or file fields.
- Future media support should be designed as a dedicated V2 package concern.

Rules:

- Do not block structured migration because media is not available.
- Do not pretend referenced media was downloaded.
- Add media only when the standard has clear rules for availability, file packaging, privacy, and parent context.

### 14. Exclude Secrets And Dangerous Operational Data

The standard is for coach-owned fitness coaching business data, not credentials or platform internals.

Applied in the current draft:

- The plan explicitly excludes passwords, session cookies, OAuth tokens, API keys, payment credentials, raw payment processor objects, webhook logs, and platform-internal operational logs.
- Exporter or importer reports can record why sensitive operational data was omitted, outside the open package.

Rules:

- Never put payment instruments in the open package or extractor artifacts intended for sharing.
- Preserve safe metadata only when it helps explain the migration.

### 15. Validate Before Importing

The schema should support a dry-run workflow before any data is written to a destination product.

Applied in the current draft:

- `manifest.record_counts` enables summaries.
- `warnings` captures package-level issues.
- `validation_status` records validation state.
- Package-level source platform metadata and private extractor run logs support gap reporting without exposing extractor internals in the open package.

Rules:

- A destination importer should validate schema first.
- It should report what that destination can import, partially import, or skip without writing those statuses back into the open package.
- The coach should approve the report before write.

### 16. Keep Human Inspection Possible

The package should be understandable to a coach, support person, or migration engineer without a custom UI.

Applied in the current draft:

- Domain folders mirror coaching concepts.
- JSON is canonical, but CSV projections can exist for review.
- The manifest and package summary should explain counts, omissions, and warnings.

Rules:

- Avoid deeply clever encodings.
- Prefer readable field names over compact field names.
- Keep free-text content intact.
- Include a README in exported packages.

### 17. Treat Enums As Open-Ended Product Reality

Enums are useful for validation, but real platforms will reveal values we did not predict.

Applied in the current draft:

- Many enums include `other` or `unknown`.
- Source-specific raw scalar values can be preserved in `original` when conversion could lose meaning.

Rules:

- If an enum value is unknown but harmless, preserve the original and map to `other` or `unknown`.
- If an unknown value changes meaning, keep it out of the open package until it can be represented neutrally, and surface it in extractor/importer reports.
- Adding enum values should be treated as a compatible minor-version change when consumers are expected to ignore unknown values or map them to `other`.

### 18. Prefer Composition Over Overloaded Fields

One field should not mean different things in different contexts.

Applied in the current draft:

- Workout prescriptions are modeled as structured plan/session content with embedded exercise details; standalone exercise libraries, workout logs, and exercise set logs are deferred beyond tight V1.
- Nutrition is modeled as macro-only meal plans assigned through `clients.client_assignments`; detailed meals, meal foods, and food databases are deferred beyond tight V1.
- Forms, form fields, submissions, and answers are separate objects.

Why:

- Migration packages need to represent both reusable coaching content and historical client activity.
- Overloaded fields make import behavior brittle.

### 19. Document Compatibility Rules Beside The Schema

A JSON Schema can validate shape, but it does not fully explain product semantics.

Applied in the current draft:

- The plan explains package shape and phases.
- This principles doc explains compatibility and extensibility decisions.
- Platform-specific docs explain coverage and gaps.

Rules:

- Every stable version should publish schema files, domain reference docs, examples, and changelog.
- Breaking-change rules should be documented before the standard is open sourced.

### 20. Promote Repeated Platform Learnings Into Core Fields Carefully

Platform-specific data should remain in extractor-private diagnostics until repeated platform learnings justify a neutral standard field. Tight V1 intentionally does not include a generic extension object.

Rules:

- One competitor quirk does not automatically become a standard field.
- A field should become core when it is broadly useful, clearly defined, and can evolve compatibly.
- Migration learnings should feed change proposals rather than one-off schema churn.

## Current Draft Assessment

The current `open-coaching-data.v1` schema already follows several of these principles:

- Product-neutral domain names.
- Package-scoped IDs.
- Clean domain records without per-record extractor provenance.
- Strict records with no generic extension bag.
- Explicit units and date/time formats.
- Optional sections and minimal required fields.

Areas to refine before open sourcing:

- Decide whether a future version needs a typed extension or `requires_understanding` mechanism.
- Create compatibility rules for enum expansion.
- Add examples showing source-specific values represented safely without platform internals or destination assumptions.
- Add a formal changelog and versioning policy.
- Add validator behavior for duplicate keys, number precision, and timestamp timezone requirements.

## Source Notes

- RFC 8259 says unique object names improve interoperability and that relying on member order creates parser differences.
- RFC 7493 recommends I-JSON constraints for interoperable messages, including UTF-8, duplicate-key rejection, safe numeric ranges, must-ignore handling for new protocol elements, RFC3339 timestamps, and base64url for binary data when binary must be embedded.
- JSON Schema provides tools for deciding whether a schema is closed, open, or selectively extensible.
- Semantic Versioning gives us a simple compatibility language for schema evolution.
- JSON:API and W3C distinguish ignorable extensions from extensions that must be understood.
- FHIR shows a mature pattern for structured extensions, modifier extensions, unknown-extension preservation, and warning/rejection behavior when unknown data changes meaning.
