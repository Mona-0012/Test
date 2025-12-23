I need to add two new FastAPI endpoints for TigerGraph.

Requirements:
1. Do NOT touch or modify my existing BQ APIs. They are already working.
2. Create two new GET APIs:
      - /tg/get-vertex-tables
      - /tg/get-edge-tables
3. Both APIs must call TigerGraph’s schema endpoint:
      GET http://<TG-IP>:14240/gsqlserver/gsql/schema?graph=<GRAPH_NAME>
4. Use the Bearer token: <YOUR_TOKEN>
5. Vertex API should return:
      { "tables": [list of VertexTypes.Name] }
6. Edge API should return:
      { "tables": [list of EdgeTypes.Name] }
7. If the schema fetch fails, raise HTTPException(500)
8. Keep the code clean, readable, and production-safe:
      - shared function fetch_tg_schema()
      - correct error handling
      - no duplication

Generate only the backend code for these two APIs using FastAPI and Python.
