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




@pg_router.get("/tg/get-config-columns/{table_name}")
def tg_get_config_columns(table_name: str):
    try:
        logger.debug("Loading TG config columns for table: %s", table_name)

        db_util = GoogleCloudSqlUtility(HOST_PROJECT)
        conn = db_util.get_db_connection()

        logger.info("Connecting to PostgreSQL for TG columns - connection: %s", conn)
        cur = conn.cursor()

        logger.debug("Executing query to fetch TG columns")
        print(f"SELECT column_name FROM information_schema.columns WHERE table_schema='public' AND table_name='{table_name}' ORDER BY ordinal_position")

        cur.execute(f"""
            SELECT column_name
            FROM information_schema.columns
            WHERE table_schema='public'
              AND table_name = '{table_name}'
            ORDER BY ordinal_position;
        """)

        rows = cur.fetchall()
        col_list = []

        for i in rows:
            col_list.append(i[0])

        return JSONResponse(
            status_code=200,
            content={"columns": col_list}
        )

    except Exception as e:
        logger.error("TG_COLUMNS_ERROR: Table: %s, PostgreSQL Error: %s", table_name, str(e))
        raise HTTPException(status_code=500, detail="Error fetching TG columns")

    finally:
        if conn:
            cur.close()
            conn.close()
            logger.debug("Database connection closed for TG column fetch for table: %s", table_name)

