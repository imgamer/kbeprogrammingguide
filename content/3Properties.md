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


3.1.2.2. 固定字典(FIXED_DICT)数据类型
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
（2017-1-13）当前KBE不允许在`<entity>.def`的方法参数中直接声明FIXED_DICT数据类型（BigWorld可以），ARRAY、TUPLE、FIXED_DICT在声明属性时无法指定默认值。

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

```
注意（BigWorld 1.8.7）：  
在BaseApp上，当使用Python字典来设置一个FIXED_DICT实例时（直接赋值给FIXED_DICT属性）
，实际效果是Python字典替换了FIXED_DICT对象。而且，整个Python字典被引用了，不仅仅
是它里面的value。实际效果是假定FIXED_DICT属性比所声明的拥有更多的key。这个特别的
BaseApp行为在未来的版本将会被修改和其它系统行为一样。
```
FIXED_DICT的value值修改会被高效的分发传输，无论在何处对整个属性的修改将会触发分发传输，例如，目标是ghost和client(有ownClients标签)。  

The default value of a FIXED_DICT data type can be specified at the entity property level.
For example:
<root>
<Properties> FIXED_DICT
<someProperty>
<Type> TradeLog </Type> <!-- From last example -->
<Default>
<dbIDA> 0 </dbIDA>
<itemsTypesA>
<item> 101 </item>
<item> 102 </item>
</itemsTypesA>
<goldPiecesA> 100 </goldPiecesA>
</Default>
</someProperty>
</Properties>
</root>
Example of specifying default value of a FIXED_DICT data type in an entity definition file

If the <Default> section is not specified, then the default value of a FIXED_DICT data
type is described by following the table:
<AllowNone> FIXED_DICT default value
True Python None object.
False Python dictionary with keys as specified in the type definition.
Each keyed value will have a default value according to its type. For example, a
keyed value of INT type will have a default value of 0.
Default values for a FIXED_DICT without a <Default> section



3.1.3. Custom user types
There are two ways to incorporate user‐defined Python classes into BigWorld entities:
wrapping a FIXED_DICT data type, or implementing a USER_TYPE.
The FIXED_DICT data type supports being wrapped by a user‐defined Python type. When a
FIXED_DICT is wrapped, BigWorld will instantiate the user‐defined Python type in place
of a FIXED_DICT instance. This enables the user to customise the behaviour of a
FIXED_DICT data type.
The type system can also be arbitrarily extended with the USER_TYPE type. Unlike a
wrapped FIXED_DICT type, the structure of a USER_TYPE type is completely opaque to
BigWorld. As such, the implementation of a USER_TYPE type is more involved. The
implementation of the type operations is performed by a Python object (such as an instance
of a class) written by the user. The Python object serves as a factory and serialiser for
instances of that type, and it can choose to use whatever Python representation of that type
it sees fit—it can be as simple as an integer, or it can be an instance of a Python class.
For more details on custom user types, see Implementing custom property data types on
page 31.
3.1.4. Alias of data types
BigWorld also allows aliases of types to be created. Aliases are a concept similar to C++ʹs
typedefs, and are listed in the XML file <res>/entities/defs/alias.xml. The
format is described below:
<root>
... other alias definitions ...
<ALIAS_NAME> TYPE_TO_ALIAS [<Default> Value </Default>1] </ALIAS_NAME>
</root>
<res>/entities/defs/alias.xml—Data type alias declaration syntax

