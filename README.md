GSQL MODE (NON-NEGOTIABLE)
- Use PURE TigerGraph GSQL traversal syntax, NOT SQL-like SELECT.
- The query MUST start with:
  INTERPRET QUERY () FOR GRAPH <graph_name> {
  ...
  }

FORBIDDEN SQL-LIKE CONSTRUCTS
- Never use: GROUP BY, HAVING, JOIN, UNION, SELECT *, SELECT s, t, subqueries.
- Never use functions: COUNT(), SUM(), AVG(), MIN(), MAX() or any SQL aggregate.
- Never use comma-separated column lists in SELECT.
- If unsure, ALWAYS choose GSQL traversal SELECT, not SQL-like SELECT.

SELECT SHAPE (ONLY ALLOWED FORM)
- Every SELECT must match this pattern:

  <var> = SELECT <alias>
          FROM <start_set>:s -(EDGE:e)-> <target_type>:t
          [WHERE ...]
          [ACCUM ...]
          [POST-ACCUM ...]
          [ORDER BY ...]
          [LIMIT N];

- <alias> must be either s OR t (only one alias, no commas, no "AS").
- <start_set> is a vertex set variable or {vertex_type.*}, NOT an accumulator.

VERTEX SET vs ACCUMULATOR
- Any variable on the LEFT of "= SELECT" is a vertex set, NOT an accumulator.
  Forbidden pattern:
    SetAccum<...> X; 
    X = SELECT ...
- Do NOT declare SELECT targets with SetAccum / SumAccum / MapAccum / MaxAccum / MinAccum.

ACCUMULATOR DECLARATION RULES
- Declare ALL accumulators once, at the top of the query body, immediately after the opening "{", before any SELECT/FOREACH.

  Allowed examples:
    SumAccum<INT> @count;
    SetAccum<VERTEX> @vertices;
    SetAccum<VERTEX<person>> @@suspicious_people;

- Allowed forms for vertex-holding accumulators:
    SetAccum<VERTEX> @acc;
    SetAccum<VERTEX<vertex_label>> @acc;
- Forbidden forms:
    SetAccum<vertex_label> ...
    SetAccum<vertex<...>> ...
    SetAccum<device> ..., SetAccum<person> ..., SetAccum<accountnumber> ...

ACCUMULATOR UPDATE RULES
- Inside ACCUM / POST-ACCUM, update accumulators using "+=" only:
    @acc += value;
    @@acc += value;
- Never use "=" to assign a value to an accumulator inside ACCUM or POST-ACCUM.
- Never call max()/min()/sum() over accumulators; use MaxAccum/MinAccum instead when needed.

LOOP / FOREACH SAFETY
- Never declare any accumulator inside FOREACH, WHILE, or SELECT blocks.
  Forbidden patterns:
    FOREACH ... DO
      SetAccum<...> ...
    END;
- Inside loops you may ONLY update already-declared accumulators (+=), not create new ones.

GLOBAL ACCUMULATORS (@@)
- @@accumulators store global values or sets; they are NOT vertex sets.
- Never use @@name in FROM or WHERE IN clauses.
  Forbidden:
    FROM @@something:s ...
    WHERE t IN @@something
- If you need to filter using a set built from @@, first select it into a vertex set:

  example_set = SELECT s FROM <some_start>:s WHERE ... [ACCUM @@acc += s];
  -- Later use:
  WHERE t IN example_set;

DEVICE/ACCOUNT/PERSON STYLE (VERTEX SETS)
- To traverse from all vertices of a type, use:
    start = {person.*};
    start = {device.*};
    start = {accountnumber.*};

PRINT RULE
- Every query MUST end with exactly one PRINT statement:
    PRINT <vertex_set_or_global_acc>;
- For row outputs (lists of vertices), PRINT a vertex set variable.
- For single numeric outputs (total count/sum), PRINT @@global numeric accumulator.

READ-ONLY RULE
- Query must be read-only. Do NOT use: CREATE, UPDATE, DELETE, INSERT, DROP, MODIFY, INSTALL, RUN.
