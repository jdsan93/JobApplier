# Application autofill — Design

## Overview

This feature is a Manifest V3 browser extension written in TypeScript. One unpacked package is loaded into Chrome and, separately, into Opera GX. Each browser keeps its own `chrome.storage.local` profile (unencrypted, no passphrase). The options page is where built-in pieces and user-added facts are entered. Clicking the toolbar icon turns auto-mode on and fills the current tab only when that tab is a job application. While auto-mode stays on, later application pages and steps are filled without another click. A fill writes stored facts into fields that have no value; it never overwrites a field that already has a value, never invents text, never fills or activates an agree-to-terms control, and never activates Submit, Next, or Continue. The page never reads the stored profile; it only sees values written into its own controls.

The repo has no product code yet. The parts below are the extension itself: options page, popup, service worker, content script, profile store, and fill engine.

## Architecture

One extension package serves both browsers. Storage is not shared between them because each browser has its own extension storage area.

| Component | Responsibility |
| --- | --- |
| Manifest V3 package | Declares the service worker, options page, popup, content script, and host access needed to run on application pages. Loaded unpacked from a local folder in Chrome and in Opera GX. No remote endpoints. |
| Options page | View, enter, and save built-in profile pieces and one resume file, and shows the saved resume file name. Hosts a basic, non-stylized add-fact menu (fact value plus the question strings it maps to). Reads and writes only this browser’s `chrome.storage.local`. |
| Popup | Shown when the user clicks the toolbar icon. On open, turns auto-mode on and asks the content script to fill the current tab if it is a job application. Presents an on/off control; turning it off is the only user action that ends auto-mode before a new browser session. |
| Service worker | Owns auto-mode in session-scoped storage so a new browser session starts with auto-mode off while the saved profile remains. Relays fill requests. Does not send profile contents off the machine. |
| Content script (isolated world) | Runs on the tab. Classifies the page as a job application or not. Collects field cues. Maps empty fields to stored facts. Writes values as if typed or selected. Attaches the resume only to an empty resume/CV upload. Never clicks submit/advance controls. |
| Profile store | `chrome.storage.local` JSON of built-in pieces and user-added facts. Isolated from web pages and from other extensions. No passphrase. No second derived copy of a fact (no stored full-name string alongside first/middle/last). |
| Resume store | The chosen resume file kept locally next to the profile in this browser only. The file name, MIME type, and base64 bytes are stored together under the `resume` key of `chrome.storage.local`; nothing is sent off the machine. |
| Fill engine | Pure mapping and compose/split used by the content script: name by field count and each field’s own cue (or exact English/Spanish labels when there are four or more), phone join/split, address street-numbers/details/localidad, date of birth as one field or month/day/year in the field’s format, user-added exact cue match, ambiguity → leave unchanged, skip fields that already have a value. |

Web pages, other extensions, analytics, and crash reporters are not components of this feature. The content script does not use the page’s `localStorage`.

## Data flow

Main success path: the user has a saved profile in this browser, clicks the toolbar icon on a job application, then continues to later application steps with auto-mode on.

```mermaid
sequenceDiagram
    actor User
    participant Icon as Toolbar icon / popup
    participant SW as Service worker
    participant Store as chrome.storage.local
    participant Session as Session storage
    participant CS as Content script
    participant Page as Application page

    User->>Icon: Click extension icon
    Icon->>SW: Popup opened
    SW->>Session: Set auto-mode on
    Icon-->>User: Show auto-mode on/off control
    SW->>CS: Fill current tab
    CS->>Page: Classify page (job application or not)
    alt Tab is not a job application
        CS-->>Page: Do not change any fields
    else Tab is a job application
        CS->>Store: Read profile and resume in isolated world
        CS->>Page: Collect fields and cues
        CS->>CS: Map empty fields; skip valued, ambiguous, and unmapped
        CS->>Page: Write stored values; attach resume if upload empty
        Note over CS,Page: Do not activate Submit, Next, Continue, or agree-to-terms
    end

    User->>Page: Advance to next application step
    Page->>CS: DOM or URL changes (new step)
    CS->>Session: Auto-mode still on?
    CS->>Page: Classify; fill remaining empty mapped fields only
```

Failure and boundary paths (same components, no extra UI):

- Auto-mode off, or a new browser session (session storage empty): the content script does not fill.
- Login, checkout, or settings that is not a job application: classify as not an application; leave every field unchanged, including on icon click.
- Field already has a value, including a resume/CV upload that already has a file: skip that control; still fill other empty mapped fields.
- Missing or ambiguous mapping, would-invent, open-ended with no mapped fact, dual built-in and user-added match: leave that field unchanged.
- No stored facts, or a needed piece (middle name, street numbers, resume file) is missing or unavailable: leave those fields unchanged; fill other empty mapped fields.
- Profile saved in the other browser: this browser’s store does not contain it; fills use only this browser’s profile.

Save path (options page only):

```mermaid
sequenceDiagram
    actor User
    participant Options as Options page
    participant Store as chrome.storage.local

    User->>Options: Enter built-in pieces and/or add-fact menu
    User->>Options: Save
    Options->>Store: Write pieces and user-added facts (no passphrase, no derived duplicates)
    Note over Store: Visible only inside this browser’s extension storage
```

## Interfaces

No network API. Contracts are the extension surfaces and the in-extension messages.

### Options page

- **Opens:** browser extension options UI.
- **Inputs:** built-in piece fields listed in the requirements; one resume file picker; add-fact menu with a fact value and one or more question strings.
- **Outputs:** persist to `chrome.storage.local` in this browser. After save, later fills in this browser use the new values.
- **Satisfies:** view/enter/save built-in facts; pieces not duplicates; no passphrase; add-fact menu; user-added storage; unpacked load in Chrome and Opera GX.

The add-fact menu is basic HTML (plain inputs, add/remove questions, save). It is not a stylized design system.

### Popup

- **Opens:** toolbar icon click (`default_popup`). In Manifest V3 that click opens the popup rather than `action.onClicked`, so popup-open **is** the icon click.
- **On open:** set auto-mode on; if the current tab is a job application, run a fill of empty mapped fields (including an empty resume/CV upload).
- **Control:** a toggle or equivalent that turns auto-mode on or off. Switching it only changes auto-mode; it does not fill. Off does not fill until auto-mode is on again. Icon click never turns auto-mode off.
- **Satisfies:** icon click turns auto-mode on; icon click fills only job applications; popup on/off; later icon click still fills remaining blanks.

### Service worker messages

- `SetAutoMode { on: boolean }` — popup writes session-scoped auto-mode.
- `GetAutoMode` → `{ on: boolean }` — missing session value means off (new browser session).
- `FillTab { tabId }` — popup or auto-mode trigger; content script decides whether the page is a job application and which empty fields to write.
- Profile and resume are read with `chrome.storage.*` from extension pages and the content script’s isolated world, not from page script.

### Content script fill entry

- **Input:** current DOM, this browser’s stored profile, auto-mode or explicit fill command.
- **Output:** in-page control values changed only for empty mapped fields; no navigation/submit activation.
- **Triggers:** the content script runs at `document_idle`, so it starts after the page has loaded. There is no separate DOM-ready signal; the DOM observer below catches forms that render late. When auto-mode is on (asked from the service worker with `GetAutoMode`), the script fills once the page has been quiet for about 500 ms. Later steps are found by DOM mutations and by URL changes. The isolated world cannot see the page’s own history calls, so URL changes are found by polling `location.href` every 500 ms and by listening for `popstate` and `hashchange`. Every trigger is debounced into one pass.
- **Satisfies:** identification from cues; compose/split; user-added exact cue match; no-overwrite; never submit.

### Storage keys (this browser only)

- `profile` — built-in pieces and `userAdded` list.
- `resume` — file name, MIME type, and bytes (or a pointer to the local resume record).
- Auto-mode is **not** in `chrome.storage.local`, so a browser restart does not restore it as on.

## Data models

Entities live only in this browser’s extension storage. Nothing expires except auto-mode, which is session-scoped and treated as off when absent.

### Profile

Stored as pieces, not as display strings that can drift.

