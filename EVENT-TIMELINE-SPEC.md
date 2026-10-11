# Event timeline for Typst_Template

> Keep dated project records in a dedicated timeline directory and present their full notes chronologically for later research writing.

Status: Design draft ready for final review; implementation has not started. Helper feasibility, dependencies, and editor-specific save integration remain pre-implementation gates.
Last updated: 2026-10-08.

## Motivation

The user's notes project will run for approximately five months.
Its record should preserve activities, observations, and decisions so the user can reconstruct their thinking when writing a scientific paper.
Writing an entry must remain quick enough for regular use.

## Settled requirements

1. The authoring macro is defined as `#event`!
2. Each rendered event requires a stored explicit date and title. An author may leave `date: ""` while drafting; the accepted save integration fills it permanently before saving. Compilation still rejects dates that remain empty or invalid.
3. Optional fields are `evidence`, `milestone`, and `workitem`.
   These fields describe the event now; GitHub integration remains future work.
   `milestone` and `workitem` are plain text, not structured GitHub references.
4. Event notes support rich Typst content, including lists, math, code, tables, images, citations, numbered figures, footnotes, and labels. There are no mandatory kind or rationale fields. Nested event/timeline collection is forbidden; headings inside a note stay local rather than creating globally numbered chapters.
5. Event authoring is restricted to files inside project-root `timeline/`, a sibling of `chapters/` and `assets/`. Event calls in ordinary chapters and other project files are forbidden, not merely discouraged.
6. Authors may split timeline files by fortnight, milestone, or another useful boundary without milestone-based PDF ordering.
7. Timeline sources are registered once in an explicit ordered file list. Same-date events retain that list's order between files and top-to-bottom order within each file. Physical file organization does not override chronology. Automatic file discovery is not part of the initial version; nested-source-include policy remains a proposed default for review.
8. The generated timeline sorts events by date, oldest first and newest last.
9. Milestone ordering and grouping are deferred.
10. A future GitHub Projects integration may supply milestone and work-item structure; its concrete design is deferred.
11. PDF chapter backlinks were selected for the earlier scattered-source model. The proposed single-render model needs no separate source-chapter destination; source code file/line navigation remains outside scope.
12. Focused-preview loading is reopened: dedicated sources could make the timeline complete without including ordinary chapters. Do not silently assume either the old included-chapters limitation or this proposed replacement.
13. The reusable capability must reach notes repositories through normal upstream template updates.
14. This session prepares the design and specification only. Do not implement macros, alter template behavior, compile prototypes, commit, or publish this feature yet.
15. The timeline shows the full event note and all supplied fields, rather than a compact summary.
16. Event notes should be boxed, visibly distinct from ordinary document text. The full chronological entry remains required in both output modes.
17. There is no author-facing event ID field in the initial interface.
18. Document TODOs are completely separate from events, including collection, rendering, and work-item associations.
19. Support configurable output choices: ordinary notes followed by a dedicated timeline chapter, or the timeline alone without ordinary notes chapters.
20. Both choices use the same project event sources, dates, fields, and chronological ordering. Changing output must not require rewriting or duplicating event records.
21. A build helper is acceptable for source-placement validation and checked local PDF builds. Its dependencies and supported authoring forms must be disclosed before implementation proceeds.
22. Keep the existing GitHub PDF workflow unchanged for now: no extra output variants, validation steps, dependencies, triggers, or artifact changes as part of this feature.
23. Add automatic date filling on save for event calls in `.typ` files inside `timeline/`: replace only an empty literal `date: ""` with the current local calendar date in `YYYY-MM-DD` format. Preserve all nonempty dates, including manually entered historical dates, and never recompute them on subsequent saves or builds.

> **Revised:** Dedicated `timeline/` files replace scattered authoring in ordinary chapters. The user now wants configurable notes-plus-timeline and timeline-only outputs. The proposed interpretation is one chronological rendering in either output, not separate printed source chapters plus an additional index. Plain-text associations, full notes, boxes, date ordering, and source-order ties remain.

