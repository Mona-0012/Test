import os
from src.api.db_utils import GoogleCloudSqlUtility
db = GoogleCloudSqlUtility(os.getenv("HOST_PROJECT"))
conn = db.get_db_connection()
print(conn.get_dsn_parameters())
