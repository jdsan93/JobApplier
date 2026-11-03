# Application autofill — Tasks

**Status:** Approved

Implementation tasks for an approved design. This file is also the record of the work. When a task is completed, mark it Done, write what was done in Outcome, and record any decision made while doing it in the Decision log.

Each task is one red→green→verify TDD cycle. Mark a task optional only when the feature still meets its requirements without it. Leave Decision log and Outcome empty until the task is done.

## Purpose

Break `design.md` into ordered, traceable work that an implementer can execute without inventing scope.

## How to use

1. Pick the first task that is not Done and whose dependencies are Done.
2. Write the failing test named in the TDD plan, then the smallest implementation that passes it, then run the verify step.
3. Mark the task Done. Fill Outcome and Decision log (leave Decision log empty if no new choice was required).
4. Do not start the next task until this one is recorded.

## Status legend

- `[ ]` Pending — not started
- `[x]` Done — implemented; Outcome and Decision log filled during execution
- `(optional)` — the feature still meets requirements if this task is skipped

## Task overview

Repo state at drafting: `package.json` exists with only a `typecheck` script (`tsc --noEmit`) and the `@types/chrome`, `@types/node`, and `typescript` devDependencies; `src/env.d.ts` is a placeholder; there is no product source yet. The verify command is `npm run check` as fixed by `design.md` (Assumptions); T1b adds it. See Open items.

| ID | Title | Traces to | Depends on | Status |
| --- | --- | --- | --- | --- |
| T1a | Scaffold the extension manifest, placeholder surfaces, and test setup | WHEN the profile is saved THE SYSTEM SHALL keep the stored profile so that other extensions cannot read it (manifest guard only; browser check in T42a) | — | Pending |
| T1b | Add the `build` and `check` scripts | none (build wiring; browser loading is verified in T42a and T42b) | T1a | Pending |
| T2 | Static scan for remote calls in `src/` and `dist/` | WHEN the extension runs | T1b | Pending |
| T3 | Profile store round-trip in `chrome.storage.local` | WHEN the user saves the profile THE SYSTEM SHALL save and later reuse it without requiring a passphrase; WHEN the user starts a new session in the same browser after saving a profile THE SYSTEM SHALL still have those saved facts available for filling | T1b | Pending |
| T4 | Auto-mode state in session storage | WHEN the browser starts a new session; WHEN auto-mode is on | T1b | Pending |
| T5 | Options page renders built-in fields and saved values | WHEN the user opens the extension’s options page | T3 | Pending |
| T6a | Options page saves built-in fields as pieces | WHEN the user opens the extension’s options page (save); WHEN the user saves name, phone, address, or date of birth; WHEN the user saves a profile fact | T5 | Pending |
| T6b | Blank middle name omitted; second save replaces | WHEN the user opens the extension’s options page THE SYSTEM SHALL let the user view, enter, and save these built-in profile facts: first name; middle name (optional); last name; date of birth; email; phone country code; phone national number; street numbers; address details for apartment number or similar; city; state/region; postal code; country; neighbourhood / localidad; LinkedIn URL; GitHub URL; portfolio or personal-site URL; US work authorization; whether US sponsorship is needed; gender; race/ethnicity; veteran status; disability status; a decline-to-answer option for each of those EEO facts, typed as the page shows it; one canned “how did you hear about us” value; and one resume file; WHEN the user changes a stored fact on the options page and saves THE SYSTEM SHALL use the updated fact on fills after that save (the save part; the fill part is T33a) | T6a | Pending |
| T7a | Resume store | WHEN the user starts a new session in the same browser after saving a profile | T1b | Pending |
| T7b | Options page resume file picker | WHEN the user opens the extension’s options page (one resume file) | T7a, T5 | Pending |
| T8 | Options add-fact menu | WHEN the user opens the extension’s options page THE SYSTEM SHALL present a basic, non-stylized menu | T5 | Pending |
| T9 | Save user-added facts with their questions | WHEN the user adds a fact and the questions it maps to and saves | T3, T6b, T8 | Pending |
| T10a | Field snapshot: label sources | WHEN a fill runs THE SYSTEM SHALL identify fields from the page’s labels, accessible names, placeholders, and similar cues | T1b | Pending |
| T10b | Field snapshot: emptiness and cue normalization | WHEN a fill runs THE SYSTEM SHALL identify fields from the page’s labels, accessible names, placeholders, and similar cues | T10a | Pending |
| T11 | Job-application classifier | WHEN the user clicks the extension icon and the current tab is not a job application; WHEN auto-mode is on and the current tab is a login, checkout, or settings form that is not a job application; WHEN a settings form has at least two personal-data fields and at least one job-application-specific field, and no password or payment-card fields | T10b | Pending |
| T12 | Write a stored value into an empty text field as user input | WHEN a fill runs and a field has no value and maps cleanly to a stored profile fact; WHEN a fill runs and a field already has a value | T10b | Pending |
| T13 | Built-in text facts: email and profile links | WHEN a fill runs and a field has no value and maps cleanly to a stored profile fact | T3, T12 | Pending |
| T14a | Name fill for one name field (composition, pieces, and exclusion of preferred/legal/passport) | WHEN a fill runs and the page presents exactly one field asking for the applicant’s first, middle, last, or full name; WHEN a fill runs and the page presents preferred-name, legal-name, or name-as-on-passport fields; WHEN a fill runs and the page asks for a name variant that is not stored as its own fact (preferred name, legal name, or name as on passport) | T3, T10b, T12 | Pending |
| T15 | Name fill for two name fields | WHEN a fill runs and the page presents exactly two fields asking for the applicant’s first, middle, last, or full name | T14a | Pending |
| T16 | Name fill for three name fields | WHEN a fill runs and the page presents exactly three fields asking for the applicant’s first, middle, last, or full name; WHEN a fill runs and middle name is not stored | T15 | Pending |
| T17 | Name fill for four or more name fields by exact label | WHEN a fill runs and the page presents four or more fields asking for the applicant’s first, middle, last, or full name | T16 | Pending |
| T18 | Phone compose and split | WHEN a fill runs and the page has a single phone field that has no value and no separate country-code field; WHEN a fill runs and the page has a country-code field and a phone field that has no value; WHEN a fill runs and the page has a country-code field and/or a national-number field | T3, T12 | Pending |
| T19 | Street and address-line pieces | WHEN a fill runs and the page has a single street or address-line field that has no value; WHEN a fill runs and the page has a street-numbers field and/or an address-details field | T3, T12 | Pending |
| T20a | Full address compose | WHEN a fill runs and the page has a single full-address field (not a street-only or address-line field) that has no value and that maps cleanly to stored street numbers, address details, city, state/region, or postal code | T19 | Pending |
| T20b | City, state/region, postal code, and country fields | WHEN a fill runs and the page has a city, state/region, postal code, and/or country field | T3, T12, T19 | Pending |
| T21 | Neighbourhood / localidad field | WHEN a fill runs and the page has a neighbourhood or localidad field that has no value and neighbourhood / localidad is stored; WHEN a fill runs and neighbourhood / localidad is not stored | T19 | Pending |
| T22 | Date of birth in one field | WHEN a fill runs and the page has a single date-of-birth field that has no value and date of birth is stored; WHEN a fill runs and a date-of-birth field’s expected format cannot be determined from the page; WHEN a fill runs and the page asks for a date other than date of birth and no stored fact is mapped to that question | T3, T12 | Pending |
| T23 | Date of birth in separate month, day, and year fields | WHEN a fill runs and the page has a month, day, and/or year field for date of birth and date of birth is stored | T22 | Pending |
| T24 | Native select writer for stored choices | WHEN a fill runs and a choice field has no value and maps cleanly to stored US work authorization, sponsorship, an EEO fact, or the canned “how did you hear about us” value; WHEN setting a stored value requires using a native select or radio group | T10b | Pending |
| T25 | Radio group writer for stored choices | WHEN a fill runs and a choice field has no value and maps cleanly to stored US work authorization, sponsorship, an EEO fact, or the canned “how did you hear about us” value; WHEN setting a stored value requires using a native select or radio group | T10b | Pending |
| T26 | ARIA combobox writer for stored choices (best-effort) | WHEN setting a stored value requires using an ARIA dropdown (a combobox or listbox that is not a native select) | T10b | Pending |
| T27 | Date-of-birth month and day select branch | WHEN a fill runs and the page has a month, day, and/or year field for date of birth and date of birth is stored | T23, T24 | Pending |
| T28 | Choice facts mapping | WHEN a fill runs and a choice field has no value and maps cleanly to stored US work authorization, sponsorship, an EEO fact, or the canned “how did you hear about us” value; WHEN a fill runs and “how did you hear about us” is a free-text field and no stored fact is mapped to that question; WHEN a fill runs and “how did you hear about us” is a choice field that has no value and that maps cleanly to the stored canned value; WHEN a fill runs and the page asks about work authorization or sponsorship for a country other than the United States and no stored fact is mapped to that question | T3, T24, T25, T26 | Pending |
| T29 | User-added fact writes into an empty field matched by exact question | WHEN a fill runs and a field has no value and that field’s label, accessible name, or placeholder is a question the user recorded for a stored user-added fact | T9, T10b, T12, T24, T25, T26 | Pending |
| T30 | User-added fact is not used for non-matching cues; agree-to-terms never matched | WHEN a fill runs and a field’s label, accessible name, and placeholder are not the question the user recorded for a stored user-added fact; WHEN a fill runs and a field is an agree-to-terms checkbox or control, including one that a user-added fact would match | T29 | Pending |
| T31 | Ambiguity guard: dual or multiple matches stay unchanged | WHEN a fill runs and a field’s mapping to the profile is missing or ambiguous; WHEN a fill runs and a field maps to both a built-in fact and a user-added fact | T13, T30 | Pending |
| T32 | Fill pass gate | WHEN the user clicks the extension icon and the current tab is not a job application; WHEN auto-mode is off and the user has not clicked the extension icon; WHEN auto-mode is off, the user has not clicked the extension icon, and the current tab is a job application; WHEN a fill runs and no profile facts are stored; WHEN auto-mode is on and the current tab is not a job application | T3, T4, T11 | Pending |
| T33a | Fill pass wiring: all mappers into the fill pass | WHEN the user clicks the extension icon and the current tab is a job application; WHEN auto-mode is on and the current tab is a job application; WHEN a fill runs on a job application hosted by a job board the user did not name or configure; WHEN a fill runs and a field has no value and maps cleanly to a stored profile fact; WHEN a fill runs and a field already has a value; WHEN the user has not added any extra facts; WHEN the user changes a stored fact on the options page and saves; WHEN a later fill runs from an icon click or from auto-mode and some fields already have values; WHEN a fill runs and filling a field would require generating, paraphrasing, or otherwise inventing text that is not already stored; WHEN a fill runs and a field is an open-ended prompt for original writing and no stored fact is mapped to that question; WHEN a fill runs and only some profile facts are stored; WHEN a fill runs THE SYSTEM SHALL not activate Submit, Next, or Continue when doing so would advance or complete the application; WHEN a fill runs and a field is an agree-to-terms checkbox or control, including one that a user-added fact would match | T11, T13, T17, T18, T20a, T20b, T21, T23, T27, T28, T29, T31, T32 | Pending |
| T33b | Fill pass uses user-added facts saved earlier | WHEN the user adds a fact and the questions it maps to and saves THE SYSTEM SHALL store that fact and those mappings with the profile and use them on fills after that save | T9, T33a | Pending |
| T36a | Debounced auto-mode triggers for later steps (DOM mutation path) | WHEN auto-mode is on and the user advances to another step of a job application; WHEN auto-mode is on and the current tab is a job application | T4, T33b | Pending |
| T36b | Debounced auto-mode triggers for later steps (URL change path) | WHEN auto-mode is on and the user advances to another step of a job application; WHEN auto-mode is on and the current tab is a job application | T36a | Pending |
| T37a | Service worker relays FillTab | none (wiring for T37b; no acceptance criterion of its own) | T4 | Pending |
| T37b | Content script runs the icon-trigger fill for job applications only | WHEN the user clicks the extension icon and the current tab is a job application; WHEN the user clicks the extension icon and the current tab is not a job application | T32, T33b, T37a | Pending |
| T38 | Popup open turns auto-mode on and requests a fill | WHEN the user clicks the extension icon THE SYSTEM SHALL turn auto-mode on | T37a, T37b | Pending |
| T39 | Popup on/off control without fill | WHEN the user opens the extension popup; WHEN the user turns auto-mode off from the popup | T38 | Pending |
| T40 | Attach the chosen resume to an empty resume/CV upload | WHEN a fill runs, the user has chosen a resume file, that file is available, and the page has a resume or CV upload field that has no file; WHEN a later fill runs from an icon click or from auto-mode and some fields already have values | T7a, T10b, T33b | Pending |
| T41 | Resume attach negatives | WHEN a fill runs and the page has no resume or CV upload field; WHEN a fill runs and the page has a file upload that is not a resume or CV upload; WHEN a fill runs and a resume or CV upload field already has a file; WHEN a fill runs, the page has a resume or CV upload field, and the user has not chosen a resume file; WHEN a fill runs, the page has a resume or CV upload field, the user has chosen a resume file, and that file is not available | T40 | Pending |
| T42a | Manual browser checklist in Chrome | WHEN the extension runs; WHEN the user loads the extension from a local folder into Chrome; WHEN the profile is saved THE SYSTEM SHALL keep the stored profile on the user’s machine so that web pages cannot read it; WHEN the profile is saved THE SYSTEM SHALL keep the stored profile so that other extensions cannot read it | T2, T9, T33b, T36a, T36b, T39, T41 | Pending |
| T42b | Manual browser checklist in Opera GX | WHEN the user saves a profile in Chrome; WHEN the user saves a profile in Opera GX; WHEN the user loads the extension from a local folder into Opera GX | T42a | Pending |

