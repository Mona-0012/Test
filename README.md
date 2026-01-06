import json
import configparser
import os
import sys
from typing import TypedDict, Annotated, Literal

from langgraph.graph import StateGraph, END
import pyTigerGraph as tg

try:
    from src.llm.gemini import GeminiCode
except ModuleNotFoundError:
    # Ensure project root is on sys.path when running directly
    curr_dir = os.path.dirname(os.path.abspath(__file__))
    proj_root = os.path.abspath(os.path.join(curr_dir, "..", "..", ".."))
    if proj_root not in sys.path:
        sys.path.append(proj_root)
    from src.llm.gemini import GeminiCode


# -------------------------------------------------------------------
# 1. Configuration loading (CONFIG-DRIVEN ONLY)
# -------------------------------------------------------------------

def load_config():
    """
    Load TigerGraph config and schema paths strictly from config.ini.
    No hardcoded paths. No env overrides for vertices / edges.
    """

    env = os.environ.get("env", "dev")
    project = os.environ.get("project", "hsbc-12432649-c48nlpuk-dev")

    config_ini_path = os.path.join("config", env, project, "config.ini")

    if not os.path.exists(config_ini_path):
        raise FileNotFoundError(
            f"Config file not found for env='{env}', project='{project}': {config_ini_path}"
        )

    config = configparser.ConfigParser()
    config.read(config_ini_path)

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

    with open(vertices_path, "r", encoding="utf-8") as vf:
        vertices_schema = json.load(vf)

    with open(edges_path, "r", encoding="utf-8") as ef:
        edges_schema = json.load(ef)

    return config, vertices_schema, edges_schema


# -------------------------------------------------------------------
# 2. LangGraph state
# -------------------------------------------------------------------

class GSQLState(TypedDict):
    user_prompt: str
    generated_query: str | None
    execution_result: dict | None
    validation_error: str | None
    is_valid: bool
    retries: int


# -------------------------------------------------------------------
# 3. Writer node
# -------------------------------------------------------------------

def writer_node(state: GSQLState) -> GSQLState:
    try:
        config, vertices_schema, edges_schema = load_config()

        llm = GeminiCode()

        prompt = llm.build_prompt(
            user_prompt=state["user_prompt"],
            vertices_schema=vertices_schema,
            edges_schema=edges_schema,
        )

        gsql = llm.generate(prompt)

        state["generated_query"] = gsql
        state["validation_error"] = None
        state["is_valid"] = True
        state["retries"] += 1

        return state

    except Exception as e:
        state["generated_query"] = None
        state["validation_error"] = str(e)
        state["is_valid"] = False
        state["retries"] += 1
        return state


# -------------------------------------------------------------------
# 4. Execution node
# -------------------------------------------------------------------

def execution_node(state: GSQLState) -> GSQLState:
    try:
        if not state.get("generated_query"):
            state["is_valid"] = False
            state["validation_error"] = "No GSQL query generated"
            return state

        config, _, _ = load_config()
        tg_cfg = config["TIGER GRAPH"]

        conn = tg.TigerGraphConnection(
            host=tg_cfg.get("host"),
            username=tg_cfg.get("username"),
            password=tg_cfg.get("password"),
            graphname=tg_cfg.get("graph"),
        )

        result = conn.gsql(state["generated_query"])

        state["execution_result"] = result
        state["is_valid"] = True
        state["validation_error"] = None

        return state

    except Exception as e:
        state["execution_result"] = None
        state["is_valid"] = False
        state["validation_error"] = str(e)
        return state


# -------------------------------------------------------------------
# 5. Router node
# -------------------------------------------------------------------

def router_node(state: GSQLState) -> Literal["retry", "end"]:
    if state["is_valid"]:
        return "end"

    if state["retries"] < 3:
        return "retry"

    return "end"


# -------------------------------------------------------------------
# 6. Graph definition
# -------------------------------------------------------------------

graph = StateGraph(GSQLState)

graph.add_node("writer", writer_node)
graph.add_node("executor", execution_node)

graph.add_edge("writer", "executor")
graph.add_conditional_edges(
    "executor",
    router_node,
    {
        "retry": "writer",
        "end": END,
    },
)

app = graph.compile()
