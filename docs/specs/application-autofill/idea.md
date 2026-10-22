# Application autofill

## Intent

JDash42 applies to jobs in the browser and spends most of the time retyping the same facts: name, contact details, work authorization, veteran status, and other questions he has already answered hundreds of times. Open-ended questions hurt too, but they are a later problem. This idea is a Chrome / Opera GX extension that fills the repeated, already-known fields on whatever application page is in front of him, without generating any new text.

It exists to make the mechanical part of applying faster than he can type, while leaving judgment (what to submit, whether this job is worth it, anything he has not already stored) with him.

## Who it is for

One person: JDash42, applying from his own Chrome and Opera GX browsers, with the project checkout at `/home/jd/JobApplier` on WSL Ubuntu.

## Goals

- Fill previously stored facts on a job application faster than typing them.
- Work on most application pages, not a short list of named job boards. Boards (Greenhouse, Lever, and the rest) are just branded form hosts; the user should not have to know which one a company uses.
- Store each fact once, in pieces, and compose or split it to match the page (name by how many name fields are on the page, with exact English/Spanish labels when there are four or more; phone vs country code plus number; address street numbers vs apartment-style details, plus city / state / zip / country / localidad; date of birth as a single date vs month / day / year).
- Let him add more facts and the questions they map to from a basic, non-stylized menu, so the profile can grow as new questions appear.
- Leave unrecognized or ambiguous fields blank rather than guess.
- Never invent text. Reuse what he has already saved.
- Never submit the form. He reviews and presses Submit.
- Keep profile data on his machine. Websites and other extensions must not be able to read the stored profile.
- Load unpacked from a local folder while it is being built. Chrome Web Store / Opera add-ons listing is not part of this idea.

## Non-goals (this idea)

- An LLM that writes new answers, cover letters, or emails.
- A bank of past open-ended answers with closest-match suggestions. That is the next feature after this one.
- A career database that builds a tailored CV per job. That is a separate project, brainstormed later.
- Auto-submitting applications, applying on his behalf, or running unattended.
- Firefox, Safari, or mobile browsers.
- A Rails app, a website, or a server backend.
- Scraping or searching job listings.
- Shipping in the Chrome Web Store or Opera add-ons catalog.
- Syncing the profile across devices or to the cloud.
- Encrypting the profile with a passphrase. Disk-theft and malware are not in scope after discussion.

## Anti-goals

These would count as failure even if the extension “works”:

- Generating a sentence he did not already save (including “helpful” paraphrases).
- Submitting a form, or filling a login / checkout / settings form because auto-mode was on or because he clicked the extension icon on that page.
- Sending profile data to a remote server.
- Storing a second copy of the same fact that can drift out of date (one “full name” string plus separate first / last that disagree).
- A giant keyword-stuffed CV used as a substitute for this tool.

## How it is made (decided)

These are decided. A builder should not reopen them without a change to this idea.

- **Medium:** a browser extension, Manifest V3, written in TypeScript (user wants a typed language). Not Rails. Opera GX is Chromium, so one extension serves Chrome and Opera GX.
- **Profile storage:** `chrome.storage.local`. Small JSON of facts. Unencrypted. No passphrase. Efficient for this size; isolated from web pages and from other extensions. Do not use the page’s `localStorage`.
- **Where he types the profile:** the extension’s own options page. Built-in facts are entered there. A very basic, non-stylized menu on that page lets him add more facts and which questions they map to. After that, the filler reads the built-in facts and the user-added facts.
- **How a fill starts:** auto-mode is **off on every browser restart** (each new browser session). He must turn it on again each day / each session. Clicking the extension icon turns auto-mode on, and fills the current tab if and only if that tab is a job application. It does not fill login, checkout, settings, or other non-application pages. After that click, auto-mode stays on for later application tabs until he turns it off or the browser restarts. Turning auto-mode off is the popup control, not the icon click. The popup still has an on/off control. On: fill when the page looks like a job application, without an extra click.
- **Auto-mode scope (author’s call):** auto-mode must not fill every form on the internet. It only auto-fills when the page looks like a job application (several matching application-style fields, or an apply-style page). Exact detection is left to the builder; the behavior above is not.
- **Submit:** never. He always presses Submit, including in auto-mode.
- **Unknown fields:** leave blank. Do not invent a middle name, nickname, or a value he never stored.
- **Compose / split (day one):** name, phone, address, and date of birth only. Name fill follows field count: 3 name-related fields → first, middle, last; 2 → first/name and last name (not full name); 1 → full name. When there are 4 or more name-related fields, do not use that counting strategy. Map each field only to an exact label match: `first name` / `primer nombre`, `last name` / `apellido`, `middle name` / `segundo nombre`. If a field has no exact match, leave it empty. Exact or nothing per field; no guessing and no larger translation table. Address compose/split includes street numbers and address details (apartment number or similar) as one combined field vs separate fields, the same way name and phone already work. Odd variants (preferred name, legal name, name as on passport) stay blank unless that fact exists in the profile (including as a user-added fact). Other dates (available to start, graduation, and so on) are not built-in.
- **Target pages:** any site. No Greenhouse-only adapter as the MVP strategy. Site-specific adapters for weird widgets can come later; they are not required to call this done.

