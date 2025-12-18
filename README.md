from fastapi import FastAPI
import mysql.connector

app = FastAPI()

# ------------------------------------------------------
# DB CONNECTION
# ------------------------------------------------------
def get_db():
    return mysql.connector.connect(
        host="localhost",
        user="root",
        password="password",
        database="metadata_db"
    )

# ------------------------------------------------------
# SIMPLE id_key GENERATOR
# ------------------------------------------------------
def key(market, kind, name, attr):
    return f"{market}~{kind}~{name}~{attr}"

# ------------------------------------------------------
# API: Accept schema + insert metadata
# ------------------------------------------------------
@app.post("/upload-graph-schema")
async def upload_graph_schema(payload: dict):

    market   = payload.get("market")
    vertices = payload.get("vertices", [])
    edges    = payload.get("edges", [])

    conn = get_db()
    cur  = conn.cursor()

    # ======================================================
    # INSERT VERTEX METADATA
    # ======================================================
    for v in vertices:
        vname = v["name"]

        for attr_name, attr_type in v.get("attributes", {}).items():

            id_key = key(market, "vertex", vname, attr_name)

            # ---- vertex_config ----
            cur.execute("""
                INSERT INTO vertex_config (
                    id_key, market, vertex_name, attribute_name, attribute_type, description
                ) VALUES (%s, %s, %s, %s, %s, %s)
            """, (
                id_key,
                market,
                vname,
                attr_name,
                attr_type,
                f"Attribute {attr_name} of vertex {vname}"
            ))

            # ---- vertex_context ----
            cur.execute("""
                INSERT INTO vertex_context (
                    id_key, embedding, raw_text
                ) VALUES (%s, %s, %s)
            """, (
                id_key,
                None,
                f"Vertex {vname} attribute {attr_name} ({attr_type})"
            ))

    # ======================================================
    # INSERT EDGE METADATA
    # ======================================================
    for e in edges:
        ename = e["name"]
        fv    = e["from"]
        tv    = e["to"]

        for attr_name, attr_type in e.get("attributes", {}).items():

            id_key = key(market, "edge", ename, attr_name)

            # ---- edge_config ----
            cur.execute("""
                INSERT INTO edge_config (
                    id_key, market, edge_name, from_vertex, to_vertex,
                    attribute_name, attribute_type, description
                ) VALUES (%s, %s, %s, %s, %s, %s, %s, %s)
            """, (
                id_key,
                market,
                ename,
                fv,
                tv,
                attr_name,
                attr_type,
                f"Edge {ename} attribute {attr_name}"
            ))

            # ---- edge_context ----
            cur.execute("""
                INSERT INTO edge_context (
                    id_key, embedding, raw_text
                ) VALUES (%s, %s, %s)
            """, (
                id_key,
                None,
                f"Edge {ename} connects {fv} to {tv}, attribute {attr_name} ({attr_type})"
            ))

    conn.commit()
    cur.close()
    conn.close()

    return {"message": "Graph schema uploaded successfully"}
