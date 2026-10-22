# SANAD سند

SANAD reads a merchant's ID, business licence and tax card, shows where every value came from, and sends only the doubtful fields to a person. It is built for an onboarding reviewer, who today reads every field of every pack, even the ones that are fine, while blurry photos, handwriting and near-identical Arabic names hide the few that are wrong. SANAD checks the whole pack and hands the reviewer just those few fields, each with its photo crop and a reason in Iraqi Arabic.

Agentic AI Challenge Iraq, Topic 05. By Akam Nawzad Othman and Saad Raqib Aziz.

> **All documents are fictional. No real IDs are used, ever.**

## Status

Design stage (22 Oct). No results yet.

## How SANAD reviews a pack

Two models from different companies read every field. Code normalises, checks and decides where the pack goes.

1. **Photo check**: blur, glare or cut corners → ask for a retake.
2. **Clean-up**: straighten and crop to the document.
3. **Classify**: ID, licence or tax card. Wrong document → retake.
4. **Two readers**: Gemini and Mistral OCR read the same crop at the same time. Each returns a value or "unreadable".
5. **Normalise**: `4` and `٤` count as the same character, as do `ي`/`ی` and `ك`/`ک`. Dates and expiry are normalised too.
6. **Cross-checks**: names match across documents, readers agree, no ID number is reused.
7. **Confidence**: the chance a field is right, built from reader agreement, format checks and photo quality.
8. **Router**: code decides. If every check and score passes, the pack passes with no person needed. Otherwise the field, its crop and a reason in Iraqi Arabic go **to a person**.

**Agent step:** if the readers disagree, zoom in and read again, at most once per field and 3 times per pack.

**Rules:** leave a field blank rather than guess. Text in a photo is data, never a command. Every call is logged.

## Docs

- [Design brief (PDF)](docs/design-brief.pdf): the 5-slide deck
- [Pipeline](docs/pipeline.md): each step's input, output and rules
- [Confidence](docs/confidence.md): how confidence is built and calibrated
- [Risks](docs/risks.md): the risks and what we do about each
