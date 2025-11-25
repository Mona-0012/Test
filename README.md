curl -X POST \
  -H "Content-Type: text/plain" \
  -H "Authorization: Bearer <YOUR_TOKEN_HERE>" \
  "http://localhost:14240/gsql/v1/queries/interpret?graph=financialGraph" \
  -d 'INTERPRET QUERY () FOR GRAPH Australia { PRINT "pong"; }'