## Requirements coverage

Every acceptance criterion from `requirements.md` appears exactly once in the left column.

| Criterion | Task(s) |
| --- | --- |
| WHEN the user opens the extension’s options page THE SYSTEM SHALL let the user view, enter, and save these built-in profile facts: first name; middle name (optional); last name; date of birth; email; phone country code; phone national number; street numbers; address details for apartment number or similar; city; state/region; postal code; country; neighbourhood / localidad; LinkedIn URL; GitHub URL; portfolio or personal-site URL; US work authorization; whether US sponsorship is needed; gender; race/ethnicity; veteran status; disability status; a decline-to-answer option for each of those EEO facts, typed as the page shows it; one canned “how did you hear about us” value; and one resume file | T5, T6a, T6b, T7b |
| WHEN the user saves name, phone, address, or date of birth THE SYSTEM SHALL store that fact as the pieces listed above so a later fill can compose or split it to match the page | T6a |
| WHEN the user saves a profile fact THE SYSTEM SHALL not store a second copy of that fact that can disagree with the saved pieces | T6a |
| WHEN the user changes a stored fact on the options page and saves THE SYSTEM SHALL use the updated fact on fills after that save | T6b, T33a |
| WHEN the user starts a new session in the same browser after saving a profile THE SYSTEM SHALL still have those saved facts available for filling | T3, T7a |
| WHEN the user saves the profile THE SYSTEM SHALL save and later reuse it without requiring a passphrase | T3 |
| WHEN the user saves a profile in Chrome THE SYSTEM SHALL keep that profile only in Chrome and shall not make it available in Opera GX | T42b |
| WHEN the user saves a profile in Opera GX THE SYSTEM SHALL keep that profile only in Opera GX and shall not make it available in Chrome | T42b |
| WHEN the user opens the extension’s options page THE SYSTEM SHALL present a basic, non-stylized menu to add a fact and the question or questions that fact maps to | T8 |
| WHEN the user adds a fact and the questions it maps to and saves THE SYSTEM SHALL store that fact and those mappings with the profile and use them on fills after that save | T9, T33b |
| WHEN a fill runs and a field has no value and that field’s label, accessible name, or placeholder is a question the user recorded for a stored user-added fact THE SYSTEM SHALL write that stored value into the field as if the user had typed or selected it | T29 |
| WHEN a fill runs and a field’s label, accessible name, and placeholder are not the question the user recorded for a stored user-added fact THE SYSTEM SHALL not fill that field from that user-added fact | T30 |
| WHEN the user has not added any extra facts THE SYSTEM SHALL still fill from the built-in facts | T33a |
| WHEN the user clicks the extension icon THE SYSTEM SHALL turn auto-mode on | T38 |
| WHEN the user clicks the extension icon and the current tab is a job application (a page with at least two personal-data fields and at least one job-application-specific field, and no password or payment-card fields) THE SYSTEM SHALL fill that tab’s fields that have no value and that map cleanly to stored profile facts | T33a, T37b |
| WHEN the user clicks the extension icon and the current tab is not a job application THE SYSTEM SHALL not fill any fields on that tab | T11, T32, T37b |
| WHEN auto-mode is on THE SYSTEM SHALL keep auto-mode on until the user turns it off from the popup or the browser starts a new session | T4 |
| WHEN the browser starts a new session THE SYSTEM SHALL set auto-mode off | T4 |
| WHEN auto-mode is off and the user has not clicked the extension icon THE SYSTEM SHALL not fill any fields | T32 |
| WHEN the user opens the extension popup THE SYSTEM SHALL present a control that turns auto-mode on or off | T39 |
| WHEN the user turns auto-mode off from the popup THE SYSTEM SHALL leave auto-mode off and not fill fields until auto-mode is turned on again | T39 |
| WHEN auto-mode is on and the current tab is a job application (a page with at least two personal-data fields and at least one job-application-specific field, and no password or payment-card fields) THE SYSTEM SHALL fill fields that have no value and that map cleanly to stored profile facts without requiring a further click | T33a, T36a |
| WHEN auto-mode is on and the current tab is not a job application THE SYSTEM SHALL not fill any fields | T32 |
| WHEN auto-mode is on and the user advances to another step of a job application THE SYSTEM SHALL fill fields on that step that have no value and that map cleanly to stored profile facts without requiring another click | T36a, T36b |
| WHEN a fill runs THE SYSTEM SHALL identify fields from the page’s labels, accessible names, placeholders, and similar cues | T10a, T10b |
| WHEN a fill runs on a job application hosted by a job board the user did not name or configure THE SYSTEM SHALL still fill fields that have no value and that map cleanly to stored facts | T33a |
| WHEN a fill runs and a field has no value and maps cleanly to a stored profile fact THE SYSTEM SHALL write that stored value into the field as if the user had typed or selected it, so the page treats the field as filled | T12, T13, T33a |
| WHEN a fill runs and a choice field has no value and maps cleanly to stored US work authorization, sponsorship, an EEO fact, or the canned “how did you hear about us” value THE SYSTEM SHALL set that choice as if the user had selected it | T24, T25, T28 |
| WHEN a fill runs and the page presents exactly one field asking for the applicant’s first, middle, last, or full name THE SYSTEM SHALL fill that field if it has no value: with the composed stored name, including middle name only if it is stored, when the field asks for the full name, or with the matching stored piece when the field asks for first, middle, or last name | T14a |
| WHEN a fill runs and the page presents exactly two fields asking for the applicant’s first, middle, last, or full name THE SYSTEM SHALL fill each of those fields that has no value and whose own label, accessible name, or placeholder says first name or last name with that stored piece; a field whose cue says neither stays unchanged | T15 |
| WHEN a fill runs and the page presents exactly two fields asking for the applicant’s first, middle, last, or full name THE SYSTEM SHALL not write a composed full name into either field | T15 |
| WHEN a fill runs and the page presents exactly three fields asking for the applicant’s first, middle, last, or full name THE SYSTEM SHALL fill each of those fields that has no value and whose own cue says first name, middle name, or last name with that stored piece; a field whose cue does not say which piece it is stays unchanged | T16 |
| WHEN a fill runs and the page presents four or more fields asking for the applicant’s first, middle, last, or full name THE SYSTEM SHALL not use the one-, two-, or three-field name fills | T17 |
| WHEN a fill runs, the page presents four or more fields asking for the applicant’s first, middle, last, or full name, and a field’s label, after trimming and case-folding, is exactly `first name` or `primer nombre` THE SYSTEM SHALL fill that field from the stored first name if that field has no value | T17 |
| WHEN a fill runs, the page presents four or more fields asking for the applicant’s first, middle, last, or full name, and a field’s label is exactly `last name` or `apellido` THE SYSTEM SHALL fill that field from the stored last name if that field has no value | T17 |
| WHEN a fill runs, the page presents four or more fields asking for the applicant’s first, middle, last, or full name, and a field’s label is exactly `middle name` or `segundo nombre` THE SYSTEM SHALL fill that field from the stored middle name if that field has no value | T17 |
| WHEN a fill runs, the page presents four or more fields asking for the applicant’s first, middle, last, or full name, and a field’s label, after trimming and case-folding, is not exactly `first name`, `primer nombre`, `last name`, `apellido`, `middle name`, or `segundo nombre` THE SYSTEM SHALL not fill that field from the stored first, middle, or last name | T17 |
| WHEN a fill runs and the page has a single phone field that has no value and no separate country-code field THE SYSTEM SHALL leave that field unchanged, and SHALL NOT join the stored country code into it | T18 |
| WHEN a fill runs and the page has a country-code field and a phone field that has no value THE SYSTEM SHALL fill the phone field with the stored national number | T18 |
| WHEN a fill runs and the page has a country-code field and/or a national-number field THE SYSTEM SHALL fill each of those fields that has no value from the matching stored piece | T18 |
| WHEN a fill runs and the page has a single street or address-line field that has no value THE SYSTEM SHALL fill that field by composing the stored street-numbers piece and the stored address-details piece that exist | T19 |
| WHEN a fill runs and the page has a street-numbers field and/or an address-details field (apartment number or similar) THE SYSTEM SHALL fill each of those fields that has no value from the matching stored piece | T19 |
| WHEN a fill runs and the page has a neighbourhood or localidad field that has no value and neighbourhood / localidad is stored THE SYSTEM SHALL fill that field from the stored neighbourhood / localidad | T21 |
| WHEN a fill runs and the page has a single full-address field (not a street-only or address-line field) that has no value and that maps cleanly to stored street numbers, address details, city, state/region, or postal code THE SYSTEM SHALL fill that field by composing those stored pieces, writing only the stored parts when some are missing | T20a |
| WHEN a fill runs and the page has a city, state/region, postal code, and/or country field THE SYSTEM SHALL fill each of those fields that has no value from the matching stored piece | T20b |
| WHEN a fill runs and the page has a single date-of-birth field that has no value and date of birth is stored THE SYSTEM SHALL fill that field from the stored date of birth, written in the date format that field asks for (month and day zero-padded, and a two-digit year where the field asks for one) | T22 |
| WHEN a fill runs and the page has a month, day, and/or year field for date of birth and date of birth is stored THE SYSTEM SHALL fill each of those fields that has no value from the stored date of birth, with a zero-padded month and day and a two-digit year where that field asks for one | T23, T27 |
| WHEN a fill runs and a date-of-birth field’s expected format cannot be determined from the page THE SYSTEM SHALL leave that field unchanged | T22 |
| WHEN a fill runs, the user has chosen a resume file, that file is available, and the page has a resume or CV upload field that has no file THE SYSTEM SHALL attach that file to the upload field | T40 |
| WHEN a fill runs and the page has no resume or CV upload field THE SYSTEM SHALL not attach a file | T41 |
| WHEN a fill runs and the page has a file upload that is not a resume or CV upload THE SYSTEM SHALL not attach the resume file to that upload | T41 |
| WHEN a fill runs and a resume or CV upload field already has a file THE SYSTEM SHALL leave that upload field unchanged | T41 |
| WHEN a fill runs and a field already has a value THE SYSTEM SHALL leave that field unchanged | T12, T33a |
| WHEN a later fill runs from an icon click or from auto-mode and some fields already have values THE SYSTEM SHALL fill remaining fields that have no value and that map cleanly to stored profile facts, including attaching the chosen resume file to a resume or CV upload that still has no file | T33a, T40 |
| WHEN a fill runs and a field’s mapping to the profile is missing or ambiguous THE SYSTEM SHALL leave that field unchanged | T31 |
| WHEN a fill runs and filling a field would require generating, paraphrasing, or otherwise inventing text that is not already stored THE SYSTEM SHALL leave that field unchanged | T33a |
| WHEN a fill runs and a field is an open-ended prompt for original writing (cover letter, why the applicant wants the job, tell-us-about-a-time, or similar) and no stored fact is mapped to that question THE SYSTEM SHALL leave that field unchanged | T33a |
| WHEN a fill runs THE SYSTEM SHALL not activate Submit, Next, or Continue when doing so would advance or complete the application | T33a |
| WHEN a fill runs and a field is an agree-to-terms checkbox or control, including one that a user-added fact would match THE SYSTEM SHALL not fill it and not activate it | T30, T33a |
| WHEN setting a stored value requires using a native select or radio group THE SYSTEM SHALL set the stored value through that widget without advancing or completing the application | T24, T25 |
| WHEN setting a stored value requires using an ARIA dropdown (a combobox or listbox that is not a native select) THE SYSTEM SHALL attempt to set the stored value by exact option match without advancing or completing the application, and in version 1 some sites will leave that field unchanged | T26 |
| WHEN the profile is saved THE SYSTEM SHALL keep the stored profile on the user’s machine so that web pages cannot read it | T42a |
| WHEN the profile is saved THE SYSTEM SHALL keep the stored profile so that other extensions cannot read it | T1a, T42a |
| WHEN the extension runs THE SYSTEM SHALL not send profile contents off the user’s machine, including through analytics, crash reports, or other remote calls that include profile contents | T2, T42a |
| WHEN the user loads the extension from a local folder into Chrome THE SYSTEM SHALL make the options page, the auto-mode control, and filling available in Chrome | T42a |
| WHEN the user loads the extension from a local folder into Opera GX THE SYSTEM SHALL make the options page, the auto-mode control, and filling available in Opera GX | T42b |
| WHEN a fill runs and no profile facts are stored THE SYSTEM SHALL leave every field unchanged | T32 |
| WHEN a fill runs and only some profile facts are stored THE SYSTEM SHALL fill fields that have no value and that map cleanly to stored facts and leave the rest unchanged | T33a |
| WHEN a fill runs and middle name is not stored THE SYSTEM SHALL leave middle-name fields unchanged | T16 |
| WHEN a fill runs and the page presents preferred-name, legal-name, or name-as-on-passport fields THE SYSTEM SHALL not count those fields toward the one-, two-, three-, or four-or-more name fills | T14a |
| WHEN a fill runs and the page asks for a name variant that is not stored as its own fact (preferred name, legal name, or name as on passport) THE SYSTEM SHALL leave that field unchanged | T14a |
| WHEN a fill runs and the page asks for a date other than date of birth and no stored fact is mapped to that question THE SYSTEM SHALL leave that field unchanged | T22 |
| WHEN a fill runs and “how did you hear about us” is a free-text field and no stored fact is mapped to that question THE SYSTEM SHALL leave that field unchanged | T28 |
| WHEN a fill runs and “how did you hear about us” is a choice field that has no value and that maps cleanly to the stored canned value THE SYSTEM SHALL select that stored value | T28 |
| WHEN a fill runs and the page asks about work authorization or sponsorship for a country other than the United States and no stored fact is mapped to that question THE SYSTEM SHALL leave those fields unchanged | T28 |
| WHEN a fill runs and street numbers are not stored THE SYSTEM SHALL leave street-numbers fields unchanged | T19 |
| WHEN a fill runs and address details (apartment number or similar) are not stored THE SYSTEM SHALL leave address-details fields unchanged | T19 |
| WHEN a fill runs and neighbourhood / localidad is not stored THE SYSTEM SHALL leave neighbourhood and localidad fields unchanged | T21 |
| WHEN a fill runs, the page has a resume or CV upload field, and the user has not chosen a resume file THE SYSTEM SHALL leave the upload field unchanged | T41 |
| WHEN a fill runs, the page has a resume or CV upload field, the user has chosen a resume file, and that file is not available THE SYSTEM SHALL leave the upload field unchanged | T41 |
| WHEN auto-mode is on and the current tab is a login, checkout, or settings form that is not a job application THE SYSTEM SHALL not fill any fields | T11 |
| WHEN a settings form has at least two personal-data fields and at least one job-application-specific field, and no password or payment-card fields THE SYSTEM SHALL treat it as a job application in version 1 and fill its empty mapped fields | T11 |
| WHEN auto-mode is off, the user has not clicked the extension icon, and the current tab is a job application THE SYSTEM SHALL not fill any fields until auto-mode is turned on | T32 |
| WHEN a fill runs and a field maps to both a built-in fact and a user-added fact THE SYSTEM SHALL leave that field unchanged | T31 |

