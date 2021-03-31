# Schema Compatibility

The SymetryML Kafka Connector assigns a SymetryML attribute type to each datum in every record that it receives. The type is assigned according to the Kafka schema data type of the datum's field.

Note that we are concerned only with the value schema of each topic. We assume that the key schema data type is a simple string.  

The SymetryML Kafka Connector supports the following schema data types:
* Schema.Type.BOOLEAN  
** Maps to SymetryML attribute type BINARY  
* Schema.Type.FLOAT64  
** Maps to SymetryML attribute type CONTINOUS  
* Schema.Type.FLOAT32  
** Maps to SymetryML attribute type CONTINOUS  
* Schema.Type.INT16  
** Maps to SymetryML attribute type CONTINOUS 
* Schema.Type.INT32  
** Maps to SymetryML attribute type CONTINOUS  
* Schema.Type.INT64  
** Maps to SymetryML attribute type CONTINOUS  
* Schema.Type.STRING  
** Maps to SymetryML attribute type STRING  

When the connector is required to map another Schema type, it will throw a ConnectException.

##### User SymetryML Attribute Types

The user can override the above mapping of Kafka Schema data types to SymetryML attribute types by setting the `sml.attribute.types` configuration property.

Note that the connector will revert to guessing the SymetryML attribute types through the above mapping if the number of types listed in `sml.attribute.types` is different than the number of fields in the record schema.

##### Schema Evolution and Compatibility  
The SymetryML Kafka Connector assumes that the compatibility type of the schema is BACKWARD, which is the default for Confluent Schema Registry. This allows for "Delete fields" and "Add optional fields". It checks against the last version.
