INTERPRET QUERY () FOR GRAPH Query_Genie {

  SumAccum<FLOAT> @@totalPayments;

  start = {accountnumber.*};

  result = SELECT s FROM start:s
           WHERE s.account_number == "X"
           ACCUM @@totalPayments += s.TOTAL_SENT_VOLUME;

  PRINT @@totalPayments;
}
