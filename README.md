Update the existing /tg/get-config-tables API to be tab-aware.

Requirements:
- Do NOT change DB connection, utilities, logging, or error handling.
- Accept a query parameter: type=vertex or type=edge.

Behavior:
- If type=vertex:
  - Query Cloud SQL table `vertex_config`
  - SELECT vertex_name, primary_id, attributes
  - Parse attributes JSON
  - Return:
    {
      "vertices": [
        {
          "vertex_name": string,
          "primary_id": string,
          "attributes": list
        }
      ]
    }

- If type=edge:
  - Query Cloud SQL table `edge_config`
  - SELECT edge_name, from_vertex, to_vertex, directed, attributes
  - Parse attributes JSON
  - Return:
    {
      "edges": [
        {
          "edge_name": string,
          "from_vertex": string,
          "to_vertex": string,
          "directed": boolean,
          "attributes": list
        }
      ]
    }

- Remove old vertex_names / edge_names logic.
- Keep everything else unchanged.