| Field | Shape | Notes |
| --- | --- | --- |
| `name.first` | string | Required for name fills that need first/name. |
| `name.middle` | string, optional | Absent means middle-name fields stay unchanged. |
| `name.last` | string | |
| `dateOfBirth` | `{ year, month, day }` | Only built-in date. Composed to one field or split to month/day/year. Not stored again as a locale string. |
| `email` | string | |
| `phone.countryCode` | string | Written only into a country-code field. Never joined into a phone field. |
| `phone.nationalNumber` | string | |
| `address.streetNumbers` | string | Combined with details for a single street/address-line. |
| `address.details` | string | Apartment number or similar. |
| `address.city` | string | |
| `address.region` | string | State/region. |
| `address.postalCode` | string | |
| `address.country` | string | |
| `address.locality` | string | Neighbourhood / localidad; not included in the full-address compose that uses the other address pieces. |
| `links.linkedin` | string | URL. |
| `links.github` | string | URL. |
| `links.portfolio` | string | Portfolio or personal-site URL. |
| `usWorkAuthorization` | stored choice | Includes a decline-to-answer value if the user saved one. |
| `usSponsorshipNeeded` | stored choice | US only. |
| `eeo.gender` | stored choice | Each EEO fact may be a decline-to-answer choice. |
| `eeo.raceEthnicity` | stored choice | |
| `eeo.veteran` | stored choice | |
| `eeo.disability` | stored choice | |
| `howHeard` | string | One canned choice value. Not a free-text generator. |
| `resume` | `{ fileName, mimeType }` plus local bytes | One file. Absent or unreadable means resume uploads stay unchanged. |
| `userAdded` | `UserAddedFact[]` | Empty list is valid; built-in fills still run. |

There is no `fullName`, `fullPhone`, or `fullAddress` stored field. Compose happens at fill time from pieces.

### UserAddedFact

| Field | Shape | Notes |
| --- | --- | --- |
| `id` | string | Stable within this browser’s profile. |
| `value` | string | Reused as typed/selected text. Never paraphrased. |
| `questions` | string[] | Each string is a question the user recorded. A field matches when its **label**, **accessible name**, or **placeholder** equals one of these strings. |

### AutoMode

| Field | Shape | Lifetime |
| --- | --- | --- |
| `on` | boolean | Session only. Absent at browser start ⇒ off. Cleared by a new browser session. Turned on by icon click (popup open) or by the popup control. Turned off only by the popup control. |

### Page field snapshot (ephemeral)

Built at fill time, not stored:

- Control reference, current value emptiness, cues (`label`, `accessibleName`, `placeholder`, plus similar cues for **identification** and **built-in** mapping only).
- Name-field membership, excluding preferred-name, legal-name, and name-as-on-passport fields from the 1/2/3/4+ count.
- Input type, placeholder, label, and maxlength, read in the same pass as the control reference. This is defined here, so the name, phone, and date tasks depend on the snapshot, not on the fill tasks.
- For four or more name fields, the label is compared after trimming and ASCII case-folding.

## Error handling

Observable result in every case is: leave the affected control unchanged, and still fill other empty mapped fields when the fill is allowed to run.

| Condition | Observable result |
| --- | --- |
| Field already has a value (text, selection, or file) | That control is not written. Remaining empty mapped fields may still be filled, including an empty resume/CV upload. |
| Resume/CV upload already has a file | Upload left unchanged. |
| Page is not a job application (including login, checkout, settings) | No fields filled, even on icon click, even if auto-mode is on. |
| Auto-mode off and no icon click | No fields filled, including on a job application, until auto-mode is on. |
| New browser session | Auto-mode off. Saved profile in **this** browser is still present. |
| No stored facts | Every field left unchanged. |
| Only some facts stored; a piece is missing (middle name, street numbers, details, localidad, resume) | Empty fields that map cleanly to stored pieces are filled; fields that need the missing piece stay unchanged. |
| Resume chosen but bytes not available | Resume/CV upload left unchanged. |
| Mapping missing, ambiguous, or both built-in and user-added | Field unchanged. |
| Would require inventing or paraphrasing text | Field unchanged. |
| Open-ended original writing with no mapped user-added question | Field unchanged. |
| User-added fact whose recorded questions are not the field’s label, accessible name, or placeholder | That fact is not used for that field. |
| Preferred / legal / passport name | Not counted in name-field totals; left unchanged unless stored as its own (user-added) fact. |
| Date other than date of birth with no mapped fact | Unchanged. |
| Free-text “how did you hear about us” with no mapped fact | Unchanged. |
| Work authorization or sponsorship for a non-US country with no mapped fact | Unchanged. |
| Four or more name fields with a label outside the six exact English/Spanish strings | That field is not filled from first, middle, or last. |
| Profile saved in the other browser | This browser does not show or fill from that profile. |
| Widget (dropdown) needed to set a stored choice | Set the choice through the widget; do not activate submit/advance/agree. |
| Two or three name fields, and a field’s own cue does not say first, middle, or last name | That field stays unchanged; fields whose cue does say it are still filled. |
| Date field whose expected format cannot be determined from the page | Unchanged. |
| Stored choice has no exactly matching option in a dropdown or radio group | Unchanged. |
| Agree-to-terms checkbox or control, including one a user-added fact would match | Never filled and never activated. |

A fill never treats a failed mapping as a reason to click Continue or Submit.

## Testing strategy

A **property** must hold for any input that matches the WHEN clause. An **example** pins one concrete situation (field counts, exact labels, a login page). This section plans checks; it is not test code and not a task list.

