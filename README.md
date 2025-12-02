RULE (FILTERS):
- Only add filters when the NLP clearly mentions a specific value.
- When a filter is needed, express it as a boolean condition in a WHERE clause on a vertex/edge alias, e.g.
  WHERE alias.<attribute> <operator> <value>
- Do NOT encode the value inside the vertex set literal (no patterns like {Type."value"} or {Type.value}).
- Never skip a filter when the NLP includes one.
- Never invent a filter when the NLP doesn’t say one.
