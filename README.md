# Reflection in C# - Hands-on Technical Assignment

## Overview
This assignment provides practical experience in understanding and implementing various aspects of Reflection in C#. It covers tasks such as inspecting assembly metadata, manipulating objects dynamically, invoking methods, creating types at runtime, building a plugin system, and more.

---

## Tasks

### **Task 1: Inspect Assembly Metadata**
Inspect the metadata of an assembly, including types, methods, properties, fields, and events.

#### Steps:
1. Load the assembly using `Assembly.LoadFile`.
2. Retrieve all types in the assembly using `GetTypes`.
3. For each type, retrieve methods, properties, fields, and events using `GetMethods`, `GetProperties`, `GetFields`, and `GetEvents`.

#### Code:
```csharp
using System;
using System.Reflection;

class AssemblyInspector
{
    public static void InspectAssembly(string assemblyPath)
    {
        Assembly assembly = Assembly.LoadFile(assemblyPath);
        Type[] types = assembly.GetTypes();
        foreach (Type type in types)
        {
            Console.WriteLine($"Type: {type.FullName}");

            Console.WriteLine("Methods:");
            foreach (MethodInfo method in type.GetMethods())
            {
                Console.WriteLine($"  {method.ReturnType} {method.Name}");
            }

            Console.WriteLine("Properties:");
            foreach (PropertyInfo property in type.GetProperties())
            {
                Console.WriteLine($"  {property.PropertyType} {property.Name}");
            }

            Console.WriteLine("Fields:");
            foreach (FieldInfo field in type.GetFields())
            {
                Console.WriteLine($"  {field.FieldType} {field.Name}");
            }

            Console.WriteLine("Events:");
            foreach (EventInfo eventInfo in type.GetEvents())
            {
                Console.WriteLine($"  {eventInfo.EventHandlerType} {eventInfo.Name}");
            }

            Console.WriteLine();
        }
    }
}

// Usage
AssemblyInspector.InspectAssembly(@"C:\path\to\your\assembly.dll");
```

---
# Task 2: Dynamic Object Inspector
Create a tool to inspect and manipulate object properties at runtime.

## Steps:
Use GetType to retrieve the object's type.

Use GetProperties to retrieve all properties.

Display the name and value of each property.

Implement a method to set a property's value using SetValue.

## code
```csharp
using System;
using System.Reflection;

class ObjectInspector
{
    public static void InspectObject(object obj)
    {
        Type type = obj.GetType();
        Console.WriteLine("Properties:");
        foreach (PropertyInfo property in type.GetProperties())
        {
            object value = property.GetValue(obj);
            Console.WriteLine($"  {property.Name}: {value}");
        }
    }

    public static void SetPropertyValue(object obj, string propertyName, object newValue)
    {
        Type type = obj.GetType();
        PropertyInfo property = type.GetProperty(propertyName);

        if (property != null && property.CanWrite)
        {
            property.SetValue(obj, newValue);
            Console.WriteLine($"Property {propertyName} set to {newValue}");
        }
        else
        {
            Console.WriteLine($"Property {propertyName} not found or is read-only.");
        }
    }
}

// Usage
var example = new { Name = "John", Age = 30 };
ObjectInspector.InspectObject(example);
ObjectInspector.SetPropertyValue(example, "Age", 31);
```

---

# Task 3: Dynamic Method Invoker
Invoke a method on an object dynamically using its name.

## Steps:
Use GetType to retrieve the object's type.

Use GetMethod to retrieve the MethodInfo object.

Use Invoke to call the method.

## code
```csharp
using System;
using System.Reflection;

class MethodInvoker
{
    public static void InvokeMethod(object obj, string methodName, params object[] parameters)
    {
        Type type = obj.GetType();
        MethodInfo method = type.GetMethod(methodName);

        if (method != null)
        {
            object result = method.Invoke(obj, parameters);
            Console.WriteLine($"Method {methodName} invoked. Result: {result}");
        }
        else
        {
            Console.WriteLine($"Method {methodName} not found.");
        }
    }
}

// Usage
var example = new { Name = "Alice" };
MethodInvoker.InvokeMethod(example, "ToString");
```
---
# Task 4: Dynamic Type Builder
Create a new type at runtime with properties and methods.

## Steps:
Use AssemblyBuilder and ModuleBuilder to create an assembly and module.

Use TypeBuilder to define a new type.

Use DefineProperty and DefineMethod to add properties and methods.

Use CreateType to finalize the type.

