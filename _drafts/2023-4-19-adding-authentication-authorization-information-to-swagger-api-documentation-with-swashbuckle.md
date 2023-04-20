---
layout: single
title:  "Adding Authentication/Authorization Information to Swagger API Documentation with Swashbuckle"
date:   2023-4-19 18:41:00 -0600
category: .NET
tags: Swashbuckle Swagger IOperationFilter Authorize Policy
header:
  image: /assets/images/headers/20220809_204319-1280x320.jpg
---

I've always wished swagger documentation included authentication and more importantly authorization information for each API call. Fortunately, Swashbuckle can be configured with various methods and filters to generate your very own customized Swagger documentation. Unfortunately, while the Swashbuckle documentation is good, it is often hard to find good examples.

This example started with a search of the Internet, with the thought that surely someone else had thought of this exact thing. But to my dismay, the only thing I found that came remotely close to what I was looking for I found in the GitHub repository [Swashbuckle.AspNetCore.Filters](https://github.com/mattfrear/Swashbuckle.AspNetCore.Filters) published by Matt Frear. 

In the code repository is an ```AppendAuthorizeToSummaryOperationFilter``` which appends authorization information to the API summary. While this was close to what I was looking for I was concerned about space, some of the APIs I need to document can have 5-10 policies.  I wanted a bit more verbose description of the roles and/or policies required with the understanding that roles are ORed, while policies are ANDed.  And I need it to handle a custom authentication attribute. 

With an example to guide me I’ve put together the following operation filter which appends detailed information about authentication/authorization to the end of the API description that appears when you click on an API call in the Swagger documentation. The description is the first thing you see following the summary and is populated with the text found inside the triple-slash comments ```<remarks></remarks>``` tags for the API call.

```csharp
using Microsoft.AspNetCore.Authorization;
using Microsoft.OpenApi.Models;
using Swashbuckle.AspNetCore.SwaggerGen;
using TGolla.AspNetCore.Mvc.Filters;

namespace TGolla.Swashbuckle.AspNetCore.SwaggerGen
{
    /// <summary>
    /// Appends the API authentication/authorization information to the operation description.
    /// </summary>
    public class AppendAuthorizationToDescription : IOperationFilter
    {
        /// <summary>
        /// Applys the appended API authentication/authorization information to the operation description.
        /// </summary>
        /// <param name="operation">An open API operation.</param>
        /// <param name="context">The operation filter context.</param>
        /// <remarks>Basic concepts pulled from https://github.com/mattfrear/Swashbuckle.AspNetCore.Filters.</remarks>
        public void Apply(OpenApiOperation operation, OperationFilterContext context)
        {
            if (context.GetControllerAndActionAttributes<AllowAnonymousAttribute>().Any())
            {
                operation.Description += "\r\n\r\nAuthentication/authorization is not required.";
                return;
            }

            var authorizeAttributes = context.GetControllerAndActionAttributes<AuthorizeAttribute>();
            var authorizeOnAnyOnePolicyAttributes = context.GetControllerAndActionAttributes<AuthorizeOnAnyOnePolicyAttribute>();

            // Get list of authorize policies.
            List<string> authorizeAttributePolicies = authorizeAttributes.Where(x => !string.IsNullOrEmpty(x.Policy))
                .OrderBy(x => x.Policy).Select(x => x.Policy).ToList();

            // Get a list of roles.
            List<string> authorizeAttributeRoles = authorizeAttributes.Where(x => !string.IsNullOrEmpty(x.Roles))
                .OrderBy(x => x.Roles).Select(x => x.Roles).ToList();

            // Get list of authorize on any one policy policies. 
            List<string> authorizeOnAnyOnePolicyAttributePolicies = new List<string>();
            if (authorizeOnAnyOnePolicyAttributes.Any())
            {
                authorizeOnAnyOnePolicyAttributePolicies = authorizeOnAnyOnePolicyAttributes.First().Arguments[0]
                    .ToString().Split(",").OrderBy(x => x).ToList();
            }

            if (authorizeAttributePolicies.Any())
                operation.Description += $"\r\n\r\nAuthorization requires {((authorizeAttributePolicies.Count > 1) ? "each of " : "")} the following {((authorizeAttributePolicies.Count > 1) ? "policies" : "policy")}: <b>{string.Join("</b>, <b>", authorizeAttributePolicies)}</b>";

            if (authorizeAttributeRoles.Any())
                operation.Description += $"\r\n\r\nAuthorization requires {((authorizeAttributeRoles.Count > 1) ? "any one of " : "")} the following {((authorizeAttributeRoles.Count > 1) ? "roles" : "role")}: <b>{string.Join("</b>, <b>", authorizeAttributeRoles)}</b>";

            if (authorizeOnAnyOnePolicyAttributePolicies.Any())
                operation.Description += $"\r\n\r\nAuthorization requires {((authorizeOnAnyOnePolicyAttributePolicies.Count > 1) ? "any one of " : "")} the following {((authorizeOnAnyOnePolicyAttributePolicies.Count > 1) ? "policies" : "policy")}: <b>{string.Join("</b>, <b>", authorizeOnAnyOnePolicyAttributePolicies)}</b>";

            if (authorizeAttributes.Any() && !authorizeAttributePolicies.Any() && !authorizeAttributeRoles.Any() && !authorizeOnAnyOnePolicyAttributePolicies.Any())
                operation.Description += "\r\n\r\nAuthentication, but no authorization is required.";
        }
    }
}
```

The code is relatively simple.  The ```AppendAuthorizationToDescription``` operation filter is created using the ```IOperationFilter``` interface. The interface requires that you define the ```Apply()``` method. This method is handed an ```OpenApiOperation``` which gives you access to things like the API description and an ```OperationFilterContext``` which gives you access to the API attributes.
The first thing the ```Apply()``` method does is to look to see if the API method is decorated with an ```AllowAnonymousAttribute```.
```csharp
[AllowAnonymous]
```
This is done by calling the ```OperationFilterContext``` extension method  ```GetControllerAndActionAttributes```. This was pulled directly from the GitHub repository Swashbuckle.AspNetCore.Filters published by Matt Frear.  The method takes an ```Attribute``` type and returns an ``` IEnumerable``` list of any attributes of the type passed. If an ```AllowAnonymousAttribute``` is found the message “Authentication/authorization is not required.” is appended to the operation description.

If an ```AllowAnonymousAttribute``` was not found the code continues on to again use the ```GetControllerAndActionAttributes``` method, this time to collect a list of attributes type ```AuthorizeAttribute``` and a list of attributes type ```AuthorizeOnAnyOnePolicyAttribute```.  These lists are then queried using Linq to build string lists of policies, roles and authorize on any one policies.  These string lists are then used to generate a verbose message concerning the authorization required which is appended to the description and if there are no required policies, roles or authorize on any one policies the message “Authentication, but no authorization is required.” is returned.