| Criterion | Kind of test | What is asserted |
| --- | --- | --- |
| WHEN the user opens the extension’s options page THE SYSTEM SHALL let the user view, enter, and save the listed built-in profile facts | Example | Options page exposes each listed built-in fact, including optional middle name, EEO decline-to-answer, canned how-heard, and one resume file, and a save persists them. |
| WHEN the user saves name, phone, address, or date of birth THE SYSTEM SHALL store that fact as the pieces listed above so a later fill can compose or split it to match the page | Property | For any saved name, phone, address, or date of birth, storage contains only the pieces, and a later fill can compose or split from those pieces. |
| WHEN the user saves a profile fact THE SYSTEM SHALL not store a second copy of that fact that can disagree with the saved pieces | Property | For any saved fact, no derived duplicate (full name, full phone, full address string) exists that could disagree with the pieces. |
| WHEN the user changes a stored fact on the options page and saves THE SYSTEM SHALL use the updated fact on fills after that save | Property | For any fact edited and saved, fills after that save use the new value and not the previous one. |
| WHEN the user starts a new session in the same browser after saving a profile THE SYSTEM SHALL still have those saved facts available for filling | Property | For any profile saved in a browser, a new session of that same browser can still fill from it. |
| WHEN the user saves the profile THE SYSTEM SHALL save and later reuse it without requiring a passphrase | Property | Save and later fill succeed with no passphrase prompt or check. |
| WHEN the user saves a profile in Chrome THE SYSTEM SHALL keep that profile only in Chrome and shall not make it available in Opera GX | Property | For any profile saved in Chrome, Opera GX does not contain, show, or fill from it. |
| WHEN the user saves a profile in Opera GX THE SYSTEM SHALL keep that profile only in Opera GX and shall not make it available in Chrome | Property | For any profile saved in Opera GX, Chrome does not contain, show, or fill from it. |
| WHEN the user opens the extension’s options page THE SYSTEM SHALL present a basic, non-stylized menu to add a fact and the question or questions that fact maps to | Example | Options page shows a plain add-fact menu that accepts a value and one or more questions. |
| WHEN the user adds a fact and the questions it maps to and saves THE SYSTEM SHALL store that fact and those mappings with the profile and use them on fills after that save | Property | For any saved user-added fact and questions, later fills in that browser can use them. |
| WHEN a fill runs and a field has no value and that field’s label, accessible name, or placeholder is a question the user recorded for a stored user-added fact THE SYSTEM SHALL write that stored value into the field as if the user had typed or selected it | Property | For any empty field whose label, accessible name, or placeholder equals a recorded question, the stored value is written as a user edit. |
| WHEN a fill runs and a field’s label, accessible name, and placeholder are not the question the user recorded for a stored user-added fact THE SYSTEM SHALL not fill that field from that user-added fact | Property | For any field whose three cues are not the recorded question, that user-added fact is not written. |
| WHEN the user has not added any extra facts THE SYSTEM SHALL still fill from the built-in facts | Property | Built-in empty-field fills run when `userAdded` is empty. |
| WHEN the user clicks the extension icon THE SYSTEM SHALL turn auto-mode on | Property | Every icon click results in auto-mode on. |
| WHEN the user clicks the extension icon and the current tab is a job application THE SYSTEM SHALL fill that tab’s fields that have no value and that map cleanly to stored profile facts | Property | For any job-application tab, icon click fills empty mapped fields only. |
| WHEN the user clicks the extension icon and the current tab is not a job application THE SYSTEM SHALL not fill any fields on that tab | Property | For any non-application tab, icon click changes no fields. |
| WHEN auto-mode is on THE SYSTEM SHALL keep auto-mode on until the user turns it off from the popup or the browser starts a new session | Property | Auto-mode remains on across tab changes until popup-off or a new session. |
| WHEN the browser starts a new session THE SYSTEM SHALL set auto-mode off | Property | Every new session starts with auto-mode off. |
| WHEN auto-mode is off and the user has not clicked the extension icon THE SYSTEM SHALL not fill any fields | Property | No fill runs while auto-mode is off (except the icon-click path, which turns it on first). |
| WHEN the user opens the extension popup THE SYSTEM SHALL present a control that turns auto-mode on or off | Example | Popup shows an on/off control that changes auto-mode. |
| WHEN the user turns auto-mode off from the popup THE SYSTEM SHALL leave auto-mode off and not fill fields until auto-mode is turned on again | Property | After popup-off, no fills occur until a later on. |
| WHEN auto-mode is on and the current tab is a job application THE SYSTEM SHALL fill fields that have no value and that map cleanly to stored profile facts without requiring a further click | Property | Any job-application tab seen while auto-mode is on is filled on empty mapped fields without another click. |
| WHEN auto-mode is on and the current tab is not a job application THE SYSTEM SHALL not fill any fields | Property | Non-application tabs are untouched while auto-mode is on. |
| WHEN auto-mode is on and the user advances to another step of a job application THE SYSTEM SHALL fill fields on that step that have no value and that map cleanly to stored profile facts without requiring another click | Property | Each later application step, found by a DOM or URL change, is filled on empty mapped fields without another click. |
| WHEN a fill runs THE SYSTEM SHALL identify fields from the page’s labels, accessible names, placeholders, and similar cues | Property | Field discovery uses those cues on any filled page. |
| WHEN a fill runs on a job application hosted by a job board the user did not name or configure THE SYSTEM SHALL still fill fields that have no value and that map cleanly to stored facts | Property | Clean empty-field fills do not require a configured board name. |
| WHEN a fill runs and a field has no value and maps cleanly to a stored profile fact THE SYSTEM SHALL write that stored value into the field as if the user had typed or selected it, so the page treats the field as filled | Property | A clean empty mapping is written with user-like input/change so the page treats it as filled. |
| WHEN a fill runs and a choice field has no value and maps cleanly to stored US work authorization, sponsorship, an EEO fact, or the canned “how did you hear about us” value THE SYSTEM SHALL set that choice as if the user had selected it | Property | Empty matching choice controls receive the stored choice as a user selection. |
| WHEN a fill runs and the page presents exactly one field asking for the applicant’s first, middle, last, or full name THE SYSTEM SHALL fill that field if it has no value: with the composed stored name, including middle name only if it is stored, when it asks for the full name, or with the matching piece when it asks for first, middle, or last name | Example | A full-name field gets composed first + optional middle + last; a first-name-only field gets the first name; a valued field is left. |
| WHEN a fill runs and the page presents exactly two fields asking for the applicant’s first, middle, last, or full name THE SYSTEM SHALL fill each of those fields that has no value and whose own cue says first name or last name with that stored piece; a field whose cue says neither stays unchanged | Example | Two name fields: each empty field whose cue says first or last gets that piece; two fields with no such cue are both left unchanged. |
| WHEN a fill runs and the page presents exactly two fields asking for the applicant’s first, middle, last, or full name THE SYSTEM SHALL not write a composed full name into either field | Property | For any two-name-field page, neither field receives the composed full name. |
| WHEN a fill runs and the page presents exactly three fields asking for the applicant’s first, middle, last, or full name THE SYSTEM SHALL fill each of those fields that has no value and whose own cue says first name, middle name, or last name with that stored piece; a field whose cue does not say which piece it is stays unchanged | Example | Three name fields: each empty field gets the piece its own cue names (first, middle, or last); a field with no such cue is left unchanged. |
| WHEN a fill runs and the page presents four or more fields asking for the applicant’s first, middle, last, or full name THE SYSTEM SHALL not use the one-, two-, or three-field name fills | Property | For any page with four or more name fields, the 1/2/3 compose/split strategies are not used. |
| WHEN a fill runs, the page presents four or more name fields, and a field’s label is exactly `first name` or `primer nombre` THE SYSTEM SHALL fill that field from the stored first name if that field has no value | Example | Empty field labeled exactly those strings gets first name. |
| WHEN a fill runs, the page presents four or more name fields, and a field’s label is exactly `last name` or `apellido` THE SYSTEM SHALL fill that field from the stored last name if that field has no value | Example | Empty field labeled exactly those strings gets last name. |
| WHEN a fill runs, the page presents four or more name fields, and a field’s label is exactly `middle name` or `segundo nombre` THE SYSTEM SHALL fill that field from the stored middle name if that field has no value | Example | Empty field labeled exactly those strings gets middle name when stored. |
| WHEN a fill runs, the page presents four or more name fields, and a field’s label is not exactly those six strings THE SYSTEM SHALL not fill that field from the stored first, middle, or last name | Property | Any 4+ name field whose label is not one of the six strings is not filled from name pieces. |
| WHEN a fill runs and the page has a single phone field that has no value and no separate country-code field THE SYSTEM SHALL leave that field unchanged, and SHALL NOT join the stored country code into it | Property | A single phone field with no country-code field on the page is never written. |
| WHEN a fill runs and the page has a country-code field and a phone field that has no value THE SYSTEM SHALL fill the phone field with the stored national number | Property | A phone field on a page with a country-code field gets only the national number. |
| WHEN a fill runs and the page has a country-code field and/or a national-number field THE SYSTEM SHALL fill each of those fields that has no value from the matching stored piece | Property | Each empty present phone piece field gets only its matching piece. |
| WHEN a fill runs and the page has a single street or address-line field that has no value THE SYSTEM SHALL fill that field by composing the stored street-numbers piece and the stored address-details piece that exist | Property | Empty single street/address-line is composed from whichever of those two pieces exist. |
| WHEN a fill runs and the page has a street-numbers field and/or an address-details field THE SYSTEM SHALL fill each of those fields that has no value from the matching stored piece | Property | Each empty present street-numbers or details field gets only its piece. |
| WHEN a fill runs and the page has a neighbourhood or localidad field that has no value and neighbourhood / localidad is stored THE SYSTEM SHALL fill that field from the stored neighbourhood / localidad | Property | Empty neighbourhood/localidad fields get the stored locality piece. |
| WHEN a fill runs and the page has a single full-address field (not a street-only or address-line field) that has no value and that maps cleanly to stored street numbers, address details, city, state/region, or postal code THE SYSTEM SHALL fill that field by composing those stored pieces, writing only the stored parts when some are missing | Property | Empty full-address fields are composed from non-locality address pieces only, writing only the stored parts when some are missing. |
| WHEN a fill runs and the page has a city, state/region, postal code, and/or country field THE SYSTEM SHALL fill each of those fields that has no value from the matching stored piece | Property | Each empty present city/region/postal/country field gets its piece. |
| WHEN a fill runs and the page has a single date-of-birth field that has no value and date of birth is stored THE SYSTEM SHALL fill that field from the stored date of birth, written in the date format that field asks for (month and day zero-padded, and a two-digit year where the field asks for one) | Property | Empty single DOB fields get the stored date in the field’s format; an undetermined format leaves the field unchanged. |
| WHEN a fill runs and the page has a month, day, and/or year field for date of birth and date of birth is stored THE SYSTEM SHALL fill each of those fields that has no value from the stored date of birth, with a zero-padded month and day and a two-digit year where that field asks for one | Property | Each empty present DOB part field gets the matching part, month and day zero-padded. |
| WHEN a fill runs and a date-of-birth field’s expected format cannot be determined from the page THE SYSTEM SHALL leave that field unchanged | Property | Undetermined date formats are never guessed. |
| WHEN a fill runs, the user has chosen a resume file, that file is available, and the page has a resume or CV upload field that has no file THE SYSTEM SHALL attach that file to the upload field | Property | Any empty resume/CV upload receives the chosen available file. |
| WHEN a fill runs and the page has no resume or CV upload field THE SYSTEM SHALL not attach a file | Property | No file is attached when no resume/CV upload exists. |
| WHEN a fill runs and the page has a file upload that is not a resume or CV upload THE SYSTEM SHALL not attach the resume file to that upload | Property | Non-resume uploads never receive the resume file. |
| WHEN a fill runs and a resume or CV upload field already has a file THE SYSTEM SHALL leave that upload field unchanged | Property | A resume/CV upload that already has a file is never replaced. |
| WHEN a fill runs and a field already has a value THE SYSTEM SHALL leave that field unchanged | Property | For any field with a value, a fill does not change it. |
| WHEN a later fill runs from an icon click or from auto-mode and some fields already have values THE SYSTEM SHALL fill remaining fields that have no value and that map cleanly to stored profile facts, including attaching the chosen resume file to a resume or CV upload that still has no file | Property | A later fill completes remaining blanks only, including an empty resume/CV upload. |
| WHEN a fill runs and a field’s mapping to the profile is missing or ambiguous THE SYSTEM SHALL leave that field unchanged | Property | Missing and ambiguous mappings are never written. |
| WHEN a fill runs and filling a field would require generating, paraphrasing, or otherwise inventing text that is not already stored THE SYSTEM SHALL leave that field unchanged | Property | No generated or paraphrased text is written. |
| WHEN a fill runs and a field is an open-ended prompt for original writing and no stored fact is mapped to that question THE SYSTEM SHALL leave that field unchanged | Property | Unmapped cover-letter / why-this-job / tell-us-about-a-time fields stay unchanged. |
| WHEN a fill runs THE SYSTEM SHALL not activate Submit, Next, or Continue when doing so would advance or complete the application | Property | Every fill leaves Submit, Next, and Continue unactivated when they would advance or complete. |
| WHEN a fill runs and a field is an agree-to-terms checkbox or control, including one that a user-added fact would match THE SYSTEM SHALL not fill it and not activate it | Property | Agree-to-terms controls are never written or clicked, even when a user-added question matches. |
| WHEN setting a stored value requires using a native select or radio group THE SYSTEM SHALL set the stored value through that widget without advancing or completing the application | Property | Widget interaction sets the value and does not advance or complete. |
| WHEN the profile is saved THE SYSTEM SHALL keep the stored profile on the user’s machine so that web pages cannot read it | Property | Page script cannot read extension storage. |
| WHEN the profile is saved THE SYSTEM SHALL keep the stored profile so that other extensions cannot read it | Property | Other extensions cannot read this extension’s storage. |
| WHEN the extension runs THE SYSTEM SHALL not send profile contents off the user’s machine, including through analytics, crash reports, or other remote calls that include profile contents | Property | No outbound call includes profile contents. |
| WHEN the user loads the extension from a local folder into Chrome THE SYSTEM SHALL make the options page, the auto-mode control, and filling available in Chrome | Example | Unpacked load in Chrome exposes options, popup auto-mode, and fill. |
| WHEN the user loads the extension from a local folder into Opera GX THE SYSTEM SHALL make the options page, the auto-mode control, and filling available in Opera GX | Example | Unpacked load in Opera GX exposes options, popup auto-mode, and fill. |
| WHEN a fill runs and no profile facts are stored THE SYSTEM SHALL leave every field unchanged | Property | Empty profile ⇒ no field writes. |
| WHEN a fill runs and only some profile facts are stored THE SYSTEM SHALL fill fields that have no value and that map cleanly to stored facts and leave the rest unchanged | Property | Partial profiles fill only empty clean mappings. |
| WHEN a fill runs and middle name is not stored THE SYSTEM SHALL leave middle-name fields unchanged | Property | Absent middle name ⇒ middle-name fields unchanged. |
| WHEN a fill runs and the page presents preferred-name, legal-name, or name-as-on-passport fields THE SYSTEM SHALL not count those fields toward the one-, two-, three-, or four-or-more name fills | Property | Those variants are excluded from name-field counts. |
| WHEN a fill runs and the page asks for a name variant that is not stored as its own fact THE SYSTEM SHALL leave that field unchanged | Property | Unstored preferred/legal/passport name fields are not filled from first/middle/last. |
| WHEN a fill runs and the page asks for a date other than date of birth and no stored fact is mapped to that question THE SYSTEM SHALL leave that field unchanged | Property | Unmapped non-DOB dates stay unchanged. |
| WHEN a fill runs and “how did you hear about us” is a free-text field and no stored fact is mapped to that question THE SYSTEM SHALL leave that field unchanged | Property | Unmapped free-text how-heard stays unchanged. |
| WHEN a fill runs and “how did you hear about us” is a choice field that has no value and that maps cleanly to the stored canned value THE SYSTEM SHALL select that stored value | Property | Empty matching how-heard choices get the canned value. |
| WHEN a fill runs and the page asks about work authorization or sponsorship for a country other than the United States and no stored fact is mapped to that question THE SYSTEM SHALL leave those fields unchanged | Property | Unmapped non-US auth/sponsorship fields stay unchanged. |
| WHEN a fill runs and street numbers are not stored THE SYSTEM SHALL leave street-numbers fields unchanged | Property | Absent street numbers ⇒ those fields unchanged. |
| WHEN a fill runs and address details are not stored THE SYSTEM SHALL leave address-details fields unchanged | Property | Absent details ⇒ those fields unchanged. |
| WHEN a fill runs and neighbourhood / localidad is not stored THE SYSTEM SHALL leave neighbourhood and localidad fields unchanged | Property | Absent locality ⇒ those fields unchanged. |
| WHEN a fill runs, the page has a resume or CV upload field, and the user has not chosen a resume file THE SYSTEM SHALL leave the upload field unchanged | Property | No chosen resume ⇒ upload unchanged. |
| WHEN a fill runs, the page has a resume or CV upload field, the user has chosen a resume file, and that file is not available THE SYSTEM SHALL leave the upload field unchanged | Property | Unavailable resume bytes ⇒ upload unchanged. |
| WHEN auto-mode is on and the current tab is a login, checkout, or settings form that is not a job application THE SYSTEM SHALL not fill any fields | Property | Those excluded forms are never filled in auto-mode. |
| WHEN auto-mode is off, the user has not clicked the extension icon, and the current tab is a job application THE SYSTEM SHALL not fill any fields until auto-mode is turned on | Property | Job applications are not filled while auto-mode is off. |
| WHEN a fill runs and a field maps to both a built-in fact and a user-added fact THE SYSTEM SHALL leave that field unchanged | Property | Dual mapping is treated as ambiguous and not written. |
| WHEN a settings form has at least two personal-data fields and at least one job-application-specific field, and no password or payment-card fields THE SYSTEM SHALL treat it as a job application in version 1 and fill its empty mapped fields | Property | A settings form meeting the rule is filled; a page with a password field is not. |

