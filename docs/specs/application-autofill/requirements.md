# Application autofill

## Summary

JDash42 applies to jobs from his own Chrome and Opera GX browsers and spends most of the time retyping facts he already knows. This feature is a browser extension that fills stored facts on the application in front of him — built-in pieces plus facts he adds — composing or splitting name, phone, address, and date of birth to match the page, without inventing text, submitting the form, overwriting a field that already has a value, or letting websites read the stored profile. Chrome and Opera GX each keep their own local profile. He still decides what to send and presses Submit himself.

## User stories

### Enter the built-in profile

As JDash42, I want to enter my reusable application facts on the extension’s options page, so that I type each built-in fact once and every later fill reuses it.

#### Acceptance criteria

WHEN the user opens the extension’s options page
THE SYSTEM SHALL let the user view, enter, and save these built-in profile facts: first name; middle name (optional); last name; date of birth; email; phone country code; phone national number; street numbers; address details for apartment number or similar; city; state/region; postal code; country; neighbourhood / localidad; LinkedIn URL; GitHub URL; portfolio or personal-site URL; US work authorization; whether US sponsorship is needed; gender; race/ethnicity; veteran status; disability status; a decline-to-answer choice for each of those EEO facts; one canned “how did you hear about us” value; and one resume file

WHEN the user saves name, phone, address, or date of birth
THE SYSTEM SHALL store that fact as the pieces listed above so a later fill can compose or split it to match the page

WHEN the user saves a profile fact
THE SYSTEM SHALL not store a second copy of that fact that can disagree with the saved pieces

WHEN the user changes a stored fact on the options page and saves
THE SYSTEM SHALL use the updated fact on fills after that save

WHEN the user starts a new session in the same browser after saving a profile
THE SYSTEM SHALL still have those saved facts available for filling

WHEN the user saves the profile
THE SYSTEM SHALL save and later reuse it without requiring a passphrase

WHEN the user saves a profile in Chrome
THE SYSTEM SHALL keep that profile only in Chrome and shall not make it available in Opera GX

WHEN the user saves a profile in Opera GX
THE SYSTEM SHALL keep that profile only in Opera GX and shall not make it available in Chrome

### Add facts as new questions appear

As JDash42, I want a basic, non-stylized menu to add facts and the questions they map to, so the profile can grow when applications ask something I do not already store.

#### Acceptance criteria

WHEN the user opens the extension’s options page
THE SYSTEM SHALL present a basic, non-stylized menu to add a fact and the question or questions that fact maps to

WHEN the user adds a fact and the questions it maps to and saves
THE SYSTEM SHALL store that fact and those mappings with the profile and use them on fills after that save

WHEN a fill runs and a field has no value and that field’s label, accessible name, or placeholder is a question the user recorded for a stored user-added fact
THE SYSTEM SHALL write that stored value into the field as if the user had typed or selected it

WHEN a fill runs and a field’s label, accessible name, and placeholder are not the question the user recorded for a stored user-added fact
THE SYSTEM SHALL not fill that field from that user-added fact

WHEN the user has not added any extra facts
THE SYSTEM SHALL still fill from the built-in facts

### Start auto-mode from the extension icon

As JDash42, I want clicking the extension icon to turn auto-mode on and fill the current tab only when it is a job application, so that I can start a hunting session without filling login, checkout, or settings pages.

#### Acceptance criteria

WHEN the user clicks the extension icon
THE SYSTEM SHALL turn auto-mode on

WHEN the user clicks the extension icon and the current tab is a job application (an apply-style page or a page of application-style fields, not a login, checkout, or settings form)
THE SYSTEM SHALL fill that tab’s fields that have no value and that map cleanly to stored profile facts

WHEN the user clicks the extension icon and the current tab is not a job application
THE SYSTEM SHALL not fill any fields on that tab

WHEN auto-mode is on
THE SYSTEM SHALL keep auto-mode on until the user turns it off from the popup or the browser starts a new session

### Auto-fill while job hunting

As JDash42, I want auto-mode that fills job applications without an extra click after I turn it on, so that a hunting session is faster than clicking on every page, without filling arbitrary web forms, and so that I must turn it on again after each browser restart.

