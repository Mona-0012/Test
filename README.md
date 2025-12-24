@pg_router.get("/tg/get-config-tables")
def tg_get_config_tables():
    try:
        logger.debug("Loading TG config table names from Cloud SQL")

        db_util = GoogleCloudSqlUtility(HOST_PROJECT)
        conn = db_util.get_db_connection()

        logger.info("Connecting to PostgreSQL for TG tables - connection: %s", conn)
        cur = conn.cursor()

        logger.debug("Executing query to fetch TG related tables")
        print("SELECT table_name FROM information_schema.tables WHERE table_schema='public' AND (table_name LIKE 'vertex_%' OR table_name LIKE 'edge_%') ORDER BY table_name")

        cur.execute("""
            SELECT table_name 
            FROM information_schema.tables 
            WHERE table_schema='public'
              AND (table_name LIKE 'vertex_%%' OR table_name LIKE 'edge_%%')
            ORDER BY table_name;
        """)

        rows = cur.fetchall()
        row_list = []

        for i in rows:
            row_list.append(i[0])

        return JSONResponse(
            status_code=200,
            content={"tables": row_list}
        )

    except Exception as e:
        logger.error("TG_TABLES_ERROR: PostgreSQL Error: %s", str(e))
        raise HTTPException(status_code=500, detail="Error fetching TG tables")

    finally:
        if conn:
            cur.close()
            conn.close()
            logger.debug("Database connection closed for TG table fetch")