## Tasks

### T1a: Scaffold the extension manifest, placeholder surfaces, and test setup

- Status: [ ] Pending
- Traces to: WHEN the profile is saved THE SYSTEM SHALL keep the stored profile so that other extensions cannot read it (manifest guard only; browser check in T42a)
- Depends on: none
- Objective: After this task, `package.json` lists `vitest` and `jsdom` as devDependencies, has a `test` script (`vitest run`, jsdom environment), and keeps the existing `typecheck` script. `tsconfig.json` has `strict` enabled (create it if absent). `src/manifest.json` is Manifest V3 and declares a service worker, an options page, `default_popup`, a content script at `document_idle` with broad host access, the `storage` and `unlimitedStorage` permissions, no `externally_connectable`, no `web_accessible_resources`, and no icon files. Placeholder `src/service-worker.ts`, `src/content-script.ts`, `src/popup.html` with `src/popup.ts`, and `src/options.html` with `src/options.ts` do nothing yet. `tests/` is created at the repo root.
- TDD plan:
  - Setup: add `vitest` and `jsdom` to `package.json` devDependencies and add a `test` script (`vitest run`, jsdom environment), keeping the existing `typecheck` script, so the Red test can run.
  - Red: `tests/manifest.test.ts` "src/manifest.json is Manifest V3, declares the service worker, options page, popup, and content script at document_idle, requests storage and unlimitedStorage, and declares no externally_connectable, no web_accessible_resources, and no icons". It fails because `src/manifest.json` does not exist.
  - Green: create the manifest, the placeholder files, the strict `tsconfig.json`, and the `test` script and devDependencies listed in Objective.
  - Verify: `npm test` and `npm run typecheck`. `npm run check` does not exist until T1b.
- Decision log:
- Outcome:

### T1b: Add the `build` and `check` scripts

- Status: [ ] Pending
- Traces to: none (build wiring; browser loading is verified in T42a and T42b)
- Depends on: T1a
- Objective: After this task, `package.json` has a `build` script that bundles the service worker, content script, popup, and options entry points from `src/` into `dist/` with esbuild and copies `src/manifest.json`, `src/popup.html`, and `src/options.html` into `dist/`. It has a `check` script that runs `typecheck`, `test`, and `build` in that order. `esbuild` is listed as a devDependency. The static scan is added to `check` in T2.
- TDD plan:
  - Red: `tests/build.test.ts` "npm run build writes dist/manifest.json, dist/options.html, and dist/popup.html, and bundles the service worker, content script, popup, and options entry points into dist/". It fails because no `build` script exists.
  - Green: the `build` and `check` scripts and the `esbuild` devDependency.
  - Verify: `npm run check`.
- Decision log:
- Outcome:

### T2: Static scan for remote calls in `src/` and `dist/`

- Status: [ ] Pending
- Traces to: WHEN the extension runs THE SYSTEM SHALL not send profile contents off the user’s machine, including through analytics, crash reports, or other remote calls that include profile contents
- Depends on: T1b
- Objective: `npm run check` fails when any file in `src/` or `dist/` contains `fetch`, `XMLHttpRequest`, `WebSocket`, `EventSource`, `navigator.sendBeacon`, or an `http://` or `https://` string literal. There are no exceptions. The scan runs after build and is part of `check`.
- TDD plan:
  - Red: `tests/static-scan.test.ts` "scan reports fetch, XMLHttpRequest, WebSocket, EventSource, navigator.sendBeacon, and an http:// or https:// literal in a file under src/ or dist/, and reports nothing for a clean file". It fails because the scanner does not exist.
  - Green: a scanner module, for example `scripts/static-scan.ts`, that exports the scan function and a CLI entry, wired into the `check` script after `build`.
  - Verify: `npm run check`.
- Decision log:
- Outcome:

### T3: Profile store round-trip in `chrome.storage.local`

- Status: [ ] Pending
- Traces to: WHEN the user saves the profile THE SYSTEM SHALL save and later reuse it without requiring a passphrase; WHEN the user starts a new session in the same browser after saving a profile THE SYSTEM SHALL still have those saved facts available for filling
- Depends on: T1b
- Objective: `src/profile-store.ts` defines the `Profile` type with the pieces in design.md Data models (`name.first`, `name.middle?`, `name.last`, `dateOfBirth {year, month, day}`, `email`, `phone.countryCode`, `phone.nationalNumber`, `address.streetNumbers`, `address.details`, `address.city`, `address.region`, `address.postalCode`, `address.country`, `address.locality`, `links.linkedin`, `links.github`, `links.portfolio`, `usWorkAuthorization`, `usSponsorshipNeeded`, `eeo.gender`, `eeo.raceEthnicity`, `eeo.veteran`, `eeo.disability`, `howHeard`, `userAdded`). It also defines and exports `export interface UserAddedFact { id: string; value: string; questions: string[] }`, used by `Profile.userAdded` and imported by T9. `saveProfile` and `loadProfile` read and write the `profile` key of `chrome.storage.local`. No passphrase or encryption step.
- TDD plan:
  - Red: `tests/profile-store.test.ts` "saveProfile then loadProfile returns the same pieces from chrome.storage.local key profile with no passphrase step, and a fresh module instance still reads them". It uses a stub of `chrome.storage.local`.
  - Green: `src/profile-store.ts` with the type and the two functions.
  - Verify: `npm run check`.
- Decision log:
- Outcome:

### T4: Auto-mode state in session storage

- Status: [ ] Pending
- Traces to: WHEN the browser starts a new session THE SYSTEM SHALL set auto-mode off; WHEN auto-mode is on THE SYSTEM SHALL keep auto-mode on until the user turns it off from the popup or the browser starts a new session
- Depends on: T1b
- Objective: The service worker keeps auto-mode in `chrome.storage.session`. A missing value reads as off. `SetAutoMode { on }` writes it and `GetAutoMode` returns `{ on }`. The value is never written to `chrome.storage.local`. Only a `SetAutoMode { on: false }` turns it off during a session.
- TDD plan:
  - Red: `tests/auto-mode.test.ts` "GetAutoMode returns off when no session value exists, returns on after SetAutoMode on and still on after a fresh read, and never writes to chrome.storage.local, and SetAutoMode { on: false } makes GetAutoMode return { on: false }". It uses stubs for `chrome.storage.session` and `chrome.storage.local`.
  - Green: `src/auto-mode.ts` with get and set functions, and the message handler in `src/service-worker.ts`.
  - Verify: `npm run check`.
- Decision log:
- Outcome:

### T5: Options page renders built-in fields and saved values

