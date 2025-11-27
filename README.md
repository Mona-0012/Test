# CALL LLM BASED ON USER llm_type
logger.info("Calling LLM type: %s", req.llm_type)

if req.llm_type.lower() == "gemini":
    gemini_client = GeminiCode(prompt=prompt)
    gsql_raw = gemini_client.optimize(prompt)
    logger.info("Received Gemini response %s", str(gsql_raw)[:100])

elif req.llm_type.lower() == "openai":
    messages = [{"role": "user", "content": prompt}]
    gsql_raw = LLMConnector(messages, config=cfg).get_llm_response()
    logger.info("Received OpenAI response %s", gsql_raw[:100])

else:
    return JSONResponse(
        status_code=400,
        content={"status": "error", "message": f"Unsupported llm_type '{req.llm_type}'"}
    )
