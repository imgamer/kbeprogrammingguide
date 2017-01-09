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

3.1. Property types
BigWorld needs to efficiently transmit data over a network between its various components.
For this purpose, BigWorld definition file describes the type of each property of an entity
(despite the fact that BigWorld is scripted using Python—an untyped language).
Because bandwidth conservation is important in implementing an MMOG, property types
should be selected such that they are the smallest type (in terms of number of bits) that can
represent the data.