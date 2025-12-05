# ACCUM DECLARATION SAFETY RULE:
- The model must declare ALL accumulators ONLY at the top of the INTERPRET QUERY block.
- Never declare SetAccum, SumAccum, MapAccum, or any accumulator inside FOREACH, WHILE, or SELECT blocks.
- Inside FOREACH, the model may only UPDATE accumulators (e.g., +=), never DECLARE them.
