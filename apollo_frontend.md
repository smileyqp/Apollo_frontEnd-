前台消息接收方式为:监听端口49923,在`webworker`接收消息`message`,并判断消息的`source`,消息的`source`分为三种`map`,`map_cloud`以及`realtime`,此时给消息`type`,即`realtime`为`SimWorldUpdate`,`map`为`MapData`,`point_cloud`为


接收数据message:


---------------------------------realtime接收数据部分---------------------------------------------

#####1.HMIConfig

######接收的为导航栏部分数据

> - dockerImage:字符串文本格式,显示为弹出框版本信息.EX:apolloauto/apollo:dev-x86_64-20180906_2002
> - modes:模式setup mode的String数组,是mode的下拉列表菜单
>         传输过去:{type:ChangeMode,mode:value值}
> - utmZoneId:数据,其中初始值为10(?I can not find it's function?where?what?)
> - availableVehicles:string数组,将其sort().map(name)排序后就是页面上vehicles的下拉列表
>           传输过去:{type:ChangeVehicle,new_vehicle: vehcile选项的value值}
> - availableMaps:string数组,将其sort().map(name)排序后就是页面上的map的下拉列表  

传输过去:`{type:ChangeMap,new_map: map选项的value值}`

######接收的为左侧为module controller的模块数据

modules:接收的为对象数据结构为` {key1：value1, key2：value2, ...} `,键值对结构.接收过来的是键值对形式的对象,之后将其传入到映射中取值
        传输过去:这边

```shell
  {type: "ExecuteModuleCommand",
            module: module,//即其中对应的相应模块名称即key
            command: command,}//command即其对应的状态
//备注WS.executeModuleCommand(id, command)======>>>key值,key的value值是一个状态,即在其中对应的某讴
歌时间对应的状态
//executeModuleCommand(module, command)
```

hardware:该模块与上面模块接收相同都是键值对,但是该模块没有发送值只有接收其status在update状态的时候
改变其值

#####2.HMIStatus
主要是上一步的初始化之后的状态更新(涉及function有`updateStatus()`和`updateGroundImage()`)

`currentMode`:string类型字符串,推测为初始化之后服务器端穿过来的一系列mode即模式之后前段选择那种模式>给后端传过去,再之后后端讲这种模式返回来传给前端即在更新状态时候显示

`currentMap`:string类型字符串,同上

`currentVehicle`:同上string类型,以及采取的方式同上

`systemStatus`:在这个`systemstatus`中,传过来的也是一个键值对对象,该键值对对象中有两个参数传过来的两个key以及其数据格式如下:
`modules`:modules也是一个键值对的格式,将这个键值对的值存到`moduleStatus`,`moduleStatus`中的key为
其中取出来的key,但是其值就`modules[key].processStatus.running`;`moduleStatus`是一个键值对结构,之前提到
过,存储了key为`Modules`这边的值,然后他的`value`则是状态值
 `hardware`:键值对格式,同上实时更新值

`passengerMsg`:这是一个string类型的消息

#####3.VehicleParam
json格式数据类似于 

```shell
vehicleParam = {
    frontEdgeToCenter: 3.89,
    backEdgeToCenter: 1.04,
    leftEdgeToCenter: 1.055,
    rightEdgeToCenter: 1.055,
    height: 1.48,
    width: 2.11,
    length: 4.933,
    steerRatio: 16,
}
```


这个中是直接传过来一个键值对数值,将这个键值对的值直接传给`vehicleParam`参数,之后在store的`update(world){}`这个function中调用

#####4.SimControlStatus
穿过来的值只是触发button的enabled

```shell
STORE.setOptionStatus('simControlEnabled', message.enabled);
```

#####5.SimWorldUpdate
```shell
timestamp
world.sequenceNum
```

    //autoDrivingCar键值对结构:
    positionX,positionY,heading


        navigationPath键值对map
        laneMarker
        planningTrajectory

#####6.MapElementIds

```shell
mapHash,mapElementIds,mapRadius
```

#####7.DefaultEndPoint
​        `poi`数组,数组里面包含`map`,`map`包含`name`和`waypoint`格式如下:

```shell
Map from POI name to its x,y coordinates,
{POI-1: [{x: 1.0, y: 1.2}, {x: 101.0, y: 10.2}]}
```

#####8.RoutePath
`routingTime,routePath`监听三种传过来的数据`source`,三种`source`分别为`realtime,map,point_cloud:
reaktime`的`message type`为`SimWorldUpdate`

`map`的`message type`为·MapData·

`point_cloud`的`message type`?此处没有写明

-----------------------------point cloud点云接收数据的部分-------------------------------------
type:PointCloudStatus:传进去的而数据为数组点集合

-------------------------------websocket map数据接收部分-----------------------------
传过来一串map相关数据的数组

-------------------------offline数据接收部分;这个是离线部分咩------------------------------------
`GroundMetadata`传过来的数据中包含这些,在`render`的`tileground`中,初始化使用这些数据

```shell
this.metadata = {
            tileLength: metadata.tile * metadata.mpp,
            left: metadata.left,
            top: metadata.top,
            numCols: metadata.wnum,
            numRows: metadata.hnum,
            mpp: metadata.mpp,
            tile: metadata.tile,
            imageUrl: metadata.image_url,

};
```

`FrameCount`:传过来的数据为一串string,但是这个string是只含有数字的,之后会用`parseInt`将其解析出来使用

`RoutePath`:
传过来的数据为一个对象有两个属性`routingTime`和`rouotePath`,分别有值,键值对形式

`SimWorldUpdate`:
传过来的数据用于`message`的`check`,含有数据为`timestamp`和`world(其中world中还有sequenceNum)`

--------------------------------------------------------------------------------------------
                                数据发送部分
--------------------------------------------------------------------------------------

------------------------------map------------------------------------------
type:RetrieveMapData
传过去的element是一个对象,对象中每个属性即key的值即value又是一个数组

```shell
type:RetrieveRelativeMapData
```

--------------------------------point_cloud----------------------------------

```shell
type:TogglePointCloud
```


传过去的值`enable`的值为布尔值

```shell
//realtime
function:clearInterval
type:RequestSimulationWorld
planning : requestPlanningData,

function:requestRoute
        type: "SendRoutingRequest",
            start: start,
            end: end,
            waypoint: waypoint,

function:requestDefaultRoutingEndPoint
        type:GetDefaultEndPoint,没有传其他数据

function:resetBackend
        type: "Reset",没有其他数据

function:dumpMessages
        type:Dump

function:changeSetupMode
        type: "ChangeMode",
            new_mode: mode,

function:changeMap
        type: "ChangeMap",
            new_map: map,

function:changeVeicle
        type: "ChangeVehicle",
            new_vehicle: vehcile,

function:executeModeCommand
        type: "ExecuteModeCommand",
            command: command,

function:executeModuleCommand
        type: "ExecuteModuleCommand",
            module: module,
            command: command,

function:executeToolCommand
         type: "ExecuteToolCommand",
            tool: tool,
            command: command,

function:changeDrivingMode
        type: "ChangeDrivingMode",
            new_mode: mode,

function:submitDriveEvent
        type: "SubmitDriveEvent",
            event_time_ms: eventTimeMs,
            event_msg: eventMessage,
            event_type: eventTypes,

function:sendAudioPiece
        type: "AudioPiece",
            data: btoa(String.fromCharCode(...data)),

function:toggleSimControl
        type: "ToggleSimControl",
            enable: enable,
                                                                               
function:requestRoutePath
        type: "RequestRoutePath",

function:publishNavigationInfo
        this.websocket.send(data);这个function中没有type是直接send data过去的

```

```shell
//offline传过去的数值
 type: "requestRoutePath"
                recordId: recordId,
                frameId: frameId,
```




