## 3. 属性
属性描述了entity的状态。和传统的对象系统一样，一个KBE属性有类型和名字，不同的是，属性也具有属性，描述自身会分布在何处和在系统中分发的频率（存疑）。  
属性被声明在entity的def文件中，在`<Properties>`段。如下表格描述了property的定义语法：  

```
<root> 
	...
	<Properties>
		<propertyName> 
			<!-- 属性的类型 -->
			<Type> TYPE_NAME </Type> 
			
			<!-- 分布方式 -->
			<Flags> DISTRIBUTION_FLAGS </Flags> 

			<!-- 默认值 (可选) -->
			<Default> DEFAULT_VALUE </Default>

			<!-- 这个属性的LOD (可选) -->
			<DetailLevel> LOD </DetailLevel> 

			<!-- 是否持久化 -->
			<Persistent> [true|false] </Persistent> 
		</propertyName>
	</Properties>
	... 
</root>
```

### 3.1. 属性类型
KBE需要通过网络在各个不同组件间高效的传送数据。为了实现这个目标，KBE定义文件描述了entity每个属性的类型（尽管KBE使用的是弱类型脚本语言python）。  
在MMOG游戏的实现中，节约带宽非常重要，属性类型应当选择为能描述数据的最小类型。

#### 3.1.1. 原始(Primitive)类型
下面的表格总结了KBE属性的可用原始类型：  

Type | Size(bytes) | Description | 最小值 | 最大值
- | - | - | - | -
BLOB | *N+K* | 二进制数据。类似string，但可以包含*NULL*字符。当写到XML时以base-64编码存储。例如在XML文件中，N为BLOB中的字节数，k<=4. | |
FLOAT | 4 | IEEE 32-bit floating-point number. | | 
INT8 | 1 | Signed 8-bit integer. | -128 | 127
INT16 | 2 | Signed 16-bit integer. | -32,768 | 32,767
INT32 | 4 | Signed 32-bit integer. | -2,147,483,648 | 2,147,483,647
INT64 | 8 | Signed 64-bit integer. | -9,223,372,036,854,775,808 | 9,223,372,036,854,775,807
MAILBOX | 12 | KBE mailbox，把entity传给一个MAILBOX参数会自动转换为MAILBOX（可设置不自动转换），详细请看MAILBOX的说明 | |
PYTHON  | 为pickled生成的字符串大小 | 使用python的pickler模块把任何python类型打包成一个字符串并传输。不要用于客户端和服务端之间的传输，因为不安全和很低效。而推荐使用用户自定义数据类型。具体请看相关的章节。 | |
STRING | *N+K* | 字符串。*N*是string中的字符数，*K<=4* | |
UINT8 | 1 | Unsigned 8-bit integer. | 0 | 255
UINT16 | 2 | Unsigned 16-bit integer. | 0 | 65,535
UINT32 | 4 | Unsigned 32-bit integer. | 0 | 4,294,967,295
UINT64 | 8 | Unsigned 64-bit integer. | 0 | 18,446,744,073,709,551,615
VECTOR2 | 8 | float类型的2维向量。在python中表现为2个数字的元组（或者Math.Vector2 ）.
VECTOR3 | 12 | float类型的3维向量。在python中表现为3个数字的元组（或者Math.Vector3 ）.
VECTOR4 | 16 | float类型的4维向量。在python中表现为3个数字的元组（或者Math.Vector4 ）.

*注意：UINT32类型使用python的long类型来替代int类型，因此可能会比INT32类型效率低。*

#### 3.1.2. 复合类型( Composite types )
接下来的消解描述了KBE中可用的复合类型。  

##### 3.1.2.1. ARRAY and TUPLE types
KBE也有队列(ARRAY)和元组(TUPLE)类型，能够创建由KBE原始类型值组成的队列(ARRAY)。  
ARRAY属性类型可由以下公式计算字节长度：  
N * t + K

公式各部分解释如下：  

* N........array中的元素数量
* t.........array中的类型大小
* k.........常数

KBE TUPLE类型在python脚本中表现为tuple类型，而ARRAY类型对应python脚本中的list类型。  
Tuple的定义方式如下：  
`<Type> TUPLE <of> [TYPE_NAME|TYPE_ALIAS] </of> [<size> n </size>] </Type>`  
Array的定义方式如下:
`<Type> ARRAY <of> [TYPE_NAME|TYPE_ALIAS] </of> [<size> n </size>] </Type>`  

一旦定义了ARRAY和TUPLE的长度，属性就必须声明n个元素。从固定长度ARRAY或TUPLE增加或删除元素都是不被允许的。如果初始值没有被指定，固定长度ARRAY或TUPLE则会包含n个所定义元素类型的默认值。  
Array不仅能够包含别名(alias)数据类型，自身也能够被定义别名。更多细节请看`数据类型的别名`章节。


