# MobileIndex (Mobile Server)
### Table of contents
- [DynamicRuleAdded,DynamicRuleUpdated,DynamicRule,Removed,DynamicAllRulesRemoved,RuleList,AddRule,
UpdateRule,RemoveRule,RemoveAllRules,ValidateRule,DeviceList,GetDeviceIndex,UpdateDeviceName,
UpdateAlmondName,DynamicAlmondModeUpdated,DynamicIndexUpdated,DynamicDeviceRemoved,
DynamicAllDeviceRemoved,DeviceOnlineCheck,DynamicClientAdded,DynamicClientJoined,DynamicClientLeft,
DynamicClientUpdate,DynamicClientRemoved,DynamicRemoveAllClient,ClientList,UpdateClient,RemoveClient,
RemoveAllClients,WifiClients,DynamicSceneAdded,DynamicSceneUpdated,DynamicSceneActivated,
DynamicSceneRemoved,DynamicAllSceneRemoved,AddScene,SetScene,ActivateScene,DeleteScene,
DeleteAllScene (Command 1061)](#1061) 

<a name="1061"></a>
## 1) Command 1061
    Command no 
    1061- JSON format
 
    Required 
    Command,CommandType,Payload

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




