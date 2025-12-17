INTERPRET QUERY () FOR GRAPH Query_Genie {

  // Select the same person
  person_set = SELECT p FROM person:p
               WHERE p.customer_id == "0019b853c1e2a24d3b84cd554885f24a69bc42913b89646df7d7454986f912a3";

  // Get their accounts
  accounts = SELECT a
             FROM person_set:p -(HAS_ACCOUNT:e)-> accountnumber:a;

  // Check if any account has sent payments
  sent_payments = SELECT a2
                  FROM accounts:a1 -(HAS_PAID:e)-> accountnumber:a2;

  PRINT sent_payments;
}