Two test-plan rules apply. The open-ended exclusion is asserted in the fill-pass test, against a version that writes default text into unmapped textareas. The agree-to-terms exclusion asserts that the matcher returns no target for an agree-to-terms checkbox; that test fails until the exclusion exists.

### Checks and manual checklist

The repo check command is `npm run check` (see Assumptions). Its static scan of `src/` fails on network APIs (`fetch`, `XMLHttpRequest`, `WebSocket`, `EventSource`, `navigator.sendBeacon`) and on any `http://` or `https://` string literal. There are no exceptions. Version 1 has no SVG and no images in any format: the options page and popup use plain text labels, so no SVG namespace string appears in the code. The user-added exact-cue fill and the missing or ambiguous guard are tested in one module with one fixture set. jsdom has no `DataTransfer`, so resume attach is unit-tested against a stub and confirmed by hand.

Chrome and Opera GX loading (the unpacked-load Example rows) is a manual checklist, not a TDD exception. The browser-level Example rows (loading, icon and popup behaviour, persistence across restart, and cross-browser isolation) are verified by this checklist. Run it before each release, in each browser, and record each step's observed result in the task's Outcome:

1. Load the built `dist/` folder unpacked (`chrome://extensions` in Chrome, `opera://extensions` in Opera GX) with developer mode on. It loads with no errors.
2. Open the options page, enter built-in facts and one user-added fact, save, close, and reopen. The values are still there.
3. Click the toolbar icon on a job application. Empty mapped fields fill, valued fields do not change, and Submit, Next, Continue, and agree-to-terms are untouched.
4. Click the icon on a login, checkout, or settings page. No field changes.
5. With auto-mode on, move to the next step of a multi-step application. Empty mapped fields fill without a click.
6. Choose a resume on the options page. An empty Resume/CV upload receives it. Other file uploads do not.
7. Restart the browser. Auto-mode is off, and the saved profile is still there.
8. Save a profile in Chrome. Open Opera GX's options page. The profile saved in Chrome does not appear there. Repeat with the other browser.
9. During a fill, open DevTools' Network tab. No request carries profile contents.
10. Open the built `manifest.json`. It declares no `externally_connectable` and no `web_accessible_resources`.
11. Open an application page with the extension installed. In the page's DevTools console, `chrome.storage` is not available to page scripts, and the profile key cannot be read from the page. Record the console output in the task's Outcome.

