USE GRAPH Query_Genie
INTERPRET QUERY () FOR GRAPH Query_Genie {
  // Start with all persons
  start = {person.*};
  
  // 1. Select ONLY the specific person returned in your result
  target_person = SELECT p 
                  FROM start:p 
                  WHERE p.customer_id == "0291c8db099dc115baa31592b6af7056b314eaecdf615baa478082e5a76617ce";

  // 2. Traverse to find their devices
  // If this returns ANY device, the relationship exists.
  devices = SELECT d
            FROM target_person:p -(HAS_USED)-> device:d;
            
  PRINT devices;
}
