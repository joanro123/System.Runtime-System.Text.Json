# Migrating from System.Runtime.Serialization to System.Text.Json

.NET Core 3.0 introduced namespace [System.Text.Json](https://devblogs.microsoft.com/dotnet/try-the-new-system-text-json-apis/) which is meant to replace [System.Runtime.Serialization](https://docs.microsoft.com/en-us/dotnet/api/system.runtime.serialization?view=net-6.0) and [Newtonsoft.Json](https://www.newtonsoft.com/json) as the primary tool used when serializing and deserializing json.

If you have an older project you still might use `System.Runtime.Serialization` to serialize and deserialize json. In that case it is time to make the change to `System.Text.Json`. Here you will find a couple of things that we came by when we made the migration and which can be useful to know.

## Controlling serialization with attributes

Let's use this class that uses `System.Runtime.Serialization` as an example:

```c#
// Using System.Runtime.Serialization
[Datacontract]
public class Request
{
    [DataMember(Name = "userName")]
    public string UserName { get; set; }

    [DataMember(Name = "orderReference", EmitDefaultValue = false)]
    public string? OrderReference { get; set; }
}
```

When using `System.Text.Json` you can no longer use the `Datacontract` and `Datamember` attributes. If you need to specify property name when serializing and deserializing you should instead use  `JsonPropertyName` attribute.

Sometimes you want a value to be excluded from being serialized. `EmitDefaultValue = false` means that if a data member has its default value, then it will not be serialized. `JsonIgnore(Condition = JsonIgnoreCondition.WhenWritingDefault)` does the same in `System.Text.Json`. In this specific example it means that when `OrderReference` is `null` the property will not be serialized in to json. 

The class above will now look like:

```c#
// Using System.Text.Json
public class Request
{
    [JsonPropertyName("userName")]
    public string UserName { get; set; }

    [JsonPropertyName("orderReference"), JsonIgnore(Condition = JsonIgnoreCondition.WhenWritingDefault)]
    public string? OrderReference { get; set; }
}
```

## Serializing objects

When serializing with `System.Runtime.Serialization` you have to first create an instance of `DataContractJsonSerializer` with the type which you want to use and after that use the `WriteObject` method to write the data to a memory stream. This data can then be encoded to a string. 

```c# 
// Using System.Runtime.Serialization
public static string SerializeWithSystemRuntime<T>(T value)
{
    using var stream = new MemoryStream();
    var serializer = new DataContractJsonSerializer(typeof(T));
    serializer.WriteObject(stream, value);
    var json = stream.ToArray();
    return Encoding.UTF8.GetString(json, 0, json.Length);
}
```

`System.Text.Json` has both `JsonSerializer.Serialize` and `JsonSerializer.SerializeAsync` methods which can be used to serialize. The latter one works in a similar way as `DataContractJsonSerializer` and writes the data to a stream. 

`JsonSerializer.Serialize` is a bit simpler as it serializes the object directly to a string.

```c#
// Using System.Text.Json
public static string SerializeWithSystemText<T>(T value)
{
    return JsonSerializer.Serialize(value, value.GetType());
}
```

## Specifying constructor

When using `System.Text.Json` and when the class has a public parameterized constructor, that constructor will be used to deserialize the class. If the class has multiple constructors then you need to use the `JsonConstructor` attribute to specify which constructor that will be used. 

```c#
// Using System.Text.Json
public Request(string userName)
{
    UserName = userName;
}

[JsonConstructor]
public Request(string userName, string orderReference)
{
    UserName = userName;
    OrderNumber = orderNumber;
}
```

The advantage of `System.Text.Json` is its performance, it is much faster and it is also being improved continuously so making the change should not be in vain!
