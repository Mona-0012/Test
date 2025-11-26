 # 1) New: if model is Gemini, use GeminiCode and return directly
    if str(self.LLM_MODEL).lower().startswith("gemini"):
        logger.info("Using Gemini LLM via GeminiCode wrapper")

        # self.messages_prompt is a list of messages; build a single prompt string
        prompt_text = "\n".join(
            m.get("content", "") for m in self.messages_prompt
        )

        gemini_client = GeminiCode(prompt=prompt_text)
        result = gemini_client.optimize(prompt_text)

        # Normalize to string for the rest of the pipeline
        if isinstance(result, str):
            return result.strip()
        else:
            # if GeminiCode returns parsed JSON, convert to text
            return json.dumps(result)