> **Confirmed:** The user selected an explicit ordered file list and rich Typst event bodies. The implementation must preserve rich content rather than flattening notes into strings or restricting them to simple prose.

> **Revised:** A local build helper is accepted, but the GitHub workflow must remain as it is. Earlier proposals to integrate the helper into CI or offer extra workflow exports are deferred. The current direct-compiler workflow does not run helper-only source checks; document this boundary rather than claiming identical validation locally and on GitHub.

> **Confirmed:** Date filling on save is part of the implementation plan. It writes a permanent literal into the saved source rather than using a compilation-time `datetime.today()` fallback. The editor has not yet been specified; its save adapter remains a technical integration gate.

## Existing integration points

- `main.typ` registers content files through `include-chapters`.
- `include-chapters` excludes unselected files during chapter-focused builds.
- It currently loads only declared files, without automatic directory discovery.
- `template.typ` already uses metadata, contextual queries, and internal PDF links for TODO summaries. Events can use the same Typst primitives with an independent metadata namespace.
- The template supports a portrait summary profile and a landscape profile with four narrow columns.
- Current config files define `print-profile` dictionaries for layout, typography, front matter, and TODO visibility. `main.typ` supports project-specific overrides. The new content selection should not hard-code a relationship between a layout preset and an output mode.
- Project event records belong to the notes project; shared event functions and defaults belong to the template.
- The existing PDF workflow compiles the complete document but has no source-placement validation step.
- `AGENTS.md` currently directs authored content to `chapters/`. Implementation must document `timeline/` as a specific exception without broadening ordinary notes search to unrelated files.
- This planning document is separate from ordinary authored content and template behavior.

## Proposed structure, pending confirmation

### Event authoring

Conceptual interface, not implemented; used only inside `timeline/`:

```typst
#event(
  date: "2026-10-02",
  title: [Changed the sampling strategy],
  evidence: [Pilot run 12; commit `a1b2c3`], // optional
  milestone: "Prototype validated", // optional plain text
  workitem: "Sampling investigation", // optional plain text
)[
  Project notes in ordinary Typst markup.
]
```

Chronological direction is settled: oldest first.
Proposed date representation: validate ISO calendar dates in `YYYY-MM-DD` format.
Same-date tie handling: retain expanded source encounter order, not alphabetical filenames or titles.
The date representation remains a proposed default for final specification review.
Only date and title are required; omitted optional fields leave no empty field rows.

### Permanent date filling on save — behavior settled, ⏳ editor adapter

Implement this as an editor-assisted authoring convenience, separate from the rendering macro and final-build validation.
The target editor is not yet known; do not silently select an editor, install an extension, or promise universal save-hook support.

Required behavior:

- Inspect only the `.typ` document being saved inside the project-root `timeline/` boundary, including nested directories. Do not rewrite unrelated files or scan all project files for date replacements.
- Identify actual event calls syntactically; fill only their empty literal `date: ""` arguments. Leave comments, code examples, other macros, body text, and other date fields unchanged.
- Read the current calendar date in the author's local timezone, format it as `YYYY-MM-DD`, and write that literal into the saved source. Compute it once for a save operation so multiple blank events receive the same date.
- Leave every nonempty date unchanged, even if it is historical or invalid. Invalid nonempty dates remain validation errors, not permission to silently correct them.
- Do not infer dates from filenames, Git history, title text, or missing date arguments. Missing arguments remain actionable authoring/validation errors.
- Keep title, rich body, evidence, milestone, workitem, and surrounding formatting untouched. Allow ordinary editor undo, and integrate with the current document buffer instead of rewriting a file behind unsaved changes.
- Make repeated saves idempotent: after the date is stamped, future saves, recompiles, and GitHub builds must not change it.
- Use safe editor save integration where supported and document whether manual saves, autosave, and remote editing are supported. If parsing is ambiguous or a save hook cannot safely run, do not guess or apply a partial replacement; keep validation able to report the remaining empty date.

