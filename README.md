Accumulator Rules (Very Important):

1. Use @@ (global accumulator) when the NLP asks for:
   - total count
   - count all
   - number of X
   - sum of X
   - average of X
   - overall value
   - final single number
   In all these cases, output should be ONE final value, so use @@.

2. Use @ (vertex accumulator) only when NLP clearly says:
   - "for each ..."
   - "per person ..."
   - "per account ..."
   - "for every vertex ..."
   These cases need a separate value for every vertex, so use @.

3. If NLP is not clear whether it is per-vertex or global:
   Always choose @@ because global is safer and matches most user questions.

4. For simple vertex counting like "count persons", always use:
   SumAccum<INT> @@count;
   start = {Person.*};
   SELECT s FROM start:s ACCUM @@count += 1;
   PRINT @@count;
