except ValueError:
    cleaned = exec_resp.text.replace("\n", " ").replace("\r", " ")
    tg_result = {"raw_response": cleaned}
