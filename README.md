text = exec_resp.text

# 1) Try direct JSON
try:
    tg_result = exec_resp.json()
except:
    # 2) If that fails, try to cut out the JSON part from the text
    start = text.find("{")
    end = text.rfind("}")

    if start != -1 and end != -1:
        try:
            json_part = text[start:end+1]
            tg_result = json.loads(json_part)
        except:
            tg_result = {"raw_response": text}
    else:
        tg_result = {"raw_response": text}

return {
    "status": "success",
    "clean_gsql": query,
    "tg_result": tg_result
}
