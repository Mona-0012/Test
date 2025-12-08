Example 6: People who share a device AND receive money from the same user
------------------------------------------------------------------------------
INTERPRET QUERY () FOR GRAPH {graph_name} {{
  BoolAccum @shareDeviceAndReceiveMoney;

  start = {{person.*}};

  pairs =
    SELECT p1
    FROM start:p1
         -(HAS_USED:e1)-> device:d
         -(reverse_HAS_USED:e2)-> person:p2
         -(HAS_ACCOUNT:e3)-> accountnumber:a2
         -(HAS_PAID:e4)->  accountnumber:a1
         -(reverse_HAS_ACCOUNT:e5)-> person:p3
    WHERE p1.person_id != p2.person_id
      AND p1.person_id == p3.person_id
    ACCUM p1.@shareDeviceAndReceiveMoney = true;

  suspicious =
    SELECT s
    FROM start:s
    WHERE s.@shareDeviceAndReceiveMoney == true
    LIMIT 100;

  PRINT suspicious;
}}