- Status: [ ] Pending
- Traces to: WHEN the user opens the extension’s options page THE SYSTEM SHALL let the user view, enter, and save these built-in profile facts: first name; middle name (optional); last name; date of birth; email; phone country code; phone national number; street numbers; address details for apartment number or similar; city; state/region; postal code; country; neighbourhood / localidad; LinkedIn URL; GitHub URL; portfolio or personal-site URL; US work authorization; whether US sponsorship is needed; gender; race/ethnicity; veteran status; disability status; a decline-to-answer option for each of those EEO facts, typed as the page shows it; one canned “how did you hear about us” value; and one resume file
- Depends on: T3
- Objective: `options.html` and `src/options.ts` show one plain text input for each built-in fact in the list above. The date of birth uses a native date input. Each EEO fact has one plain text input in which the user types the decline-to-answer wording the page shows. Values load from `loadProfile` when the page opens. The resume file picker is T7b.
- TDD plan:
  - Red: `tests/options-render.test.ts` "options page renders one input per built-in fact and shows values loaded from the saved profile". It fails because the options page has no fields.
  - Green: a field-list constant and a render function in `src/options.ts`, with `options.html` as the host.
  - Verify: `npm run check`.
- Decision log:
- Outcome:

### T6a: Options page saves built-in fields as pieces

- Status: [ ] Pending
- Traces to: WHEN the user opens the extension’s options page THE SYSTEM SHALL let the user view, enter, and save these built-in profile facts: first name; middle name (optional); last name; date of birth; email; phone country code; phone national number; street numbers; address details for apartment number or similar; city; state/region; postal code; country; neighbourhood / localidad; LinkedIn URL; GitHub URL; portfolio or personal-site URL; US work authorization; whether US sponsorship is needed; gender; race/ethnicity; veteran status; disability status; a decline-to-answer option for each of those EEO facts, typed as the page shows it; one canned “how did you hear about us” value; and one resume file; WHEN the user saves name, phone, address, or date of birth THE SYSTEM SHALL store that fact as the pieces listed above so a later fill can compose or split it to match the page; WHEN the user saves a profile fact THE SYSTEM SHALL not store a second copy of that fact that can disagree with the saved pieces
- Depends on: T5
- Objective: The save handler builds a `Profile` from the form. Date of birth is split from the `YYYY-MM-DD` date input into `{ year, month, day }`. Name, phone, and address stay as pieces. The stored object has no `fullName`, `fullPhone`, or `fullAddress` key.
- TDD plan:
  - Red: `tests/options-save.test.ts` "saving the form stores dateOfBirth as { year, month, day } from a 1990-03-07 input, stores phone and address as pieces, links.linkedin, links.github, links.portfolio, eeo.gender, eeo.raceEthnicity, eeo.veteran, eeo.disability, howHeard, usWorkAuthorization, usSponsorshipNeeded, and address.locality survive a save and loadProfile round trip unchanged, and has no fullName, fullPhone, or fullAddress key".
  - Green: `buildProfileFromForm` in `src/profile-form.ts`, called from the save handler.
  - Verify: `npm run check`.
- Decision log:
- Outcome:

### T6b: Blank middle name omitted; second save replaces

- Status: [ ] Pending
- Traces to: WHEN the user opens the extension’s options page THE SYSTEM SHALL let the user view, enter, and save these built-in profile facts: first name; middle name (optional); last name; date of birth; email; phone country code; phone national number; street numbers; address details for apartment number or similar; city; state/region; postal code; country; neighbourhood / localidad; LinkedIn URL; GitHub URL; portfolio or personal-site URL; US work authorization; whether US sponsorship is needed; gender; race/ethnicity; veteran status; disability status; a decline-to-answer option for each of those EEO facts, typed as the page shows it; one canned “how did you hear about us” value; and one resume file; WHEN the user changes a stored fact on the options page and saves THE SYSTEM SHALL use the updated fact on fills after that save (the save part; the fill part is T33a)
- Depends on: T6a
- Objective: A blank middle name is not stored. A second save replaces the earlier values.
- TDD plan:
  - Red: `tests/options-save.test.ts` "a blank middle name is not stored, and a second save replaces the first value".
  - Green: the middle-name omission and replacement behaviour in `buildProfileFromForm` in `src/profile-form.ts`.
  - Verify: `npm run check`.
- Decision log:
- Outcome:

### T7a: Resume store

- Status: [ ] Pending
- Traces to: WHEN the user starts a new session in the same browser after saving a profile THE SYSTEM SHALL still have those saved facts available for filling
- Depends on: T1b
- Objective: `src/resume-store.ts` defines `saveResume({ fileName, mimeType, bytes })` and `loadResume()`. `saveResume` writes `fileName`, `mimeType`, and the base64-encoded bytes under the `resume` key of `chrome.storage.local`. `loadResume` returns the same `fileName`, `mimeType`, and decoded bytes. Base64 conversion uses `ArrayBuffer`/`Uint8Array`. No passphrase or encryption step.
- TDD plan:
  - Red: `tests/resume-store.test.ts` "saveResume stores fileName, mimeType, and base64 bytes under the resume key of chrome.storage.local, and loadResume returns the same fileName, mimeType, and bytes after a fresh module instance". It uses a stub of `chrome.storage.local`.
  - Green: `src/resume-store.ts` with the two functions and base64 conversion.
  - Verify: `npm run check`.
- Decision log:
- Outcome:

### T7b: Options page resume file picker

- Status: [ ] Pending
- Traces to: WHEN the user opens the extension’s options page THE SYSTEM SHALL let the user view, enter, and save these built-in profile facts: first name; middle name (optional); last name; date of birth; email; phone country code; phone national number; street numbers; address details for apartment number or similar; city; state/region; postal code; country; neighbourhood / localidad; LinkedIn URL; GitHub URL; portfolio or personal-site URL; US work authorization; whether US sponsorship is needed; gender; race/ethnicity; veteran status; disability status; a decline-to-answer option for each of those EEO facts, typed as the page shows it; one canned “how did you hear about us” value; and one resume file
- Depends on: T7a, T5
- Objective: The options page has one file input for the resume and shows the saved file name. On save, the handler reads the chosen file and calls `saveResume` from T7a with `fileName`, `mimeType`, and bytes. When a saved resume exists, the page displays its `fileName` on load.
- TDD plan:
  - Red: `tests/options-resume.test.ts` "the options page file input saves through saveResume and shows the saved file name".
  - Green: the file input and saved-name display in `src/options.ts`, wired to `saveResume`/`loadResume`.
  - Verify: `npm run check`.
- Decision log:
- Outcome:

### T8: Options add-fact menu

- Status: [ ] Pending
- Traces to: WHEN the user opens the extension’s options page THE SYSTEM SHALL present a basic, non-stylized menu to add a fact and the question or questions that fact maps to
- Depends on: T5
- Objective: The options page has a plain section with one value input, one question input, an "add question" button that adds another question input, a remove button for each question input, and a save button. It uses default browser styling and no theme.
- TDD plan:
  - Red: `tests/add-fact-menu.test.ts` "add-fact menu renders a value input and one question input, an add-question button adds another question input, and each question input has a remove button".
  - Green: the menu render function in `src/options.ts`, with markup in `options.html`.
  - Verify: `npm run check`.
- Decision log:
- Outcome:

### T9: Save user-added facts with their questions

- Status: [ ] Pending
- Traces to: WHEN the user adds a fact and the questions it maps to and saves THE SYSTEM SHALL store that fact and those mappings with the profile and use them on fills after that save
- Depends on: T3, T6b, T8
- Objective: Save writes `profile.userAdded` as entries `{ id, value, questions }`. Each question is stored as the user typed it, and a blank question row is not stored as a question. An empty list is a valid saved value. The list loads back through `loadProfile`.
- TDD plan:
  - Red: `tests/user-added-save.test.ts` "saving a fact with two questions stores one UserAddedFact with both questions in profile.userAdded, a blank question row is not stored as a question, and saving with no facts stores an empty list".
  - Green: the save path in `src/user-added.ts` wired into the options save handler.
  - Verify: `npm run check`.
- Decision log:
- Outcome:

### T10a: Field snapshot: label sources

- Status: [ ] Pending
- Traces to: WHEN a fill runs THE SYSTEM SHALL identify fields from the page’s labels, accessible names, placeholders, and similar cues
- Depends on: T1b
- Objective: `src/field-snapshot.ts` builds, in one pass over the DOM, an entry per text-like input, textarea, select, radio group, ARIA combobox, and file input. Each entry holds the control reference, `label`, `accessibleName`, `placeholder`, `type`, `maxlength`, `autocomplete`, and an `empty` flag. Labels come from `for`/`id`, an enclosing `label`, `aria-labelledby`, and `legend`. `autocomplete` is held on each entry for identification and for the payment-card rule in T11.
- TDD plan:
  - Red: `tests/field-snapshot.test.ts` "snapshot reads the label from a for/id association, an enclosing label, aria-labelledby, a legend, and a placeholder", and "snapshot exposes autocomplete on each entry, with cc-number read from a control whose autocomplete is cc-number". It fails because the snapshot module does not exist.
  - Green: `src/field-snapshot.ts`.
  - Verify: `npm run check`.
- Decision log:
- Outcome:

### T10b: Field snapshot: emptiness and cue normalization

- Status: [ ] Pending
- Traces to: WHEN a fill runs THE SYSTEM SHALL identify fields from the page’s labels, accessible names, placeholders, and similar cues
- Depends on: T10a
- Objective: `src/field-snapshot.ts` reports emptiness per control type: a text field with a value is not empty; an empty select, an unchecked radio group, and a file input with no file are empty. Cue matching trims whitespace, removes trailing `*`, `:`, or `?`, and case-folds ASCII. Emptiness means no user-visible value, no selected choice, or no file.
- TDD plan:
  - Red: `tests/field-snapshot.test.ts` "snapshot reports a text field with a value as not empty, an empty select, an unchecked radio group, and a file input with no file as empty, and normalizes cue text by trimming, removing a trailing ':', and case-folding ASCII". It fails because the emptiness and normalization logic does not exist.
  - Green: `src/field-snapshot.ts`.
  - Verify: `npm run check`.
- Decision log:
- Outcome:

### T11: Job-application classifier

- Status: [ ] Pending
- Traces to: WHEN the user clicks the extension icon and the current tab is not a job application THE SYSTEM SHALL not fill any fields on that tab; WHEN auto-mode is on and the current tab is a login, checkout, or settings form that is not a job application THE SYSTEM SHALL not fill any fields; WHEN a settings form has at least two personal-data fields and at least one job-application-specific field, and no password or payment-card fields THE SYSTEM SHALL treat it as a job application in version 1 and fill its empty mapped fields
- Depends on: T10b
- Objective: `isJobApplication(snapshot)` returns true only when the snapshot has at least two personal-data fields (name, email, phone, address, or date-of-birth cues) and at least one job-application field (LinkedIn, GitHub, portfolio, resume/CV upload, US work authorization or sponsorship, EEO, how-heard, or cover letter), and has no password field and no payment-card field. It reads only the snapshot fields, so a settings-style form with the same field set counts the same as any other page. Detection uses no job-board list. A password field is an `input` whose `type` is `password`. A payment-card field is a control whose `autocomplete` is `cc-number`, `cc-exp`, or `cc-csc`, or whose label, accessible name, or placeholder exactly equals one of: card number, expiration date, expiry, CVV, CVC, security code.
- TDD plan:
  - Red: `tests/classify.test.ts` "isJobApplication is true for two personal fields plus a LinkedIn field, including a settings-style form with the same fields and no password or payment-card field, false for one personal field, false for two personal fields with no job-application field, false for a page with a password field, false for a page with a payment-card field whose label exactly equals a card cue, and false for a settings-style page with two personal fields and a LinkedIn field plus a control whose autocomplete is cc-number and whose label is not a card cue". It fails because the classifier does not exist.
  - Green: `src/classify.ts`.
  - Verify: `npm run check`.
