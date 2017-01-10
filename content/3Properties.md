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
BLOB | *N + K* | 二进制数据。类似string，但可以包含*NULL*字符。当写到XML时以base-64编码存储。例如在XML文件中，N为BLOB中的字节数，k<=4. | |
FLOAT | 4 | IEEE 32-bit floating-point number. | | 
INT8 | 1 | Signed 8-bit integer. | -128 | 127
INT16 | 2 | Signed 16-bit integer. | -32,768 | 32,767
INT32 | 4 | Signed 32-bit integer. | -2,147,483,648 | 2,147,483,647
INT64 | 8 | Signed 64-bit integer. | -9,223,372,036,854,775,808 | 9,223,372,036,854,775,807
MAILBOX | 12 | KBE mailbox，把entity传给一个MAILBOX参数会自动转换为MAILBOX（可设置不自动转换），详细请看MAILBOX的说明 | |
PYTHON  | 为pickled生成的字符串大小 | 使用python的pickler模块把人恶化python类型打包成一个字符串并传输。这不应用于客户端和服务端之间的传输，因为不安全和很低效。而推荐使用用户自定义数据类型。具体请看相关的章节。 | |




3.1.2. Composite types
The following sections describe the composite types available in BigWorld.
3.1.2.1. ARRAY and TUPLE types
BigWorld also has ARRAY and TUPLE types, which can create an array of values of any of
the BigWorld primitive types.
Properties of ARRAY type have a byte size calculated by the formula below:
N * t + k
The components of the formula are described below:
􀂃 N......Number of elements in the array.
􀂃 t........ Size of the type contained in the array.
􀂃 k.......Constant.
The BigWorld TUPLE type is represented in script by the Python tuple type, while the
BigWorld ARRAY type is represented in script by Python list type.
Tuples are specified as follows:
<Type> TUPLE <of> [TYPE_NAME|TYPE_ALIAS] </of> [<size> n </size>] </Type>
<res>/entities/defs/<entity>.def—TUPLE declaration syntax
Arrays are specified as follows:
<Type> ARRAY <of> [TYPE_NAME|TYPE_ALIAS] </of> [<size> n </size>] </Type>
<res>/entities/defs/<entity>.def—ARRAY declaration syntax
In case the size of ARRAY or TUPLE is specified, then it must have the declared n elements.
Adding or deleting elements to fixed‐sized ARRAY or TUPLE is not allowed. If the default
value is not specified, then a fixed‐sized ARRAY or TUPLE will contain n default values of
the element type.
Arrays not only can contain aliased data types, but also can be aliased themselves. For more
details, see Alias of data types on page 24.