Then repeat steps 1 to 7 in the other browser and confirm its profile is separate.

The browser-level criteria are verified only by this checklist, and each run's observed results are logged in the task Outcome. Other-extension access is covered by step 10 together with the browser's extension-private storage. No further check is added for it.

## Requirement coverage

Every acceptance criterion appears once.

| Criterion | Where the design satisfies it |
| --- | --- |
| WHEN the user opens the extension’s options page THE SYSTEM SHALL let the user view, enter, and save the listed built-in profile facts | Interfaces (Options page); Data models (Profile) |
| WHEN the user saves name, phone, address, or date of birth THE SYSTEM SHALL store that fact as the pieces listed above so a later fill can compose or split it to match the page | Data models (Profile pieces, no derived strings); Architecture (Profile store, Fill engine) |
| WHEN the user saves a profile fact THE SYSTEM SHALL not store a second copy of that fact that can disagree with the saved pieces | Data models (no fullName/fullPhone/fullAddress); Interfaces (Options page save) |
| WHEN the user changes a stored fact on the options page and saves THE SYSTEM SHALL use the updated fact on fills after that save | Data flow (Save path); Architecture (Profile store read at fill time) |
| WHEN the user starts a new session in the same browser after saving a profile THE SYSTEM SHALL still have those saved facts available for filling | Architecture (Profile store in chrome.storage.local); Data models (Profile lifetime) |
| WHEN the user saves the profile THE SYSTEM SHALL save and later reuse it without requiring a passphrase | Overview; Architecture (Profile store); Interfaces (Options page) |
| WHEN the user saves a profile in Chrome THE SYSTEM SHALL keep that profile only in Chrome and shall not make it available in Opera GX | Architecture (per-browser storage); Overview |
| WHEN the user saves a profile in Opera GX THE SYSTEM SHALL keep that profile only in Opera GX and shall not make it available in Chrome | Architecture (per-browser storage); Overview |
| WHEN the user opens the extension’s options page THE SYSTEM SHALL present a basic, non-stylized menu to add a fact and the question or questions that fact maps to | Interfaces (Options page add-fact menu) |
| WHEN the user adds a fact and the questions it maps to and saves THE SYSTEM SHALL store that fact and those mappings with the profile and use them on fills after that save | Data models (UserAddedFact); Data flow (Save path); Fill engine |
| WHEN a fill runs and a field has no value and that field’s label, accessible name, or placeholder is a question the user recorded for a stored user-added fact THE SYSTEM SHALL write that stored value into the field as if the user had typed or selected it | Fill algorithm (user-added exact cue match); Data models (UserAddedFact.questions) |
| WHEN a fill runs and a field’s label, accessible name, and placeholder are not the question the user recorded for a stored user-added fact THE SYSTEM SHALL not fill that field from that user-added fact | Fill algorithm (user-added match uses only those three cues) |
| WHEN the user has not added any extra facts THE SYSTEM SHALL still fill from the built-in facts | Fill algorithm (built-in path with empty userAdded) |
| WHEN the user clicks the extension icon THE SYSTEM SHALL turn auto-mode on | Interfaces (Popup on open); Data flow |
| WHEN the user clicks the extension icon and the current tab is a job application THE SYSTEM SHALL fill that tab’s fields that have no value and that map cleanly to stored profile facts | Data flow (icon click → fill); Fill algorithm |
| WHEN the user clicks the extension icon and the current tab is not a job application THE SYSTEM SHALL not fill any fields on that tab | Data flow (classify not application); Error handling |
| WHEN auto-mode is on THE SYSTEM SHALL keep auto-mode on until the user turns it off from the popup or the browser starts a new session | Data models (AutoMode session lifetime); Interfaces (Popup) |
| WHEN the browser starts a new session THE SYSTEM SHALL set auto-mode off | Data models (AutoMode absent ⇒ off); Architecture (Service worker) |
| WHEN auto-mode is off and the user has not clicked the extension icon THE SYSTEM SHALL not fill any fields | Data flow (failure paths); Error handling |
| WHEN the user opens the extension popup THE SYSTEM SHALL present a control that turns auto-mode on or off | Interfaces (Popup control) |
| WHEN the user turns auto-mode off from the popup THE SYSTEM SHALL leave auto-mode off and not fill fields until auto-mode is turned on again | Interfaces (Popup); Data models (AutoMode) |
| WHEN auto-mode is on and the current tab is a job application THE SYSTEM SHALL fill fields that have no value and that map cleanly to stored profile facts without requiring a further click | Data flow (auto-mode on); Fill algorithm |
| WHEN auto-mode is on and the current tab is not a job application THE SYSTEM SHALL not fill any fields | Data flow (classify); Error handling |
| WHEN auto-mode is on and the user advances to another step of a job application THE SYSTEM SHALL fill fields on that step that have no value and that map cleanly to stored profile facts without requiring another click | Data flow (next step); Fill algorithm |
| WHEN a fill runs THE SYSTEM SHALL identify fields from the page’s labels, accessible names, placeholders, and similar cues | Fill algorithm (identification) |
| WHEN a fill runs on a job application hosted by a job board the user did not name or configure THE SYSTEM SHALL still fill fields that have no value and that map cleanly to stored facts | Architecture (no board adapters); Fill algorithm |
| WHEN a fill runs and a field has no value and maps cleanly to a stored profile fact THE SYSTEM SHALL write that stored value into the field as if the user had typed or selected it, so the page treats the field as filled | Fill algorithm (write path) |
| WHEN a fill runs and a choice field has no value and maps cleanly to stored US work authorization, sponsorship, an EEO fact, or the canned “how did you hear about us” value THE SYSTEM SHALL set that choice as if the user had selected it | Fill algorithm (choice widgets) |
| WHEN a fill runs and the page presents exactly one field asking for the applicant’s first, middle, last, or full name THE SYSTEM SHALL fill that field if it has no value: with the composed stored name, including middle name only if it is stored, when it asks for the full name, or with the matching piece when it asks for first, middle, or last name | Fill algorithm (name count = 1) |
| WHEN a fill runs and the page presents exactly two fields asking for the applicant’s first, middle, last, or full name THE SYSTEM SHALL fill each of those fields that has no value and whose own cue says first name or last name with that stored piece; a field whose cue says neither stays unchanged | Fill algorithm (name count = 2) |
| WHEN a fill runs and the page presents exactly two fields asking for the applicant’s first, middle, last, or full name THE SYSTEM SHALL not write a composed full name into either field | Fill algorithm (name count = 2) |
| WHEN a fill runs and the page presents exactly three fields asking for the applicant’s first, middle, last, or full name THE SYSTEM SHALL fill each of those fields that has no value and whose own cue says first name, middle name, or last name with that stored piece; a field whose cue does not say which piece it is stays unchanged | Fill algorithm (name count = 3) |
| WHEN a fill runs and the page presents four or more fields asking for the applicant’s first, middle, last, or full name THE SYSTEM SHALL not use the one-, two-, or three-field name fills | Fill algorithm (name count ≥ 4) |
| WHEN a fill runs, four or more name fields, label exactly `first name` or `primer nombre` THE SYSTEM SHALL fill that field from the stored first name if that field has no value | Fill algorithm (exact EN/ES labels) |
| WHEN a fill runs, four or more name fields, label exactly `last name` or `apellido` THE SYSTEM SHALL fill that field from the stored last name if that field has no value | Fill algorithm (exact EN/ES labels) |
| WHEN a fill runs, four or more name fields, label exactly `middle name` or `segundo nombre` THE SYSTEM SHALL fill that field from the stored middle name if that field has no value | Fill algorithm (exact EN/ES labels) |
| WHEN a fill runs, four or more name fields, label not exactly those six strings THE SYSTEM SHALL not fill that field from the stored first, middle, or last name | Fill algorithm (exact EN/ES labels) |
| WHEN a fill runs and the page has a single phone field that has no value and no separate country-code field THE SYSTEM SHALL leave that field unchanged, and SHALL NOT join the stored country code into it | Fill algorithm (phone rule) |
| WHEN a fill runs and the page has a country-code field and a phone field that has no value THE SYSTEM SHALL fill the phone field with the stored national number | Fill algorithm (phone rule) |
| WHEN a fill runs and the page has a country-code field and/or a national-number field THE SYSTEM SHALL fill each of those fields that has no value from the matching stored piece | Fill algorithm (phone split) |
| WHEN a fill runs and the page has a single street or address-line field that has no value THE SYSTEM SHALL fill that field by composing the stored street-numbers piece and the stored address-details piece that exist | Fill algorithm (address join) |
| WHEN a fill runs and the page has a street-numbers field and/or an address-details field THE SYSTEM SHALL fill each of those fields that has no value from the matching stored piece | Fill algorithm (address split) |
| WHEN a fill runs and the page has a neighbourhood or localidad field that has no value and neighbourhood / localidad is stored THE SYSTEM SHALL fill that field from the stored neighbourhood / localidad | Fill algorithm (locality) |
| WHEN a fill runs and the page has a single full-address field that has no value and that maps cleanly to stored street numbers, address details, city, state/region, or postal code THE SYSTEM SHALL fill that field by composing those stored pieces, writing only the stored parts when some are missing | Fill algorithm (full address without locality) |
| WHEN a fill runs and the page has a city, state/region, postal code, and/or country field THE SYSTEM SHALL fill each of those fields that has no value from the matching stored piece | Fill algorithm (address pieces) |
| WHEN a fill runs and the page has a single date-of-birth field that has no value and date of birth is stored THE SYSTEM SHALL fill that field from the stored date of birth, written in the date format that field asks for (month and day zero-padded, and a two-digit year where the field asks for one) | Fill algorithm (DOB compose) |
| WHEN a fill runs and the page has a month, day, and/or year field for date of birth and date of birth is stored THE SYSTEM SHALL fill each of those fields that has no value from the stored date of birth, with a zero-padded month and day and a two-digit year where that field asks for one | Fill algorithm (DOB split) |
| WHEN a fill runs and a date-of-birth field’s expected format cannot be determined from the page THE SYSTEM SHALL leave that field unchanged | Fill algorithm (date formats) |
| WHEN a fill runs, the user has chosen a resume file, that file is available, and the page has a resume or CV upload field that has no file THE SYSTEM SHALL attach that file to the upload field | Fill algorithm (resume attach) |
| WHEN a fill runs and the page has no resume or CV upload field THE SYSTEM SHALL not attach a file | Fill algorithm (resume attach) |
| WHEN a fill runs and the page has a file upload that is not a resume or CV upload THE SYSTEM SHALL not attach the resume file to that upload | Fill algorithm (resume attach) |
| WHEN a fill runs and a resume or CV upload field already has a file THE SYSTEM SHALL leave that upload field unchanged | Fill algorithm (no-overwrite); Error handling |
| WHEN a fill runs and a field already has a value THE SYSTEM SHALL leave that field unchanged | Fill algorithm (no-overwrite first); Error handling |
| WHEN a later fill runs from an icon click or from auto-mode and some fields already have values THE SYSTEM SHALL fill remaining fields that have no value and that map cleanly, including an empty resume/CV upload | Fill algorithm (no-overwrite + remaining blanks); Data flow |
| WHEN a fill runs and a field’s mapping to the profile is missing or ambiguous THE SYSTEM SHALL leave that field unchanged | Fill algorithm (ambiguity); Error handling |
| WHEN a fill runs and filling a field would require generating, paraphrasing, or otherwise inventing text that is not already stored THE SYSTEM SHALL leave that field unchanged | Fill algorithm (no invent); Error handling |
| WHEN a fill runs and a field is an open-ended prompt for original writing and no stored fact is mapped to that question THE SYSTEM SHALL leave that field unchanged | Fill algorithm (open-ended guard) |
| WHEN a fill runs THE SYSTEM SHALL not activate Submit, Next, or Continue when doing so would advance or complete the application | Fill algorithm (never submit); Architecture (Content script) |
| WHEN a fill runs and a field is an agree-to-terms checkbox or control, including one that a user-added fact would match THE SYSTEM SHALL not fill it and not activate it | Fill algorithm (agree-to-terms never filled); Error handling |
| WHEN setting a stored value requires using a native select or radio group THE SYSTEM SHALL set the stored value through that widget without advancing or completing the application | Fill algorithm (choice widgets) |
| WHEN the profile is saved THE SYSTEM SHALL keep the stored profile on the user’s machine so that web pages cannot read it | Architecture (isolated world, chrome.storage.local, not page localStorage) |
| WHEN the profile is saved THE SYSTEM SHALL keep the stored profile so that other extensions cannot read it | Architecture (extension-private storage) |
| WHEN the extension runs THE SYSTEM SHALL not send profile contents off the user’s machine, including through analytics, crash reports, or other remote calls that include profile contents | Architecture (no remote endpoints); Overview |
| WHEN the user loads the extension from a local folder into Chrome THE SYSTEM SHALL make the options page, the auto-mode control, and filling available in Chrome | Architecture (Manifest V3 unpacked package) |
| WHEN the user loads the extension from a local folder into Opera GX THE SYSTEM SHALL make the options page, the auto-mode control, and filling available in Opera GX | Architecture (same package, Opera GX Chromium) |
| WHEN a fill runs and no profile facts are stored THE SYSTEM SHALL leave every field unchanged | Error handling; Fill algorithm |
| WHEN a fill runs and only some profile facts are stored THE SYSTEM SHALL fill fields that have no value and that map cleanly to stored facts and leave the rest unchanged | Error handling; Fill algorithm |
| WHEN a fill runs and middle name is not stored THE SYSTEM SHALL leave middle-name fields unchanged | Error handling; Data models (name.middle optional) |
| WHEN a fill runs and the page presents preferred-name, legal-name, or name-as-on-passport fields THE SYSTEM SHALL not count those fields toward the one-, two-, three-, or four-or-more name fills | Fill algorithm (name-field census) |
| WHEN a fill runs and the page asks for a name variant that is not stored as its own fact THE SYSTEM SHALL leave that field unchanged | Fill algorithm (name variants); Error handling |
| WHEN a fill runs and the page asks for a date other than date of birth and no stored fact is mapped to that question THE SYSTEM SHALL leave that field unchanged | Fill algorithm (DOB-only built-in dates) |
| WHEN a fill runs and “how did you hear about us” is a free-text field and no stored fact is mapped to that question THE SYSTEM SHALL leave that field unchanged | Fill algorithm (how-heard) |
| WHEN a fill runs and “how did you hear about us” is a choice field that has no value and that maps cleanly to the stored canned value THE SYSTEM SHALL select that stored value | Fill algorithm (how-heard choice) |
| WHEN a fill runs and the page asks about work authorization or sponsorship for a country other than the United States and no stored fact is mapped to that question THE SYSTEM SHALL leave those fields unchanged | Fill algorithm (US-only built-in auth) |
| WHEN a fill runs and street numbers are not stored THE SYSTEM SHALL leave street-numbers fields unchanged | Error handling |
| WHEN a fill runs and address details are not stored THE SYSTEM SHALL leave address-details fields unchanged | Error handling |
| WHEN a fill runs and neighbourhood / localidad is not stored THE SYSTEM SHALL leave neighbourhood and localidad fields unchanged | Error handling |
| WHEN a fill runs, the page has a resume or CV upload field, and the user has not chosen a resume file THE SYSTEM SHALL leave the upload field unchanged | Error handling |
| WHEN a fill runs, the page has a resume or CV upload field, the user has chosen a resume file, and that file is not available THE SYSTEM SHALL leave the upload field unchanged | Error handling |
| WHEN auto-mode is on and the current tab is a login, checkout, or settings form that is not a job application THE SYSTEM SHALL not fill any fields | Fill algorithm (page classification); Error handling |
| WHEN auto-mode is off, the user has not clicked the extension icon, and the current tab is a job application THE SYSTEM SHALL not fill any fields until auto-mode is turned on | Data models (AutoMode); Error handling |
| WHEN a fill runs and a field maps to both a built-in fact and a user-added fact THE SYSTEM SHALL leave that field unchanged | Fill algorithm (dual-map ⇒ ambiguous) |
| WHEN a settings form has at least two personal-data fields and at least one job-application-specific field, and no password or payment-card fields THE SYSTEM SHALL treat it as a job application in version 1 and fill its empty mapped fields | Fill algorithm (page classification); Assumptions (Known limit) |