##### 3.1.2.2. 固定字典(FIXED_DICT)数据类型
固定字典数据类型允许你使用字符串做为key的固定集合定义类字典属性。key和key的类型是提前定义好的。  
固定字典的声明如下所示：  

```
<Type> FIXED_DICT
	<Parent> ParentFixedDictTypeDeclaration </Parent>
	<Properties>
		<field>
		<Type> FieldTypeDeclaration </Type>
		</field>
	</Properties>
	<AllowNone> true|false </AllowNone>
</Type>
```
说明如下：
`<Parent>`是可选的，可以继承其他FIXED_DICT；`<AllowNone>`是可选的，默认值为false，如果设为true，则整个FIXED_DICT可以为None；`</field>`可以有多个。FIXED_DICT会在`<res>/entities/entity_defs/alias.xml`中被声明。  
**（2017-1-13）当前KBE不允许在`<entity>.def`的方法参数中直接声明FIXED_DICT数据类型（BigWorld可以），ARRAY、TUPLE、FIXED_DICT在声明属性时无法指定默认值。**  

以下的代码片段展示了FIXED_DICT的属性声明：
```
<root>
	<TradeLog> FIXED_DICT
	<Properties>
		<dbIDA>
			<Type> INT64 </Type>
		</dbIDA>
		<itemsTypesA>
			<Type> ARRAY <of> ITEM </of> </Type>
		</itemsTypesA>
		<goldPiecesA>
			<Type> GOLDPIECES </Type>
		</goldPiecesA>
	</Properties>
	</TradeLog>
</root>
```
FIXED_DICT实例能够像一个Python字典那样被访问和修改，限制如下：  

* 无法增加或删除key
* value的类型必须和声明匹配。  

例如：  
```
if (entity.TradeLog[ "dbIDA" ] == 0):
	entity.TradeLog[ "dbIDA" ] = 100
```

当使用一个Python字典来设置一个FIXED_DICT实例，Python字典的值会被FIXED_DICT实例引用。  

>**注意（BigWorld 1.8.7）**：  
>在BaseApp上，当使用Python字典来设置一个FIXED_DICT实例时（直接赋值给FIXED_DICT属性），实际效果是Python字典替换了FIXED_DICT对象。而且，整个Python字典被引用了，不仅仅是它里面的value。实际效果是假定FIXED_DICT属性比所声明的拥有更多的key。这个特别的BaseApp行为**在未来的版本将会被修改和其它组件系统行为一样**。


FIXED_DICT的value值修改会被高效的分发传输，无论在何处对整个属性的修改将会触发分发传输，例如，目标是ghost和client(有ownClients标签)。  

（当前未实现）如果没有定义`<Default>`块，FIXED_DICT数据类型的默认值如下表所示：  

<AllowNone> | FIXED_DICT default value
- | -
True | Python None object.
False | 已定义类型的python字典，每个key的value对应python类型的默认值。例如，INT类型的value默认值会是0.FIXED_DICT默认值没有<Default>块。


#### 3.1.3. 用户自定义类型
有2种方式把用户自定义的python类纳入(incorporate ... into)KBE实体:包装一个FIXED_DICT数据类型，或者实现一个USER_TYPE。  
FIXED_DICT数据类型支持被用户定义的python类型包装。当一个FIXED_DICT被打包，KBE将实例化用户自定义python类型来替代FIXED_DICT实例。这是允许用户定制FIXED_DICT数据类型的行为。  

类型系统能够使用USER_TYPE类型任意的扩展。和包装的FIXED_DICT类型不同，USER_TYPE类型的结构对KBE而言是完全黑盒的。因此，USER_TYPE类型的实现更复杂。操作类型的实现展示为一个用户实现的python对象（比如一个类的实例）。对类型实例而言，这个python对象行为就像是一个工厂和数据转换器，可以选择用什么python类型来表现更合适，可以简单的一个整型或者是一个python类的实例。  
更多用户自定义类型的细节，请看`实现用户属性数据类型`章节。  

#### 3.1.4. 数据类型别名
KBE允许创建类型的别名。别名的概念类似C++的`typedef`，定义在`<res>/entities/defs/alias.xml`。格式如下：  
```
<root>
	... other alias definitions ...
	<ALIAS_NAME> TYPE_TO_ALIAS [<Default> Value </Default>] </ALIAS_NAME>
</root>
默认值的说明请看接下来的章节。
```  

使用的别名定义如下表：  