Implementation recommendation: share syntax-aware date-edit calculation with the local helper's parsing layer, and use a thin editor adapter to apply the edits before saving. The parsing/edit operation itself should not require a PDF build or network access.
The exact adapter, installation steps, and dependencies depend on the selected editor and must be disclosed before implementation. A VS Code pre-save adapter is a researched candidate, not a selected dependency or a guarantee for other editors.

The saved date records the save day when no date was supplied, not proof of the actual activity day. Authors can set a past date explicitly before saving.
Editors without the adapter can still use the template by entering dates manually; the event macro must not depend on the editor integration.
Neither the checked build helper nor ordinary PDF compilation should silently stamp dates or mutate authored sources. No compilation-time current-date fallback is permitted for event records.
The existing GitHub workflow stays unchanged and never backfills event dates.

### Source organization and loading — explicit list settled

Timeline files are separate from ordinary chapters and must not be included as ordinary content through `include-chapters` by accident.
The author can choose file boundaries independently of PDF chronology.

Use an explicit ordered list of timeline source files, declared once in project-owned configuration and passed to a dedicated timeline loader.
This makes same-date ties deterministic and avoids scanning undeclared files or introducing generated manifests initially.
Registration syntax and list location are proposed below for final review, not another unsettled product requirement.
Recommended initial contract: list every event-bearing source file directly, including files in nested directories under `timeline/`; do not nest source `include` calls between event files. This keeps registration, tie order, and duplicate detection explicit. Ordinary asset references remain allowed and are not event source registrations.

Automatic discovery is out of scope for the first version, following the user's choice.
Unlisted files are not collected merely because they exist inside `timeline/`.
Proposed list name and shape, for final review: `timeline-files` is an ordered tuple of project-root-relative source paths, with shared definitions in `main.typ` or one project-owned config. Both output modes reuse it unchanged.

### Directory restriction and validation

The authoring rule is settled; its precise enforcement mechanism requires technical validation during implementation planning.
Typst's documented location API describes PDF positions and does not expose a source filename. A PDF location must not be treated as proof that an event call originated in `timeline/`.

Recommended layers:

- Validate registered source paths against the project-root `timeline/` boundary, including traversal and duplicate entries.
- Reject event output outside the approved timeline-loading context where feasible.
- Use source-aware validation in the accepted local build helper to detect event authoring outside `timeline/` and disallowed includes from timeline files. Integration into the existing PDF workflow is deferred at the user's request.
- Specify supported source forms, including aliases and dynamically constructed includes. Do not promise that a simple text search or path-prefix check proves arbitrary Typst code obeys the restriction.

This is a repository-authoring constraint, not a security sandbox against deliberately hostile Typst code.
Unsupported authoring forms must fail clearly rather than silently weakening the rule.
Any added dependency or required build command must be disclosed before implementation approval.

The user accepts a build helper. Proposed local interface: one checked-build entry point runs source validation before the Typst compiler. Direct editor/compiler previews remain useful but must not be described as full source-placement validation when they bypass the helper.
The unchanged GitHub workflow also bypasses the helper for now; only compiler-enforced checks run there. Do not modify other entry points just to conceal this distinction.
Technical gate before implementation: select and validate the implementation language/parser, disclose installation and maintenance footprint, and prove the supported source checks. Typst's own syntax library is a candidate, not an already selected or installed dependency. If feasibility needs a spike, obtain separate authorization; this design session does not authorize a prototype.

### Collection

- Collect records only from approved, registered timeline sources, preserving date, title, full body, optional fields, and source encounter order.
- Metadata is a possible collection mechanism, not a settled requirement now that the sources have a dedicated boundary.
- If metadata is used, its namespace must remain separate from TODO metadata.
- It does not scan undeclared files, search raw source text, or infer events from TODOs or Git commits.
- Keep collection and rendering separate so a future GitHub adapter can enrich records without changing chronological ordering or forcing new mandatory authoring fields.
- No author-facing `id` is exposed in the initial interface. Any internal identity belongs to the implementation and must not become author-maintained configuration.
- Internal records should identify their type and schema version. This is an implementation boundary, not project configuration the author maintains.
- The optional `milestone` and `workitem` values are plain-text strings displayed as supplied. Evidence remains Typst content capable of containing explicit links.
- Capture source encounter order at original registration only. Rendering must not advance that sequence. Sort by calendar date and then the sequence defined by the explicit file list and within-file positions.
- They do not control ordering, trigger network requests, or resolve against a local TODO system.
- GitHub-specific IDs, tokens, APIs, synchronization direction, and milestone semantics remain undefined until that integration is discussed.

