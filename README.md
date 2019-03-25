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
- [AlmondModeChange (Command 61)](#61) 
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
- [Command 800](#800)
- [ChangeUser (Command 1060,Action:Add)](#1060a)
- [ChangeUser (Command 1060,Action:Update)](#1060b)
- [Super login (Command 1004)](#1004)
- [UnlinkAlmondRequest (Command 1110)](#1110a) 
- [UserInviteRequest (Command 1110](#1110b)
- [UpdateUserProfileRequest (Command 1110)](#1110c)
- [DeleteSecondaryUserRequest (Command 1110)](#1110d)
- [DeleteMeAsSecondaryUserRequest (Command 1110)](#1110e)
- [ChangePasswordRequest (Command 1110)](#1110f) 
- [DeleteAccountRequest (Command 1110)](#1110g)
- [PaymentDetails (Command 1011)](#1011a)
- [UpdateCard (Command 1011)](#1011b)
- [DeleteSubscription (Command 1011)](#1011c)

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
    7.Send response to config.SERVER_NAME        // (payload,command,almondMAC) to queue

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
    3.hgetall on (MAC:<AlmondMAC>,DeviceValues) 
    //Here, multi is done on all AlmondMAC and the IDs present in DeviceValues  

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
## 7) SubscribeMe (Command 1011) 
    Command no
    1011- JSON format 

    Required
    Command,CommandType,Payload

    Redis  
    2.hgetall on UID_<packet.userid>      // values = PMAC_<AlmondMAC>
     
    Queue
    4.Send SubscribeMeResponse to config.HTTP_SERVER_NAME
     
    Functional
    1.Command 1011
    3.delete store[data.UnicastID]                           
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
    5.hgetall on UID_<userID>             /here, multi is done on every userID in UserList

    SQl
    2.Delete on NotificationPreferences
      params: AlmondMAC,UserID

    Queue
    6.Send DynamicDevicePreferencesResponse to MobileQueue   

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
    5.hgetall on UID_<userID>          //here, multi is done on every userID in UserList

    SQl
    2.Delete on ClientPreferences
      params: AlmondMAC,UserID
    
    Queue
    6.Send DynamicClientPreferencesResponse to MobileQueue            

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
    7.Send response to config.SERVER_NAME        // (payload,command,almondMAC) to queue

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
    8.Send AffiliationUserRequestResponse to rows.server
 
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
      params: mac

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
    socket(packet)->validator(do)->processor(do)->notificationNew(do)->genericModel(insertOrUpdate)->oldRowBuilder(notificationNew)->dispatcher(dispatchResponse)

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
## 17)AlmondModeChange (Command 61)
    Command no 
    61- JSON format
 
    Required 
    Command,CommandType,Payload,almondMAC

    Redis
    3.hgetall on AL_<AlmondMAC>          //(AlmondMAC = <packet.parsedPayload.AlmondMAC>)
    4.get on ICID_<string>              // here <string> = random string data)
    5.setex on ICID_<string>            // (prefix + key), value = SERVER_NAME

    Queue
    7.Send response to config.SERVER_NAME       // (payload,command,almondMAC) to queue

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
    //(prefix+key),values =res.mac,SERVER_NAME, socket.userid,commandID,res.emailID
        
    SQl 
    3.Select on Users
      params: UserID
    6.Select on SCSIDB.CMSAffiliations
      params:AlmondMAC
 
    Queue
    10. Send AffiliationAlmondCompleteResponse to rows.server  
   
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

    Cassandra
    2.Select on almondhistory
    params: mac,type

    Redis
    4.hgetall on AL_<AlmondMAC>
    5.get on ICID_<code>                //where code = random string
    6.setex on ICID_<code>            //where code = random string

    Queue
    8.Send RestoreResponse to  config.SERVER_NAME

    Functional
    1.Command 2222
    3.Send listResponse,commandLengthType ToMobile       //where listResponse = payload
    7.delete store[commandID] 

    Flow
    socket(packet)->validator(do)->processor(do)->notificationFetcher(almondHistroy)->newRowBuilder(almondHistroy)->dispatcher(dispatchResponse)->dispatcher(unicast)->newRowBuilder(restoreData)->broadcaster(unicast)

<a name="1525"></a>
## 21)UpdateClientPreferences (Command 1525)
    Command no 
    1525- JSON format
 
    Required 
    Command,CommandType,Payload,UserID

    Redis

    multi
    5.hgetall on UID_<userID>          //here, multi is done on every userID in UserList

    SQl
    2.Insert on WifiClientsNotificationPreferences
      params:AlmondMAC,ClientID,UserID,NotificationType

    Queue
    6.Send UpdateClientPreferencesResponse to MobileQueue
    
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

    Cassandra
    2.Insert on logging.error_log
      params: date,time,ip,server,category,error
    
    SQl
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
    socket(packet)->validator(do)->validator(login)->logging(errorLog)->processor(do)->login(Mob_Login)->redisManager(getAllAlmonds)->oldRowBuilder(loginJSON)->dispatcher(dispatchResponse)->mongo-store(add)->redisManager(redisExecute)->login(GetSubscriptions)->oldRowBuilder(subscriptions)->dispatcher(dispatchResponse)

<a name="1"></a>
## 24)Login (Command 1)
    Command no 
    1- JSON format
 
    Required 
    Command,CommandType,Payload,UserID,AlmondMAC

    Redis
    5.hgetall on UID_<data.UserID> 
    7.hincrby on UID_<data.UserID>         //values = (Q_<config.SERVER_NAME>,1)

    Cassandra
    2.Insert on logging.error_log
      params: date,time,ip,server,category,error

    SQl  
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
    socket(packet)->validator(do)->validator(login)->logging(errorLog)->processor(do)->login(Mob_Login)->redisManager(getAllAlmonds)->oldRowBuilder(loginJSON)->dispatcher(dispatchResponse)->mongo-store(add)->redisManager(redisExecute)->login(GetSubscriptions)->oldRowBuilder(subscriptions)->dispatcher(dispatchResponse)

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
    8.hgetall on UID_<userID>          //here, multi is done on every userID in UserList

    SQl
    2.Select on Users
      params: UserID
    3.Delete on UserTempPasswords
      params: UserID
    4.Delete on NotificationID
     params: UserID

    Queue
    9.Send UserProfileResponse to MobileQueue            

    Functional
    1.Command 4
    5.Send listResponse,commandLengthType ToMobile       //where listResponse = payload
    6.delete socketStore[userid]

    Flow
    socket(packet)->validator(do)->validator(checkCredentials)->sqlManager(getUser)->processor(do)->login(logoutAll)->connection-pool(queryFunction)->oldRowBuilder(logoutAll)->dispacher(dispatchResponse)->mongo-store(removeAll)->dispatcher(broadcast)->broadCastBuilder(removeAll)->broadcaster(broadcast)

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
   
    //if (userSession.length == 1)
    5.delete socketStore[socket.userid]

    Flow
    socket(packet)->validator(do)->processor(do)->login(logout)->oldRowBuilder(logout)->dispacher(dispatchResponse)->mongo-store(remove)

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

    //if (userSession.length == 1)
    5.delete socketStore[socket.userid]
    
    Flow
    socket(packet)->validator(do)->processor(do)->accountSetup(Mob_Signup)->oldRowBuilder(accountSetup)->dispacher(dispatchResponse)->mongo-store(remove)

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
    socket(packet)->validator(do)->processor(do)->almond(sanity_check)->oldRowBuilder(dummy)->dispacher(dispatchResponse)

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
    socket(packet)->validator(do)->processor(do)->notificationPreferences(get_notification_preference_list)->oldRowBuilder(get_notification_preference_list)->dispacher(dispatchResponse)

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
    socket(packet)->validator(do)->processor(do)->almond(get_almondmode)->oldRowBuilder(get_almondmode)->dispacher(dispatchResponse)

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
    socket(packet)->validator(do)->processor(do)->notification(Mobile_Notification_Registration)->oldRowBuilder(notificationAddRegistration)->dispacher(dispatchResponse)

<a name="283"></a>
## 33)NotificationDeleteRegistration (Command 283)
    Command no 
    283- JSON format
 
    Required 
    Command,CommandType,Payload

    SQl
    2.Delete on NotificationID
      params:HashVal,RegID,UserID,Platform

    Functional
    1.Commad 283
    3.Send listResponse,commandLengthType ToMobile       //where listResponse = payload

    Flow
    socket(packet)->validator(do)->processor(do)->notification(Mobile_Notification_Delete_Registration)->oldRowBuilder(notificationDeleteRegistration)->dispacher(dispatchResponse)

<a name="804a"></a>
## 34.Command 804 (Client)
    Command no 
    804- JSON format
 
    Required 
    Command,CommandType,Payload,AlmondMAC,ClientID

    SQl
    2.Select on WifiClients
      params:AlmondMAC,ClientID
   
    Cassandra
    3.Select on dynamic_log
      params: mac,id
   
    Functional
    1.Command 804
    4.Send listResponse,commandLengthType ToMobile       //where listResponse = payload

    Flow
    socket(packet)->validator(do)->processor(do)->notification(get_logs)->notificationFetcher(getLogs)->oldRowBuilder(getLogs)->dispacher(dispatchResponse)

<a name="804b"></a>
## 35.Command 804 (Device)
    Command no 
    804- JSON format
 
    Required 
    Command,CommandType,Payload,DeviceID
       
    Cassandra
    2.Select on dynamic_log
      params: mac,id

    Functional
    1.Command 804
    3.Send listResponse,commandLengthType ToMobile       //where listResponse = payload

    Flow
    socket(packet)->validator(do)->processor(do)->notification(get_logs)->notificationFetcher(getLogs)->oldRowBuilder(getLogs)->dispacher(dispatchResponse)

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
    socket(packet)->validator(do)->processor(do)->notificationFetcher(makeBadgeZero)->oldRowBuilder(clear_the_badge)->dispacher(dispatchResponse)

<a name="300"></a>
## 37.NotificationPreferences(Command 300)
    Command no 
    300- JSON format
 
    Required 
    Command,CommandType,Payload

    Redis

    multi
    5.hgetall on UID_<userID>     //here, multi is done on every userID in UserList

    Queue
    6.Send UserProfileResponse to MobileQueue            

    SQl
    2.Insert on NotificationPreferences
      params: AlmondMAC, DeviceID, UserID,IndexVal, NotificationType

    Functional
    1.Command 300
    3.Send listResponse,commandLengthType ToMobile       //where listResponse = payload
    4.Send listResponse,commandLengthType ToMobile       //where listResponse = payload

    Flow
    socket(packet)->validator(do)->processor(do)->notificationPreferences(update_notification_preferences)->oldRowBuilder(notificationPreferences)->dispacher(dispatchResponse)->dispatcher(broadcast)->broadcastBuilder(defaultXML)->broadcaster(broadcast)

<a name="800"></a>
## 38.Command 800
    Command no 
    800- JSON format
 
    Required 
    Command,CommandType,Payload

    Cassandra
    2.Select on notification_store.notification_records 
    params:usr_id

    Functional
    1.Command 800
    3.Send listResponse,commandLengthType ToMobile       //where listResponse = payload

    Flow
    socket(packet)->validator(do)->processor(do)->notificationFetcher(getNotifications)->oldRowBuilder(get_notifications)->dispacher(dispatchResponse)

<a name="1060a"></a>
## 39.ChangeUser (Command 1060,Action:Add) 
    Command no 
    1060- JSON format
 
    Required 
    Command,CommandType,Payload,AlmondMAC,UserID

    Redis
    3.hgetall on AL_<payload.AlmondMAC>

    multi
    6.hgetall on UID_<userID>          //here, multi is done on every userID in UserList

    Queue
    7.Send UserProfileResponse to MobileQueue            

    SQl
    2.update on AlmondplusDB.WifiClients
    params:UserName,AlmondMAC,ClientID

    Functional
    1.Command 1060
    4.Send listResponse,commandLengthType ToMobile       //where listResponse = payload
    5.Send listResponse,commandLengthType ToMobile       //where listResponse = payload

    Flow
    socket(packet)->validator(do)->processor(do)->clientModel(update)->newRowBuilder(change_user)->dispatcher(broadcast)->broadcastBuilder(change_user)->broadcaster(broadcast)

<a name="1060b"></a>
## 40.ChangeUser (Command 1060,Action:Update) 
    Command no 
    1060- JSON format
 
    Required 
    Command,CommandType,Payload,AlmondMAC,UserID

    Redis
    4.hgetall on AL_<payload.AlmondMAC>

    multi
    7.hgetall on UID_<userID>         //here, multi is done on every userID in UserList

    Queue
    8.Send UserProfileResponse to MobileQueue           

    SQl
    2.Update on AlmondplusDB.WifiClients 
      params: UserName,AlmondMAC, UserName

    3.Update on AlmondplusDB.WifiClients 
      params: UserName,AlmondMAC,ClientID

    Functional
    1.Command 1060
    5.Send listResponse,commandLengthType ToMobile       //where listResponse = payload
    6.Send listResponse,commandLengthType ToMobile       //where listResponse = payload

    Flow
    socket(packet)->validator(do)->processor(do)->clientModel(update)->newRowBuilder(change_user)->dispatcher(broadcast)->broadcastBuilder(change_user)->broadcaster(broadcast)
    

<a name="1004"></a>
## 41.SUPER_LOGIN (Command 1004) 
    Command no 
    1004- JSON format
 
    Required 
    Command,CommandType,Payload

    Redis
    5.hgetall on UID_<data.UserID> 
    7.hincrby on UID_<data.UserID>         //values = (Q_<config.SERVER_NAME>,1)

   
    Cassandra
    2.Insert on logging.error_log
      params: date,time,ip,server,category,error
    
    SQl
    3.Select on Users
      params: EmailID
    4.Insert on UserTempPasswords
      params:UserID,TempPassword,LastUsedTime
    

    Functional
    1.Command 1004
    6.Send listResponse,commandLengthType ToMobile       //where listResponse = payload

    Flow
    socket(packet)->validator(do)->validator(login)->logging(errorLog)->processor(do)->login(Mob_Login)->redisManager(getAllAlmonds)->oldRowBuilder(loginJSON)->dispatcher(dispatchResponse)->mongo-store(add)->redisManager(redisExecute)

<a name="1110a"></a>
## 42.UnlinkAlmondRequest (Command 1110) 
    Command no 
    1110- JSON format
 
    Required 
    Command,CommandType,Payload,AlmondMAC
     
    Redis
    4.hgetall on AL_<data.AlmondMAC>

    SQl
    2.Select on Users
      params:EmailID

    Queue
    5.Send UnlinkAlmondRequestResponse to AlmondServer

    Functional
    1.Command 1110
    3.Send listResponse,commandLengthType ToMobile       //where listResponse = payload
 
    Flow
    socket(packet)->validator(do)->validator(unlink),validator(checkCredentials)->sqlManager(getUser)->processor(do)->account-manager-json(UnlinkAlmond)->rowBuilder(defaultReply)->dispatcher(unicast)->dispatcher(dispatchResponse)->account-manager-json(getAlmond)->broadcastBuilder(unlink)->broadcaster(unicast)

<a name="1110b"></a>
## 43.UserInviteRequest (Command 1110) 
    Command no 
    1110- JSON format
 
    Required 
    Command,CommandType,Payload,AlmondMAC

    SQl
    2.Select on Users
      params: UserID
    4.Select on AlmondSecondaryUsers
      params: AlmondMAC,UserID
    5.Insert on AlmondSecondaryUsers
      params:AlmondMAC,userID
    8.Select on Users
      params: UserID

    Redis
    3.hgetall on UID_<userid>
    6.hmset on AL_<JsonObj.AlmondMAC>      //where value = [SUSER_<UserID>,1]
    7.hmset on UID_<userid>               //where value = [SMAC_<JsonObj.AlmondMAC>,1]
    9.hgetall on AL_<AlmondMAC>
    11.hgetall on AL_<AlmondMAC>
    12.get on ICID_<code>                //where code = random string
    13.setex on ICID_<code>            //where code = random string

    multi
    17.hgetall on UID_<data.SecondaryUsers>     //here, multi is done on every SecondaryUsers 

    Queue
    15.Send UserInviteRequestResponse to config.SERVER_NAME
    14.Send UserInviteRequestResponse to MobileQueue
    
    Functional
    1.Command 1110
    10.Send listResponse,commandLengthType ToMobile       //where listResponse = payload
    14.delete store[commandID]
    16.Send listResponse,commandLengthType ToMobile       //where listResponse = payload

    Flow
    socket(packet)->validator(do)->processor(do)->account-manager-json(UserInvite)->sqlManager(checkSecondary),sqlManager(addSecondaryAlmond)->redisManager(addSecondaryAlmond)->sqlManager(getEmail)->redisManager(getAlmond)->rowBuilder(defaultReply)->dispatcher(dispatchResponse)->dispatcher(unicast)->rowBuilder(userChange)->rowBuilder(userChange)->broadcaster(unicast)->dispatcher(broadcast)->broadcastBuilder(almondAdd),broadcastBuilder(userAdd)->broadcaster(broadcast)

<a name="1110c"></a>
## 44.UpdateUserProfileRequest (Command 1110) 
    Command no 
    1110- JSON format
 
    Required 
    Command,CommandType,Payload,UserID

    Redis

    multi
    5.hgetall on UID_<userID>         //here, multi is done on every userID in UserList
    
    SQl
    2.Update on Users
      params: UserID

    Queue
    6.Send UserProfileResponse to MobileQueue 
   
    Functional
    1.Command 1110
    3.Send listResponse,commandLengthType ToMobile       //where listResponse = payload
    4.Send listResponse,commandLengthType ToMobile       //where listResponse = payload

    Flow
    socket(packet)->validator(do)->processor(do)->account-manager-json(UpdateUserProfile)->rowBuilder(UpdateUserProfile)->dispatcher(dispatchResponse)-> dispatcher(broadcast)->broadcastBuilder(userProfileUpdate)->broadcaster(broadcast)

<a name="1110d"></a>
## 45.DeleteSecondaryUserRequest (Command 1110) 
    Command no 
    1110- JSON format
 
    Required 
    Command,CommandType,Payload,UserID,AlmondMAC

    SQl
    2.Delete on AlmondSecondaryUsers
      params: AlmondMAC, userID
    3.Delete on NotificationPreferences
      params:  AlmondMAC, userID

    Redis
    4.hdel on  AL_<AlmondMAC>             // value = SUSER_<secondaryUser>
    5.hdel on UID_<secondaryUser>         //value = SMAC_<AlmondMAC>
    7.hgetall on AL_<AlmondMAC>
    8.get on ICID_<code>                //where code = random string
    9.setex on ICID_<code>            //where code = random string    

    multi
    13.hgetall on UID_<userList>   
      /* here, multi is done on userList where userList = data.userListAlmondDelete,data.userListUserDelete */

    Queue
    11.Send DynamicUserChangeResponse to config.SERVER_NAME
    14.Send DynamicAlmondDeleteResponse,DynamicUserDeleteResponse to MobileQueue

    Functional
    1.Command 1110
    6.Send listResponse,commandLengthType ToMobile       //where listResponse = payload
    10.delete store[commandID]
    12.Send listResponse,commandLengthType ToMobile       //where listResponse = payload

    Flow
    socket(packet)->validator(do)->processor(do)->account-manager-json(DeleteSecondaryUser)->sqlManager(deleteUser)->redisManager(redisExecute),redisManager(deleteAccount)->rowBuilder(defaultReply)->dispatcher(dispatchResponse)->dispatcher(unicast)-> rowBuilder(userChange)->broadcaster(unicast)->dispatcher(broadcast)->broadcastBuilder(almondDelete),broadcastBuilder(userDelete)->broadcaster(broadcast)

<a name="1110e"></a>
## 46.DeleteMeAsSecondaryUserRequest (Command 1110) 
    Command no 
    1110- JSON format
 
    Required 
    Command,CommandType,Payload,UserID,AlmondMAC

    SQl    
    3.Delete on AlmondSecondaryUsers
      params: AlmondMAC, userID
    4.Delete on NotificationPreferences
      params:  AlmondMAC, userID

    Redis
    2.hgetall on AL_<AlmondMAC>
    5.hdel on  AL_<AlmondMAC>             // value = SUSER_<secondaryUser>
    6.hdel on UID_<secondaryUser>         //value = SMAC_<AlmondMAC>
    8.hgetall on AL_<AlmondMAC>
    9.get on ICID_<code>                //where code = random string
    10.setex on ICID_<code>            //where code = random string

    multi
    14.hgetall on UID_<userList>   
      /* here, multi is done on userList where userList = data.userListAlmondDelete,data.userListUserDelete */

    Queue
    12.Send DynamicUserChangeResponse to config.SERVER_NAME
    15.Send DynamicAlmondDeleteResponse,DynamicUserDeleteResponse to MobileQueue
    
    Functional
    1.Command 1110
    7.Send listResponse,commandLengthType ToMobile       //where listResponse = payload
    11.delete store[commandID] 
    13.Send listResponse,commandLengthType ToMobile       //where listResponse = payload

    Flow
    socket(packet)->validator(do)->processor(do)->account-manager-json(DeleteMeAsSecondaryUserResponse)->redisManager(getAlmond)->sqlManager(deleteUser)->redisManager(redisExecute),redisManager(deleteAccount)->rowBuilder(defaultReply)->dispatcher(dispatchResponse)->dispatcher(unicast)-> rowBuilder(userChange)->broadcaster(unicast)->dispatcher(broadcast)->broadcastBuilder(almondDelete),broadcastBuilder(userDelete)->broadcaster(broadcast)

<a name="1110f"></a>
## 47.ChangePasswordRequest (Command 1110) 
    Command no 
    1110- JSON format
 
    Required 
    Command,CommandType,Payload,UserID,AlmondMAC

    SQL 
    2.Select on Users
     params:EmailID

    3.Update on Users
     params:Password,EmailID
     
    4.Delete on UserTempPasswords         //if (rows.affectedRows == 1)
     params:UserID
    5.Delete on NotificationID
     params:UserID

            (or)

     4.return            //if (rows.affectedRows == 0)

    /* note: considered that rows are affected */

    Redis
    7.hmset on UID_<socket.userid>        //values = Q_<config.SERVER_NAME,userSession.length-1>
    
    multi
    9.hgetall on UID_<userID>          //here multi is done on every userID in userList

    Queue
    10.Send ChangePasswordRequestResponse to MobileQueue

    Functional
    1.Command 1110
    6.Send listResponse,commandLengthType ToMobile       //where listResponse = payload
    8.delete socketStore[userid]

    Flow
    socket(packet)->validator(do)->validator(checkCredentials)->processor(do)->account-manager-json(ChangePassword)->connection-pool(queryFunction)->rowBuilder(defaultReply)->dispatcher(dispatchResponse)->mongo-store(removeAllExceptCurrent)->dispatcher(broadcast)->broadcastBuilder(removeAll)->broadcaster(broadcast)

<a name="1110g"></a>
## 48.DeleteAccountRequest (Command 1110) 
    Command no 
    1110- JSON format
 
    Required 
    Command,CommandType,Payload,UserID,AlmondMAC

    SQL 
    2.Select on Users
      params:EmailID
    3.Delete on Users
      params: UserID
    9.Delete on AlmondUsers
      params: AlmondMAC
    10.Update on AllAlmondPlus
      params: AlmondMAC 

    Redis
    4.hgetall on UID_<packet.userid>
    5.del on UID_<packet.userid>

    multi
    6.hgetall on AL_<pMACs>

    multi
    7.del on AL_<pMACs>

    multi
    8.hdel on UID_<Entry[1]>           //values = SMAC_<AlmondMAC>, Entry[1] =Secondary UserID 

    13.hmset on UID_<data.userid>   //values = (Q_<config.SERVER_NAME>,0)

    multi
    14.hgetall on UID_<userID>          //here multi is done on every userID in userList

    Queue
    15.Send DeleteAccountRequestResponse to MobileQueue
    16.Send DeleteAccountResponse to config.HTTP_SERVER_NAME

    Functional
    1.Command 1110
    11.Send listResponse,commandLengthType ToMobile       //where listResponse = payload
    12.delete socketStore[data.userid]

    Flow
    socket(packet)->validator(do)->validator(checkCredentials)->processor(do)->account-manager-json(DeleteAccount)->sqlManager(deleteUser)->redisManager(redisExecute),redisManager(deleteAccount)->rowBuilder(defaultReply)->dispatcher(dispatchResponse)->mongo-store(removeAll)->dispatcher(broadcast)->broadcastBuilder(removeAll)->broadcaster(broadcast)->dispatcher(broadcastToAllAlmonds)->broadcaster(broadcastModel)

<a name="1011a"></a>
## 49.PaymentDetails (Command 1011) 
    Command no 
    1011- JSON format
 
    Required 
    Command,CommandType,Payload,UserID,AlmondMAC
    
    Redis
    2.hgetall on UID_<packet.userid>      // values = PMAC_<AlmondMAC>

    Queue
    4.Send PaymentDetailsResponse to config.HTTP_SERVER_NAME

    Functional
    1.Command 1011
    3.delete store[data.UnicastID]
    5.Send listResponse,commandLengthType ToMobile           //where listResponse = payload

    Flow 
    socket(packet)->validator(do)->processor(do)->subscriptionCommands(subscriptionCommands)-
    >redisManager(getAlmonds)->mongo-store(getSocket)->requestQueue(set)->producer(sendToQueue)->dispatcher(dispatchResponse)

<a name="1011b"></a>
## 50.UpdateCard (Command 1011) 
    Command no 
    1011- JSON format
 
    Required 
    Command,CommandType,Payload,UserID,AlmondMAC

    Redis
    2.hgetall on UID_<packet.userid>      // values = PMAC_<AlmondMAC>

    Queue
    4.Send UpdateCardResponse to config.HTTP_SERVER_NAME

    Functional
    1.Command 1011
    3.delete store[data.UnicastID]
    5.Send listResponse,commandLengthType ToMobile           //where listResponse = payload

    Flow 
    socket(packet)->validator(do)->processor(do)->subscriptionCommands(subscriptionCommands)-
    >redisManager(getAlmonds)->mongo-store(getSocket)->requestQueue(set)->producer(sendToQueue)->dispatcher(dispatchResponse)
    
<a name="1011c"></a>
## 51.DeleteSubscription (Command 1011) 
    Command no 
    1011- JSON format
 
    Required 
    Command,CommandType,Payload,UserID,AlmondMAC

    Redis
    2.hgetall on UID_<packet.userid>      // values = PMAC_<AlmondMAC>

    Queue
    4.Send DeleteSubscriptionResponse to config.HTTP_SERVER_NAME

    Functional
    1.Command 1011
    3.delete store[data.UnicastID]
    5.Send listResponse,commandLengthType ToMobile           //where listResponse = payload

    Flow 
    socket(packet)->validator(do)->processor(do)->subscriptionCommands(subscriptionCommands)-
    >redisManager(getAlmonds)->mongo-store(getSocket)->requestQueue(set)->producer(sendToQueue)->dispatcher(dispatchResponse)
