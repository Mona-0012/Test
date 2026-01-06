def load_config():
    env = os.environ.get('env', 'dev')
    config_ini_path = os.path.join("config", env, "hsbc-12432649-c48nlpuk-dev", "config.ini")

    config = configparser.ConfigParser()
    if not os.path.exists(config_ini_path):
        raise FileNotFoundError("Config file not found: " + str(config_ini_path))

    config.read(config_ini_path)

    # ===================== ADD THIS BLOCK HERE =====================

    if "TIGER GRAPH" not in config:
        raise KeyError("[TIGER GRAPH] section missing in config.ini")

    tg_cfg = config["TIGER GRAPH"]

    vertices_path = tg_cfg.get("vertices_path")
    edges_path = tg_cfg.get("edges_path")

    if not vertices_path or not edges_path:
        raise ValueError(
            "Both 'vertices_path' and 'edges_path' must be set in [TIGER GRAPH]"
        )

    if not os.path.exists(vertices_path):
        raise FileNotFoundError(f"Vertices schema file not found: {vertices_path}")

    if not os.path.exists(edges_path):
        raise FileNotFoundError(f"Edges schema file not found: {edges_path}")

    with open(vertices_path, "r", encoding="utf-8") as f:
        vertices_schema = json.load(f)

    with open(edges_path, "r", encoding="utf-8") as f:
        edges_schema = json.load(f)

    # ===================== END BLOCK =====================

    return config, vertices_schema, edges_schema