- Decision log:
- Outcome:

### T12: Write a stored value into an empty text field as user input

- Status: [ ] Pending
- Traces to: WHEN a fill runs and a field has no value and maps cleanly to a stored profile fact THE SYSTEM SHALL write that stored value into the field as if the user had typed or selected it, so the page treats the field as filled; WHEN a fill runs and a field already has a value THE SYSTEM SHALL leave that field unchanged
- Depends on: T10b
- Objective: A writer sets the value of an empty input or textarea through the native value setter and dispatches `input` and `change`. A field that already has a value is never passed to the writer. The writer returns without setting the value or dispatching input or change when the control already has a value.
- TDD plan:
  - Red: `tests/write-text.test.ts` "writes through the native value setter and dispatches input and change, and an input whose value is 'Maybe' receives no value-setter call and no input or change event".
  - Green: `src/write-text.ts`.
  - Verify: `npm run check`.
- Decision log:
- Outcome:

### T13: Built-in text facts: email and profile links

- Status: [ ] Pending
- Traces to: WHEN a fill runs and a field has no value and maps cleanly to a stored profile fact THE SYSTEM SHALL write that stored value into the field as if the user had typed or selected it, so the page treats the field as filled
- Depends on: T3, T12
- Objective: The Email, LinkedIn, GitHub, and Portfolio cue strings from design.md Cue vocabulary map to `email`, `links.linkedin`, `links.github`, and `links.portfolio`. Only empty fields are written.
- TDD plan:
  - Red: `tests/text-mapping.test.ts` "an empty field labelled 'LinkedIn Profile URL' gets links.linkedin, an empty field labelled 'Email address' gets email, an empty field labelled 'GitHub URL' gets links.github, an empty field labelled 'Portfolio website' gets links.portfolio, and a valued Email field is unchanged". It fails because the cue table and mapper do not exist.
  - Green: the cue table and mapper in `src/mappers/text.ts`.
  - Verify: `npm run check`.
- Decision log:
- Outcome:

### T14a: Name fill for one name field (composition, pieces, and exclusion of preferred/legal/passport)

- Status: [ ] Pending
- Traces to: WHEN a fill runs and the page presents exactly one field asking for the applicant’s first, middle, last, or full name THE SYSTEM SHALL fill that field if it has no value: with the composed stored name, including middle name only if it is stored, when the field asks for the full name, or with the matching stored piece when the field asks for first, middle, or last name; WHEN a fill runs and the page presents preferred-name, legal-name, or name-as-on-passport fields THE SYSTEM SHALL not count those fields toward the one-, two-, three-, or four-or-more name fills; WHEN a fill runs and the page asks for a name variant that is not stored as its own fact (preferred name, legal name, or name as on passport) THE SYSTEM SHALL leave that field unchanged
- Depends on: T3, T10b, T12
- Objective: The name census counts only fields whose cue exactly matches one of the name cues in design.md Cue vocabulary (full name, your name, nombre completo, first name, given name, primer nombre, middle name, segundo nombre, last name, family name, surname, apellido). Preferred-name, legal-name, and passport-name fields are not counted and are left unchanged. With exactly one counted field, an empty field is filled with `First Middle Last` (middle only if stored) for a full-name cue, or with the matching piece for a first, middle, or last cue. A valued name field is never overwritten.
- TDD plan:
  - Red: `tests/name-one-field.test.ts` "with one 'Full name' field, the field gets 'First Last' when middle is not stored and 'First Middle Last' when it is; a single empty 'First name', 'Middle name', or 'Last name' field gets the matching stored piece; a valued name field is unchanged; with one Full name field and a Preferred name field, the Preferred name field stays empty; with Legal name and Name as on passport also present, both stay empty and the Full name field still receives the composed name". It fails because the census, count-1 branch, and exclusion of preferred, legal, and passport fields do not exist.
  - Green: the census, count-1 branch, and exclusion of preferred, legal, and passport name fields from the census in `src/mappers/name.ts`.
  - Verify: `npm run check`.
- Decision log:
- Outcome:

### T15: Name fill for two name fields

- Status: [ ] Pending
- Traces to: WHEN a fill runs and the page presents exactly two fields asking for the applicant’s first, middle, last, or full name THE SYSTEM SHALL fill each of those fields that has no value and whose own label, accessible name, or placeholder says first name or last name with that stored piece; a field whose cue says neither stays unchanged; WHEN a fill runs and the page presents exactly two fields asking for the applicant’s first, middle, last, or full name THE SYSTEM SHALL not write a composed full name into either field
- Depends on: T14a
- Objective: With exactly two counted name fields, each empty field whose own cue is a first-name or last-name cue receives that piece. A field whose cue is neither is unchanged. No composed full name is written.
- TDD plan:
  - Red: `tests/name-two-fields.test.ts` "two empty fields labelled 'First name' and 'Last name' get the first and last piece, and two fields labelled 'Full name' and 'Your name' both stay empty".
  - Green: the count-2 branch in `src/mappers/name.ts`.
  - Verify: `npm run check`.
- Decision log:
- Outcome:

### T16: Name fill for three name fields

- Status: [ ] Pending
- Traces to: WHEN a fill runs and the page presents exactly three fields asking for the applicant’s first, middle, last, or full name THE SYSTEM SHALL fill each of those fields that has no value and whose own cue says first name, middle name, or last name with that stored piece; a field whose cue does not say which piece it is stays unchanged; WHEN a fill runs and middle name is not stored THE SYSTEM SHALL leave middle-name fields unchanged
- Depends on: T15
- Objective: With exactly three counted name fields, each empty field whose own cue is a first, middle, or last cue receives that piece. A field whose cue says none of these stays unchanged. When middle name is not stored, the middle-name field stays unchanged.
- TDD plan:
  - Red: `tests/name-three-fields.test.ts` "fields labelled 'First name', 'Middle name', and 'Full name' get the first piece and the middle piece, the 'Full name' field stays empty, and with no middle name stored the middle field stays empty".
  - Green: the count-3 branch in `src/mappers/name.ts`.
  - Verify: `npm run check`.
- Decision log:
- Outcome:

### T17: Name fill for four or more name fields by exact label

- Status: [ ] Pending
- Traces to: WHEN a fill runs and the page presents four or more fields asking for the applicant’s first, middle, last, or full name THE SYSTEM SHALL not use the one-, two-, or three-field name fills; WHEN a fill runs, the page presents four or more fields asking for the applicant’s first, middle, last, or full name, and a field’s label, after trimming and case-folding, is exactly `first name` or `primer nombre` THE SYSTEM SHALL fill that field from the stored first name if that field has no value; WHEN a fill runs, the page presents four or more fields asking for the applicant’s first, middle, last, or full name, and a field’s label is exactly `last name` or `apellido` THE SYSTEM SHALL fill that field from the stored last name if that field has no value; WHEN a fill runs, the page presents four or more fields asking for the applicant’s first, middle, last, or full name, and a field’s label is exactly `middle name` or `segundo nombre` THE SYSTEM SHALL fill that field from the stored middle name if that field has no value; WHEN a fill runs, the page presents four or more fields asking for the applicant’s first, middle, last, or full name, and a field’s label, after trimming and case-folding, is not exactly `first name`, `primer nombre`, `last name`, `apellido`, `middle name`, or `segundo nombre` THE SYSTEM SHALL not fill that field from the stored first, middle, or last name
- Depends on: T16
- Objective: With four or more counted name fields, the 1/2/3 strategies are not used. An empty field is filled only when its label, trimmed and case-folded, is exactly `first name`, `primer nombre`, `last name`, `apellido`, `middle name`, or `segundo nombre`. Any other label is not filled from first, middle, or last.
- TDD plan:
  - Red: `tests/name-four-plus.test.ts` "with four name fields labelled 'First Name', 'Apellido', 'Middle name', and 'Full name' and a stored middle name, the first three get first, last, and middle and the 'Full name' field stays empty; with four name fields labelled 'Primer nombre', 'Segundo nombre', 'Apellido', and 'Full name', the first three get first, middle, and last".
  - Green: the count-4+ branch in `src/mappers/name.ts`.
  - Verify: `npm run check`.
- Decision log:
- Outcome:

### T18: Phone compose and split

- Status: [ ] Pending
- Traces to: WHEN a fill runs and the page has a single phone field that has no value and no separate country-code field THE SYSTEM SHALL leave that field unchanged, and SHALL NOT join the stored country code into it; WHEN a fill runs and the page has a country-code field and a phone field that has no value THE SYSTEM SHALL fill the phone field with the stored national number; WHEN a fill runs and the page has a country-code field and/or a national-number field THE SYSTEM SHALL fill each of those fields that has no value from the matching stored piece
- Depends on: T3, T12
- Objective: A phone-cue field gets `phone.nationalNumber` only when a country-code field is on the page. A country-code field gets `phone.countryCode`, and a national-number field gets `phone.nationalNumber` even when no country-code field is present. A single phone field with no country-code field stays unchanged, and the country code is never joined into it.
- TDD plan:
  - Red: `tests/phone-pieces.test.ts` "a country-code field and a phone field get countryCode and nationalNumber, a lone empty National number field gets phone.nationalNumber with no country-code field present, and a single empty phone field with no country-code field stays unchanged".
  - Green: `src/mappers/phone.ts`.
  - Verify: `npm run check`.
- Decision log:
- Outcome:

### T19: Street and address-line pieces

- Status: [ ] Pending
- Traces to: WHEN a fill runs and the page has a single street or address-line field that has no value THE SYSTEM SHALL fill that field by composing the stored street-numbers piece and the stored address-details piece that exist; WHEN a fill runs and the page has a street-numbers field and/or an address-details field (apartment number or similar) THE SYSTEM SHALL fill each of those fields that has no value from the matching stored piece; WHEN a fill runs and street numbers are not stored THE SYSTEM SHALL leave street-numbers fields unchanged; WHEN a fill runs and address details (apartment number or similar) are not stored THE SYSTEM SHALL leave address-details fields unchanged
- Depends on: T3, T12
- Objective: A single address-line field gets `streetNumbers, details` (each piece only when stored, with `,`  between them). A street-number field gets `streetNumbers` and an address-details field gets `details`. When a piece is not stored, the field for that piece stays unchanged.
- TDD plan:
  - Red: `tests/address-line.test.ts` "an empty 'Address' field gets 'streetNumbers, details' when both are stored, an empty 'Street number' field gets streetNumbers, an empty 'Apt' field gets details, with no street numbers stored the 'Street number' field stays empty, with no address details stored an empty 'Apt' field stays empty, with only address details stored an empty Address field gets details alone, and with only street numbers stored it gets streetNumbers alone".
  - Green: `src/mappers/address.ts`.
  - Verify: `npm run check`.
- Decision log:
- Outcome:

### T20a: Full address compose

- Status: [ ] Pending
- Traces to: WHEN a fill runs and the page has a single full-address field (not a street-only or address-line field) that has no value and that maps cleanly to stored street numbers, address details, city, state/region, or postal code THE SYSTEM SHALL fill that field by composing those stored pieces, writing only the stored parts when some are missing
- Depends on: T19
- Objective: A full-address field (cue from design.md Cue vocabulary) gets the composed value: the street line, then one space, then city, one space, region, one space, and postal code, with missing parts skipped. Country and locality are excluded from the compose. Extends `src/mappers/address.ts`.
- TDD plan:
  - Red: `tests/address-full.test.ts` "an empty 'Full address' field gets the street line, one space, city, one space, region, one space, and postal code with the missing parts skipped and no country or locality".
  - Green: the full-address compose branch in `src/mappers/address.ts`.
  - Verify: `npm run check`.
- Decision log:
- Outcome:

### T20b: City, state/region, postal code, and country fields