### Output configuration — settled modes, ⏳ interface defaults

Required outputs:

- Notes plus timeline: selected ordinary notes chapters first, followed by one dedicated chronological timeline chapter.
- Timeline only: that same chronological timeline without ordinary notes chapters.

Recommended third value: notes only, as the backward-compatible default when no timeline configuration is supplied. This adds no new feature to existing notes behavior.
Working interface proposal: a document-level mode with values `notes`, `notes-with-timeline`, and `timeline`; names are provisional, not implemented API.

Keep output selection logically separate from the print/layout profile. A landscape layout must not imply notes-only content, and a portrait layout must not imply a timeline.
Project configs can compose the output choice and print profile, allowing useful named presets without duplicating chapter lists or event sources.
Recommended placement: shared source declarations remain project-owned and defined once; configuration chooses the output and layout, not a second copy of the records.
Recommended convenience: a temporary build input can override the saved mode for preview or export. Precedence would be explicit build override, then project config, then backward-compatible notes-only default.
Do not require separate independent document entry points merely to obtain the two PDFs.

Proposed layout default: use the selected print profile in either mode. A timeline-specific layout override is a possible later option, not a requirement established by the current request.
Front matter remains governed by the selected config, but references and summaries must describe only content actually included in that output. Timeline-only must not evaluate ordinary notes just to hide their pages.
Unknown modes should fail clearly rather than falling back silently.

### Timeline rendering — rich-body support settled

The dedicated loader/renderer handles the approved event sources without registering additional events when projecting records.

The timeline displays the full date, title, body, and each supplied optional field.
Compact timeline summaries are not the selected first-version behavior.

Proposed interpretation of the selected outputs: treat `timeline/` files as record sources, not separately printed source chapters. Collect events and render each full note exactly once as a box in either the appended timeline chapter or timeline-only output.
File grouping is an authoring convenience, not another PDF hierarchy. This interpretation must be included in the final spec review rather than silently reviving the earlier doubled-note model.

Recommended box presentation: a date/title header, full body, and supplied optional fields, using existing profile-aware callout styling. Long boxes must support page and column breaks.

Single rendering removes the duplication-specific obstacle to figures, footnotes, and labels, but ordinary Typst content validation and date-reordering effects still need testing.
Rich event bodies also require avoiding self-referential collection: a nested `#event` or timeline renderer inside an event body must not create an expanding query result.
Settled body policy: support ordinary rich Typst notes, including lists, math, code, tables, images, citations, numbered figures, footnotes, and labels; disallow nested event/timeline collection. Any technical limitation must be disclosed rather than silently omitting content.
Headings inside a note remain visually subordinate and local, without globally numbered chapters or global outline entries.
Keep bodies as Typst content; do not serialize/reparse them as strings. Asset references must remain valid relative to the authoring source file even when the event is rendered in chronological order.
Numbered figures, footnotes, and other layout-dependent numbering follow the rendered chronology; their numbers may differ between output modes as ordinary notes are added or omitted. Labels rather than handwritten numbers should be used for internal references.
Timeline-only cannot preserve internal links to ordinary notes chapters that it does not include. Such unresolved references must fail clearly, not silently include hidden notes, drop the link, or omit part of the event. References within the timeline and external evidence links remain supported.

Layout recommendation: a vertical chronology with profile-dependent spacing and date placement; long content must remain readable across page and column breaks.
Detailed layout choices follow after the content projection is settled.

### Origin navigation