### Fill algorithm

Used by the content script. No extra user-visible behavior.

1. **Permit:** Run only if this pass is an icon-click fill, or auto-mode is on. Icon click has already set auto-mode on. If auto-mode is off and this is not that click, stop.
2. **Classify the page.** A page is a job application when it has at least two personal-data fields (name, email, phone, address, or date of birth cues) and at least one job-application-specific field (LinkedIn, GitHub, portfolio, resume/CV upload, US work authorization or sponsorship, EEO, how-heard, or cover letter). Otherwise change no fields and stop. A page with a password field or a payment-card field is not an application, because those are unrelated fields. Settings pages are not separately identified and are a known limit. No cue list identifies any page type. Apply-style words in the URL, title, or heading do not qualify a page alone. Detection does not use a list of job boards.
3. **Read** this browser’s profile and resume in the isolated world. If no facts are stored, change no fields and stop.
4. **Identify fields** from labels, accessible names, placeholders, and similar cues (`for`/`id` associations, `aria-labelledby`, legend, `autocomplete` as a similar cue for identification and built-in mapping only). Built-in cues match the Cue vocabulary exactly, after trimming whitespace, removing trailing `*`, `:`, or `?`, and ASCII case-folding.
5. **Emptiness:** A field has no value when it has no user-visible value (empty text, no selected choice, no file). A field that already has a value is skipped for the rest of this pass.
6. **Name census:** Count fields whose cue is one of the name strings in the Cue vocabulary, matched exactly. Preferred-name, legal-name, and name-as-on-passport labels are not in that list, so they are never counted. Those variants are filled only if a stored fact of their own exists (user-added exact cue match).
    - Count 1: if empty, fill by its own cue. A full-name cue gets the composed first + middle-if-stored + last; a first, middle, or last cue gets that piece. Do not invent a middle name.
    - Count 2: fill each empty field whose own cue says first name or last name with that piece. A field whose cue says neither stays unchanged, so two such fields with no first or last cue are both unchanged. Never write the composed full name into either field.
    - Count 3: fill each empty field whose own cue says first, middle, or last name with that piece. A field whose cue does not say which piece it is stays unchanged. If middle is not stored, leave the middle field unchanged.
    - Count ≥ 4: do not use the 1/2/3 strategies. Fill an empty field only when its **label** is exactly `first name`, `primer nombre`, `last name`, `apellido`, `middle name`, or `segundo nombre`. Any other label is not filled from first, middle, or last.