## Profile facts (author’s call unless noted)

Built-in facts, stored as pieces, not display strings, so compose/split can work. He can also add more facts later, with the questions they map to:

- Name: first, middle (optional), last
- Date of birth (decided: this is the only built-in date; store so it can be filled as one field or as month / day / year)
- Email, phone (country code and national number stored so they can be joined or split)
- Address: street numbers; address details for apartment number or similar; city; state/region; postal code; country; neighbourhood / localidad as its own address fact (filled when the page asks for neighbourhood or localidad)
- Links he reuses: LinkedIn, GitHub, portfolio / personal site
- Work authorization for the US, and whether he needs sponsorship
- EEO / self-identification he is willing to save: gender, race/ethnicity, veteran status, disability status, plus the usual “decline to answer” choice
- A single canned “how did you hear about us” value if he always picks the same thing; free-text versions of that question are not a built-in fill (they belong with the later answer bank unless he adds a mapped fact)

**Resume file (author’s call):** one resume file he chooses on the options page. If the page has a resume/CV upload, attach that file. If there is no upload field, do nothing. Tailored-per-job resumes are the separate CV project.

## What a fill does

1. Read the stored profile.
2. Find fields on the page (labels, accessible names, placeholders, and similar). Matching is left to the builder except where this idea already decides it (name field count; exact English/Spanish name labels when there are four or more name fields; user-added question mappings); it must not require a per-site list to be useful.
3. For each field, if a stored fact maps cleanly — including a user-added fact mapped to that question — write that value into the field as if he had typed it (so the page’s own scripts notice the change). Name fields follow the 3 / 2 / 1 count above, or the exact-label pairs when there are four or more name fields.
4. If mapping is missing or ambiguous, leave the field alone.
5. Do not touch open-ended text areas that ask for original writing (cover letter, “why do you want to work here”, “tell us about a time…”) unless he has already stored a fact mapped to that question. Unmapped ones stay empty in this idea.
6. Do not press Submit, Next, Continue, or agree-to-terms on his behalf if those would advance or complete the application. (Builder may still need to interact with a widget to set a value, e.g. open a dropdown to pick “No”. That is filling, not submitting.)

## Constraints

- Personal data stays on the machine. No analytics, crash reports, or APIs that include profile contents.
- Permission to run on application pages is required for “most pages”; do not interpret that as permission to exfiltrate data.
- JDash42’s local checkout is `/home/jd/JobApplier` on WSL Ubuntu; GitHub remote is https://github.com/jdsan93/JobApplier.
- No product code until this idea is approved and a later specify pass has produced requirements and design. This file is not a design.

## Left to the builder

- Exact TypeScript tooling, bundler, extension folder layout, and how unpacked load works in Chrome vs Opera GX.
- How “this page looks like a job application” is detected, as long as auto-mode does not fill arbitrary web forms.
- How labels are matched to built-in profile facts, as long as the name field-count rule, the exact English/Spanish name-label pairs when there are four or more name fields, compose/split for name, phone, address, and date of birth, user-added question mappings, and ambiguity-means-blank all hold. Not regex, ML, or other learned matching for user-added questions. Not a larger name-label translation table than those pairs.
- Visual details of the options page and popup (toggle, edit profile, maybe a “filled N fields” hint after a fill), except the add-fact menu is basic and non-stylized.
- Whether a filled field is highlighted; if yes, it is a convenience, not a substitute for him reviewing before Submit.
- How the resume file is held next to the JSON profile (still local, still inside the extension’s storage model).
- Tests, fixtures, and which real application pages to use as examples (CentralReach Greenhouse posting was the conversation example, not a scope limit).

## Later, not this idea

1. Answer bank: store how he has answered similar open questions; an LLM suggests the closest answer he has already given, if any.
2. Career database and per-job CV instead of one keyword-stuffed resume.
