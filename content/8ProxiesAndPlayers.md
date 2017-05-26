## 8. Proxies和玩家

### 8.1. Proxies
BaseApp上的KBEngine.Proxy继承于KBEngine.Base，目的是为玩家控制entity提供支持。从KBEngine.Proxy继承的entity，能够实现玩家角色，玩家账号和服务器任何其他玩家控制对象相关。  
无论何时一个客户端登录到服务器，一个从KBEngine.Proxy衍生的entity会从数据库被创建。KBE如何确定加载哪个Proxy的细节，请看`User Authentication And Proxy Selection`章节。以这个方式创建的Proxies会有一个叫做`password`的属性，这个属性的值会被设置为登录密码。  
一个KBEngine.Proxy实例既不需要cell entity或者客户端。和其他entity一样，一个proxy entity能够使用KBEngine.createBase来创建。详细请看BaseAPp的`Entity Instantiation and Destruction`章节。  

和其它Base entity一样，保存proxy entity到数据库和从其中加载entity都是可以的。开始时，这些重新加载的proxy entity是没有客户端依附的。接着，一个已经存在的proxy可以把它的客户端转交给重载的proxy，重载的entity将负责处理这个客户端连接。  
这个可以实现功能，例如，玩家登陆时连接到Account proxy，然后从菜单选择另一个proxy来作为游戏中的角色。把控制权转角给另一个proxy使用方法giveClientTo，例子如下：  
`clientControlledProxy.giveClientTo( nonClientControlledProxy )`  
注意，两个proxy必须在相同的BaseApp。   
无论何时客户端在两个proxy之间转换，客户端依附的cell entity会被销毁，客户端会接收到一个onEntitiesReset调用去清理它的当前的世界数据（knowledge of the world）。这会很快中断所有的游戏通信同时强制客户端刷新。如果只是cell entity被销毁，proxy的客户端数据还会被保持下来。
![BaseApp managing bases and proxies](../image/BaseApp managing bases and proxies.png)

### 8.2. Witnesses
当一个有依附客户端的proxy拥有一个对应的cell entity时，一个叫做witness的附加对象会被依附到cell entity。  
这个对象维护entity的AoI，且发送更新到proxy，proxy会把更新转发到客户端。  
更新包括大量的游戏相关信息。例如：  

* entity位置更新。  
* entity属性更新。
* 方法调用。
* 空间数据改变。
* entity进入和离开AoI的通知。

### 8.3. Entity控制
默认情况下，每个cell entity都被认为是服务端控制。当一个entity结合了一个witness对象，被视作由依附到相应proxy的客户端来控制。  
尽管如此，使用entity属性controlledBy，对一个entity的控制可以显示的分配和查询。这个属性可以设置为None，表示由服务端控制，或设置为Base.MailBox表明被依附到proxy的客户端控制。“控制”在上下文里的意思是控制和负责entity的位置和方向。客户端（和proxy）会被告知它们允许控制entity的集合改变，proxy可以通过它们的属性区读取这个集合。

### 8.4. 物理校正
当一个entity被客户端控制，设置topSpeed属性为大于0的值会允许物理检查。所有entity的运动将会被检查，在以下的方面：  

* 速度  
第一个检查是关于速度，保证它不超过topSpeed值。考虑到150ms的网络抖动速度下，会有一个很小的误差。小心处理避免这个延迟被利用——玩家会被允许在非常短的时间内快速移动，由这个抖动导致的，但不允许长时间如此。  
* 场景几何（**这个说明有待改进**）
第二个检查是场景几何，保证entity只能通过一个明确定义(well-defined)的入口离开当前chunk。  
尽管速度非常快，这个检查为级别设计产生一个后果。控制角色移动（mobility）的障碍必须在chunks的级别上被表现（而不是chunk内部）。例如，一个有一堵墙的chunk，会有一个门用来进入到另外一边，不会被这个物理检查系统保护。替代方案是，两个chunk应当用来处理这个情况——墙的每一边有一个，门作为两者之间的入口。  
* ~~定制的物理验证（未实现)~~
如果安装了定制的物理验证，它将会在速度和几何检查之间被调用。会使用以下参数调用定制的物理验证：  
	* entity的指针
	* vehicle的指针——如果entity不在vehicle上则为NULL。
	* entity想要移动到的位置
	* 上次物理检查后过去的时间  
如果entity被允许移动到新位置，定制物理验证应当返回true，否则返回false。  
要安装一个自定义物理验证，必须写一个CellApp扩展模块。扩展模块初始化时，全局函数指针g_customPhysicsValidator（在bigworld/src/server/cellapp/entity.hpp中声明）应该被设置只想自定义物理验证器函数。更多细节，请看相关的扩展章节。  

如果启用，物理检查也可以用于以下情形：  

* 一个客户端控制多个entity。  
* 在坐骑（vehicle）之间移动。
* 移动到另一个空间。  
当一个服务端脚本直接设置一个客户端控制的entity的位置或者朝向时，CellApp会把这个行为当作一个物理校正。一个新的位置和朝向会强制发送给客户端，同时不会更新未来的数据知道这个校正被周知了。在为响应服务端的一些行为或者事件而传送玩家时，这个特性非常有用。尤其是，Entity.teleport, Entity.boardVehicle和 Entity.alightVehicle方法也遵循这个机制。在bigworld1.6版，当网关不使用时，在空间之间移动一个entity，服务端传送是唯一的方式。虽然在大多数情况下，运动控制器也会导致强制下行位置，这些用于客户端控制entity上既不推荐也不支持，因为它们不断的通过一次性调整设置位置。

