Rules to enforce:
1. No SQL syntax (no SELECT *, no JOIN, no subqueries, no USING results from previous SELECT in FROM).
2. Each SELECT block must end with a semicolon.
3. Use only traversal patterns: src -(EDGE)-> tgt.
4. Use only vertex types, edge types, and attributes defined in vertices.json and edges.json.
5. Keywords must be uppercase (SELECT, FROM, WHERE, ACCUM, POST-ACCUM, PRINT).
6. ACCUM updates must use valid operators (+=, =, etc.) and never `=+`.
7. Do not merge tokens (PRINT @@var; not PRINT@@var;).
8. Do not make up vertex types, edges, or attributes.
9. No DML or schema operations: forbid CREATE, ALTER, UPDATE, DELETE, DROP.
10. Output GSQL must always be a read-only INTERPRET QUERY block.

Rewrite my existing prompt so it cleanly enforces all rules without being overly rigid. 
Focus on clarity, correctness, and preventing parse errors.