The earlier destination was the original chapter heading in the PDF, not the actual source file position.
Under the proposed single-render model, there is no separately printed source chapter; recommendation: omit redundant backlinks in both output modes.
If the user later requests printed source chapters as a third timeline presentation, revisit navigation and duplicated-body behavior explicitly rather than adding those copies by default.
Do not promise actual `.typ` file/line navigation or introduce editor-specific links.

### Focused previews — ⏳ recommended source completeness

Previously, focused builds could omit events scattered across excluded chapters; documenting that limitation was accepted.
With dedicated sources, recommendation: load all registered timeline files whenever the timeline chapter is selected, even if ordinary chapters are excluded.
In timeline-only mode, ordinary chapter selection must not filter timeline sources or accidentally include notes chapters.
In notes-plus-timeline mode, the existing chapter filter applies to ordinary notes; the appended timeline still uses all registered event sources.
Recommended interaction: explicitly requesting an ordinary chapter-only focused preview suppresses the appended timeline unless it is also requested. Final builds without the preview filter honor the saved output mode.
This is not yet a settled replacement for the earlier preview rule.
No serialized external event index or hidden full-document layout is proposed for the initial version.

### Automated PDFs — unchanged workflow

The existing workflow produces one PDF from the saved project configuration on every push and never commits generated output.
The user explicitly requests keeping it unchanged for now. Do not add alternate export controls, a build matrix, extra artifacts, or helper invocation to `.github/workflows/build-pdf.yml`.
The workflow continues compiling `main.typ` directly and uploading its existing PDF artifact. Future `main.typ` configuration naturally determines that PDF's content, but there is no new workflow-level mode selection.
PDFs and construction output remain ignored and uncommitted; existing read-only permissions and artifact retention remain unchanged.
Local verification can exercise all modes and both profiles without editing the workflow.
CI integration of strict source-placement validation and automatic multi-PDF publication require a later request.

## Draft acceptance criteria

- Event authoring is accepted inside `timeline/` and rejected outside it by the approved validation workflow, with an actionable source-file diagnostic.
- Paths escaping `timeline/`, duplicate registrations, and unsupported loading forms fail clearly rather than silently violating the restriction.
- Events from multiple registered timeline files form one date-sorted list.
- A valid calendar date and a nonempty title are required; invalid inputs produce an actionable error.
- Saving a supported timeline document changes an actual event's empty `date: ""` to the author's current local `YYYY-MM-DD` date and saves the literal in the source; all events stamped by one save use the same captured date.
- Existing dates, manual historical dates, omitted date arguments, comments, examples, non-event date fields, files outside `timeline/`, and all rich bodies/other fields remain unchanged by stamping.
- Repeated saves and later recompiles leave stamped dates unchanged; the edit is undoable and does not overwrite concurrent or unsaved buffer changes.
- Tests use a controlled clock/timezone, including local midnight and daylight-saving boundaries, and cover multiple events, malformed/incomplete source, repeated saves, and the supported manual-save/autosave behavior.
- With the editor adapter absent or bypassed, manual date authoring still works and compilation reports empty/invalid dates rather than mutating sources or substituting the build day.
- Evidence may be omitted without requiring a placeholder.
- Evidence, milestone, and workitem appear in full when supplied, and produce no empty placeholders when omitted.
- The timeline renders the complete event body with the agreed Typst content support.
- Rich-body tests cover images with source-relative asset paths, tables, citations, numbered figures, footnotes, labels, and local headings in both modes, including long boxed notes across page and column boundaries.
- Nested event/timeline collection is rejected; local headings do not add globally numbered chapters or outline entries.
- Each collected source event appears exactly once in the timeline.
- Rendering the timeline does not add new event metadata or collect unrelated TODO metadata. No document TODO is automatically converted into an event or work item.
- Each full event note appears boxed, rather than blending into ordinary chapter text, with one chronological copy in either output mode under the proposed single-render interpretation.
- Events are sorted oldest first; ties follow declared source-loading order and within-file order.
- No automatic file discovery occurs; an unlisted timeline file does not contribute events.
- Source sequence registration happens once per authored event and is unaffected by timeline placement or rendering copies.
- Milestone and workitem accept plain text without GitHub lookups or required URL/ID formatting.
- Authors do not have to supply IDs, kinds, or rationale fields.
- Focused timeline builds follow the agreed source-loading rule, documented with a working example.
- Existing documents without events retain their current behavior.
- Notes-plus-timeline output places the timeline after ordinary notes. Timeline-only output excludes ordinary notes entirely rather than merely hiding their layout.
- Switching output modes does not duplicate source declarations or alter record contents, ordering, and optional field visibility.
- Notes-only is the proposed backward-compatible default; unknown configured modes fail clearly.
- Output selection is independent of portrait/landscape print profiles. Test both required modes with both existing profiles.
- Outline entries, links, and front-matter summaries refer only to included content; timeline-only does not import omitted notes solely to populate their summaries.
- Both preset profiles remain usable, including page and column boundaries.
- Shared updates do not overwrite project-owned timeline files or alter existing tutorial text.
- The local checked build runs placement validation before producing a PDF; direct compiler builds, including the unchanged workflow, are documented as bypassing helper-only checks.
- `.github/workflows/build-pdf.yml` remains unchanged, including its single direct compilation, artifact, permissions, and retention. No PDFs or construction artifacts are added to tracked files.