- Status: [ ] Pending
- Traces to: WHEN a fill runs and the page has a city, state/region, postal code, and/or country field THE SYSTEM SHALL fill each of those fields that has no value from the matching stored piece
- Depends on: T3, T12, T19
- Objective: City, state/region, postal code, and country fields each get their own stored piece when empty. Extends `src/mappers/address.ts`.
- TDD plan:
  - Red: `tests/address-city-region-postal-country.test.ts` "empty 'City', 'State/Province', 'Postal code', and 'Country' fields each get their piece".
  - Green: the city, region, postal code, and country mapping in `src/mappers/address.ts`.
  - Verify: `npm run check`.
- Decision log:
- Outcome:

### T21: Neighbourhood / localidad field

- Status: [ ] Pending
- Traces to: WHEN a fill runs and the page has a neighbourhood or localidad field that has no value and neighbourhood / localidad is stored THE SYSTEM SHALL fill that field from the stored neighbourhood / localidad; WHEN a fill runs and neighbourhood / localidad is not stored THE SYSTEM SHALL leave neighbourhood and localidad fields unchanged
- Depends on: T19
- Objective: An empty field whose cue is a neighbourhood or localidad cue gets `address.locality` when stored. When it is not stored, the field stays unchanged.
- TDD plan:
  - Red: `tests/locality.test.ts` "an empty 'Neighbourhood' field and an empty 'Localidad' field get address.locality when stored, and both stay empty when it is not stored".
  - Green: the locality mapping in `src/mappers/address.ts`.
  - Verify: `npm run check`.
- Decision log:
- Outcome:

### T22: Date of birth in one field

- Status: [ ] Pending
- Traces to: WHEN a fill runs and the page has a single date-of-birth field that has no value and date of birth is stored THE SYSTEM SHALL fill that field from the stored date of birth, written in the date format that field asks for (month and day zero-padded, and a two-digit year where the field asks for one); WHEN a fill runs and a date-of-birth field’s expected format cannot be determined from the page THE SYSTEM SHALL leave that field unchanged; WHEN a fill runs and the page asks for a date other than date of birth and no stored fact is mapped to that question THE SYSTEM SHALL leave that field unchanged
- Depends on: T3, T12
- Objective: A `type="date"` field gets `YYYY-MM-DD`. Otherwise the format is read from the field's placeholder or label tokens (`dd`, `mm`, `yyyy` or `yy`, `aaaa` or `aa`, separated by `/`, `-`, or `.`), and the token order sets the output order. Month and day are always zero-padded, and the year is two digits when the pattern says `yy` or `maxlength="2"`. A field with no recognizable pattern stays unchanged. A "Start date" field is never filled from the date of birth.
- TDD plan:
  - Red: `tests/dob-single.test.ts` "an empty field labelled Date of birth with placeholder MM/DD/YYYY gets 03/07/1990, labelled Date of birth with placeholder DD.MM.YY gets 07.03.90, labelled Date of birth with no recognizable pattern stays empty, an empty Start date field stays empty, and an empty input with type=date labelled Date of birth gets 1990-03-07".
  - Green: the format parser and single-field writer in `src/mappers/dob.ts`.
  - Verify: `npm run check`.
- Decision log:
- Outcome:

### T23: Date of birth in separate month, day, and year fields

- Status: [ ] Pending
- Traces to: WHEN a fill runs and the page has a month, day, and/or year field for date of birth and date of birth is stored THE SYSTEM SHALL fill each of those fields that has no value from the stored date of birth, with a zero-padded month and day and a two-digit year where that field asks for one
- Depends on: T22
- Objective: Empty month and day text fields get zero-padded values. The year field gets four digits, or two digits when `maxlength="2"`. Each field is written only when it is empty.
- TDD plan:
  - Red: `tests/dob-parts.test.ts` "empty 'Birth month', 'Birth day', and 'Birth year' fields get '03', '07', and '1990', and a 'Birth year' field with maxlength 2 gets '90'".
  - Green: the parts writer in `src/mappers/dob.ts`.
  - Verify: `npm run check`.
- Decision log:
- Outcome:

### T24: Native select writer for stored choices

- Status: [ ] Pending
- Traces to: WHEN a fill runs and a choice field has no value and maps cleanly to stored US work authorization, sponsorship, an EEO fact, or the canned “how did you hear about us” value THE SYSTEM SHALL set that choice as if the user had selected it; WHEN setting a stored value requires using a native select or radio group THE SYSTEM SHALL set the stored value through that widget without advancing or completing the application
- Depends on: T10b
- Objective: For an empty native `<select>`, the option whose visible text equals the stored value (trimmed, case-folded) is selected, and `change` is dispatched. The option value attribute is not matched. With no exact match the select stays unchanged. Submit, Next, and Continue are never touched.
- TDD plan:
  - Red: `tests/select-writer.test.ts` "an empty select with an option whose text is 'no' gets selected for stored 'No' and change fires, an option whose value is 'no' but text is 'Maybe' is not chosen, and no exact match leaves the select unchanged, and with a Next button in the same form the select writer records zero click calls on it". It fails because the select writer does not exist.
  - Green: `src/widgets/select.ts`.
  - Verify: `npm run check`.
- Decision log:
- Outcome:

### T25: Radio group writer for stored choices

- Status: [ ] Pending
- Traces to: WHEN a fill runs and a choice field has no value and maps cleanly to stored US work authorization, sponsorship, an EEO fact, or the canned “how did you hear about us” value THE SYSTEM SHALL set that choice as if the user had selected it; WHEN setting a stored value requires using a native select or radio group THE SYSTEM SHALL set the stored value through that widget without advancing or completing the application
- Depends on: T10b
- Objective: For an unchecked radio group with no selection, the radio whose label equals the stored value (trimmed, case-folded) is checked and `change` is dispatched. With no exact label match the group stays unchanged. Submit, Next, and Continue are never touched.
- TDD plan:
  - Red: `tests/radio-writer.test.ts` "a radio group with an option labelled 'Yes' gets checked for stored 'Yes', and a group with no exact label match stays unselected; with a Next button in the same form, the radio writer records zero click calls on it".
  - Green: `src/widgets/radio.ts`.
  - Verify: `npm run check`.
- Decision log:
- Outcome:

### T26: ARIA combobox writer for stored choices (best-effort)

- Status: [ ] Pending
- Traces to: WHEN setting a stored value requires using an ARIA dropdown (a combobox or listbox that is not a native select) THE SYSTEM SHALL attempt to set the stored value by exact option match without advancing or completing the application, and in version 1 some sites will leave that field unchanged
- Depends on: T10b
- Objective: For an empty `role="combobox"` or `listbox` control, the writer opens it, chooses the option whose text equals the stored value (trimmed, case-folded), and closes it. A search box may receive the stored value, but only an exact-match option is chosen. With no exact match the control stays unchanged and no other option is clicked. This is best-effort per design.md and the requirement's version 1 wording.
- TDD plan:
  - Red: `tests/combobox-writer.test.ts` "a role=combobox control opens, chooses the option whose text exactly equals the stored value, and closes; with no exact option it stays unchanged and no option is clicked".
  - Green: `src/widgets/combobox.ts`.
  - Verify: `npm run check`.
- Decision log:
- Outcome:

### T27: Date-of-birth month and day select branch

- Status: [ ] Pending
- Traces to: WHEN a fill runs and the page has a month, day, and/or year field for date of birth and date of birth is stored THE SYSTEM SHALL fill each of those fields that has no value from the stored date of birth, with a zero-padded month and day and a two-digit year where that field asks for one
- Depends on: T23, T24
- Objective: An empty month or day `<select>` chooses the option whose value or visible text equals the padded number or the unpadded number. A year `<select>` chooses the option whose value or visible text equals the stored year in the format the field asks for. Month names are not read. T24's text-only native select writer is left unchanged; this task adds a separate exported matcher in `src/widgets/select.ts` that matches by value or visible text and is used only by the date-of-birth select branch.
- TDD plan:
  - Red: `tests/dob-select.test.ts` "a month select with option values 01 to 12 gets '03' for month 3, a day select with option text '7' gets chosen for day 7, and a year select with option '1990' gets chosen".
  - Green: the select branch in `src/mappers/dob.ts`, using the new value-or-text matcher exported from `src/widgets/select.ts`. T24's text-only writer remains unchanged.
  - Verify: `npm run check`.
- Decision log:
- Outcome:

### T28: Choice facts mapping

- Status: [ ] Pending
- Traces to: WHEN a fill runs and a choice field has no value and maps cleanly to stored US work authorization, sponsorship, an EEO fact, or the canned “how did you hear about us” value THE SYSTEM SHALL set that choice as if the user had selected it; WHEN a fill runs and “how did you hear about us” is a free-text field and no stored fact is mapped to that question THE SYSTEM SHALL leave that field unchanged; WHEN a fill runs and “how did you hear about us” is a choice field that has no value and that maps cleanly to the stored canned value THE SYSTEM SHALL select that stored value; WHEN a fill runs and the page asks about work authorization or sponsorship for a country other than the United States and no stored fact is mapped to that question THE SYSTEM SHALL leave those fields unchanged
- Depends on: T3, T24, T25, T26
- Objective: The US work authorization, US sponsorship, EEO (gender, race/ethnicity, veteran status, disability status, with decline-to-answer text), and how-heard cues from design.md Cue vocabulary map to their stored choices and go through the widget writers. A free-text how-heard field, and a work-authorization or sponsorship question for a non-US country, are not mapped from built-in facts.
- TDD plan:
  - Red: `tests/choice-facts.test.ts` "an empty select labelled 'Are you legally authorized to work in the United States' gets the stored usWorkAuthorization, an empty 'Gender' select gets eeo.gender, an unchecked radio group labelled 'Do you require sponsorship' with an option labelled 'No' is checked for stored usSponsorshipNeeded 'No', a role=combobox labelled 'Gender' with option 'Man' gets eeo.gender 'Man', and an empty 'How did you hear about us' select gets howHeard, while a free-text 'How did you hear about us' textarea and an 'Are you legally authorized to work in Canada' select stay unchanged".
  - Green: `src/mappers/choice.ts`.
  - Verify: `npm run check`.
- Decision log:
- Outcome:

### T29: User-added fact writes into an empty field matched by exact question

- Status: [ ] Pending
- Traces to: WHEN a fill runs and a field has no value and that field’s label, accessible name, or placeholder is a question the user recorded for a stored user-added fact THE SYSTEM SHALL write that stored value into the field as if the user had typed or selected it
- Depends on: T9, T10b, T12, T24, T25, T26
- Objective: For each `userAdded` fact, an empty field matches when its label, accessible name, or placeholder equals one of the fact's `questions` after trimming leading and trailing whitespace. The match is exact and case-sensitive. A text input or textarea gets `value` written as text. A select, radio, or combobox is set through the widget writers from T24 to T26.
- TDD plan:
  - Red: `tests/user-added-match.test.ts` "an empty input whose placeholder equals the recorded question 'Notice period?' gets the stored value, an empty select whose label equals a recorded question gets the stored option set through the select writer, and a radio group whose label equals a recorded question is checked with the stored option through the radio writer".
  - Green: `src/mappers/user-added.ts`.
  - Verify: `npm run check`.
- Decision log:
- Outcome:

### T30: User-added fact is not used for non-matching cues; agree-to-terms never matched

- Status: [ ] Pending
- Traces to: WHEN a fill runs and a field’s label, accessible name, and placeholder are not the question the user recorded for a stored user-added fact THE SYSTEM SHALL not fill that field from that user-added fact; WHEN a fill runs and a field is an agree-to-terms checkbox or control, including one that a user-added fact would match THE SYSTEM SHALL not fill it and not activate it
- Depends on: T29
- Objective: A user-added fact matches only when one of the three cues equals a recorded question exactly. Substring, similar wording, and regex never match. A checkbox or `role="checkbox"` control is never a target, and the matcher returns no target for an agree-to-terms checkbox.
- TDD plan:
  - Red: `tests/user-added-exclusion.test.ts` "a field whose label contains a recorded question but is not equal gets no value, and the matcher returns no target for an agree-to-terms checkbox whose label equals a recorded question".
  - Green: the exact-match guard and the checkbox exclusion in `src/mappers/user-added.ts`.
  - Verify: `npm run check`.
