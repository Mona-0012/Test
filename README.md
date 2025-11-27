curl -X POST "http://127.0.0.1:8000/execute_gsql" ^
  -H "Content-Type: application/json" ^
  -d "{\"market\": \"Australia\", \"gsql\": \"INTERPRET QUERY () FOR GRAPH Graph_new { PRINT 1; }\"}"
