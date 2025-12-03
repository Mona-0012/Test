
You
,
Nov 10, 11:50 AM
,
logger.py
,Nov 10, 11:50 AM,
Tuesday, Nov 18
You
,
Nov 18, 6:04 PM
,
sometimes i m getting this error. is it because of the free version
image.png
,Nov 18, 6:04 PM,
Wednesday, Nov 19
You
,
Nov 19, 12:57 PM
,
Hi Abhijeet, I checked this , i think it has some memory issue.
,Nov 19, 12:57 PM,
is it because of the free version?
,Nov 19, 12:57 PM,
Abhijeet Srivastava
,
Nov 19, 12:58 PM
,
Hi Darshana, yes it seems like its due to non-enterprise version we are using.
but i am checking for the work around at the moment. Will let you know if anything can prevent this in future.
,Nov 19, 12:58 PM,
You
,
Nov 19, 12:59 PM
,
sure
,Nov 19, 12:59 PM,
thnks
,Nov 19, 12:59 PM,
Abhijeet Srivastava
,
Nov 19, 1:21 PM
,
Hi Darshana, quick call if free now?
,Nov 19, 1:21 PM,
You
,
Nov 19, 1:24 PM
,
sure
,Nov 19, 1:24 PM,
Abhijeet Srivastava
,
Nov 19, 1:28 PM
,
Video meeting
Google Meet
Join video meeting
,Nov 19, 1:28 PM,
You
,
Nov 19, 1:35 PM
,
on vdi i did curl and checked and i got the error.
image.png
,Nov 19, 1:35 PM,
You
,
Nov 19, 1:39 PM
,
uska api galat hai esa bol rha hai i checked. so i am checking ki kese kar sakte hai
,Nov 19, 1:39 PM,
Abhijeet Srivastava
,
Nov 19, 1:43 PM
,
ok
,Nov 19, 1:43 PM,
let me know if you need any help/assistance
,Nov 19, 1:43 PM,
You
,
Nov 19, 3:10 PM
,
Yeah sure
,Nov 19, 3:10 PM,
You
,
Nov 19, 9:35 PM
,
Sir please wo container ka memory increase kar dijiye. Right now i m not working on it so.
,Nov 19, 9:35 PM,
Abhijeet Srivastava
,
Nov 19, 10:44 PM
,
Sure Darshana
,Nov 19, 10:44 PM,
Thursday, Nov 20
You
,
Nov 20, 2:03 PM
,
Thanks
,Nov 20, 2:03 PM,
Abhijeet Srivastava
,
Nov 20, 2:04 PM
,
Have changed it to no limit for the container, if you still see that error then there is only one last thing we can do - run it as a service in linux rather than running on docker/containers
,Nov 20, 2:04 PM,
You
,
Nov 20, 2:04 PM
,
ok let me check that.
,Nov 20, 2:04 PM,
Today
You
,
10 min
,
i am getting this error :
image.png
,11 min,
for
image.png
image.png
,9 min,
Abhijeet Srivastava
,
2 min
,
Can you try this instead - 

INTERPRET QUERY () FOR GRAPH Query_Genie {
  SetAccum<person> @@suspiciousPeople;
  SetAccum<accountnumber> @@paid_accounts;
  
  // Find devices and map to people
  MapAccum<VERTEX<device>, SetAccum<VERTEX<person>>> @@deviceToPeople;
  
  start_devices = {device.*};
  
  devices_with_users = SELECT d
                       FROM start_devices:d -(reverse_HAS_USED:e)-> person:p
                       ACCUM @@deviceToPeople += (d -> p);
  
  // Find people sharing devices
  SetAccum<person> @@people_sharing_device;
  
  shared_devices = SELECT d
                   FROM devices_with_users:d
                   WHERE @@deviceToPeople.get(d).size() > 1
                   ACCUM 
                     FOREACH person_var IN @@deviceToPeople.get(d) DO
                       @@people_sharing_device += person_var
                     END;
  
  // Find accounts from shared devices
  accounts_from_shared = SELECT acc
                         FROM shared_devices:d -(HAS_ACCOUNT)-> accountnumber:acc
                         ACCUM @@paid_accounts += acc;
  
  // Filter for paid accounts
  result = SELECT p
           FROM @@people_sharing_device:p -(HAS_ACCOUNT)-> accountnumber:acc
           WHERE acc IN @@paid_accounts
           ACCUM @@suspiciousPeople += p
           LIMIT 100;
  
  PRINT @@suspiciousPeople;
}
Message read by Abhijeet Srivastava

,2 min,



History is on

