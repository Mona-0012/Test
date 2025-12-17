
INTERPRET QUERY () FOR GRAPH Query_Genie {
  SumAccum<INT> @@muleCount;
  start = {person.*};
  
  result = SELECT p FROM start:p 
           WHERE p.mule == true
           ACCUM @@muleCount += 1;
           
  PRINT @@muleCount;
}




INTERPRET QUERY () FOR GRAPH Query_Genie {
  SumAccum<INT> @@mobileCount;
  start = {device.*};
  
  result = SELECT d FROM start:d 
           WHERE d.devicetype == "Mobile"
           ACCUM @@mobileCount += 1;
           
  PRINT @@mobileCount;
}



USE GRAPH Query_Genie
INTERPRET QUERY () FOR GRAPH Query_Genie {
  start = {person.*};
  
  // 1. Get Mules
  mules = SELECT p FROM start:p WHERE p.mule == true;
  
  // 2. See their devices (regardless of type)
  mule_devices = SELECT d FROM mules:p -(HAS_USED)-> device:d;
  
  PRINT mule_devices;
}
