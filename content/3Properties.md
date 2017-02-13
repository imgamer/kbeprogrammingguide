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
每个属性有一个分布类型定义，以便确定在KBE中哪个执行上下文(cell, base或者client)负责更新，并把值传输到何处。  
在文件`<res>/entities/defs/<entity>.def`的`<Properties>`子块中的`<Flags>`设置数据分布。  

entity最多有256个exposed属性（属性同时存在于client和server），而不超过61个时是最好的。  

附BigWorld定义在源码`data_description.hpp`中的数据分布字节标记描述如下：

Flag | Required flags | Excluded flags | Master value on | Description
- | - | - | - |-
DATA_BASE | N/A | DATA_GHOSTED | Base | 数据在Base上，在Cell上不可用
DATA_GHOSTED | N/A | DATA_BASE | Cell | 数据在Cell上并能够镜像（ghost）给其它Cell。<br>这意味着从其它entity上获取这个属性值是安全的。因为bw保证了跨Cell边界的安全使用。
DATA_OTHER_CLIENT | DATA_GHOSTED | N/A | Cell | 数据在Cell上，其它entity的AoI中拥有这个entity时，数据也会更新给它们的客户端。其它entity的客户端读取这个属性是安全的，除了这个数据所在Cell客户端上的player entity。这个标记经常和DATA_OWN_CLIENT组合使用来创建一个在所有客户端上分布的属性。
DATA_OWN_CLIENT | N/A | N/A | Base(如果设置了DATA_BASE)，否则Cell | 数据传输给这个entity对应的客户端。仅对player entity有效。

下面列出了上表字节标志的有效组合：

* ALL_CLIENTS  
属性对所有的cell entity及其client都是可见的。相当于同时设置了OWN_CLIENT和OTHER_CLIENT标记。例如玩家的名字属性，玩家或者一个creature的血条等。
* BASE  
属性只在Base上可见。例如，聊天房间列表，玩家仓库中的物品。  
* BASE_AND_CLIENT  
属性在Base和其对应的客户端可见。相当同时设置了BASE和OWN_CLIENT属性。注意：这个类型的属性只会在客户端entity创建时同步。当属性改变时，client和base都不会自动更新。需要定义一个方法来传输新值，只有一个玩家需要接收它，因此比较简单。  
* CELL_PRIVATE  
属性只在Cell上对其本身的entity可见。例如，NPC AI算法的思考数据；涉及游戏的玩家属性，但让其它玩家看见是危险的，例如战斗后的恢复时间。  
* CELL_PUBLIC  
属性在Cell上对自己和其它entity可见。例如，一个玩家的魔法级别（能够被敌人可见但不能被玩家客户端可见）；其它敌对NPC的呼叫范围。  
* CELL_PUBLIC_AND_OWN  
属性对Cell其它entity可见，同时对本身Cell和客户端也可见。与OWN_CLIENT不同，这个属性也可以被镜像，因此对Cell的其它entity可见。  
* OTHER_CLIENTS
属性对非自身entity的玩家客户端和Cell上的其它entity可见。例如，世界中动态物品的状态（一个门或者一个按钮）；粒子系统的效果类型；玩家当前是否坐在椅子上。   
* OWN_CLIENT  
属性仅对本entity的客户端和Cell可见。例如，玩家的角色类型；玩家的经验点数值。  

注意：对于拥有ALL_CLIENTS,OTHER_CLIENTS,OWN_CLIENT等分布标记的属性，会隐性的触发一个客户端方法调用`set_<property_name>`，请参看后面的相关章节。   

当为一个属性选择了分布标记，考虑以下几点：  

* 那些方法需要这些属性？  
如果相关的执行环境(cell, base, or client)有一个维护这个属性的方法，你必须在相关上下文让这个属性可用。  
* 这个属性是否被其它entity访问？  
这应该需要定义方法去访问属性值。如果是这样的情况，需要让这个属性能够被镜像(ghosted)。  
需要注意，镜像entity的属性会有一点延迟，例如，在某个时刻，不能精确的描述一个entity的状态；同时，这个属性对于其它entity来说是只读的；只有拥有这个属性的entity能够改变它。
* 客户端是否需要直接访问这个属性？  
Client/server的带宽很宝贵，客户端能直接访问的属性要尽量小。  
有时，cell维护的一组属性只需要发送由其衍生的额外属性给客户端。例如，客户端部分可能并不需要知道让一个守卫愤怒的6个AI状态变量组合，而只需要知道衍生的值：守卫挥舞着一把斧头。  
* 玩家是否能获得这个属性来作弊？  
要注意是否需要把这个属性发给客户端。  
* 任何属性只能有一个主值。  
主值必须是在base或者cell上。如果同样的属性需要在base和cell上可用，通常需要通过方法把属性返送给另外一端。




[^1]: 等于python列表['Health potion', 'Bear skin', 'Wooden shield' ].
[^2]: 基于base6编码的字符串必须被定义。
[^3]: 值定义必须没有被引用。