## Code 
```csharp
using System;
using System.Reflection;
using System.Reflection.Emit;

class DynamicTypeBuilder
{
    public static void CreateDynamicType()
    {
        AssemblyName assemblyName = new AssemblyName("DynamicAssembly");
        AssemblyBuilder assemblyBuilder = AssemblyBuilder.DefineDynamicAssembly(assemblyName, AssemblyBuilderAccess.Run);
        ModuleBuilder moduleBuilder = assemblyBuilder.DefineDynamicModule("DynamicModule");

        TypeBuilder typeBuilder = moduleBuilder.DefineType("DynamicType", TypeAttributes.Public);

        PropertyBuilder propertyBuilder = typeBuilder.DefineProperty("DynamicProperty", PropertyAttributes.None, typeof(string), null);

        MethodBuilder methodBuilder = typeBuilder.DefineMethod("GetGreeting", MethodAttributes.Public, typeof(string), null);
        ILGenerator il = methodBuilder.GetILGenerator();
        il.Emit(OpCodes.Ldstr, "Hello, Dynamic World!");
        il.Emit(OpCodes.Ret);

        Type dynamicType = typeBuilder.CreateType();
        object instance = Activator.CreateInstance(dynamicType);
        MethodInfo method = dynamicType.GetMethod("GetGreeting");
        string result = (string)method.Invoke(instance, null);
        Console.WriteLine(result); // Outputs: Hello, Dynamic World!
    }
}

// Usage
DynamicTypeBuilder.CreateDynamicType();
```
---

# Task 5: Plugin System
Create a plugin system that loads and invokes methods from external assemblies.

## Steps:
Define a common interface for plugins.

Create plugins that implement the interface.

Load plugins using Assembly.LoadFile.

Use GetType and GetInterface to find plugin types.

Create instances and call methods.

## code 
```csharp
using System;
using System.Reflection;

public interface IPlugin
{
    void Execute();
}

public class SamplePlugin : IPlugin
{
    public void Execute()
    {
        Console.WriteLine("SamplePlugin executed.");
    }
}

class PluginSystem
{
    public static void LoadAndExecutePlugin(string assemblyPath)
    {
        Assembly assembly = Assembly.LoadFile(assemblyPath);
        foreach (Type type in assembly.GetTypes())
        {
            if (typeof(IPlugin).IsAssignableFrom(type))
            {
                IPlugin plugin = (IPlugin)Activator.CreateInstance(type);
                plugin.Execute();
            }
        }
    }
}

// Usage
PluginSystem.LoadAndExecutePlugin(@"C:\path\to\plugin.dll");
```

---
# Task 6: Mocking Framework
Create a mock object for unit testing using Reflection.

## Steps:
Define an interface to mock.

Use TypeBuilder to create a dynamic type that implements the interface.

Use MethodBuilder to define methods that return default values.

Create an instance and use it in tests.

## code

```csharp
using System;
using System.Reflection;
using System.Reflection.Emit;

class MockingFramework
{
    public static object CreateMock(Type interfaceType)
    {
        AssemblyName assemblyName = new AssemblyName("MockAssembly");
        AssemblyBuilder assemblyBuilder = AssemblyBuilder.DefineDynamicAssembly(assemblyName, AssemblyBuilderAccess.Run);
        ModuleBuilder moduleBuilder = assemblyBuilder.DefineDynamicModule("MockModule");

        TypeBuilder typeBuilder = moduleBuilder.DefineType("MockType", TypeAttributes.Public, null, new[] { interfaceType });

        foreach (MethodInfo method in interfaceType.GetMethods())
        {
            MethodBuilder methodBuilder = typeBuilder.DefineMethod(method.Name, MethodAttributes.Public | MethodAttributes.Virtual, method.ReturnType, null);
            ILGenerator il = methodBuilder.GetILGenerator();
            if (method.ReturnType == typeof(int))
            {
                il.Emit(OpCodes.Ldc_I4_0); // Return default int value
            }
            else if (method.ReturnType == typeof(string))
            {
                il.Emit(OpCodes.Ldnull); // Return default string value
            }
            il.Emit(OpCodes.Ret);
        }

        Type mockType = typeBuilder.CreateType();
        return Activator.CreateInstance(mockType);
    }
}

// Usage
public interface ISampleInterface
{
    int GetValue();
    string GetName();
}

object mock = MockingFramework.CreateMock(typeof(ISampleInterface));
Console.WriteLine(((ISampleInterface)mock).GetValue()); // Outputs: 0
Console.WriteLine(((ISampleInterface)mock).GetName());  // Outputs: null
```
---
# Task 7: Serialization API
Create a serialization API using Reflection and optimize it using Reflection.Emit.

## Steps:
Implement a simple serialization API using Reflection.

Identify limitations (e.g., performance, nested objects).

Rewrite the API using Reflection.Emit for better performance.

## Code (Reflection-based Serialization):
```csharp
using System;
using System.Text;

class SimpleSerializer
{
    public static string Serialize(object obj)
    {
        StringBuilder sb = new StringBuilder();
        Type type = obj.GetType();

        foreach (PropertyInfo property in type.GetProperties())
        {
            object value = property.GetValue(obj);
            sb.AppendLine($"{property.Name}: {value}");
        }

        return sb.ToString();
    }
}

// Usage
var example = new { Name = "Alice", Age = 25 };
Console.WriteLine(SimpleSerializer.Serialize(example));
```