#### Acceptance criteria

WHEN the browser starts a new session
THE SYSTEM SHALL set auto-mode off

WHEN auto-mode is off
THE SYSTEM SHALL not fill any fields

WHEN the user opens the extension popup
THE SYSTEM SHALL present a control that turns auto-mode on or off

WHEN the user turns auto-mode off from the popup
THE SYSTEM SHALL leave auto-mode off and not fill fields until auto-mode is turned on again

WHEN auto-mode is on and the current tab is a job application (an apply-style page or a page of application-style fields, not a login, checkout, or settings form)
THE SYSTEM SHALL fill fields that have no value and that map cleanly to stored profile facts without requiring a further click

WHEN auto-mode is on and the current tab is not a job application
THE SYSTEM SHALL not fill any fields

WHEN auto-mode is on and the user advances to another step of a job application
THE SYSTEM SHALL fill fields on that step that have no value and that map cleanly to stored profile facts without requiring another click

### Match fields on any application page

As JDash42, I want fills to work from the page’s own field cues on whatever application is in front of me, so that I do not have to know which job board a company uses.

#### Acceptance criteria

WHEN a fill runs
THE SYSTEM SHALL identify fields from the page’s labels, accessible names, placeholders, and similar cues

WHEN a fill runs on a job application hosted by a job board the user did not name or configure
THE SYSTEM SHALL still fill fields that have no value and that map cleanly to stored facts

WHEN a fill runs and a field has no value and maps cleanly to a stored profile fact
THE SYSTEM SHALL write that stored value into the field as if the user had typed or selected it, so the page treats the field as filled

WHEN a fill runs and a choice field has no value and maps cleanly to stored US work authorization, sponsorship, an EEO fact, or the canned “how did you hear about us” value
THE SYSTEM SHALL set that choice as if the user had selected it

### Fill name by how many name fields are on the page

As JDash42, I want name fields filled from the stored pieces according to how many name fields the page presents, so that two fields are never treated as a full-name box, and so that four or more name fields are filled only from exact English or Spanish labels.

#### Acceptance criteria

WHEN a fill runs and the page presents exactly one field asking for the applicant’s first, middle, last, or full name
THE SYSTEM SHALL fill that field with the composed stored name, including middle name only if it is stored, if that field has no value

WHEN a fill runs and the page presents exactly two fields asking for the applicant’s first, middle, last, or full name
THE SYSTEM SHALL fill each of those fields that has no value as first/name and last name from the stored pieces

WHEN a fill runs and the page presents exactly two fields asking for the applicant’s first, middle, last, or full name
THE SYSTEM SHALL not write a composed full name into either field

WHEN a fill runs and the page presents exactly three fields asking for the applicant’s first, middle, last, or full name
THE SYSTEM SHALL fill each of those fields that has no value as first name, middle name, and last name from the stored pieces

WHEN a fill runs and the page presents four or more fields asking for the applicant’s first, middle, last, or full name
THE SYSTEM SHALL not use the one-, two-, or three-field name fills

WHEN a fill runs, the page presents four or more fields asking for the applicant’s first, middle, last, or full name, and a field’s label is exactly `first name` or `primer nombre`
THE SYSTEM SHALL fill that field from the stored first name if that field has no value

WHEN a fill runs, the page presents four or more fields asking for the applicant’s first, middle, last, or full name, and a field’s label is exactly `last name` or `apellido`
THE SYSTEM SHALL fill that field from the stored last name if that field has no value

WHEN a fill runs, the page presents four or more fields asking for the applicant’s first, middle, last, or full name, and a field’s label is exactly `middle name` or `segundo nombre`
THE SYSTEM SHALL fill that field from the stored middle name if that field has no value

WHEN a fill runs, the page presents four or more fields asking for the applicant’s first, middle, last, or full name, and a field’s label is not exactly `first name`, `primer nombre`, `last name`, `apellido`, `middle name`, or `segundo nombre`
THE SYSTEM SHALL not fill that field from the stored first, middle, or last name

### Compose and split phone, address, and date of birth

As JDash42, I want those facts stored in pieces and shaped to the page, so that I do not maintain duplicate strings that can drift.

#### Acceptance criteria

