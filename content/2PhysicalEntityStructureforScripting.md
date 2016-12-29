## 2. Entity脚本文件组织结构

Entity是组成游戏世界的对象。可以使用entity来创建玩家，NPC，战利品你，聊天室和其它交互元素。  
每个entity类型由python脚本脚本的集合实现，基于一个XML定义文件绑定在一起。脚本位于Scripts目录的资源树中（i.e,`<res>/Scripts`,`<res>`是定义在环境变量中的虚拟路径，一般命名为assets目录。）。  

### 2.1. KBEngine引擎默认资产库（额外加入这一小节）
KBEngine引擎默认资产库`<assets>`目录，如果用户没有设置环境变量指向，引擎默认会尝试读取引擎根目录assets作为默认的资产库。资产库的概念类似于Unity3D中的Assets，不过其中一些文件夹名称结构被固定了。  
要想引擎启动时读取到对应的项目资产库，必须在环境变量中指定。  
下表总结了`<res>`中entity的重要文件和目录:

### 2.2. entity.xml文件介绍
**kbe还能指定是否有客户端脚本**  
`<res>/scripts/entities.xml`文件被引擎用来确定可用的entity类型。  
这个文件的每个标签描述一种entity类型，同时要求在`<res>/scripts/defs`目录中必须有相应的定义文件，在`<res>/scripts/base`或者`<res>/scripts/cell`至少存在一个python脚本文件。可能在`<res>/entities/client`也存在脚本文件。  
定义在此文件中的entity类型的顺序对应entity的类型ID。最简单的形式是，entity文件有一个标签列表来加载每一个entity。  

定义一个名为NewEntityType的entity，如下在entities.xml中简单的增加一行即可：
```xml
<root>
    ...
    <NewEntityType/>
</root>
```

### 2.3. definition文件
`<res>/entities/defs/<entity>.def`文件确定你的脚本在KBE中如何关联。这个机制允许KBE系统把发送和接收消息的任务抽象成简单的调用entity的不同脚本方法。某种意义上，definition文件提供了entity的接口，而在python脚本中具体实现。下图展示了kbe entity的概念部分：  
![Conceptual parts of an entity](../image/Conceptual parts of an entity.png)  

每个entity类型有一个对应的定义文件，命名为entity名，后缀为`.def`。例如，一个Avatar entity类型会有一个叫做`Avatar.def`的文件。  
接下来是一个新entity的最小定义文件，用于说明本章节想要表达的内容：  
![Minimal entity definition file]()

By the end of this chapter, we should be able to replace all placeholders (denoted by italics)
in the example file above with actual code.

2.3. The script files
Big World Technology divides processing of entities in a game world into three different
execution contexts:
Entity
type Script file location Description
Cell <res>/entities/cell Takes care of the portions of an entity that affect the space
around it.
Processing takes place on the server cluster.
Base <res>/entities/base Takes care of the portions of an entity that do not affect the
space around it (as well as possibly acting as a proxy for a
player).
Processing takes place on the server cluster.
Client <res>/entities/client Takes care of the portions of an entity that require heavy
awareness of the surrounding environment.
Entity types
It is possible for some entity instances to not have one of these three parts. Furthermore,
some entity types may not support ever having one of these parts. For each entity type,
there is a script file for each of CellApp, BaseApp, and Client, if that type supports that
execution context.
These script files are named after the entity type, followed by the extension .py. This file
must contain a class with the name of the entity type.
For example, if you have an entity type Seat that can have cell, base and client execution
contexts, there would be three script files, each with the implementation of the class:
􀂃 <res>/entities/cell/Seat.py
􀂃 <res>/entities/base/Seat.py
􀂃 <res>/entities/client/Seat.py
The entityʹs base class defined in the script file is determined by the execution context that
the file represents, as described below:

Script file
execution context
Entity's base class
Cell BigWorld.Entity
Base BigWorld.Base or BigWorld.Proxy
Client BigWorld.Entity
Entity's base class per execution context
For more details about the difference between the Base and Proxy classes, see Proxies and
Players on page 85.