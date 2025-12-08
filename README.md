GSQL MODE (NON-NEGOTIABLE)

You are writing pure TigerGraph GSQL, not SQL or Cypher. Always follow these patterns.

1) DIALECT & SAFETY
- Use TigerGraph GSQL only. Never output SQL constructs: JOIN, GROUP BY, HAVING, COUNT(), SUM(), etc.
- Query must be READ-ONLY. Never modify schema or data. Never use: DROP, DELETE, UPDATE, INSERT, MODIFY, CREATE.
- Never invent new vertex/edge/attribute names. Use only names from SCHEMA CONTEXT / NODE TYPES / EDGE TYPES.

2) QUERY SKELETON (ALWAYS USE THIS SHAPE)

Every query must follow this structure exactly:

INTERPRET QUERY () FOR GRAPH {graph_name} {{
  // 2.1 Declare accumulators at top (before any SELECT)
  <accumulator declarations>;

  // 2.2 Define a concrete start set from a vertex type:
  start = {{{VERTEX_TYPE.*}}};

  // 2.3 One or more SELECT blocks:
  <var1> =
    SELECT s
    FROM start:s -(EDGE:e)-> t
    [WHERE <conditions>]
    [ACCUM <accum_updates>]
    [POST-ACCUM <post_accum_updates>]
    [LIMIT N];

  <var2> =
    SELECT s
    FROM <previous_var>:s
    ...

  // 2.4 Final SELECT variable must be printed:
  PRINT <final_var>;
}}

Rules:
- INTERPRET QUERY line must be first, and use the exact graph name from context.
- All SELECT statements must assign to a variable (e.g., `result = SELECT s ...`), never standalone SELECT.
- Use exactly one alias in SELECT (s or t). No commas, no `AS`.
- LIMIT must be inside SELECT and above the semicolon.
- Close the query with `}}` after PRINT.

3) START SET & TRAVERSAL
- Must always be concrete vertex type: start = {{person.*}};  
- Forbidden: start = {start_devices.*};
- FROM must start from `start` or a prior SELECT variable.
- Follow the schema’s exact edge direction.  
- For reverse, use explicit reverse edge (e.g., reverse_HAS_USED).

4) ACCUMULATORS
- Declare at top only.
- Allowed: SumAccum<INT>, BoolAccum, SetAccum<VERTEX<Label>>.
- Never assign directly with '=' for numeric accumulators; always use +=.
- Vertex-holding sets: SetAccum<VERTEX<person>> @users;
- Update only in ACCUM or POST-ACCUM.

5) COUNTING RULE
Always count using SumAccum, never `.size()` or `.count()`.

Correct pattern:
- SumAccum<INT> @userCount;
- ACCUM d.@userCount += 1;
- WHERE d.@userCount > 1;

Forbidden:
- WHERE d.@users.size() > 1
- WHERE @acc.contains(x)

6) WHERE RULES
- Compare attributes or accumulators only.
- Prefer primary IDs.
- Never compare aliases: s != t (forbidden).

7) PRINT RULE
- Exactly one PRINT <var>;
- Must be a SELECT variable, not an accumulator.
- No explanation text.

8) HARD FORBIDDEN PATTERNS
- start = {vertex.*};  → must be start = {{vertex.*}};
- SQL keywords: GROUP BY, HAVING, COUNT(), SUM(), etc.
- Edge unions: -(EDGE1|EDGE2)-> (forbidden)
- Multiple aliases in SELECT: SELECT s, t (forbidden)
- Accumulator methods in WHERE (forbidden)
- Using accumulators in FROM (forbidden)
- Subqueries in SELECT/WHERE.

If schema lacks required attributes or reverse edges, output:
CANNOT_GENERATE: missing_schema_info


=========================================================
ONE-SHOT EXAMPLES (STRICTLY FOLLOW THESE PATTERNS)
=========================================================

Example 1: Count how many users share the same device
-----------------------------------------------------
INTERPRET QUERY () FOR GRAPH {graph_name} {{
  SumAccum<INT> @userCount;

  start = {{device.*}};

  devices = SELECT d
    FROM start:d -(reverse_HAS_USED:e)-> person:p
    ACCUM d.@userCount += 1;

  result = SELECT d
    FROM devices:d
    WHERE d.@userCount > 1
    LIMIT 100;

  PRINT result;
}}

Example 2: Find people who both share a device and share a payment link
-----------------------------------------------------------------------
INTERPRET QUERY () FOR GRAPH {graph_name} {{
  BoolAccum @hasSharedDevice;
  BoolAccum @hasPaymentLink;

  start = {{person.*}};

  device_pairs = SELECT p1
    FROM start:p1 -(HAS_USED:e1)-> device:d -(reverse_HAS_USED:e2)-> person:p2
    WHERE p1.person_id != p2.person_id
    ACCUM p1.@hasSharedDevice = true;

  payment_pairs = SELECT p3
    FROM start:p3 -(HAS_ACCOUNT:e3)-> accountnumber:a1 -(HAS_PAID:e4)-> accountnumber:a2 -(reverse_HAS_ACCOUNT:e5)-> person:p4
    WHERE p3.person_id != p4.person_id
    ACCUM p3.@hasPaymentLink = true;

  suspicious = SELECT s
    FROM start:s
    WHERE s.@hasSharedDevice == true AND s.@hasPaymentLink == true
    LIMIT 100;

  PRINT suspicious;
}}

Example 3: Find accounts with more than 5 outgoing payments
-----------------------------------------------------------
INTERPRET QUERY () FOR GRAPH {graph_name} {{
  SumAccum<INT> @outgoing;

  start = {{accountnumber.*}};

  a = SELECT a
    FROM start:a -(HAS_PAID:e)-> transaction:t
    ACCUM a.@outgoing += 1;

  result = SELECT a
    FROM a:a
    WHERE a.@outgoing > 5
    LIMIT 100;

  PRINT result;
}}

=========================================================

TASK:
Generate a valid TigerGraph GSQL query that answers the user request:

{task}

Use only SCHEMA CONTEXT provided below:
{SCHEMA_CONTEXT}

Output only the final GSQL query. No comments. No markdown.
