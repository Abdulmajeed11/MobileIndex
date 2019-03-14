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
    socket(packet)->validator(do)->processor(do)->commandMapping(almond.onlyUnicast)->dispatcher(dispatchResponse)->dispatcher(unicast)->broadcaster(unicast)

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
    3.hgetall on MAC:%s<AlmondMAC>

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
    2.Select on ALMOND_USERS
      params: StaticSceneListHash, AlmondMAC

     //if (rows && rows[0] && rows[0].Hash != payload.HashNow)
    
    3.Update on ALMOND_USERS
      params: StaticSceneListHash, AlmondMAC

    4.Select on SCENES 
      params: AlmondMAC

    Functional 
    1.Command 1300
    5.Send listResponse,commandLengthType ToMobile          //where listResponse = payload
    
    Flow
    socket(packet)->validator(do)->processor(do)->genericModel(execute),genericModel(get)->newRowBuilder(scenes)->dispatcher(dispatchResponse)

<a name="1400"></a>
## 4) RuleList (Command 1400)
    Command no 
    1400- JSON format
 
    Required 
    Command,CommandType,Payload

    SQl
    2.Select on ALMOND_USERS
    params: StaticRuleListHash, AlmondMAC

    //if (rows && rows[0] && rows[0].Hash != payload.HashNow)
    
    3.Update on ALMOND_USERS
      params: StaticRuleListHash, AlmondMAC

    4.Select on RULE 
      params: AlmondMAC

    Functional 
    1.Command 1400
    5.Send listResponse,commandLengthType ToMobile         //where listResponse = payload
    
    Flow
    socket(packet)->validator(do)->processor(do)->genericModel(execute),genericModel(get)->newRowBuilder(Rules)->dispatcher(dispatchResponse)

<a name="1500"></a>
## 5) ClientList (Command 1500)
    Command no 
    1500- JSON format
 
    Required 
    Command,CommandType,Payload

    SQl
    2.Select on ALMOND_USERS
    params: StaticWifiClientListHash, AlmondMAC

    //if (rows && rows[0] && rows[0].Hash != payload.HashNow)
    
    3.Update on ALMOND_USERS
      params: StaticWifiClientListHash, AlmondMAC

    4.Select on WIFICLIENTS 
      params: AlmondMAC

    Functional 
    1.Command 1500
    5.Send listResponse,commandLengthType ToMobile         //where listResponse = payload
    
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
    socket(packet)->validator(do)->processor(do)->commandMapping(almond.AlmondProperties)->newRowBuilder(almondProperties)->dispatcher(dispatchResponse)

<a name="1011"></a>
## 7) Command 1011 
    Command no
    1011- JSON format 

    Required
    Command,CommandType,Payload

    Redis  
    2.hgetall on UID_<UserID>          // where UserID = packet.userid        
 
    Queue
    5.Send SubscribeMeResponse to config.HTTP_SERVER_NAME
     
    Functional
    1.Command 1011
    3.socketStore[packet.userid]         // MS.getSocket->MS.hget
    4.delete store[data.UnicastID]       // requestQueue->cleanup
    6.Send listResponse,commandLengthType ToMobile           //where listResponse = payload
  
    Flow 
    socket(packet)->validator(do)->processor(do)->commandMapping(SC.subscriptionCommands)-
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
    socket(packet)->validator(do)->processor(do)->commandMapping(newPref.do)->genericModel(select)->oldRowBuilder(newPref)->dispatcher(dispatchResponse)->dispatcher(broadcast)->broadcastBuilder(preferences)

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
    6.hgetall on UID_<userlist>

    SQl
    2.Delete on NotificationPreferences
      params: AlmondMAC,UserID

    Queue
    7.Send DynamicDevicePreferencesResponse to (key.substring(redisConstants.QUEUE.length, key.length))

    Functional
    1.Command 1700
    3.Send listResponse,commandLengthType ToMobile               //where listResponse = payload
    4.socketStore(userList)
    5.Send DynamicDevicePreferencesResponse ToMobile

    Flow
    socket(packet)->validator(do)->processor(do)->notiPrefs(do)->genericModel(delete)->oldRowBuilder(newPref)->dispatcher(dispatchResponse)->dispatcher(broadcast)->broadcastBuilder(preferences)

<a name="1700d"></a>
## 11)UpdateClientPreferences (Command 1700) 
    Command no
    1700- JSON format 

    Required
    Command,CommandType,Payload

    Redis
    6.hgetall on UID_<userlist>

    SQl
    2.Delete on ClientPreferences
      params: AlmondMAC,UserID
    
    Queue
    7.Send DynamicClientPreferencesResponse to (key.substring(redisConstants.QUEUE.length, key.length))

    Functional
    1.Command 1700
    3.Send listResponse,commandLengthType ToMobile             //where listResponse = payload
    4.socketStore(userList)
    5.Send DynamicClientPreferencesResponse ToMobile

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
    socket(packet)->validator(do)->processor(do)->commandMapping(almond.onlyUnicast)->dispatcher(dispatchResponse)->dispatcher(unicast)->broadcaster(unicast)

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
    socket(packet)->validator(do)->processor(do)->commandMapping(affiliation.execute)->redisManager(getCode)->sqlManager(getEmail)->cid-bid(incCommandID)->newRowBuilder(affiliationError)->dispatcher(dispatchResponse)->dispatcher(unicast)->broadcaster(unicast)



