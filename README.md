SELECT count(*) 
FROM person:p -(HAS_USED:e)-> device:d
WHERE e.devicetype == "Mobile"
