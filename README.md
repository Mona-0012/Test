USE GRAPH Query_Genie
INTERPRET QUERY () FOR GRAPH Query_Genie {
  SumAccum<INT> @@count;
  start = {person.*};

  result = SELECT p 
           FROM start:p -(HAS_USED:e)-> device:d
           WHERE e.devicetype == "Mobile" 
           ACCUM @@count += 1;
           
  PRINT @@count;
}
