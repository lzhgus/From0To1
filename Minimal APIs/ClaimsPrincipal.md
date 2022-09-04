# IPrincipal Interface

-   Reference

## [Definition](https://docs.microsoft.com/en-us/dotnet/api/system.security.principal.iprincipal?view=net-6.0#definition)

Namespace:

[System.Security.Principal](https://docs.microsoft.com/en-us/dotnet/api/system.security.principal?view=net-6.0)

Assembly:

System.Runtime.dll

Defines the basic functionality of a principal object.

C#Copy

```
public interface IPrincipal
```


## Remarks

A principal object represents the security context of the user on whose behalf the code is running, including that user's identity ([IIdentity](https://docs.microsoft.com/en-us/dotnet/api/system.security.principal.iidentity?view=net-6.0)) and any roles to which they belong.

All principal objects are required to implement the [IPrincipal](https://docs.microsoft.com/en-us/dotnet/api/system.security.principal.iprincipal?view=net-6.0) interface. For more information about [IPrincipal](https://docs.microsoft.com/en-us/dotnet/api/system.security.principal.iprincipal?view=net-6.0) implementations, see the [ClaimsPrincipal](https://docs.microsoft.com/en-us/dotnet/api/system.security.claims.claimsprincipal?view=net-6.0) class.

---