- Decision log:
- Outcome:

### T31: Ambiguity guard: dual or multiple matches stay unchanged

- Status: [ ] Pending
- Traces to: WHEN a fill runs and a field’s mapping to the profile is missing or ambiguous THE SYSTEM SHALL leave that field unchanged; WHEN a fill runs and a field maps to both a built-in fact and a user-added fact THE SYSTEM SHALL leave that field unchanged
- Depends on: T13, T30
- Objective: A field that maps to a built-in fact and to a user-added fact, or to more than one user-added fact, gets no target. A field whose mapping is missing gets no target.
- TDD plan:
  - Red: `tests/ambiguity.test.ts` "an empty field labelled 'Email' whose placeholder equals a recorded question for a user-added fact gets no target, an empty field matched by two user-added facts gets no target, and an empty field with no mapping gets no target".
  - Green: the conflict check in the mapper pipeline (`src/mappers/resolve.ts`).
  - Verify: `npm run check`.
- Decision log:
- Outcome:

### T32: Fill pass gate

- Status: [ ] Pending
- Traces to: WHEN the user clicks the extension icon and the current tab is not a job application THE SYSTEM SHALL not fill any fields on that tab; WHEN auto-mode is off and the user has not clicked the extension icon THE SYSTEM SHALL not fill any fields; WHEN auto-mode is off, the user has not clicked the extension icon, and the current tab is a job application THE SYSTEM SHALL not fill any fields until auto-mode is turned on; WHEN a fill runs and no profile facts are stored THE SYSTEM SHALL leave every field unchanged; WHEN auto-mode is on and the current tab is not a job application THE SYSTEM SHALL not fill any fields
- Depends on: T3, T4, T11
- Objective: `shouldRunPass({ trigger, autoModeOn, isJobApplication, profileStored })` returns true only when the trigger is an icon click or auto-mode is on, the page is a job application, and a profile is stored. A profile is not stored when every built-in piece is absent or blank after trimming and `userAdded` is empty. `isProfileStored(profile: Profile): boolean` returns false when every built-in piece is absent or blank after trimming and `userAdded` is empty, and true when any built-in piece is non-blank or `userAdded` has at least one entry.
- TDD plan:
  - Red: `tests/pass-gate.test.ts` "shouldRunPass is false for an auto trigger with autoModeOn false even when isJobApplication is true and profileStored is true, false for a non-application page on icon or auto trigger, false when the profile is empty, and true for an icon trigger on a job application with a stored profile"; "isProfileStored(profile: Profile): boolean returns false when every piece is blank after trimming and userAdded is empty, and true for a profile whose only non-blank piece is email, and true for userAdded with two entries". It uses plain objects and no chrome stubs.
  - Green: `src/fill-gate.ts`.
  - Verify: `npm run check`.
- Decision log:
- Outcome:

### T33a: Fill pass wiring: all mappers into the fill pass

- Status: [ ] Pending
- Traces to: WHEN the user clicks the extension icon and the current tab is a job application (a page with at least two personal-data fields and at least one job-application-specific field, and no password or payment-card fields) THE SYSTEM SHALL fill that tab’s fields that have no value and that map cleanly to stored profile facts; WHEN auto-mode is on and the current tab is a job application (a page with at least two personal-data fields and at least one job-application-specific field, and no password or payment-card fields) THE SYSTEM SHALL fill fields that have no value and that map cleanly to stored profile facts without requiring a further click; WHEN a fill runs on a job application hosted by a job board the user did not name or configure THE SYSTEM SHALL still fill fields that have no value and that map cleanly to stored facts; WHEN a fill runs and a field has no value and maps cleanly to a stored profile fact THE SYSTEM SHALL write that stored value into the field as if the user had typed or selected it, so the page treats the field as filled; WHEN a fill runs and a field already has a value THE SYSTEM SHALL leave that field unchanged; WHEN the user has not added any extra facts THE SYSTEM SHALL still fill from the built-in facts; WHEN the user changes a stored fact on the options page and saves THE SYSTEM SHALL use the updated fact on fills after that save; WHEN a later fill runs from an icon click or from auto-mode and some fields already have values THE SYSTEM SHALL fill remaining fields that have no value and that map cleanly to stored profile facts; WHEN a fill runs and filling a field would require generating, paraphrasing, or otherwise inventing text that is not already stored THE SYSTEM SHALL leave that field unchanged; WHEN a fill runs and a field is an open-ended prompt for original writing (cover letter, why the applicant wants the job, tell-us-about-a-time, or similar) and no stored fact is mapped to that question THE SYSTEM SHALL leave that field unchanged; WHEN a fill runs and only some profile facts are stored THE SYSTEM SHALL fill fields that have no value and that map cleanly to stored facts and leave the rest unchanged; WHEN a fill runs THE SYSTEM SHALL not activate Submit, Next, or Continue when doing so would advance or complete the application; WHEN a fill runs and a field is an agree-to-terms checkbox or control, including one that a user-added fact would match THE SYSTEM SHALL not fill it and not activate it
- Depends on: T11, T13, T17, T18, T20a, T20b, T21, T23, T27, T28, T29, T31, T32
- Objective: `runFillPass(document, profile)` takes the snapshot, runs the gate, then the name, phone, address, locality, DOB, text, choice, and user-added mappers, and the ambiguity guard, and writes each remaining empty target. It reads the profile fresh on each pass. A page with an unnamed job board host is treated the same as any other job application. Empty textareas for cover letter, why-this-job, and tell-us-about-a-time prompts with no mapped user-added fact are never written by the pass, and an injected writer that would write default text into unmapped textareas produces no such text; every value the pass writes equals a value held in the stored profile or in userAdded. `runFillPass` dispatches no click on any Submit, Next, Continue, or agree-to-terms control. With only some profile facts stored, it writes only the mapped fields whose facts are stored and leaves every other field unchanged.
- TDD plan:
  - Red: `tests/fill-pass.test.ts` "a job-application fixture with an unnamed host gets its empty mapped fields filled, a valued field is unchanged, and after saving a changed email the next pass writes the new email into an empty field", plus "a second runFillPass on the same page leaves already-filled fields unchanged and fills a newly empty mapped field", plus "with userAdded set to [], the pass fills the empty mapped built-in fields", plus "an empty textarea labelled 'Cover letter' gets zero writer calls and stays empty when the injected writer would return 'Default text', and every value the pass returns is present in the stored profile or userAdded". `tests/fill-pass-no-click.test.ts` "a Submit button, a Continue button, a Next button, and an 'I agree' checkbox each get zero click calls, and with only email stored only the email field is filled, and the I agree checkbox stays unchecked with its value unchanged". It fails because src/fill-pass.ts does not exist yet.
  - Green: `src/fill-pass.ts` wiring the mappers and the gate, and the exclusion of Submit, Next, Continue, and agree-to-terms controls from every write.
  - Verify: `npm run check`.
- Decision log:
- Outcome:

### T33b: Fill pass uses user-added facts saved earlier

- Status: [ ] Pending
- Traces to: WHEN the user adds a fact and the questions it maps to and saves THE SYSTEM SHALL store that fact and those mappings with the profile and use them on fills after that save
- Depends on: T9, T33a
- Objective: After this task, a fill pass that runs after a user-added fact is saved writes that fact's value into an empty field whose label, accessible name, or placeholder equals one of its recorded questions, using the value held in the saved profile.
- TDD plan:
  - Red: `tests/fill-pass-user-added.test.ts` "after saving userAdded with question 'Notice period?', the next runFillPass writes its value into an empty input whose placeholder equals that question". It fails if runFillPass uses a profile captured before the save.
  - Green: the call in `src/fill-pass.ts` that loads the profile through `loadProfile` at the start of each pass, so a pass after a save sees the new `userAdded` list.
  - Verify: `npm run check`.
- Decision log:
- Outcome:

### T36a: Debounced auto-mode triggers for later steps (DOM mutation path)

- Status: [ ] Pending
- Traces to: WHEN auto-mode is on and the user advances to another step of a job application THE SYSTEM SHALL fill fields on that step that have no value and that map cleanly to stored profile facts without requiring another click; WHEN auto-mode is on and the current tab is a job application THE SYSTEM SHALL fill fields that have no value and that map cleanly to stored profile facts without requiring a further click
- Depends on: T4, T33b
- Objective: `src/content-script.ts` asks the service worker for `GetAutoMode`. When on, it runs one debounced pass after the page has been quiet for about 500 ms. DOM mutations schedule that same debounced pass. With auto-mode off, no pass is scheduled.
- TDD plan:
  - Red: `tests/auto-trigger.test.ts` "with auto-mode on at content-script start on a job application with no later DOM mutation, exactly one pass runs 500 ms after load"; "with auto-mode on, a DOM mutation adding an empty mapped field schedules exactly one pass after 500 ms of quiet, five mutations in a row run one pass, and with auto-mode off no pass is scheduled" (fake timers).
  - Green: the initial quiet pass, the DOM-mutation trigger, and debounce in `src/content-script.ts`, gated on `GetAutoMode` returning on.
  - Verify: `npm run check`.
- Decision log:
- Outcome:

### T36b: Debounced auto-mode triggers for later steps (URL change path)

- Status: [ ] Pending
- Traces to: WHEN auto-mode is on and the user advances to another step of a job application THE SYSTEM SHALL fill fields on that step that have no value and that map cleanly to stored profile facts without requiring another click; WHEN auto-mode is on and the current tab is a job application THE SYSTEM SHALL fill fields that have no value and that map cleanly to stored profile facts without requiring a further click
- Depends on: T36a
- Objective: `src/content-script.ts` detects URL changes by polling `location.href` every 500 ms and by listening for `popstate` and `hashchange`. Each of these schedules the same debounced pass as T36a. A URL change together with a DOM mutation produces one pass.
- TDD plan:
  - Red: `tests/auto-trigger.test.ts` "a pushState href change seen by 500 ms polling, a popstate event, and a hashchange event each schedule the same debounced pass, and a URL change with a DOM mutation runs one pass" (fake timers).
  - Green: the URL polling and event listeners in `src/content-script.ts`, reusing the debounce from T36a.
  - Verify: `npm run check`.
- Decision log:
- Outcome:

### T37a: Service worker relays FillTab

- Status: [ ] Pending
- Traces to: none (wiring for T37b; no acceptance criterion of its own)
- Depends on: T4
- Objective: The service worker handles `FillTab { tabId }` by sending a fill message with trigger `icon` to that tab's content script. The service worker does not read the profile and does not decide whether the page is a job application.
- TDD plan:
  - Red: `tests/fill-tab-relay.test.ts` "FillTab for tab 7 sends one fill message with trigger icon to tab 7 through chrome.tabs.sendMessage". It fails because the handler does not exist.
  - Green: the `FillTab` handler in `src/service-worker.ts`.
  - Verify: `npm run check`.
- Decision log:
- Outcome:

### T37b: Content script runs the icon-trigger fill for job applications only

- Status: [ ] Pending
- Traces to: WHEN the user clicks the extension icon and the current tab is a job application (a page with at least two personal-data fields and at least one job-application-specific field, and no password or payment-card fields) THE SYSTEM SHALL fill that tab’s fields that have no value and that map cleanly to stored profile facts; WHEN the user clicks the extension icon and the current tab is not a job application THE SYSTEM SHALL not fill any fields on that tab
- Depends on: T32, T33b, T37a
- Objective: `src/content-script.ts` listens for the fill message from T37a and runs the gate and the fill pass for trigger `icon`. On a tab that is not a job application it changes no field.
- TDD plan:
  - Red: `tests/icon-fill-listener.test.ts` "the content script with trigger icon runs the pass on a job application, and changes no field on a non-application tab".
  - Green: the message listener in `src/content-script.ts`.
  - Verify: `npm run check`.
