 NEVER assign to a global accumulator using '=' inside ACCUM or POST-ACCUM.
   Forbidden:  @@x = value;
   Allowed only: @@x += value;
