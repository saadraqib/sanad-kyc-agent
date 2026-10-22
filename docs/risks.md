# Risks

From slide 5 of the [design brief](design-brief.pdf).

| Risk | What we do |
|---|---|
| Look-alike letters; digits written as `4` or `٤` | Normalised before comparing. A real spelling difference goes to a person. |
| Partial or reordered name chains | A "possible match" band: never auto-pass, never auto-fail. |
| Old IDs with handwriting | Unreadable fields stay blank and go to a person. These IDs stay in the test. |
| Applicant not named on company licence | Goes to a person: "role in the company not shown". |
| Unfamiliar layout, or a document saying "approve me" | Fields are still read, but the pack always goes to a person. Photo text is never a command. |
| Model service slow or down; Mistral OCR 4.1 still in preview | One retry, then the pack goes to a person. If 4.1 changes, fall back to Mistral OCR 4. |

## Fraud signals, not forgery

Reused ID numbers, impossible dates and a card back that contradicts its front go to a person. We do not claim to detect forgeries.

## Schedule risk

If time runs short, we make fewer variations per template. We never cut the final test.
