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

<a name="1061"></a>
## 1) Command 1061
    Command no 
    1061- JSON format
 
    Required 
    Command,CommandType,Payload,almondMAC

    Redis
    //(AlmondMAC = <packet.parsedPayload.AlmondMAC>)
    3.hgetall on AL_<packet.parsedPayload.AlmondMAC>
    
    //(prefix+key = data.prefix + <string>, here <string> = random string data)
    4.get on data.prefix + <string>

    /*if (client.connected) {
        client["setex"](key, time, value, callback);
    }*/
    6.setex on data.prefix + string            // (prefix + key)

    Queue
     // (payload,command,almondMAC) to queue
    9.Send packet.Payload.toString(),1061+1,packet.parsedPayload.AlmondMAC to o.server

    Functional 
    1.Command 1061
    2.Send listResponse,commandLengthType ToMobile
    5.return redis.client_replica;
    7.delete store[commandID]
    8.Return commandID

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

    3.Send listResponse,commandLengthType ToMobile
    
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

    5.Send listResponse,commandLengthType ToMobile
    
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

    4.Send listResponse,commandLengthType ToMobile
    
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

    4.Send listResponse,commandLengthType ToMobile
    
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

    4.Send listResponse,commandLengthType ToMobile

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

