# --- FINAL RULES: SELECT + AGGREGATION (STRICT) ---

1. Never use SQL-style aggregation. Forbidden anywhere:
   - COUNT(   SUM(   AVG(   MIN(   MAX(
   - GROUP BY
   - HAVING
   - SELECT s AS ..., SELECT s, t, any SELECT with commas or aliases.

2. Every SELECT must follow this shape only:
   <var> = SELECT s
           FROM <start>:s -(EDGE:e)-> <target>:t
           [WHERE ...]
           [ACCUM ...]
           [POST-ACCUM ...]
           [ORDER BY ...]
           [LIMIT N];

   Only ONE alias in SELECT (s OR t). No commas. No "AS".

3. All aggregation must use accumulators:
   - Use SumAccum<INT> and "ACCUM += 1" for counts.
   - Use MaxAccum/MinAccum with "POST-ACCUM += value" for max/min.
   - Never call COUNT(), SUM(), etc.

4. Before output, self-check:
   - Remove any COUNT()/SUM()/GROUP BY/HAVING.
   - Ensure every SELECT uses exactly one alias and no commas.
