curl -X POST "http://127.0.0.1:8000/generate_gsql" \
  -H "Content-Type: application/json" \
  -d "{\"market\":\"US\", \"nlp\":\"Give me a GSQL query to count the total number of person vertices in the Australia graph.\", \"llm_type\":\"gemini\"}"