## Final review and technical gates

The agreed behavior is recorded. Review the consolidated design and proposed defaults as a whole before marking the specification approved. Editor selection remains open for the new save integration; this does not authorize an implementation or installation in an assumed editor.

Proposed defaults for final review:

- ISO `YYYY-MM-DD` input and display, validated as real calendar dates.
- One full boxed chronological copy, with no separately printed source chapters or redundant backlinks.
- `notes`, `notes-with-timeline`, and `timeline` modes; notes-only fallback for existing projects.
- Saved output config with an optional local build override, independent of the selected print profile.
- Ordered `timeline-files` tuple declared once in `main.typ` or one project-owned config. Direct registration of all source files, including subdirectories, rather than nested event-file includes.
- Every registered source is collected when the timeline is requested; ordinary chapter-only previews suppress it unless explicitly requested. Preview syntax will be documented with the implementation.
- Existing profile-aware callout defaults, breakable long entries, and no global outline entries for individual event titles or local note headings.
- Missing/unreadable files, duplicate source registration, invalid modes/dates, and disallowed nesting fail clearly. An empty event set shows a short empty-state message in explicitly selected timeline output, but no timeline appears in notes-only mode.
- Local helper validation; the GitHub workflow stays unchanged and does not gain helper-only checks.
- Permanent date stamping on save for empty event date literals, using the author's local day and preserving all existing dates; builds never rewrite dates.

Technical gates, not decisions silently assumed complete:

1. Validate the helper/parser approach and disclose its exact dependencies and limits before implementation. The source boundary is an authoring rule, not a hostile-code security sandbox.
2. Verify rich-body collection without string flattening, duplicate numbering, source-relative asset breakage, or collection feedback. Do not narrow the agreed content support without reporting the limitation and obtaining direction.
3. Verify both required outputs under both profiles and document unresolved references to omitted notes in timeline-only builds.
4. Identify the target editor, verify its safe save-edit mechanism, and disclose adapter dependencies/setup. Verify timezone semantics in remote editing and supported autosave behavior; do not claim stamping on every save if the editor can bypass the hook.

After user review, the spec can be committed separately if requested. Specification approval is not automatic authorization to implement, publish, or change the workflow.

## Candidate implementation sequence

This plan is provisional and execution is not authorized during the current design discussion.

