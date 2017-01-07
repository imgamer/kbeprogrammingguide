 3. Properties
Properties describe what the state of an entity is. Like traditional object systems, a BigWorld property has a type and a name. Unlike traditional object systems, a property also has distribution properties that affect where and how frequently it is distributed around the system.
Properties are declared in the entityʹs definition file1, on a section named Properties. The table below describes the grammar for property definition:

```
 
<root> ...
<Properties>
<propertyName>
<!-- type of this property -->
TYPE_NAME </Type>
<!-- Method of distribution -->
<Flags> DISTRIBUTION_FLAGS </Flags> <!-- Default value (optional) -->
<Default> DEFAULT_VALUE </Default>
<!-- Is the property editable? (true/false) (optional) -->
<Editable> [true|false] </Editable>
<!-- Level of detail for this property (optional) -->
LOD </DetailLevel> <!-- Is the property persistent? -->
<Persistent> [true|false] </Persistent> </propertyName>
</Properties>
... </root>
 
 <Type>
For details, see Property types on page 19.
      
      
 For details, see Data distribution on page 27.
        
       
       
 For details, see Property types, Default values on page 26.
 
 
 
 
 
 
            
<DetailLevel>
For details, see LoD (Level of detail) on properties on page 38.
            
            
 
                        
For details, see The Database Layer on page 71.
  
 

```