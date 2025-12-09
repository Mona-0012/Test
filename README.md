CREATE QUERY suspicious_network_final() FOR GRAPH Query_Genie {
  
  // ACCUMulator to store the people who share a device
  // Key: device_id (STRING), Value: Set of person vertices
  Map<STRING, SET<person>> @@device_to_people_map;
  
  // Set to store the final list of "suspicious" people based on the device link
  Set<person> @@shared_device_people;
  
  // ACCUMulator to store the final suspicious connections: Person A -> Set of Person B
  Map<person, SET<person>> @@suspicious_connections;
  
  
  // 1. Identify all people connected to a device, and map them
  start_people = {person.*};
  
  // Traverse from person to device (using HAS_USED) and accumulate the person into the device's map
  people_on_device = SELECT t
                     FROM start_people:s -(HAS_USED:e)-> device:t
                     ACCUM t.@@device_to_people_map += (t.primary_id -> s);
  
  
  // 2. Filter for devices shared by more than one person, and gather those people
  
  // Traverse the devices, filtering for those with multiple people
  shared_devices = SELECT t
                   FROM people_on_device:t
                   WHERE t.@@device_to_people_map.size() > 1
                   ACCUM FOREACH (person_node IN t.@@device_to_people_map.values()) DO
                             @@shared_device_people += person_node
                         END;
                         
                         
  // 3. Find money flow connections (HAS_PAID) *between* the accounts of these shared-device people
  
  // Start from the set of people who share a device
  shared_people_start = @@shared_device_people;
  
  // Find money transfer (HAS_PAID) between accounts, where both accounts belong to the shared set of people
  // p1 and p2 are the person vertices. We are looking for the full path:
  // Person 1 -> Account 1 -> Account 2 -> Person 2
  results = SELECT p2
            FROM shared_people_start:p1 -(HAS_ACCOUNT:e_p1_a1)-> accountnumber:a1 
                                       -(HAS_PAID:e_a1_a2)-> accountnumber:a2
                                       -(reverse_HAS_ACCOUNT:e_a2_p2)-> person:p2
            WHERE p2 IN @@shared_device_people // Ensure the recipient is also from the shared device set
            AND p1 != p2 // Exclude self-transfers (though usually impossible with two accounts)
            ACCUM p1.@@suspicious_connections += p2;

  
  // 4. Output the final list of suspicious person-to-person connections
  
  PRINT @@suspicious_connections AS Suspicious_Pairs;
}
