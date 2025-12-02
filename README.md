def load_tg_config(marketname: str):
    config = load_config(marketname)   # reuse existing loader
    tg = config["TIGER GRAPH"]         # section name from your config.ini

    return {
        "host": tg.get("host"),
        "username": tg.get("username"),
        "password": tg.get("password"),
        "graph": tg.get("graph")
    }
