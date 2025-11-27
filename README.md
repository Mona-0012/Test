# CALL LLM (Gemini only)
logger.info("Calling Gemini LLM")

gemini_client = GeminiCode(prompt=prompt)
gsql_raw = gemini_client.optimize(prompt)

logger.info("Received Gemini response %s", str(gsql_raw)[:100])
