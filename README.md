INTERPRET QUERY () FOR GRAPH {graph_name} {{
  BoolAccum @hasSharedDevice;
  BoolAccum @hasPaymentLink;

  start = {{person.*}};

  device_pairs =
    SELECT p1
    FROM start:p1 -(HAS_USED:e1)-> device:d -(reverse_HAS_USED:e2)-> person:p2
    WHERE p1.person_id != p2.person_id
    ACCUM p1.@hasSharedDevice = true;

  payment_pairs =
    SELECT p1
    FROM start:p1
         -(HAS_ACCOUNT:e3)-> accountnumber:a1
         -(HAS_PAID:e4)->  accountnumber:a2
         -(reverse_HAS_ACCOUNT:e5)-> person:p2
    WHERE p1.person_id != p2.person_id
    ACCUM p1.@hasPaymentLink = true;

  suspicious =
    SELECT s
    FROM start:s
    WHERE s.@hasSharedDevice == true
      AND s.@hasPaymentLink == true
    LIMIT 100;

  PRINT suspicious;
}}
