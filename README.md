# --- GLOBAL ACCUM SAFETY (STRICT) ---
1. NEVER assign to a global accumulator using '=' inside ACCUM or POST-ACCUM.
   Forbidden:  @@x = value;
   Allowed only: @@x += value;

2. Global accumulators must be updated using incremental logic only.
   If logic requires replacing a value (like keeping a max), 
   rewrite using MaxAccum or vertex accumulators instead.

3. If the NLP requires “max”, “min”, “highest”, “lowest”, etc:
   - ALWAYS use MaxAccum or MinAccum.
   - NEVER compare and assign to @@acc manually.

4. Before output, scan the query:
   IF you find '@@... =', rewrite it to avoid equals-assign.
