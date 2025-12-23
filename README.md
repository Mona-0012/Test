@pg_router.get("/tg/get-config-tables")
def tg_get_config_tables():
    try:
        conn = psycopg2.connect(
            host="localhost",
            dbname="metadata_db",
            user="postgres",
            password="password"
        )
        cur = conn.cursor()

        sql = """
            SELECT table_name
            FROM information_schema.tables
            WHERE table_schema = 'public'
              AND (
                   table_name LIKE 'vertex_%'
                OR table_name LIKE 'edge_%'
              )
            ORDER BY table_name;
        """

        cur.execute(sql)
        rows = cur.fetchall()

        tg_tables = [r[0] for r in rows]

        cur.close()
        conn.close()

        return {"tables": tg_tables}

    except Exception as e:
        raise HTTPException(status_code=500, detail=str(e))