1. Small: obtain final design/defaults review; separately obtain implementation or spike authorization when the user wants to proceed.
2. Medium, depends on 1: validate directory enforcement, choose/disclose helper dependencies, and test traversal, duplicate paths, nested includes, aliases, and misuse outside the approved loader. Stop for direction if the accepted boundary cannot be enforced under the supported authoring contract.
3. Medium, depends on 2: implement independent event authoring and checked local loading, calendar-date validation/sorting, and source-order ties. Keep empty-date errors explicit and do not add a compilation-time date fallback.
4. Medium, depends on 3: implement configurable notes-plus-timeline and timeline-only composition independently of print profiles, plus full boxed notes and optional fields. Existing notes-only projects must still work without a timeline declaration.
5. Medium, depends on 2–3 and editor selection: implement a syntax-aware empty-date edit operation and the approved editor's save adapter. Preserve nonempty dates and document buffers, capture the author's local date once per save, provide undoable edits, and keep ordinary compiler usage independent of the adapter. No editor installation or prototype is authorized by this plan alone.
6. Medium, depends on 3–5: verify explicit multi-file sources, diagnostics, date ties, collection convergence, rich content and asset paths, both modes across both profiles, focused/checked builds, and page/column boundaries using temporary examples. Add controlled-clock date-stamping tests for timezone boundaries, repeated saves, comments/examples, historical dates, incomplete syntax, and supported autosave. Do not depend on event IDs or flatten content for testing; do not edit the existing workflow.
7. Small, depends on 6: document `timeline/`, registration, configuration/output examples, local validation versus direct compilation, previews, upstream migration, and editor date-stamping setup/limits/manual fallback. Revise guidance without replacing authored tutorial material or project-owned content.

## References

- [Typst modules and includes](https://typst.app/docs/reference/scripting/#modules)
- [Typst paths](https://typst.app/docs/reference/foundations/path/)
- [Typst metadata](https://typst.app/docs/reference/introspection/metadata/)
- [Typst content values](https://typst.app/docs/reference/foundations/content/)
- [Typst syntax library](https://docs.rs/typst-syntax/latest/typst_syntax/)
- [Typst document queries](https://typst.app/docs/reference/introspection/query/)
- [Typst document locations](https://typst.app/docs/reference/introspection/location/)
- [Typst links](https://typst.app/docs/reference/model/link/)
- [Typst array sorting](https://typst.app/docs/reference/foundations/array/#definitions-sorted)
- [Typst breakable blocks](https://typst.app/docs/reference/layout/block/)
- [Typst hide preserves layout](https://typst.app/docs/reference/layout/hide/)
- [VS Code pre-save editing API](https://code.visualstudio.com/api/references/vscode-api#TextDocumentWillSaveEvent)
- [Typst current-date function](https://typst.app/docs/reference/foundations/datetime/#definitions-today)

## Decision notes

- Chronology remains the first-version structure because milestone organization will be revisited with GitHub Projects.
- Optional evidence is retained because it helps connect notes to experiments, commits, and supporting sources. Optional milestone and workitem fields preserve project associations without controlling chronology.
- Events and document TODOs are completely separate. A workitem field does not imply a reference to a document TODO.
- Full timeline notes replace the previous compact recommendation. The proposed single-render model removes the separate source-chapter backlink destination; actual source-file navigation remains out of scope.
- Event IDs are deferred, following the user's preference to keep the initial interface small.
- Dedicated `timeline/` files replace flexible placement in ordinary chapters to provide a structured authoring boundary.
- Boxes retain the distinction between project records and ordinary prose in either required output mode.
- Configurable notes-plus-timeline and timeline-only outputs let projects reuse the same records for different purposes. The proposed design keeps content selection separate from layout, with one chronological rendering and backward-compatible notes-only behavior.
- Plain-text milestone/workitem fields keep authoring lightweight until GitHub integration is designed.
- Source order resolves same-date ties without adding a required time or an author-maintained sequence field.
- An explicit ordered source list is selected over automatic discovery, giving both output modes the same deterministic collection scope.
- Rich Typst content is required for future-proof authoring; preserve content values and validate figures, citations, footnotes, labels, and assets rather than introducing a plain-text record format.
- A build helper is acceptable. Its full validation is local for now because the existing GitHub workflow must remain unchanged.
- Workflow-level alternate exports, helper integration, and multi-PDF publication are deferred, not silently included in this implementation plan.
- Empty event date literals will be permanently filled on save; existing dates remain untouched. Editor integration is a separate authoring adapter, not a compilation-time date default or a GitHub workflow change.