别名(Alias) | 映射的类型 | 描述
- | - | -
ANGLE | FLOAT | 弧度值，表示角度
BOOL | INT8 | 布尔类型（0表示fasle，非0表示true）。映射INT8, 最小的KBE类型。
INFO | UINT16 | 关于任务(mission)的信息元素
MISSION_STATS | ARRAY <of>INFO </of> | 任务信息数据元素的数组。这个是alias数组，同时它的元素类型也是alias类型。
OBJECT_ID | INT32 | 另一个entity的句柄。命名明确的反映了属性包含一个entity的句柄。
STATS_MATRIX | ARRAY <of> MISSION_STATS </of> | 任务信息数据元素的二维数组。注意这是一个别名的数组，它的元素是一个其它别名的数组。嵌套别名。

由以上的别名说明，定义别名的语法如以下文件(`<res>/entities/entity_defs/alias.xml`)：
```
<root>
	<!-- Aliased data types -->
	<OBJECT_ID> INT32 </OBJECT_ID>
	<BOOL> INT8 </BOOL>
	<ANGLE> FLOAT </ANGLE>
	<INFO> UINT16 </INFO>
	
	<!-- Aliased arrays -->
	<MISSION_STATS> ARRAY <of> INFO </of> </MISSION_STATS>
	<STATS_MATRIX> ARRAY <of> MISSION_STATS </of> </STATS_MATRIX>
</root>
```

还能使用别名来自定义python数据类型，其在网络上有自己的流语义（streaming semantics）。这些类型在文件`<res>/entities/entity_defs/alias.xml`中被声明，语法如下：  
```
<root>
	<ALIAS_NAME>	USER_TYPE
		<implementedBy> UserDataType.instance </implementedBy>
	</ALIAS_NAME>
</root>
```
有关这个机制的更多细节，请看用户属性数据类型实现的相关章节。


### ~~3.2. 默认值（未实现）~~
当entity被创建，它的属性会被初始化为默认值。默认值能够在属性级别(对应的def文件)或者类型声明级别(`alias.xml`中)被覆写。  
每个类型的默认值和覆写语法如下：  
（略）

数据类型 | 默认值 | 例子
- | - | -
ARRAY | [] | `<Default>`<br>`<item> Health potion </item>`<br>`<item> Bear skin </item>`<br>`<item> Wooden shield </item>`<br>`</Default>`[^1]
BLOB | '' | `<Default> SGVsbG8gV29ybGQhB </Default>`[^2]
FIXED_DICT | 见前面的章节说明
INT8 | 0 | `<Default> 99 </Default>`
INT16 | 同上
INT32 | 同上
INT64 | 同上
MAILBOX | None | 默认值不可覆写
PYTHON | None | `<Default>`<br>`{ "Strength": 90, "Agility": 77 }`<br>`</Default>`
STRING | '' | `<Default> Hello World! </Default>`[^3]
TUPLE | () | See ARRAY data type
USER_TYPE | 用户自定义的defaultValue()返回值 | 类似FIXED_DICT，可嵌套FIXED_DICT
VECTOR2 | 长度为2，值为0.0的PyVector | `<Default> 3.142 2.71 </Default>`
VECTOR3 | 长度为3，值为0.0的PyVector | `<Default> 3.142 2.71 1.4 </Default>`
VECTOR4 | 长度为4，值为0.0的PyVector | `<Default> 3.142 2.71 1.4 3.8 </Default>`

### 3.3. 数据分布
属性描述了entity的状态。有些状态只会和cell相关，有些和base相关，而有些只和客户端有关。除了以上，有些状态会和多于一个以上的部分相关。
Properties represent the state of an entity. Some states are only relevant to the cell, others only to the base, and yet others only to the client. Some states, however, are relevant to more than one of these.
Each property then has a distribution type that specifies to BigWorld which execution context (cell, base, or client) is responsible for updating the property, and where to propagate its value within the system.
Data distribution is set up by specifying the sub‐section <Flags> of the section <Properties> of the file <res>/entities/defs/<entity>.def.
Entities can have up to 256 exposed properties (i.e., properties that exist both on client and server), with the efficient number being 61.
The bit flags available are defined in src/lib/entitydef/data_description.hpp, and are described in the table below:
Properties
Flag
RF
EF
Master value on
 
Description
 DATA_BASE
 
 
Base
 
Data will be updated on the base, and will not be available on the cell.
 DATA_GHOSTED
 
 
Cell
 
Data will be updated on the cell, and will be ghosted on other cells.
This means that it is safe to read the value of this property from another entity, because BigWorld safely makes it available even across cell boundaries.
    
 DATA_OTHER_CLIENT
 
 
Cell
 
Data will be updated on the cell, and made available to clients who have this entity in their AoI.
This makes the property safe to read from the client for any entity, except for that client's player avatar entity. This flag is often combined with DATA_OWN_CLIENT to create a property that is distributed to all clients.
        
DATA_OWN_CLIENT
 
 
 
Base, if DATA_BASE is set. Otherwise, on cell.
 