7. **Phone, address, date of birth** on remaining empty fields: join or split from pieces as specified in Cue vocabulary and formats. Localidad is its own mapping and is not folded into the non-locality full-address compose. A full-address field is written with whichever of its parts are stored. Other fields whose piece is missing stay unchanged. Dates other than date of birth are not filled from built-in DOB.
8. **Other built-in facts** (email, links, US work authorization, US sponsorship, EEO including decline-to-answer, canned how-heard as a **choice**): fill empty clean mappings only. Free-text how-heard is filled only via a user-added mapping. Non-US work authorization/sponsorship is filled only via a user-added mapping.
9. **User-added facts:** a stored user-added fact maps to a field only when that field’s label, accessible name, **or** placeholder **is** a question the user recorded for that fact. No substring, similar wording, regex, or other cues. If those three strings are not the recorded question, do not use that fact for that field.
10. **Conflicts:** if a field maps to both a built-in fact and a user-added fact, or to more than one fact, or the mapping is otherwise missing or ambiguous, leave it unchanged. Open-ended original writing with no mapped stored fact is left unchanged. Never invent or paraphrase.
11. **Write:** for each remaining empty clean mapping, set the value as if the user typed or selected it (native value setter plus input/change, or the page widget such as a dropdown) so the page treats the field as filled. Interact with widgets only as needed to set the value. Do not activate Submit, Next, or Continue when that would advance or complete the application. Never fill or activate an agree-to-terms control.
12. **Resume:** if a resume/CV upload has no file and a chosen resume file is available, attach it. If there is no resume/CV upload, or the upload is not resume/CV, or it already has a file, or no file was chosen, or bytes are unavailable, attach nothing / leave the upload unchanged.
13. **Later fills:** icon click or auto-mode on a later step repeats this algorithm. Fields that already have values stay; remaining blanks, including an empty resume/CV upload, may still be filled.

### Cue vocabulary and formats

Built-in cues are exact matches (Fill algorithm step 4). A field’s label, accessible name, or placeholder must equal one of the strings in its row. Strings are lowercase here.

| Fact | Cue strings |
| --- | --- |
| First name | first name, given name, primer nombre |
| Middle name | middle name, segundo nombre |
| Last name | last name, family name, surname, apellido |
| Full name | full name, your name, nombre completo |
| Email | email, email address, e-mail, e-mail address, correo electrónico |
| Phone (single field) | phone, phone number, mobile, mobile number, cell phone, telephone, teléfono, telefono, número de teléfono, numero de telefono |
| Phone country code | country code, phone country code, country calling code, código de país |
| Phone national number | national number |
| Street numbers | street number, street numbers, house number |
| Address line (street numbers + details) | address, street address, address line 1, address 1, street, street and number, dirección, direccion |
| Address details | apartment, apartment number, apt, apt number, suite, unit, address line 2, address 2, departamento, apartamento |
| Full address (without locality) | full address, mailing address, home address, complete address, dirección completa, direccion completa |
| Neighbourhood / localidad | neighbourhood, neighborhood, localidad, barrio |
| City | city, town, ciudad |
| State / region | state, state/province, state/region, province, region, estado, provincia |
| Postal code | postal code, zip, zip code, postcode, código postal, codigo postal |
| Country | country, país, pais |
| Date of birth | date of birth, birth date, birthdate, dob, fecha de nacimiento |
| Date of birth parts | birth month, birth day, birth year, dob month, dob day, dob year |
| LinkedIn | linkedin, linkedin url, linkedin profile, linkedin profile url |
| GitHub | github, github url, github profile, github profile url |
| Portfolio | portfolio, portfolio url, portfolio website, website, website url, personal website, personal site |
| Resume / CV upload | resume, resume/cv, cv, curriculum vitae |
| US work authorization | are you legally authorized to work in the united states, are you authorized to work in the united states, work authorization |
| US sponsorship | will you now or in the future require sponsorship, do you require sponsorship, visa sponsorship |
| EEO | gender, race/ethnicity, ethnicity, veteran status, disability status |
| How did you hear about us | how did you hear about us, how did you hear about this job, referral source |
| Cover letter (open-ended, never filled) | cover letter |
| Agree-to-terms (never filled or activated) | i agree, i agree to the terms, i agree to the terms and conditions, terms and conditions, i have read and agree |

