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



3.1.2.2. FIXED_DICT data type
The FIXED_DICT data type allows you to define dictionary‐like attributes with a fixed set
of string keys. The keys and the types of the keyed values are predefined.
The declaration of a FIXED_DICT is illustrated below:
<Type> FIXED_DICT
?<Parent> ParentFixedDictTypeDeclaration </Parent>
<Properties>
+<field>
<Type> FieldTypeDeclaration </Type>
</field>
</Properties>
?<AllowNone> true|false </AllowNone>
</Type>
FIXED_DICT data type declaration
This data type may be declared anywhere a type declaration may appear, e.g., in <res>/
entitites/defs/alias.xml1, in <res>/entitites/defs/<entity>.def, as
method call arguments, etc.
The code excerpt below shows the declaration of a FIXED_DICT attribute:
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
fantasydemo/res/entities/defs/alias.xml
Instances of FIXED_DICT can be accessed and modified like a Python dictionary, with the
following exceptions:
􀂃 Keys cannot be added or deleted
􀂃 The type of the value must match the declaration.

