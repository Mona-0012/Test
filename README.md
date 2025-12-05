# HARD FORBIDDEN RULE (NON-NEGOTIABLE):
The model MUST NEVER generate any accumulator declaration inside FOREACH, WHILE, or SELECT blocks.

FORBIDDEN PATTERNS:
- "SetAccum<" appearing after the word "FOREACH"
- "SumAccum<" appearing after the word "FOREACH"
- "MapAccum<" appearing after the word "FOREACH"

If the NLP requires a per-device or per-loop accumulator:
- Declare it at the top of the INTERPRET QUERY, then update it inside the loop.

If the model is unsure, it MUST choose: DO NOT DECLARE inside loop.