WHEN a fill runs and the page has a single phone field that has no value and country code and national number are stored
THE SYSTEM SHALL fill that field by joining those pieces

WHEN a fill runs and the page has a country-code field and/or a national-number field
THE SYSTEM SHALL fill each of those fields that has no value from the matching stored piece

WHEN a fill runs and the page has a single street or address-line field that has no value
THE SYSTEM SHALL fill that field by composing the stored street-numbers piece and the stored address-details piece that exist

WHEN a fill runs and the page has a street-numbers field and/or an address-details field (apartment number or similar)
THE SYSTEM SHALL fill each of those fields that has no value from the matching stored piece

WHEN a fill runs and the page has a neighbourhood or localidad field that has no value and neighbourhood / localidad is stored
THE SYSTEM SHALL fill that field from the stored neighbourhood / localidad

WHEN a fill runs and the page has a single full-address field (not a street-only or address-line field) that has no value and that maps cleanly to stored address pieces other than neighbourhood / localidad
THE SYSTEM SHALL fill that field by composing those stored pieces

WHEN a fill runs and the page has a city, state/region, postal code, and/or country field
THE SYSTEM SHALL fill each of those fields that has no value from the matching stored piece

WHEN a fill runs and the page has a single date-of-birth field that has no value and date of birth is stored
THE SYSTEM SHALL fill that field from the stored date of birth

WHEN a fill runs and the page has a month, day, and/or year field for date of birth and date of birth is stored
THE SYSTEM SHALL fill each of those fields that has no value from the stored date of birth

### Attach the chosen resume

As JDash42, I want the resume file I chose on the options page attached when an application asks for a resume or CV, so that I do not hunt for the file on every application.

#### Acceptance criteria

WHEN a fill runs, the user has chosen a resume file, that file is available, and the page has a resume or CV upload field that has no file
THE SYSTEM SHALL attach that file to the upload field

WHEN a fill runs and the page has no resume or CV upload field
THE SYSTEM SHALL not attach a file

WHEN a fill runs and the page has a file upload that is not a resume or CV upload
THE SYSTEM SHALL not attach the resume file to that upload

WHEN a fill runs and a resume or CV upload field already has a file
THE SYSTEM SHALL leave that upload field unchanged

### Leave filled fields alone

As JDash42, I want a fill to leave fields that already have a value unchanged and only complete remaining blanks, so that a later icon click or auto-mode pass does not erase what is already on the page.

#### Acceptance criteria

WHEN a fill runs and a field already has a value
THE SYSTEM SHALL leave that field unchanged

WHEN a later fill runs from an icon click or from auto-mode and some fields already have values
THE SYSTEM SHALL fill remaining fields that have no value and that map cleanly to stored profile facts, including attaching the chosen resume file to a resume or CV upload that still has no file

### Keep judgment and submission with the user

As JDash42, I want fills to reuse only what I already saved and never submit for me, so that I still review the page and press Submit myself.

#### Acceptance criteria

WHEN a fill runs and a field’s mapping to the profile is missing or ambiguous
THE SYSTEM SHALL leave that field unchanged

WHEN a fill runs and filling a field would require generating, paraphrasing, or otherwise inventing text that is not already stored
THE SYSTEM SHALL leave that field unchanged

WHEN a fill runs and a field is an open-ended prompt for original writing (cover letter, why the applicant wants the job, tell-us-about-a-time, or similar) and no stored fact is mapped to that question
THE SYSTEM SHALL leave that field unchanged

WHEN a fill runs
THE SYSTEM SHALL not activate Submit, Next, Continue, or an agree-to-terms control when doing so would advance or complete the application

WHEN setting a stored value requires using a widget such as a dropdown
THE SYSTEM SHALL set the stored value through that widget without advancing or completing the application

### Keep the profile on the user’s machine

As JDash42, I want the stored profile to stay on my machine, so that websites and other extensions cannot read it, even though a filled page will see the values written into its own fields.

#### Acceptance criteria

WHEN the profile is saved
THE SYSTEM SHALL keep the stored profile on the user’s machine so that web pages cannot read it

WHEN the profile is saved
THE SYSTEM SHALL keep the stored profile so that other extensions cannot read it

