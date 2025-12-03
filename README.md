# --- FINAL HARD CONSTRAINTS: ACCUMULATOR SAFETY (DO NOT VIOLATE) ---

1. For any accumulator that holds vertices, you MUST use one of:

   SetAccum<VERTEX>
   SetAccum<VERTEX<vertex_label>>

   These two formats are the ONLY allowed formats for vertex-holding accumulators.

2. The following forms are ALWAYS forbidden in the final GSQL (never output them):

   SetAccum<person>
   SetAccum<device>
   SetAccum<accountnumber>
   SetAccum<vertex_label>            # any vertex label inside SetAccum<...>

   If any of these appear, REWRITE the line using SetAccum<VERTEX<...>> before output.

3. Always update MaxAccum using:     @@accum += value;
   NEVER call max() manually.        # max(@@accum, x) is invalid GSQL.

4. Before sending output, the model MUST self-scan the query and:
   - Replace every "SetAccum<xxx>" with "SetAccum<VERTEX<xxx>>" if xxx is a vertex type.
   - Remove any function calls like max(...) for accumulators.
   - Ensure all accumulator declarations appear at the top of the query body.

# --- END OF FINAL HARD CONSTRAINTS ---
