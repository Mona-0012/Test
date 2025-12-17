INTERPRET QUERY () FOR GRAPH Graph_new {

  // Select exactly one person by primary id
  start = SELECT p FROM person:p
          WHERE p.customer_id == "0019b853c1e2a24d3b84cd554885f24a69bc42913b89646df7d7454986f912a3";

  // Traverse to devices
  result = SELECT d
           FROM start:p -(HAS_USED:e)-> device:d
           WHERE d.devicetype == "Mobile";

  PRINT result;
}