- Decision log:
- Outcome:

### T38: Popup open turns auto-mode on and requests a fill

- Status: [ ] Pending
- Traces to: WHEN the user clicks the extension icon THE SYSTEM SHALL turn auto-mode on
- Depends on: T37a, T37b
- Objective: Opening the popup (the toolbar icon click under `default_popup`) sends `SetAutoMode { on: true }` and then `FillTab` for the active tab. Opening the popup never sends `on: false`.
- TDD plan:
  - Red: `tests/popup-open.test.ts` "opening the popup sends SetAutoMode on and then FillTab for the active tab, and never sends SetAutoMode off".
  - Green: the open handler in `src/popup.ts`.
  - Verify: `npm run check`.
- Decision log:
- Outcome:

### T39: Popup on/off control without fill

- Status: [ ] Pending
- Traces to: WHEN the user opens the extension popup THE SYSTEM SHALL present a control that turns auto-mode on or off; WHEN the user turns auto-mode off from the popup THE SYSTEM SHALL leave auto-mode off and not fill fields until auto-mode is turned on again
- Depends on: T38
- Objective: The popup shows an on/off control that is on after every open. Toggling it off sends `SetAutoMode { on: false }` and sends no `FillTab`. Toggling it on sends `SetAutoMode { on: true }` and sends no `FillTab`. The control shows off after an off toggle in the same open.
- TDD plan:
  - Red: `tests/popup-control.test.ts` "the control shows on after open, toggling it off sends SetAutoMode off and no FillTab, and toggling it on sends SetAutoMode on and no FillTab".
  - Green: the control in `src/popup.html` and `src/popup.ts`.
  - Verify: `npm run check`.
- Decision log:
- Outcome:

### T40: Attach the chosen resume to an empty resume/CV upload

- Status: [ ] Pending
- Traces to: WHEN a fill runs, the user has chosen a resume file, that file is available, and the page has a resume or CV upload field that has no file THE SYSTEM SHALL attach that file to the upload field; WHEN a later fill runs from an icon click or from auto-mode and some fields already have values THE SYSTEM SHALL fill remaining fields that have no value and that map cleanly to stored profile facts, including attaching the chosen resume file to a resume or CV upload that still has no file
- Depends on: T7a, T10b, T33b
- Objective: `attachResume(snapshot, resume)` builds a `File` from the stored bytes and sets it on an empty resume or CV file input through `DataTransfer.files`, then dispatches `change`. The step runs inside the fill pass after the other writes. The resume/CV cue set is `resume`, `resume/cv`, `cv`, and `curriculum vitae`.
- TDD plan:
  - Red: `tests/resume-attach.test.ts` "an empty file input labelled 'Resume/CV' gets a File with the stored name, MIME type, and bytes through a DataTransfer stub, and change is dispatched"; "a snapshot with a valued text field and an empty file input labelled Resume/CV gets the stored File through attachResume, and the valued text field is unchanged".
  - Green: `src/mappers/resume.ts` and the call in `src/fill-pass.ts`.
  - Verify: `npm run check`.
- Decision log:
- Outcome:

### T41: Resume attach negatives

- Status: [ ] Pending
- Traces to: WHEN a fill runs and the page has no resume or CV upload field THE SYSTEM SHALL not attach a file; WHEN a fill runs and the page has a file upload that is not a resume or CV upload THE SYSTEM SHALL not attach the resume file to that upload; WHEN a fill runs and a resume or CV upload field already has a file THE SYSTEM SHALL leave that upload field unchanged; WHEN a fill runs, the page has a resume or CV upload field, and the user has not chosen a resume file THE SYSTEM SHALL leave the upload field unchanged; WHEN a fill runs, the page has a resume or CV upload field, the user has chosen a resume file, and that file is not available THE SYSTEM SHALL leave the upload field unchanged
- Depends on: T40
- Objective: `attachResume` attaches nothing when there is no resume or CV upload, when the upload is a non-resume file input (for example "Cover letter"), when the upload already has a file, when no resume is stored, or when the stored bytes are unreadable.
- TDD plan:
  - Red: `tests/resume-negatives.test.ts` "with no resume upload, a 'Cover letter' upload, an upload that already has a file, no stored resume, and unreadable stored bytes, the DataTransfer stub records zero assignments".
  - Green: the guards in `src/mappers/resume.ts`.
  - Verify: `npm run check`.
- Decision log:
- Outcome:

### T42a: Manual browser checklist in Chrome

- Status: [ ] Pending
- Traces to: WHEN the extension runs THE SYSTEM SHALL not send profile contents off the user’s machine, including through analytics, crash reports, or other remote calls that include profile contents; WHEN the user loads the extension from a local folder into Chrome THE SYSTEM SHALL make the options page, the auto-mode control, and filling available in Chrome; WHEN the profile is saved THE SYSTEM SHALL keep the stored profile on the user’s machine so that web pages cannot read it; WHEN the profile is saved THE SYSTEM SHALL keep the stored profile so that other extensions cannot read it
- Depends on: T2, T9, T33b, T36a, T36b, T39, T41
- Objective: Run steps 1 to 7 and steps 9 to 11 of design.md Checks and manual checklist in Chrome against the built `dist/` folder. Record each step's observed result in Outcome. Step 1 loads with no errors. Step 3 leaves Submit, Next, Continue, and agree-to-terms untouched. Step 6 attaches the chosen resume only to an empty resume/CV upload and not to other file uploads. Step 7 shows auto-mode off after restart and the profile still present. Step 9 shows no request carrying profile contents. Step 10 shows no `externally_connectable` and no `web_accessible_resources` in the built manifest. Step 11 shows that in the page's DevTools console `chrome.storage` is not available to page scripts and the profile key cannot be read from the page, with the console output recorded in Outcome. Step 8 is run against both browsers in T42b.
- TDD plan:
  - Red: the manual checklist from design.md (Checks and manual checklist, steps 1 to 11) has no recorded results in Outcome, so the task is not done. Run `npm run check` first; it must pass on the current `dist/`.
  - Green: fix any failing step by returning to the task that owns it, then update the Decision log.
  - Verify: `npm run check`, then the manual checklist in Chrome, with each step's result recorded in Outcome.
- Decision log:
- Outcome:

### T42b: Manual browser checklist in Opera GX

- Status: [ ] Pending
- Traces to: WHEN the user saves a profile in Chrome THE SYSTEM SHALL keep that profile only in Chrome and shall not make it available in Opera GX; WHEN the user saves a profile in Opera GX THE SYSTEM SHALL keep that profile only in Opera GX and shall not make it available in Chrome; WHEN the user loads the extension from a local folder into Opera GX THE SYSTEM SHALL make the options page, the auto-mode control, and filling available in Opera GX
- Depends on: T42a
- Objective: Run steps 1 to 7 of design.md Checks and manual checklist in Opera GX against the built `dist/` folder, then run step 8 against both browsers, after T42a. Record each step's observed result in Outcome. Step 8 shows that a profile saved in one browser is absent in the other.
- TDD plan:
  - Red: the manual checklist from design.md (Checks and manual checklist, steps 1 to 7 in Opera GX plus step 8 against both browsers) has no recorded results in Outcome, so the task is not done. Run `npm run check` first; it must pass on the current `dist/`.
  - Green: fix any failing step by returning to the task that owns it, then update the Decision log.
  - Verify: `npm run check`, then the manual checklist in Opera GX (steps 1 to 7) and step 8 against both browsers, with each step's result recorded in Outcome.
- Decision log:
- Outcome:

## Open items

- `package.json` **has only a** `typecheck` **script.** `design.md` fixes `npm run check` as the verify command. T1a adds the `test` script and the `vitest` and `jsdom` devDependencies. T1b adds the `build` and `check` scripts and the `esbuild` devDependency. npm is the package manager per design.md (Assumptions), so no decision is needed before T1a starts.
- **Full-address separator: resolved.** `design.md` Formats now says the street line is followed by one space, then city, region, and postal code, each separated by one space. T20a pins that format. No decision needed.
- **Settings forms: resolved by the requirements.** `requirements.md` (Edge cases) states that a settings form with at least two personal-data fields and at least one job-application field, and no password or payment-card field, is treated as a job application in version 1. T11 implements that rule. No decision needed.
- **Payment-card detection: resolved by the design.** `design.md` (Assumptions, Payment-card and password fields) defines the payment-card cues. T11 uses that definition. No decision needed.
- **Custom dropdowns are best-effort: resolved by the requirements.** `requirements.md` (Keep judgment) states that an ARIA dropdown is attempted by exact option match and that some sites will leave the field unchanged in version 1. T26 follows that wording. No decision needed.
- **Browser-level rows are manual: specified by design.** Load, popup, restart persistence, cross-browser isolation, and "web pages and other extensions cannot read the profile" cannot be proven by unit tests. `design.md` (Checks and manual checklist) specifies that these are verified by the manual checklist in T42a and T42b, with T1a covering the manifest guard. T42a's and T42b's Red steps are manual observations rather than automated failing tests, as `design.md` specifies. No decision needed: `design.md` specifies this.
- **Web-page read criterion: covered by T42a step 11.** The criterion "WHEN the profile is saved THE SYSTEM SHALL keep the stored profile on the user’s machine so that web pages cannot read it" is mapped to T42a only. T42a step 11 (per `design.md` Checks and manual checklist) is the named check: the page's DevTools console must show `chrome.storage` unavailable to page scripts and the profile key unreadable from the page. No decision needed.
- **DataTransfer on file inputs is unverified.** T40 uses a stub. Real attachment in Chrome and Opera GX is confirmed only in T42a step 6 and, for Opera GX, in T42b. This follows `design.md` Assumptions.
- **Synthesis notes (round 1).** T6 is split into T6a (save pieces, no derived copy) and T6b (blank middle name omitted; second save replaces). T7 is split into T7a (resume store) and T7b (options file picker). T10 is split into T10a (label sources) and T10b (emptiness and cue normalization). T14 is split into T14a (one-field composition, with the preferred, legal, and passport exclusion; the former T14b is merged into it). T20 is split into T20a (full address) and T20b (city, region, postal code, country). T36 is split into T36a (DOM mutation path) and T36b (URL change path). T34 is merged into T33a. T33b is a separate task for fills that use user-added facts saved earlier, split out of T33a. T35a was merged into T33a in an earlier round. T42 is split into T42a (Chrome) and T42b (Opera GX), with step 8 run against both browsers in T42b. T11 gains two negative classifier tests. T22 gains a `type="date"` test. T27 uses a separate value-or-text matcher. T24 and T25 depend on T10b, and T40 and T41 depend on the resume store and snapshot. T40 gains a second Red test for a valued text field alongside the empty file input. T1a and T1b point their browser-check references at T42a and T42b. T1b and T37a carry no browser-loading or icon-click criterion of their own; those criteria are traced to T42a, T42b, and T37b. The Chrome isolation criterion is covered by T42a and T42b.
- **Synthesis notes (round 1, evaluation merge).** T10a's snapshot also holds `autocomplete` on each entry, which the T11 payment-card rule needs. The proposed T33b rename to T33d is not applied: it depended on a task T33c that does not exist, so T33b keeps its id. References to the nonexistent T33c are removed from T36a (now depends on T4 and T33b), T37b (now T32, T33b, T37a), T40 (now T7a, T10b, T33b), and T42a (now T2, T9, T33b, T36a, T36b, T39, T41). T42a's Traces to now include the not-send-profile criterion; the Chrome-isolation criterion maps to T42b only, since step 8 runs in T42b.

