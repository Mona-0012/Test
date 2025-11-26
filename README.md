
text = exec_resp.text

try:
    # Try reading JSON directly
    tg_result = exec_resp.json()
except:
    # If not JSON, return plain text
    tg_result = text

return {
    "status": "success",
    "clean_gsql": query,
    "tg_result": tg_result
}
