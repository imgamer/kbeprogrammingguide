## 8. Proxies和玩家

### 8.1. Proxies
BaseApp上的KBEngine.Proxy继承于KBEngine.Base，目的是为玩家控制entity提供支持。从KBEngine.Proxy继承的entity，能够实现玩家角色，玩家账号和服务器任何其他玩家控制对象相关。  
无论何时一个客户端登录到服务器，一个从KBEngine.Proxy衍生的entity会从数据库被创建。KBE如何确定加载哪个Proxy的细节，请看`User Authentication And Proxy Selection`章节。以这个方式创建的Proxies会有一个叫做`password`的属性，这个属性的值会被设置为登录密码。  
一个KBEngine.Proxy实例既不需要cell entity或者客户端。和其他entity一样，一个proxy entity能够使用KBEngine.createBase来创建。详细请看BaseAPp的`Entity Instantiation and Destruction`章节。  

Like other base entities, saving and loading proxy entities from the database is possible.
Initially, these reloaded proxy entities will be created without an attached client. An existing
proxy can later hand its client over to the reloaded one, in which case the reloaded proxy
will be the one handling the client connection.
This allows you to have, for example, an Account entity that people can log in to, and a
Character entity that people can select from a menu in order to use in the game.
To pass the control of a client from one proxy to another, use the method giveClientTo,
as in the example below:
clientControlledProxy.giveClientTo( nonClientControlledProxy )
Note that to pass the control, both proxies must be on the same BaseApp.
Whenever a client moves between proxies, or the cell entity of the proxy that a client is
attached to is destroyed, the client receives a call on onEntitiesReset to clear out its
current knowledge of the world. This effectively interrupts all game communications, and
forces the client to refresh. If only the cell entity has been destroyed, then the clientʹs
knowledge of its proxy is retained.