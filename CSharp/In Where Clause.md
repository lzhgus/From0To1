For generic type parameters, the `in` keyword specifies that the type parameter is contravariant. You can use the `in` keyword in generic interfaces and delegates.


[in (Generic Modifier) - C# Reference | Microsoft Docs](https://docs.microsoft.com/en-us/dotnet/csharp/language-reference/keywords/in-generic-modifier)

[where (generic type constraint) - C# Reference | Microsoft Docs](https://docs.microsoft.com/en-us/dotnet/csharp/language-reference/keywords/where-generic-type-constraint)

An interface that has a contravariant type parameter allows its methods to accept arguments of less derived types than those specified by the interface type parameter. For example, in the IComparer<T>(https://docs.microsoft.com/en-us/dotnet/api/system.collections.generic.icomparer-1) interface, type T is contravariant, you can assign an object of the __IComparer<Person>__ type to an object of the __IComparer<Employee>__ type without using any special conversion methods if `Employee` inherits `Person`.




---



For example, you can declare a generic class, `AGenericClass`, such that the type parameter `T` implements the [IComparable<T>](https://docs.microsoft.com/en-us/dotnet/api/system.icomparable-1) interface:

public class AGenericClass<T> where T : IComparable<T> { }