Rules for cues:

- Bare `name`, `middle initial`, `month`, `day`, and `year` are not cues. A bare name is often a company or reference field, a middle initial would need truncating, and bare month, day, or year is too ambiguous.
- Phone: when the page also has a country-code field, a phone-cue field gets the national number. When it has none, a single phone field is left unchanged. The country code is never joined into a phone field.
- Name census counts only the name cues above. Preferred, legal, and passport names are not in the table, so they are not counted and are filled only by a user-added fact.

Formats:

- Names: pieces joined with one space. Without a middle name: `First Last`.
- Street line: street numbers, then `, `, then details. A missing piece is skipped.
- Full address: street line, then one space, then city, region, and postal code, each separated by one space. Country is not part of the composed address and fills only its own field. Missing parts are skipped. Localidad is excluded.
- Date of birth in one field:
  - `input type="date"` takes `YYYY-MM-DD`.
  - Otherwise the format comes from the field’s placeholder or label. Hint text is not read in version 1: tokens `dd`, `mm`, and `yyyy` or `yy` (`aaaa` and `aa` also count, for Spanish pages), separated by `/`, `-`, or `.`. The token order sets the output order. Examples: `MM/DD/YYYY` gives `03/07/1990`; `YYYY-MM-DD` gives `1990-03-07`; `DD.MM.YY` gives `07.03.90`.
  - Month and day are always two digits. The year is two digits when the pattern says `yy` or the field has `maxlength="2"`; otherwise four.
  - No recognizable pattern: leave the field unchanged.
- Date of birth in separate fields: month and day zero-padded; year four digits unless the field has `maxlength="2"`. A `<select>` for month or day gets the option whose value or text equals the padded number or the number without padding.

Widgets:

- Native `<select>`: the option whose visible text equals the stored value (trimmed, case-folded). Set it and fire `change`, as a user selection would. The option’s value attribute is not matched.
- Radio group: the radio whose label equals the stored value.
- ARIA dropdown (`role="combobox"` or `listbox`): open it, choose the option whose text equals the stored value, then close it. A search box may receive the stored value, but only an exact-match option is chosen. No exact match: leave unchanged. These are best-effort in this version. They differ from site to site, so some will stay unchanged. Best-effort behaviour satisfies the custom-dropdown requirement for version 1.
- Stored choices are the option texts saved on the options page. Stored "Man" does not match an option "Male", so that field stays unchanged.

## Assumptions

- **Toolchain:** TypeScript in strict mode. esbuild bundles `src/` into `dist/` (`manifest.json` plus the options page, popup, service worker, and content script), and that folder is the one loaded unpacked. Vitest runs unit tests in a jsdom environment. `npm run check` runs `tsc --noEmit`, then `vitest run`, then the esbuild build, then the static scan in Checks. Every tool is free and open source; no paid service is used. npm is the package manager. The requirements do not fix exact filenames.
- **Placeholder entry points:** the first task creates one minimal source file per surface (service worker, content script, popup, options page), each doing nothing yet, plus one trivial test. The build therefore runs from the first task, and the check command is never a no-op. Real behavior replaces the placeholders in later tasks.
- **Auto-mode storage:** `chrome.storage.session` (or equivalent session-scoped extension storage) holds auto-mode so a service-worker restart during the same browser session keeps it on, and a new browser session starts off.
- **Resume bytes:** stored as base64 text under the `resume` key of this browser’s `chrome.storage.local`, next to the profile metadata. The `unlimitedStorage` permission is declared so a resume larger than the default quota still saves. How the File is assigned on `<input type="file">` is `DataTransfer` in the isolated world, which Chrome and Opera GX are assumed to allow.
- **Job-application detection:** a page is a job application when it has at least two personal-data fields and at least one job-application-specific field (Fill algorithm step 2). A page with a password field or a payment-card field is never one. Every other page that meets the rule is one. This is a lax first-version rule. No cue lists are kept for login, checkout, settings, or payment pages. Apply-style words in the URL, title, or heading do not count on their own.
- **Built-in matching:** built-in facts match the Cue vocabulary exactly, after trimming whitespace, removing trailing `*`, `:`, or `?`, and ASCII case-folding. The vocabulary is a fixed list, not a job-board adapter, and it is not used for user-added facts.
- **Exact 4+ name labels:** compared to the six required strings after trimming ASCII whitespace and folding ASCII case, so a label `First Name` matches `first name`. A larger translation table is out of scope.
- **Choice matching:** a stored choice (US work authorization, US sponsorship, EEO including decline-to-answer, canned how-heard) is written only to an option whose visible text equals the stored text exactly, after trimming and ASCII case-folding. No exact match leaves the field unchanged. No fixed option lists are kept. Date-of-birth month and day `<select>` elements are the exception: they match the option by value or visible text, as Formats describes.
- **Decline-to-answer:** each EEO fact has one text input. The user types the decline wording the page uses, and it is matched under Choice matching.
- **Fill-pass tasks:** the fill pass is planned as two tasks: a pass-gate task, and a mapper-wiring task that connects the seven mappers.
- **Month and day select branch:** its own task, placed after the native select writer task and depending on it.
- **Checkboxes:** user-added facts do not target checkboxes or role=checkbox controls in version 1. Agree-to-terms controls stay excluded. The agree-to-terms exclusion test uses a checkbox or role=checkbox control as its target, and the matcher must return no target for it.
- **Re-enabling auto-mode:** reopening the popup turns auto-mode on again. That is the only path back on after a popup-off.
- **Popup control state:** the control shows on after every open, because opening the popup turns auto-mode on. Toggling it off within the same open turns it off, and the control then shows off.
- **Choice entry:** on the options page, each choice fact (US work authorization, US sponsorship, each EEO fact including decline-to-answer, and canned how-heard) is a text input. The user types the option wording exactly as the page shows it, and the value is matched under Choice matching. No pickers.
- **Date-of-birth entry:** the options page uses a native date input (`YYYY-MM-DD`), split into `{ year, month, day }` on save.
- **Year selects for date of birth:** a year `<select>` is filled by the same rule as the month and day selects. The option whose value or visible text equals the stored year, in the format the field asks for (four digits, or two digits when the option uses that), is chosen.
- **Payment-card and password fields:** a payment-card field has `autocomplete` of `cc-number`, `cc-exp`, or `cc-csc`, or its label, accessible name, or placeholder exactly equals one of: card number, expiration date, expiry, CVV, CVC, security code. A password field is an `input` of type `password`.
- **Known limit (version 1):** a settings form with at least two personal-data fields and one application-specific field, and no password or payment-card field, is treated as a job application and may be filled. No exclusion cue lists are kept.
- **Test location:** tests live in `tests/` at the repo root, outside `src/`. The static scan covers `src/` and `dist/` only.
- **Empty profile:** a profile counts as not stored when every built-in piece is absent or blank after trimming and `userAdded` is empty. A saved profile with only blank pieces is treated the same as no profile.
- **Choice widgets for user-added facts:** when a user-added fact matches a select, radio group, or ARIA combobox, it is set through the choice widget rules (native select, radio, or combobox), not by writing text. Text inputs and textareas are written as text.
- **User-added equality:** after trimming leading and trailing whitespace, label, accessible name, or placeholder must equal the recorded question, including case. No other normalization.
- **Empty after a user clear:** if the user clears a control back to having no value, a later fill may write into it. The extension does not keep a separate “user touched this” log.
- **Popup versus icon:** Manifest V3 `default_popup` means the toolbar click opens the popup; popup-open is the icon-click event that turns auto-mode on and may fill.
- **Host access:** broad page access is declared so fills can run on unnamed application hosts. That permission is not permission to send profile contents anywhere.
- **No extra UI:** no “filled N fields” toast, no highlight of filled fields, no stylized options theme. Those are omitted because the requirements do not ask for them.
- **No images:** the manifest declares no icon files in version 1. Chrome and Opera GX show their default extension icon on the toolbar. Clicking it still opens the popup.
- **Options and popup visuals:** functional HTML/CSS sufficient to enter facts and toggle auto-mode; only the add-fact menu is constrained to be basic and non-stylized.
