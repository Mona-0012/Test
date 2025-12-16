    15. **COUNTING & TOTALS RULE**:
       - IF User asks "Total number" / "Grand Total":
         Use `SumAccum<INT> @@total;` -> `ACCUM @@total += 1` -> `PRINT @@total;`
       - IF User asks "Count per person" / "How many for each":
         Use `SumAccum<INT> @count;` -> `ACCUM p.@count += 1` -> `PRINT result[result.customer_id, result.@count];`
       - NEVER use generic `SELECT p.attr` syntax. ALWAYS use `PRINT Set[Set.attr]` to show columns.
