# MobileIndex (Mobile Server)
### Table of contents
- [DynamicRuleAdded,DynamicRuleUpdated,DynamicRule,Removed,DynamicAllRulesRemoved,AddRule,UpdateRule,
RemoveRule,RemoveAllRules,ValidateRule,GetDeviceIndex,UpdateDeviceName,UpdateAlmondName,
DynamicAlmondModeUpdated,DynamicIndexUpdated,DynamicDeviceRemoved,DynamicAllDeviceRemoved,
DeviceOnlineCheck,DynamicClientAdded,DynamicClientJoined,DynamicClientLeft,DynamicClientUpdate,
DynamicClientRemoved,DynamicRemoveAllClient,UpdateClient,RemoveClient,RemoveAllClients,WifiClients,
DynamicSceneAdded,DynamicSceneUpdated,DynamicSceneActivated,DynamicSceneRemoved,
DynamicAllSceneRemoved,AddScene,SetScene,ActivateScene,DeleteScene,DeleteAllScene (Command 1061)](#1061) 
- [DeviceList (Command 1200)](#1200)
- [SceneList (Command 1300)](#1300)
- [RuleList (Command 1400)](#1400)
- [ClientList (Command 1500)](#1500)
- [AlmondProperties (Command 1050)](#1050)
- [SubscribeMe (Command 1011)](#1011)
- [GetDevicePreferences (Command 1700)](#1700a)
- [GetClientPreferences (Command 1700)](#1700b)
- [UpdateDevicePreferences (Command 1700)](#1700c)
- [UpdateClientPreferences (Command 1700)](#1700d)
- [RouterSummary (Command 1100)](#1100)
- [AffiliationUserRequest (Command 23)](#23)
- [IOTScanResults (Command 1013)](#1013)
- [UpdateNotificationRegistration (Command 1800)](#1800)
- [Logout (Command 1900)](#1900)
- [Command 61](#61) 
- [AffiliationUserRequest (Command 1023)](1023)
- [GetAlmondList (Command 1112)](#1112) 
- [Restore (Command 2222)](#2222)
- [UpdateClientPreferences (Command 1525)](#1525)
- [GetClientPreferences (Command 1526)](#1526)
- [Login (Command 1003)](#1003)
- [Login (Command 1)](#1)
- [UserProfileRequest (Command 1110)](#1110)
- [Logoutall (Command 4)](#4)
- [Logout (Command 3)](#3)
- [Signup (Command 6)](#6)
- [CloudSanity (Command 102)](#102)
- [NotificationPreferenceList (Command 113)](#113)
- [AlmondModeRequest (Command 151)](#151)
- [NotificationAddRegistration (Command 281)](#281)
- [NotificationDeleteRegistration (Command 283)](#283)
- [Command 804 (Client)](#804a)
- [Command 804 (Device)](#804b)
- [Command 806](#806)
- [NotificationPreferences(Command 300)](#300)

<a name="1061"></a>
## 1)Command 1061
    Command no 
    1061- JSON format
 
    Required 
    Command,CommandType,Payload,almondMAC

    Redis
    3.hgetall on AL_<AlmondMAC>          //(AlmondMAC = <packet.parsedPayload.AlmondMAC>)
    4.get on ICID_<string>              // here <string> = random string data)
    5.setex on ICID_<string>            // (prefix + key), value = SERVER_NAME

    Queue
    7.Send response to server_name        // (payload,command,almondMAC) to queue

    Functional 
    1.Command 1061
    2.Send listResponse,commandLengthType ToMobile         //where listResponse = payload
    6.delete store[commandID]

    Flow
    socket(packet)->validator(do)->processor(do)->almond(onlyUnicast)->dispatcher(dispatchResponse)->dispatcher(unicast)->broadcaster(unicast)

<a name="1200"></a>
## 2) DeviceList (Command 1200)
    Command no 
    1200- JSON format
 
    Required 
    Command,CommandType,Payload

    SQl
    2.Select on DEVICE_DATA 
      params: AlmondMAC

    Redis

    multi
    3.hgetall on (MAC:%s+:%s,AlmondMAC,DeviceValues)       // Ids=DeviceValues,mac=AlmondMAC  

    Functional 
    1.Command 1200
    4.Send listResponse,commandLengthType ToMobile          //where listResponse = payload
    
    Flow
    socket(packet)->validator(do)->processor(do)->device(execute)->redisDeviceValue(get)->genericModel(get)->connection-pool(queryFunction)->newRowBuilder(devices)->dispatcher(dispatchResponse)

<a name="1300"></a>
## 3) SceneList (Command 1300)
    Command no 
    1300- JSON format
 
    Required 
    Command,CommandType,Payload,AlmondMAC

    SQl
    2.Select on SCENES 
      params: AlmondMAC

    Functional 
    1.Command 1300
    3.Send listResponse,commandLengthType ToMobile          //where listResponse = payload
    
    Flow
    socket(packet)->validator(do)->processor(do)->genericModel(execute),genericModel(get)->newRowBuilder(scenes)->dispatcher(dispatchResponse)

<a name="1400"></a>
## 4) RuleList (Command 1400)
    Command no 
    1400- JSON format
 
    Required 
    Command,CommandType,Payload

    SQl
    2.Select on RULE 
      params: AlmondMAC

    Functional 
    1.Command 1400
    3.Send listResponse,commandLengthType ToMobile         //where listResponse = payload
    
    Flow
    socket(packet)->validator(do)->processor(do)->genericModel(execute),genericModel(get)->newRowBuilder(Rules)->dispatcher(dispatchResponse)

<a name="1500"></a>
## 5) ClientList (Command 1500)
    Command no 
    1500- JSON format
 
    Required 
    Command,CommandType,Payload

    SQl
    2.Select on WIFICLIENTS 
      params: AlmondMAC

    Functional 
    1.Command 1500
    3.Send listResponse,commandLengthType ToMobile         //where listResponse = payload
    
    Flow
    socket(packet)->validator(do)->processor(do)->genericModel(execute),genericModel(get)->newRowBuilder(clients)->dispatcher(dispatchResponse)

<a name="1050"></a>
## 6) AlmondProperties (Command 1050) 
    Command no
    1050- JSON format 

    Required
    Command,CommandType,Payload,almondMAC

    SQl
    2.Select on AlmondProperties2
     params: AlmondMAC

    Functional
    1.Command 1050
    3.Send listResponse,commandLengthType ToMobile           //where listResponse = payload

    Flow 
    socket(packet)->validator(do)->processor(do)->almond(AlmondProperties)->newRowBuilder(almondProperties)->dispatcher(dispatchResponse)

<a name="1011"></a>
## 7) Command 1011 
    Command no
    1011- JSON format 

    Required
    Command,CommandType,Payload

    Redis  
    2.hgetall on UID_<UserID>          // where UserID = packet.userid        
 
    Queue
    4.Send SubscribeMeResponse to config.HTTP_SERVER_NAME
     
    Functional
    1.Command 1011
    3.delete store[data.UnicastID]                           // requestQueue->cleanup
    5.Send listResponse,commandLengthType ToMobile           //where listResponse = payload
  
    Flow 
    socket(packet)->validator(do)->processor(do)->subscriptionCommands(subscriptionCommands)-
    >redisManager(getAlmonds)->mongo-store(getSocket)->requestQueue(set)->producer(sendToQueue)->dispatcher(dispatchResponse)

<a name="1700a"></a>
## 8)GetDevicePreferences (Command 1700) 
    Command no
    1700- JSON format 

    Required
    Command,CommandType,Payload

    SQl
    2.Select on NotificationPreferences
      params: AlmondMAC,UserID

    Functional
    1.Command 1700
    3.Send listResponse,commandLengthType ToMobile            //where listResponse = payload

    Flow 
    socket(packet)->validator(do)->processor(do)->newPref(do)->genericModel(select)->oldRowBuilder(newPref)->dispatcher(dispatchResponse)->dispatcher(broadcast)->broadcastBuilder(preferences)

<a name="1700b"></a>
## 9)GetClientPreferences (Command 1700) 
    Command no
    1700- JSON format 

    Required
    Command,CommandType,Payload

    SQl
    2.Select on ClientPreferences
      params: AlmondMAC,UserID

    Functional
    1.Command 1700 
    3.Send listResponse,commandLengthType ToMobile      //where listResponse = payload
 
    Flow 
    socket(packet)->validator(do)->processor(do)->notiPrefs(do)->genericModel(select)->oldRowBuilder(newPref)->dispatcher(dispatchResponse)->dispatcher(broadcast)->broadcastBuilder(preferences)

<a name="1700c"></a>
## 10)UpdateDevicePreference (Command 1700) 
    Command no
    1700- JSON format 

    Required
    Command,CommandType,Payload

    Redis

    multi
    5.hgetall on UID_<userlist>

    SQl
    2.Delete on NotificationPreferences
      params: AlmondMAC,UserID

    Queue
    6.Send DynamicDevicePreferencesResponse to S11        //where S11 = (redisQueue)

    Functional
    1.Command 1700
    3.Send listResponse,commandLengthType ToMobile               //where listResponse = payload
    4.Send DynamicDevicePreferencesResponse ToMobile

    Flow
    socket(packet)->validator(do)->processor(do)->notiPrefs(do)->genericModel(delete)->oldRowBuilder(newPref)->dispatcher(dispatchResponse)->dispatcher(broadcast)->broadcastBuilder(preferences)

<a name="1700d"></a>
## 11)UpdateClientPreferences (Command 1700) 
    Command no
    1700- JSON format 

    Required
    Command,CommandType,Payload

    Redis

    multi
    5.hgetall on UID_<userlist>

    SQl
    2.Delete on ClientPreferences
      params: AlmondMAC,UserID
    
    Queue
    6.Send DynamicClientPreferencesResponse to S11               //where S11 = (redisQueue)

    Functional
    1.Command 1700
    3.Send listResponse,commandLengthType ToMobile             //where listResponse = payload
    4.Send DynamicClientPreferencesResponse ToMobile

    Flow 
    socket(packet)->validator(do)->processor(do)->notiPrefs(do)->genericModel(delete)->oldRowBuilder(newPref)->dispatcher(dispatchResponse)->dispatcher(broadcast)->broadcastBuilder(preferences)

<a name="1100"></a>
## 12)RouterSummary (Command 1100)
    Command no 
    1100- JSON format
 
    Required 
    Command,CommandType,Payload,almondMAC

    Redis
    3.hgetall on AL_<AlmondMAC>          //(AlmondMAC = <packet.parsedPayload.AlmondMAC>)
    4.get on ICID_<string>              // here <string> = random string data)
    5.setex on ICID_<string>            // (prefix + key), value = SERVER_NAME

    Queue
    7.Send response to server_name        // (payload,command,almondMAC) to queue

    Functional 
    1.Command 1061
    2.Send listResponse,commandLengthType ToMobile         //where listResponse = payload
    6.delete store[commandID]

    Flow
    socket(packet)->validator(do)->processor(do)->almond(onlyUnicast)->dispatcher(dispatchResponse)->dispatcher(unicast)->broadcaster(unicast)

<a name="23"></a>
## 13)AffiliationUserRequest (Command 23)
    Command no 
    23- JSON format
 
    Required 
    Command,CommandType,Payload

    Redis
    2.get on CODE:<data.code>  
    4.get on ICID_<string>              // here <string> = random string data)
    5.setex on ICID_<string>            // (prefix + key), value = SERVER_NAME
        
    SQl 
    3.Select on Users
      params: UserID
     
    Queue
    8.Send AffiliationUserRequestResponse to queue       //(where queue = payload.queue)
 
    Functional
    1.Command 23
    6.Send listResponse,commandLengthType ToMobile       //where listResponse = payload
    7.delete store[row.CommandID]

    Flow
    socket(packet)->validator(do)->processor(do)->affiliation(execute)->redisManager(getCode)->sqlManager(getEmail)->cid-bid(incCommandID)->newRowBuilder(affiliationError)->dispatcher(dispatchResponse)->dispatcher(unicast)->broadcaster(unicast)

<a name="1013"></a>
## 14)IOTScanResults (Command 1013)
    Command no 
    1013- JSON format
 
    Required 
    Command,CommandType,Payload,AlmondMAC
    
    SQl
    2.Select on IOT_Scanner
      params: AlmondMAC

    Functional
    1.Command 1013
    3.Send listResponse,commandLengthType ToMobile       //where listResponse = payload

    Flow
    socket(packet)->validator(do)->processor(do)->almond(IOTScan)->newRowBuilder(IOTScan)->dispatcher(dispatchResponse)

<a name="1800"></a>
## 15)UpdateNotificationRegistration (Command 1800)
    Command no 
    1800- JSON format
 
    Required 
    Command,CommandType,Payload

    SQl
    2.Insert on NotificationID
      // Here Params = parsedPayload

    Functional
    1.Command 1800
    3.Send listResponse,commandLengthType ToMobile       //where listResponse = payload

    Flow
    socket(packet)->validator(do)->processor(do)->notificationNew(do)->oldRowBuilder(notificationNew)->dispatcher(dispatchResponse)

<a name="1900"></a>
## 16)Logout (Command 1900)
    Command no 
    1900- JSON format
 
    Required 
    Command,CommandType,Payload

    Redis
    4.hmset on UID_<socket.userid>      // values = Q_config.SERVER_NAME,userSession.length - 1

    SQl
    2.Delete on UserTempPasswords
      params: UserID,TempPassword

    6.Delete on NotificationID
      // here, params = parsedPayload
    
    Functional
    1.Command 1900
    3.Send listResponse,commandLengthType ToMobile       //where listResponse = payload
    5.delete socketStore[userid]                  //if (userSession.length == 1) 
    7.Send listResponse,commandLengthType ToMobile       //where listResponse = payload

    Flow
    socket(packet)->validator(do)->processor(do)->Login(Logout)->connection-pool(queryFunction)->oldRowBuilder(logoutJSON)->dispatcher(dispatchResponse)->dispatcher(socketHandler)->mongo-store(remove)->redisManager(redisExecute)->secondaryModels(notificationNew.do)->oldRowBuilder(notificationNew)->dispatcher(dispatchResponse)


<a name="61"></a>
## 17)Command 61
    Command no 
    61- JSON format
 
    Required 
    Command,CommandType,Payload,almondMAC

    Redis
    3.hgetall on AL_<AlmondMAC>          //(AlmondMAC = <packet.parsedPayload.AlmondMAC>)
    4.get on ICID_<string>              // here <string> = random string data)
    5.setex on ICID_<string>            // (prefix + key), value = SERVER_NAME

    Queue
    7.Send response to server_name        // (payload,command,almondMAC) to queue

    Functional 
    1.Command 61
    2.Send listResponse,commandLengthType ToMobile         //where listResponse = payload
    6.delete store[commandID]

    Flow
    socket(packet)->validator(do)->processor(do)->almond(onlyUnicast)->dispatcher(dispatchResponse)->dispatcher(unicast)->broadcaster(unicast)

<a name="1023"></a>
## 18)AffiliationUserRequest (Command 1023)
    Command no 
    1023- JSON format
 
    Required 
    Command,CommandType,Payload,AlmondMAC,UserID

    Redis
    2.get on CODE:<data.code>  
    4.get on ICID_<string>              // here <string> = random string data)
    5.setex on ICID_<string>            // (prefix + key), value = SERVER_NAME

    7.setex on CODE:<data.code>     
    //(prefix+key),value=res.mac,SERVER_NAME, socket.userid,commandID,res.emailID
        
    SQl 
    3.Select on Users
      params: UserID
    6.Select on SCSIDB.CMSAffiliations
     params:AlmondMAC
 
    Queue
    10. Send Response to queue  
    // where repsonse=command, payload,almondMAC, queue= payload.queue 

    Functional
    1.Command 1023
    8.Send listResponse,commandLengthType ToMobile       //where listResponse = payload
    9.delete store[res.commandID]

    Flow
    socket(packet)->validator(do)->processor(do)->affiliation(execute)->redisManager(getCode)->sqlManager(getEmail)->cid-bid(incCommandID)->newRowBuilder(affiliationError)->dispatcher(dispatchResponse)->dispatcher(unicast)->broadcaster(unicast)
    
<a name="1112"></a>
## 19)GetAlmondList (Command 1112)
    Command no 
    1112- JSON format
 
    Required 
    Command,CommandType,Payload,AlmondMAC,UserID

    Redis
    2.hgetall on UID_<packet.userid>
    3.hgetall on AL_<MACs>

    SQl
    4.Select on SCSIDB.CMS
      params: CMSCode
    6.Select on Users
      params: UserID

    Functional
    1.Command 1112
    5.Send listResponse,commandLengthType ToMobile       //where listResponse = payload
    7.Send listResponse,commandLengthType ToMobile       //where listResponse = payload

    Flow
    socket(packet)->validator(do)->processor(do)->almond(create_almond_list)->redisManager(getAllAlmonds),redisManager(redisExecuteAll)->oldRowBuilder(create_almond_list)->dispatcher(dispatchResponse)->almond(AlmondAffiliationData)->oldRowBuilder(AffiliationData)->dispatcher(dispatchResponse)

<a name="2222"></a>
## 20)Restore (Command 2222)
    Command no 
    2222- JSON format
 
    Required 
    Command,CommandType,Payload,AlmondMAC

    SQl
    2.Select on almondhistory
    params: mac,type

    Queue
    4. Send Response to queue  
    // where repsonse=command, payload,almondMAC, queue= payload.queue 

    Functional
    1.Command 2222
    3.Send listResponse,commandLengthType ToMobile       //where listResponse = payload

    Flow
    socket(packet)->validator(do)->processor(do)->notificationFetcher(almondHistroy)->newRowBuilder(almondHistroy)->dispatcher(dispatchResponse)

<a name="1525"></a>
## 21)UpdateClientPreferences (Command 1525)
    Command no 
    1525- JSON format
 
    Required 
    Command,CommandType,Payload,UserID

    Redis

    multi
    5.hgetall on UID_<userID>          // (redisConstants)

    SQl
    2.Insert on WifiClientsNotificationPreferences
      params:AlmondMAC,ClientID,UserID,NotificationType

    Queue
    6.Send UpdateClientPreferencesResponse to S11            //where S11 = (redisQueue)
    
    Functional
    1.Command 1525
    3.Send listResponse,commandLengthType ToMobile       //where listResponse = payload
    4.Send Response ToMobile                            //where Response = command,payload

    Flow
    socket(packet)->validator(do)->processor(do)->notificationPreferences(change_wificlient_notification_preferences)->oldRowBuilder(wifiNotificationPreferences)->dispatcher(dispatchResponse)->broadcastBuilder(defaultXML)->broadcaster(broadcast)

<a name="1526"></a>
## 22)GetClientPreferences (Command 1526)
    Command no 
    1526- JSON format
 
    Required 
    Command,CommandType,Payload,UserID

    SQl
    2.Select on NotificationPreferences
     params: AlmondMAC,UserID

    Functional
    1.Command 1526
    3.Send listResponse,commandLengthType ToMobile       //where listResponse = payload

    Flow
    socket(packet)->validator(do)->processor(do)->notificationPreferences(get_wifi_notification_preferences)->oldRowBuilder(wifiNotificationPreferences)->dispatcher(dispatchResponse)

<a name="1003"></a>
## 23)Login (Command 1003)
    Command no 
    1003- JSON format
 
    Required 
    Command,CommandType,Payload,UserID,AlmondMAC

    Redis
    5.hgetall on UID_<data.UserID> 
    7.hincrby on UID_<data.UserID>         //values = (Q_<config.SERVER_NAME>,1)

    SQl
    2.Insert on logging.error_log
      params: date,time,ip,server,category,error
    3.Select on Users
      params: EmailID
    4.Insert on UserTempPasswords
      params:UserID,TempPassword,LastUsedTime
    8.Select on Subscriptions
      params: AlmondMAC
     
    Functional
    1.Command 1003
    6.Send listResponse,commandLengthType ToMobile       //where listResponse = payload
    9.Send listResponse,commandLengthType ToMobile       //where listResponse = payload

    Flow
    socket(packet)->validator(do)->validator(login)->logging(errorLog)->login(Mob_Login)->redisManager(getAllAlmonds)->oldRowBuilder(loginJSON)->dispatcher(dispatchResponse)->mongo-store(add)->redisManager(redisExecute)->login(GetSubscriptions)->oldRowBuilder(subscriptions)->dispatcher(dispatchResponse)

<a name="1"></a>
## 24)Login (Command 1)
    Command no 
    1- JSON format
 
    Required 
    Command,CommandType,Payload,UserID,AlmondMAC

    Redis
    5.hgetall on UID_<data.UserID> 
    7.hincrby on UID_<data.UserID>         //values = (Q_<config.SERVER_NAME>,1)

    SQl
    2.Insert on logging.error_log
      params: date,time,ip,server,category,error
    3.Select on Users
      params: EmailID
    4.Insert on UserTempPasswords
      params:UserID,TempPassword,LastUsedTime
    8.Select on Subscriptions
      params: AlmondMAC

    Functional
    1.Command 1
    6.Send listResponse,commandLengthType ToMobile       //where listResponse = payload
    9.Send listResponse,commandLengthType ToMobile       //where listResponse = payload

    Flow
    socket(packet)->validator(do)->validator(login)->logging(errorLog)->login(Mob_Login)->redisManager(getAllAlmonds)->oldRowBuilder(loginJSON)->dispatcher(dispatchResponse)->mongo-store(add)->redisManager(redisExecute)->login(GetSubscriptions)->oldRowBuilder(subscriptions)->dispatcher(dispatchResponse)

<a name="1110"></a>
## 25)UserProfileRequest (Command 1110)
    Command no 
    1110- JSON format
 
    Required 
    Command,CommandType,Payload,UserID

    SQl
    2.Select on Users
      params: UserID

    Functional
    1.Command 1110
    3.Send listResponse,commandLengthType ToMobile       //where listResponse = payload

    Flow
    socket(packet)->validator(do)->processor(do)->account-manager-json(UserProfile)->newRowBuilder(UserProfile)->dispacher(dispatchResponse)

<a name="4"></a>
## 26)Logoutall (Command 4)
    Command no 
    4- JSON format
 
    Required 
    Command,CommandType,Payload,UserID
  
    Redis
    7.hmset on UID_<userid>        // where values = [Q_config.SERVER_NAME,0]
  
    multi
    8.hgetall on UID_<userID>          // (redisConstants)

    SQl
    2.Select on Users
      params: UserID
    3.Delete on UserTempPasswords
      params: UserID
    4.Delete on NotificationID
     params: UserID

    Queue
    9.Send UserProfileResponse to S11            //where S11 = (redisQueue)

    Functional
    1.Command 4
    5.Send listResponse,commandLengthType ToMobile       //where listResponse = payload
    6.delete socketStore[userid]

    Flow
    socket(packet)->validator(do)->validator(checkCredentials)->sqlManager(getUser)->login(logoutAll)->connection-pool(queryFunction)->oldRowBuilder(logoutAll)->dispacher(dispatchResponse)->mongo-store(removeAll)->dispatcher(broadcast)->broadCastBuilder(removeAll)->broadcaster(broadcast)

<a name="3"></a>
## 27)Logout (Command 3)
    Command no 
    3- JSON format
 
    Required 
    Command,CommandType,Payload,UserID
 
    Redis
    4.hmset on UID_<socket.userid>      //values = [Q_config.SERVER_NAME,userSession.length-1]

    SQl
    2.Delete on UserTempPasswords
     params: UserID,TemmpPassword

    Functional
    1.Command 3
    3.Send listResponse,commandLengthType ToMobile       //where listResponse = payload
    5.delete socketStore[socket.userid]

    Flow
    socket(packet)->validator(do)->login(logout)->oldRowBuilder(logout)->dispacher(dispatchResponse)->mongo-store(remove)

<a name="6"></a>
## 28)Signup (Command 6)
    Command no 
    6- JSON format
 
    Required 
    Command,CommandType,Payload,UserID

    Redis
    4.hmset on UID_<socket.userid>      //values = [Q_config.SERVER_NAME,userSession.length-1] 

    SQl
    2.Select on Users
      params: EmailID 

    Functional 
    1.Command 6
    3.Send listResponse,commandLengthType ToMobile       //where listResponse = payload
    5.delete socketStore[socket.userid]
    
    Flow
    socket(packet)->validator(do)->accountSetup(Mob_Signup)->oldRowBuilder(accountSetup)->dispacher(dispatchResponse)->mongo-store(remove)

<a name="102"></a>
## 29)CloudSanity (Command 102)
    Command no 
    102- JSON format
 
    Required 
    Command,CommandType,Payload

    Functional
    1.Command 102
    2.Send listResponse,commandLengthType ToMobile       //where listResponse = payload

    Flow
    socket(packet)->validator(do)->almond(sanity_check)->oldRowBuilder(dummy)->dispacher(dispatchResponse)

<a name="113"></a>
## 30)NotificationPreferenceList (Command 113)
    Command no 
    113- JSON format
 
    Required 
    Command,CommandType,Payload

    SQl
    2.Select on NotificationPreferences
      params: AlmondMAC,UserID

    Functional
    1.Command 113
    3.Send listResponse,commandLengthType ToMobile       //where listResponse = payload

    Flow
    socket(packet)->validator(do)->notificationPreferences(get_notification_preference_list)->oldRowBuilder(get_notification_preference_list)->dispacher(dispatchResponse)

<a name="151"></a>
## 31)AlmondModeRequest (Command 151)
    Command no 
    151- JSON format
 
    Required 
    Command,CommandType,Payload

    SQl
    2.select on AlmondPreferences
    params:T1.AlmondMAC

    Functional
    1.Command 151
    3.Send listResponse,commandLengthType ToMobile       //where listResponse = payload

    Flow
    socket(packet)->validator(do)->almond(get_almondmode)->oldRowBuilder(get_almondmode)->dispacher(dispatchResponse)

<a name="281"></a>
## 32)NotificationAddRegistration (Command 281)
    Command no 
    281- JSON format
 
    Required 
    Command,CommandType,Payload

    SQl
    2.Select on NotificationID
      params:HashVal

    //if (rows.length == 0)
    3.Insert on NotificationID
      params: HashVal, RegID, UserID, Platform

            (or)

    //if (rows.length == 1)
    3.Update on NotificationID
      params:  UserID,Platform,RegID

    Functional
    1.Command 281   
    4.Send listResponse,commandLengthType ToMobile       //where listResponse = payload

    Flow
    socket(packet)->validator(do)->notification(Mobile_Notification_Registration)->oldRowBuilder(notificationAddRegistration)->dispacher(dispatchResponse)

<a name="283"></a>
## 33)NotificationDeleteRegistration (Command 283)
    Command no 
    283- JSON format
 
    Required 
    Command,CommandType,Payload

    SQl
    2.Delete on NotificationID
      params:HashVal,RegID,UserID, 

    Functional
    1.Commad 283
    3.Send listResponse,commandLengthType ToMobile       //where listResponse = payload

    Flow
    socket(packet)->validator(do)->notification(Mobile_Notification_Delete_Registration)->oldRowBuilder(notificationDeleteRegistration)->dispacher(dispatchResponse)

<a name="804a"></a>
## 34.Command 804 (Client)
    Command no 
    804- JSON format
 
    Required 
    Command,CommandType,Payload,AlmondMAC,ClientID

    SQl
    2.Select on WifiClients
      params:AlmondMAC,ClientID
    3.Select on dynamic_log
      params: mac,id
    4.Select on dynamic_log           //if (data.type != undefined && data.type == "wifi_client"
      params: mac,id

    Functional
    1.Command 804
    5.Send listResponse,commandLengthType ToMobile       //where listResponse = payload

    Flow
    socket(packet)->validator(do)->notification(get_logs)->notificationFetcher(getLogs)->oldRowBuilder(getLogs)->dispacher(dispatchResponse)

<a name="804b"></a>
## 35.Command 804 (Device)
    Command no 
    804- JSON format
 
    Required 
    Command,CommandType,Payload,DeviceID
       
    SQl
    2.Select on dynamic_log
      params: mac,id

    Functional
    1.Command 804
    3.Send listResponse,commandLengthType ToMobile       //where listResponse = payload

    Flow
    socket(packet)->validator(do)->notification(get_logs)->notificationFetcher(getLogs)->oldRowBuilder(getLogs)->dispacher(dispatchResponse)

<a name="806"></a>
## 36.Command 806 
    Command no 
    806- JSON format
 
    Required 
    Command,CommandType,Payload

    Functional
    1.Command 806
    2.Send listResponse,commandLengthType ToMobile       //where listResponse = payload

    Flow
    socket(packet)->validator(do)->notificationFetcher(makeBadgeZero)->oldRowBuilder(clear_the_badge)->dispacher(dispatchResponse)

<a name="300"></a>
## 37.NotificationPreferences(Command 300)
    Command no 
    300- JSON format
 
    Required 
    Command,CommandType,Payload

    Redis

    multi
    5.hgetall on UID_<userID>          // (redisConstants)

    Queue
    6.Send UserProfileResponse to S11            //where S11 = (redisQueue)

    SQl
    2.Insert on NotificationPreferences
      params: AlmondMAC, DeviceID, UserID,IndexVal, NotificationType

    Functional
    1.Command 300
    3.Send listResponse,commandLengthType ToMobile       //where listResponse = payload
    4.Send listResponse,commandLengthType ToMobile       //where listResponse = payload

    Flow
    socket(packet)->validator(do)->notificationPreferences(update_notification_preferences)->oldRowBuilder(notificationPreferences)->dispacher(dispatchResponse)->dispatcher(broadcast)->broadcastBuilder(defaultXML)->broadcaster(broadcast)
