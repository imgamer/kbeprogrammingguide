## 8. Proxies和玩家

### 8.1. Proxies
BaseApp上的KBEngine.Proxy继承于KBEngine.Base，目的是为玩家控制entity提供支持。从KBEngine.Proxy继承的entity，能够实现玩家角色，玩家账号和服务器任何其他玩家控制对象相关。  
无论何时一个客户端登录到服务器，一个从KBEngine.Proxy衍生的entity会从数据库被创建。KBE如何确定加载哪个Proxy的细节，请看`User Authentication And Proxy Selection`章节。以这个方式创建的Proxies会有一个叫做`password`的属性，这个属性的值会被设置为登录密码。  
一个KBEngine.Proxy实例既不需要cell entity或者客户端。和其他entity一样，一个proxy entity能够使用KBEngine.createBase来创建。详细请看BaseAPp的`Entity Instantiation and Destruction`章节。  

和其它Base entity一样，保存entity到数据库和从其中加载entity都是可以的。开始时，这些从新加载的proxy entity是没有客户端依附的。接着，一个已经存在的proxy可以把它的客户端转交给重载的proxy，重载的entity将负责处理这个客户端连接。  
这个可以实现功能，例如，玩家登陆时连接到Account proxy，然后从菜单选择另一个proxy来作为游戏中的角色。把控制权转角给另一个proxy使用方法giveClientTo，例子如下：  
`clientControlledProxy.giveClientTo( nonClientControlledProxy )`  
注意，两个proxy必须在相同的BaseApp。   
无论何时客户端在两个proxy之间转换，客户端衣服的cell entity会被销毁，客户端会接收到一个onEntitiesReset调用去清理它的当前的世界数据（knowledge of the world）。这会很快中断所有的游戏通信同时强制客户端刷新。如果只是cell entity被销毁，proxy的客户端数据还会被保持下来。
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
* 场景几何
第二个检查是场景几何，保证entity只能通过一个明确定义(well-defined)的入口离开当前chunk。

In spite of being very fast, this check does a consequence for level design. Barriers that control character mobility must be represented at the level of chunks. For example, a chunk with a wall across it, and a door giving access to the other side, is not protected by this physics checking system. Instead, two chunks should be used for this case – one on each side of the wall, with the door as a portal between them.
  Custom physics validator
If a custom physics validator is installed, then it will be called between the speed and the geometry check. The custom physics validator is called with the following parameters:
  Pointer to the entity.
  Pointer to the vehicle – NULL if the entity is not on a vehicle.   The position to which the entity wants to move.
  The time elapsed since the last physics check.
The custom physics validator should return true if the entity is allowed to move to the new position, or false if it is not allowed.

 