Data is propagated to client owning this entity. This only makes sense with player entities.
 
RF=Required flags, EF = Excluded flags
Data distribution bit flags
  27 of 177
N/A DATA_GHOSTED N/A N/A
N/A N/A
DATA_BASE DATA_GHOSTED
 Server Programming Guide
The table below list the valid combinations of the above bit flags:
        Available to:
       Enumeration
               Description
  ALL_CLIENTSA
    
   
  
    
   
   
   Property is available to all entities on cell and client. Corresponds to setting both OWN_CLIENT and OTHER_CLIENTS flags.
Examples include:
  The name of a player.
  The health status of a player or a creature.
       BASE
    
   
  
    
   
   
   Property is only available on the base. Examples include:
  List of members of a chat room.   Items in a character's inventory.
     BASE_AND_CLIENT
    
   
  
    
   
   
   Property is available on the base and on the owning client. Corresponds to setting both OWN_CLIENT and BASE flags.
NOTE: Properties of this type are only synchronised when the client entity is created. Neither the client nor the base is automatically updated when property changes. Methods must be used to propagate new value, which is simple, since only one player needs to receive it.
          CELL_PRIVATE
    
   
  
    
   
   
   Property is only available to its entity, and only on cell. Examples include:
  Properties of an NPCs 'thoughts' in AI algorithms.   Player properties relevant to game play, but
dangerous to allow players to see (e.g., healing time after battle).
       CELL_PUBLIC
    
   
  
    
   
   
   Property is available only on the cell, and is available to other entities.
Examples include:
  The mana level of a player (which can be seen only by enemies, not by other players).
  The call sign for grouping from enemy NPC.
       CELL_PUBLIC_AND_OWN
    
   
  
    
   
   
   Property is available to other entities on the cell, and to this one on both the cell and the client.
Unlike OWN_CLIENT, this data is also ghosted, and therefore available to other entities on the cell.
     DATA_EDITOR_ONLY
    
   
  
    
   
   
   This value may be useful when using BigWorld.fetchEntitiesFromChunks1 from a BaseApp. It could be used to decide programmatically whether a particular entity should be loaded.
For example, you may associate a level of difficulty with each entity, so entity will only be loaded if the mission's level of difficulty is high enough.
           A When properties with this distribution flag are updated by server, an implicit method is called on client. For details, see Implicit set_<property_name> methods on page 46.
         Data distribution constants (continues on next page...)
1 For details on this function, see Python Base API documentation, entry Modules BigWorld
28 of 177 Copyright 1999-2008 BigWorld Pty. Ltd. All rights reserved. Proprietary commercial in confidence.
  Other cells Cell
Base
Own client Other clients WorldEditor
 Data distribution constants (...continued from previous page)
Data distribution constants
The table below lists the deprecated data distribution constants:
Properties
Available to:
 
Enumeration
 
 
 
 
 
 
Description
OTHER_CLIENTSA
 
 
 
 
  
 
 
 
 
 
 
 
Property is available from client to entities that are not this player's avatar. Also available on cell to other entities.
Examples include:
  The state of dynamic world items (e.g., doors, loot containers, and buttons).
  The type of a particle system effect.
  The player who is currently sitting on a seat.
       
OWN_CLIENTA
  
  
  
  
  
  
 Property is only available to this entity, on both the cell and the client.
Examples include:
  The character class of a player.
  Number of experience points for a player.
    
  A When properties with this distribution flag are updated by server, an implicit method is called on client. For details, see Implicit set_<property_name> methods on page 46.
 
Deprecated enumeration
 
Equivalent to
 ALL_CLIENT
 
ALL_CLIENTS
 CELL
 
CELL_PUBLIC
 CELL_AND_OWN
 
CELL_PUBLIC_AND_OWN
 GHOSTED
 
CELL_PUBLIC
 GHOSTED_AND_OWN
 
CELL_PUBLIC_AND_OWN
 OTHER_CLIENT
 
OTHER_CLIENTS
 PRIVATE
 CELL_PRIVATE
Deprecated distribution constants
When choosing a distribution flag for a property, consider the points described below:
  Which methods need the property?
You have to make the property available on an execution context (cell, base, or client) if
that context has a method that manipulates the property.
  Does this property need to be accessed by other entities?
This could include methods being called to access its value. If this is the case, we need to
make the property ghosted.
When doing this, remember that the ghosted entitiesʹ properties may be a little ʹlaggedʹ, i.e., they may not represent the exact state of an entity at a given time. Also, remember that other entities can only read the property; only the entity that owns the property may change it.

[^1]: 等于python列表['Health potion', 'Bear skin', 'Wooden shield' ].
[^2]: 基于base6编码的字符串必须被定义。
[^3]: 值定义必须没有被引用。


