# Pipeline

How SANAD reviews one pack. Two models from different companies read every field. Code normalises, checks and decides where the pack goes. The running example is **pack M-0412**: one fictional merchant, 3 photos (unified ID card, business licence, tax card).

## Rules for every step

- **Blank, never guessed.** If a value can't be read, it stays blank and goes to a person.
- **Photo text is data.** Text on a document is never treated as an instruction, even if it says "approve me".
- **Every call logged.** Records are kept 5 years for audit.
- **Same photo, same result.** Each step is deterministic for a given input.

## Input

The photos of one pack:

- **ID**, one of two sets: the unified national card (front and back), or the old civil ID plus a nationality certificate.
- **Business licence**, Federal or Kurdistan Region.
- **Tax card**, Federal or Kurdistan Region.

These cover individual traders and companies. For a company, the person on the ID must be named on the licence.

## Steps

### 1. Photo check

- **Input:** the raw phone photos.
- **Output:** each photo is accepted, or a retake is requested.
- **Rules:** blur, glare or cut corners → ask for a retake.
- **Example:** the licence had glare, so it was retaken.

### 2. Clean-up

- **Input:** accepted photos.
- **Output:** one straightened image per photo, cropped to the document.
- **Example:** the ID photo was tilted and is now straight.

### 3. Classify

- **Input:** cleaned images.
- **Output:** a label for each image: ID, licence or tax card.
- **Rules:** if a document is missing or the wrong one was sent, ask for a retake.
- **Example:** photo 1 is the unified ID card.

### 4. Two readers

- **Input:** one crop per field.
- **Output:** two readings per field, one from each reader. Each reading is a value or "unreadable".
- **Rules:**
  - Reader A (Gemini 3.8 Flash) and reader B (Mistral OCR 4.1, preview) get **the same crop at the same time**.
  - The readers come from different companies, so their mistakes rarely match.
  - A reader that can't read a field returns "unreadable", not a guess.
  - If the service is slow or down: one retry, then the pack goes to a person.
- **Example:** ID number, reader A: `١٩٩٠١٢٣٤٥٦٧٨`, reader B: `199012345678`.

### 5. Normalise

- **Input:** raw readings from both readers.
- **Output:** readings in one standard form so they can be compared.
- **Rules:**
  - Arabic-Indic and Western digits count as the same character (`٤` = `4`).
  - Look-alike letters count as the same character: `ي`/`ی`, `ك`/`ک`.
  - Dates and expiry dates are put into one format.
  - Normalising only removes differences in how a character is written. A real spelling difference still goes to a person.
- **Example:** both readings become `199012345678`.

### 6. Cross-checks

- **Input:** normalised fields from all documents in the pack.
- **Output:** a pass or fail for each check, with a reason.
- **Rules:**
  - The two readers agree on each field.
  - Names match across the ID, licence and tax card. Names are 3- or 4-part chains in Arabic or Kurdish.
  - Partial or reordered name chains go into a "possible match" band. They never pass or fail automatically.
  - No ID number is reused across packs.
  - For a company, the applicant is named on the licence. If not: "role in the company not shown".
  - Impossible dates, or a card back that contradicts its front, are fraud signals and go to a person. SANAD does not claim to detect forgeries.
- **Example:** the ID number agrees, but the owner name on the licence differs (`علي حسين كاظم` vs `علي حسن كاظم`).

### Agent step: read again

- **Trigger:** the two readers disagree on a field.
- **Action:** zoom in on that field and read it again.
- **Limit:** at most once per field and 3 times per pack.

### 7. Confidence

- **Input:** each field's reader agreement, format checks and photo quality.
- **Output:** one score per field: the chance the field is right.
- **Rules:** calibrated in code. See [confidence.md](confidence.md).
- **Example:** 98%: in calibration, 196 of 200 similar fields were right.

### 8. Router

- **Input:** every check result and confidence score for the pack.
- **Output:** **PASS** or **TO A PERSON**.
- **Rules:**
  - Code decides, not a model.
  - PASS only if every check and every score passes the threshold. No person is needed.
  - Otherwise, each doubtful field goes to a person with its crop and a reason in Iraqi Arabic.
  - A document with an unfamiliar layout always goes to a person.
- **Example:** the name check failed, so the pack goes to a person with the reason `الاسم بالإجازة ما يطابق الهوية`.

## Output: the reviewer view

For every field: the value (or "left blank"), where it came from, and a status: **Checked**, or **Review** with the reason. See slide 1 of the [design brief](design-brief.pdf) for a mock-up.
