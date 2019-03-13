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
- [SubscribeMe (Command 1011)](1011)
- [GetDevicePreferences (Command 1700)](1700a)
- [GetClientPreferences (Command 1700)](1700b)
- [UpdateDevicePreferences (Command 1700)](1700c)
- [UpdateClientPreferences (Command 1700)](1700d)
- [RouterSummary (Command 1100)](#1100)
- [AffiliationUserRequest (Command 23)](#23)

<a name="1061"></a>
## 1)Command 1061
    Command no 
    1061- JSON format
 
    Required 
    Command,CommandType,Payload,almondMAC

    Redis
    //(AlmondMAC = <packet.parsedPayload.AlmondMAC>)
    3.hgetall on AL_<packet.parsedPayload.AlmondMAC>
    
    //(prefix+key = ICID_ + <string>, here <string> = random string data), value = null
    4.get on ICID_<string>

    /*if (client.connected) {
        client["setex"](key, time, value, callback);
    }*/
    5.setex on ICID_<string>            // (prefix + key), value = SERVER_NAME

    Queue
     // (payload,command,almondMAC) to queue
    8.Send response to o.server

    Functional 
    1.Command 1061
    2.Send listResponse,commandLengthType ToMobile         //where listResponse = payload
    6.delete store[commandID]
    7.Return commandID

    Flow
    socket(packet)->validator(do)->processor(do)->commandMapping(almond.onlyUnicast)->dispatcher(dispatchResponse)->dispatcher(unicast)

<a name="1200"></a>
## 2) DeviceList (Command 1200)
    Command no 
    1200- JSON format
 
    Required 
    Command,CommandType,Payload

    Functional 
    1.Command 1200
    2.Return modifiedRows       // rowBuilder->Devices

    3.Send listResponse,commandLengthType ToMobile          //where listResponse = payload
    
    Flow
    socket(packet)->validator(do)->processor(do)->commandMapping(device.execute)->dispatcher(dispatchResponse)

<a name="1300"></a>
## 3) SceneList (Command 1300)
    Command no 
    1300- JSON format
 
    Required 
    Command,CommandType,Payload,AlmondMAC

    SQl
    2.select HASH_QUERY
      params: payload.mapper.hashColumn, payload.AlmondMAC

    Functional 
    1.Command 1300
    3.return modifiedRows       //rowBuilder->getResponse
    4.return row                //rowBuilder->scenes

    5.Send listResponse,commandLengthType ToMobile          //where listResponse = payload
    
    Flow
    socket(packet)->validator(do)->processor(do)->commandMapping(genericModel.execute)->dispatcher(dispatchResponse)

<a name="1400"></a>
## 4) RuleList (Command 1400)
    Command no 
    1400- JSON format
 
    Required 
    Command,CommandType,Payload

    Functional 
    1.Command 1400
    2.return modifiedRows       //rowBuilder->getRespponse
    3.return row                //rowBuilder->rules
 
    4.Send listResponse,commandLengthType ToMobile         //where listResponse = payload
    
    Flow
    socket(packet)->validator(do)->processor(do)->commandMapping(genericModel.execute)->dispatcher(dispatchResponse)

<a name="1500"></a>
## 5) ClientList (Command 1500)
    Command no 
    1500- JSON format
 
    Required 
    Command,CommandType,Payload

    Functional 
    1.Command 1500
    2.return modifiedRows                //rowBuilder->getResponse
    3.return parsedClients               //rowBuilder->clients

    4.Send listResponse,commandLengthType ToMobile         //where listResponse = payload
    
    Flow
    socket(packet)->validator(do)->processor(do)->commandMapping(genericModel.execute)->dispatcher(dispatchResponse)

<a name="1050"></a>
## 6) AlmondProperties (Command 1050) 
    Command no
    1050- JSON format 

    Required
    Command,CommandType,Payload,almondMAC

    SQl
    2.Select on AlmondProperties2
     params: data.AlmondMAC

    Functional
    1.Command 1050
    3.return result              // rowBuilder->almondProperties

    4.Send listResponse,commandLengthType ToMobile           //where listResponse = payload

    Flow 
    socket(packet)->validator(do)->processor(do)->commandMapping(almond.AlmondProperties)->dispatcher(dispatchResponse)

<a name="1011"></a>
## 7) Command 1011 
    Command no
    1011- JSON format 

    Required
    Command,CommandType,Payload

    Redis 
    // where UserID = packet.userid
    2.hgetall on UID_<packet.userid>              //(M.USER + UserID)
 
    Queue
    8.Send SubscribeMeResponse to config.HTTP_SERVER_NAME
     
    Functional
    1.Command 1011
    3.return entries                     // RM.getAlmonds->getKeys
    4.socketStore[packet.userid]         // MS.getSocket->MS.hget
    5.return sendSocket                  // MS.getSocket
    6.delete store[data.UnicastID]       // requestQueue->cleanup
    7.return data.UnicastID              // requestQueue.set  
    9.return SubscribeMeResponse         //rowBuilder->default

    Flow 
    socket(packet)->validator(do)->processor(do)->commandMapping(SC.subscriptionCommands)->dispatcher(dispatchResponse)

<a name="1700a"></a>
## 8)GetDevicePreferences (Command 1700) 
    Command no
    1700- JSON format 

    Required
    Command,CommandType,Payload

    SQl
    2.Select on NotificationPreferences
      params: NotificationPreferences

    Functional
    1.Command 1700
    3.Send listResponse,commandLengthType ToMobile            //where listResponse = payload

    Flow 
    socket(packet)->validator(do)->processor(do)->commandMapping(newPref.do)->dispatcher(dispatchResponse)->broadcastBuilder(preferences)

<a name="1700b"></a>
## 9)GetClientPreferences (Command 1700) 
    Command no
    1700- JSON format 

    Required
    Command,CommandType,Payload

    SQl
    2.Select on ClientPreferences
      params: WifiClientsNotificationPreferences

    Functional
    1.Command 1700 
    3.return GetClientPreferences response              //oldRowBuilder.newPref
    4.Send listResponse,commandLengthType ToMobile      //where listResponse = payload
 
    Flow 
    socket(packet)->validator(do)->processor(do)->commandMapping(newPref.do)->dispatcher(dispatchResponse)->broadcastBuilder(preferences)

<a name="1700c"></a>
## 10)UpdateDevicePreference (Command 1700) 
    Command no
    1700- JSON format 

    Required
    Command,CommandType,Payload

    SQl
    2.Select on NotificationPreferences
      params: NotificationPreferences

    Functional
    1.Command 1700
    3.Send listResponse,commandLengthType ToMobile               //where listResponse = payload
    4.return DynamicDevicePreferencesResponse,socket.userid      // broadcastBuilder.preferences

    Flow 
    socket(packet)->validator(do)->processor(do)->commandMapping(newPref.do)->dispatcher(dispatchResponse)->broadcastBuilder(preferences)

<a name="1700d"></a>
## 11)UpdateClientPreferences (Command 1700) 
    Command no
    1700- JSON format 

    Required
    Command,CommandType,Payload

    SQl
    2.Select on ClientPreferences
      params: WifiClientsNotificationPreferences

    Functional
    1.Command 1700
    3.Send listResponse,commandLengthType ToMobile             //where listResponse = payload
    4.return DynamicClientPreferencesResponse,socket.userid      // broadcastBuilder.preferences

    Flow 
    socket(packet)->validator(do)->processor(do)->commandMapping(newPref.do)->dispatcher(dispatchResponse)->broadcastBuilder(preferences)

<a name="1100"></a>
## 12)RouterSummary (Command 1100)
    Command no 
    1100- JSON format
 
    Required 
    Command,CommandType,Payload,almondMAC

    Redis
    //(AlmondMAC = <packet.parsedPayload.AlmondMAC>)
    3.hgetall on AL_<packet.parsedPayload.AlmondMAC>
    
    //(prefix+key = ICID_ + <string>, here <string> = random string data), value = null
    4.get on ICID_<string>

    /*if (client.connected) {
        client["setex"](key, time, value, callback);
    }*/
    5.setex on ICID_<string>            // (prefix + key), value = SERVER_NAME

    Queue
     // (payload,command,almondMAC) to queue
    8.Send response to o.server

    Functional 
    1.Command 1100
    2.Send listResponse,commandLengthType ToMobile        //where listResponse = payload
    6.delete store[commandID]
    7.Return commandID

    Flow
    socket(packet)->validator(do)->processor(do)->commandMapping(almond.onlyUnicast)->dispatcher(dispatchResponse)->dispatcher(unicast)

<a name="23"></a>
## 13)AffiliationUserRequest (Command 23)
    Command no 
    23- JSON format
 
    Required 
    Command,CommandType,Payload

    Redis
    // where, data.code = answer
    2.get on CODE:<data.code>  

    //(prefix+key = ICID_ + <string>, here <string> = random string data), value = null
    4.get on ICID_<string>

    /*if (client.connected) {
        client["setex"](key, time, value, callback);
    }*/
    5.setex on ICID_<string>            // (prefix + key), value = SERVER_NAME       

    SQl 
    3.Select on Users
      params: UserID
     
    Queue
    11.Send AffiliationUserRequestResponse to queue
 
    Functional
    1.Command 23
    6.return AffiliationUserCompleteResponse             //newRowBuilder->affiliationError
    7.Send listResponse,commandLengthType ToMobile       //where listResponse = payload
    8.delete store[row.CommandID]
    9.return CommandID
    10.return out               //where out = response, unicast->newRowBuilder.affiliation



