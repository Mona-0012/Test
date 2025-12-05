Use this TigerGraph documentation as the first reference for correctness:
https://docs.tigergraph.com/gsql-ref/4.2/

I have a long GSQL-generation prompt with strict rules. My generated GSQL still fails with errors such as:

+= used outside ACCUM/POST-ACCUM

invalid alias names (e.g., p2_s)

lowercase end instead of END

SELECT blocks not matching GSQL traversal syntax

SQL-like GROUP BY/HAVING/JOIN

unbalanced { / }

Your tasks:

Read my prompt file and check whether the rules are strong enough to prevent syntactic errors.

Suggest the minimal additional rules needed so the LLM always generates valid TigerGraph GSQL.

Review the failing GSQL and fix it according to the documentation and my rules.

Ensure future generations strictly follow TigerGraph traversal SELECT syntax, accumulator rules, and edge direction rules.

Keep the response short, technical, and focused on rule-corrections + GSQL fixes.

If you want, I can also generate a compact rules-block that you can paste directly into your prompt file.

Chat
