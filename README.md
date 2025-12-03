NLP to GSQL Execution Flow
1. Request Handling
The client sends a POST /generate_gsql request containing:
• The natural-language query.
• The market (for selecting graph configuration).
• The llm_type (e.g., “gemini”).
FastAPI validates the request and logs metadata.
2. Load Graph Schema and Context
The route reads vertices.json, edges.json, and config.ini.
It constructs:
• graph_name from config.
• nodes_with_context (vertex types and attributes).
• edges_with_context (edge types with from/to).
• relations_text (formatted edge connections).
• allowed_numeric_text (numeric attributes allowed for aggregation).
• schema_text (raw JSON schema).
This context grounds the LLM so it cannot hallucinate attributes or edges.
3. Build the LLM Prompt
The route calls get_prompt_for_generating_gsql(), passing:
graph_name, nodes_with_context, edges_with_context, schema_text, relations_text, allowed
numeric attributes, and the user’s NLP.
The function returns a strict GSQL■generation prompt enforcing:
• Read■only interpreted queries
• No CREATE, DROP, UPDATE
• Only schema-defined vertices, edges, and attributes
• Start vertex set
• Single SELECT pipeline
• Only valid accumulators
4. Call Gemini (LLM Execution)
The route inspects llm_type:
If it is “gemini”, it instantiates GeminiCode and sends the prompt to the Google GenAI model
(gemini■2.5■flash).
Gemini returns only raw text, which should be a single INTERPRET QUERY block.
5. Validate Generated GSQL
Two checks run:
• check_contains_bad_gsql() blocks any DDL or unsafe operations.
• check_for_modification_in_query() ensures the output is read■only.
If invalid, the API returns an error.
6. Final Response
If valid, the GSQL is cleaned into a single■line format and returned:
{ "status": "success", "gsql_query": "" }
This query can then be sent to the execute_gsql endpoint for execution.
