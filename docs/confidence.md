# Confidence

Every field gets one score: the chance that the field is right. The router (step 8) uses these scores to decide whether a pack can pass without a person.

> Design stage (22 Oct). No results yet. The numbers below show how the score will be read, not measured results.

## What goes into the score

Three signals, combined in code into one score:

1. **Reader agreement.** Do Gemini and Mistral OCR return the same value after normalising? The readers come from different companies, so their mistakes rarely match. Agreement is strong evidence the value is right.
2. **Format checks.** Does the value have the right shape for the field? For example, the right number of digits for an ID number, or a real date for an expiry.
3. **Photo quality.** How sharp and clear the crop was: blur, glare, cut corners, handwriting.

A field either reader marked "unreadable" stays blank and goes to a person, whatever its score.

## Calibration

A raw score isn't a probability until we check it against data. We calibrate in code on fictional packs that we checked by hand.

The 120 fictional packs are split by person, so no identity repeats across splits:

| Split | Packs | Use |
|---|---|---|
| Build | 30 | Develop the pipeline |
| Calibrate | 30 | Map raw signals to a score; lock the threshold |
| Final test | 60 | Run once, after the threshold is locked |

The Kurdistan tax card appears only in the final test.

### Example: what "98%" means

A field with a score of 98% belongs to a group of similar fields. In calibration, **196 of 200** of them were right. Of every 100 fields given that score, we expect about 2 to be wrong.

The 200 are fields, not packs. Every pack holds many fields, so 30 calibration packs give enough fields to fill groups of this size.

## Checking it: is 90% really 90%?

Calibration uses the 30 calibration packs. We check it on the 60 final-test packs:

1. Group the test fields by their score.
2. For each group, count how many fields were actually right.
3. Compare: fields scored about 90% should be right about 90% of the time.

We publish the raw counts for each group, not just a summary number.

## How the threshold is used

- The threshold is locked on the calibration packs, **before** the final test is run.
- A pack passes only if every field clears the threshold **and** every cross-check passes.
- The target is **0 wrong packs passed**: every planted error must reach a person. A low score always means a person looks at the field.