WHEN the extension runs
THE SYSTEM SHALL not send profile contents off the user’s machine, including through analytics, crash reports, or other remote calls that include profile contents

WHEN the user loads the extension from a local folder into Chrome
THE SYSTEM SHALL make the options page, the auto-mode control, and filling available in Chrome

WHEN the user loads the extension from a local folder into Opera GX
THE SYSTEM SHALL make the options page, the auto-mode control, and filling available in Opera GX

## Edge cases and errors

WHEN a fill runs and no profile facts are stored
THE SYSTEM SHALL leave every field unchanged

WHEN a fill runs and only some profile facts are stored
THE SYSTEM SHALL fill fields that have no value and that map cleanly to stored facts and leave the rest unchanged

WHEN a fill runs and middle name is not stored
THE SYSTEM SHALL leave middle-name fields unchanged

WHEN a fill runs and the page presents preferred-name, legal-name, or name-as-on-passport fields
THE SYSTEM SHALL not count those fields toward the one-, two-, three-, or four-or-more name fills

WHEN a fill runs and the page asks for a name variant that is not stored as its own fact (preferred name, legal name, or name as on passport)
THE SYSTEM SHALL leave that field unchanged

WHEN a fill runs and the page asks for a date other than date of birth and no stored fact is mapped to that question
THE SYSTEM SHALL leave that field unchanged

WHEN a fill runs and “how did you hear about us” is a free-text field and no stored fact is mapped to that question
THE SYSTEM SHALL leave that field unchanged

WHEN a fill runs and “how did you hear about us” is a choice field that has no value and that maps cleanly to the stored canned value
THE SYSTEM SHALL select that stored value

WHEN a fill runs and the page asks about work authorization or sponsorship for a country other than the United States and no stored fact is mapped to that question
THE SYSTEM SHALL leave those fields unchanged

WHEN a fill runs and street numbers are not stored
THE SYSTEM SHALL leave street-numbers fields unchanged

WHEN a fill runs and address details (apartment number or similar) are not stored
THE SYSTEM SHALL leave address-details fields unchanged

WHEN a fill runs and neighbourhood / localidad is not stored
THE SYSTEM SHALL leave neighbourhood and localidad fields unchanged

WHEN a fill runs, the page has a resume or CV upload field, and the user has not chosen a resume file
THE SYSTEM SHALL leave the upload field unchanged

WHEN a fill runs, the page has a resume or CV upload field, the user has chosen a resume file, and that file is not available
THE SYSTEM SHALL leave the upload field unchanged

WHEN auto-mode is on and the current tab is a login, checkout, or settings form that is not a job application
THE SYSTEM SHALL not fill any fields

WHEN auto-mode is off and the current tab is a job application
THE SYSTEM SHALL not fill any fields until auto-mode is turned on

WHEN a fill runs and a field maps to both a built-in fact and a user-added fact
THE SYSTEM SHALL leave that field unchanged

## Out of scope

- Generating new sentences, paraphrases, cover letters, or emails
- Storing past open-ended answers or suggesting a closest match to a written question
- A career database or a tailored resume per job
- Submitting applications, applying on the user’s behalf, or running unattended
- Firefox, Safari, and mobile browsers
- A website, hosted app, or server backend
- Scraping or searching job listings
- Publishing in the Chrome Web Store or the Opera add-ons catalog
- Syncing the profile across devices or to the cloud
- Sharing or syncing the profile between Chrome and Opera GX
- Protecting the profile with a passphrase, or defending against disk theft and malware
- Job-board-specific adapters
- Built-in compose/split for dates other than date of birth
- Built-in preferred name, legal name, or name as on passport
- Built-in filling of free-text “how did you hear about us” answers
- Matching a user-added fact by similar wording, substring, regex, machine learning, or cues other than the field’s label, accessible name, and placeholder
- A stylized design system for the options page or the add-fact menu
- A larger name-label translation table than the listed English/Spanish pairs
- Filling a login, checkout, settings, or other non-application page from the extension icon
- Turning auto-mode off from the extension icon
- Overwriting a field that already has a value, including a resume or CV upload that already has a file
- Using a keyword-stuffed CV as a substitute for this filler
