People with heavy device usage and many outgoing payments
------------------------------------------------------------
INTERPRET QUERY () FOR GRAPH {graph_name} {{
  SumAccum<INT> @deviceUseCount;
  SumAccum<INT> @paymentCount;

  start = {{person.*}};

  device_usage =
    SELECT p
    FROM start:p -(HAS_USED:e1)-> device:d
    ACCUM p.@deviceUseCount += 1;

  outgoing_payments =
    SELECT p
    FROM start:p
         -(HAS_ACCOUNT:e2)-> accountnumber:a1
         -(HAS_PAID:e3)->  accountnumber:a2
    ACCUM p.@paymentCount += 1;

  risky =
    SELECT s
    FROM start:s
    WHERE s.@deviceUseCount >= 5
      AND s.@paymentCount > 20
    LIMIT 100;

  PRINT risky;
}}
