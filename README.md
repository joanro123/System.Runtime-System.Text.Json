## From System.Runtime.Serialization to System.Text.Json

If you have an older project you still might use System.Runtime.Serialization to serialize and deserialize json. In that case it is time to make the change to System.Text.Json, here you will find a couple of tips that can be useful to know. 

Your class that uses System.Runtime.Serialization might look something like this:

```c#
[Datacontract]
public class Request
{

[DataMember(Name = "userName")]
public string UserName { get; set; }

[DataMember(Name = "orderNumber", EmitDefaultValue = false)]
public int OrderNumber { get; set; }

}
```

When using System.Text.Json you no longer need _Datacontract_ and _Datamember_ attributes. If you need to specify property name when serializing and deserializing you can instead use  _JsonPropertyName_ attribute. ``EmitDefaultValue = false`` means that if a data member has its default value, then it will not be serialized. ``JsonIgnore(Condition = JsonIgnoreCondition.WhenWritingDefault)`` does the same in System.Text.Json:

```c#
public class Request
{

[JsonPropertyName("userName")]
public string UserName { get; set; }

[JsonPropertyName("orderNumber"), JsonIgnore(Condition = JsonIgnoreCondition.WhenWritingDefault)]
public int OrderNumber { get; set; }

}
```
When serializing with System.Runtime.Serialization you have to first create an instance of _DataContractJsonSerializer_ with the type which you want to use and after that use the _WriteObject_ method to write the data to a memory stream. This data can then be encoded to a string. 

```c# 
public static string SerializeWithSystemRuntime<T>(T value)
{
using var stream = new MemoryStream();
var serializer = new DataContractJsonSerializer(typeof(T));
serializer.WriteObject(stream, value);
var json = stream.ToArray();
return Encoding.UTF8.GetString(json, 0, json.Length);
}
```

System.Text.Json has both _JsonSerializer.Serialize_ and _JsonSerializer.SerializeAsync_ methods which can be used to serialize. The latter one works in a similar way as _DataContractJsonSerializer_ and writes the data to a stream. 

_JsonSerializer.Serialize_ is a bit simpler as it serializes the object directly to a string. Options to serialization can be added with _JsonSerializerOptions_, in this example _JsonNamingPolicy.CamelCase_ converts a property's name on an object to camel-casing. 

```c#
public static string SerializeWithSystemText<T>(T value)
{
var options = new JsonSerializerOptions { PropertyNamingPolicy = JsonNamingPolicy.CamelCase };
return JsonSerializer.Serialize(value, value.GetType(), options);
}
```
