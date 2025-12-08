def get_retry_prompt(
    graph_name: str,
    nodes: list[dict],
    edges: list[dict],
    original_text: str,
    invalid_gsql: str,
    tg_error: str,
    schema_text: str,
    allowed_numeric_text: str,
    relations_text: str,
):
    """
    Build the retry prompt using your existing GSQL prompt generator.

    Inputs required for retry:
    - original_text      (the NLP)
    - invalid_gsql       (what the LLM generated first)
    - tg_error           (error from TigerGraph)
    - all schema/prompt blocks exactly same as first call
    """

    # Build a new TASK string for the retry
    retry_task = f"""
The previous GSQL you generated for this NLP failed with a TigerGraph syntax/semantic error.
Your job is to FIX the GSQL and regenerate a syntactically valid TigerGraph GSQL query.

NATURAL LANGUAGE REQUEST:
{original_text}

PREVIOUS INVALID GSQL:
{invalid_gsql}

TIGERGRAPH ERROR MESSAGE:
{tg_error}

Follow ALL the same rules, schema, edge directions, accumulator restrictions,
and output rules exactly as specified in this prompt.  
Return ONLY the corrected GSQL query.
""".strip()

    # Reuse your existing generator exactly as-is
    return get_prompt_for_generating_gsql(
        graph_name=graph_name,
        nodes=nodes,
        edges=edges,
        task=retry_task,                   # <--- only this changes
        schema_text=schema_text,
        allowed_numeric_text=allowed_numeric_text,
        relations_text=relations_text,
    )
