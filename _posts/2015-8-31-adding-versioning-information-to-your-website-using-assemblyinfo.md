---
layout: posts
title:  "Adding Versioning Information to Your Website using AssemblyInfo"
date:   2015-8-31 12:49:00 -0600
category: .NET
tags: ASP.NET AssemblyInfo .NET
---

![Image](/assets/images/Version 2.0.1.5.png "Version 2.0.1.5")

When it comes to versioning the .NET framework has everything a developer needs to define information about an applications assembly like name, description and version.  Templates used by Visual Studio to create projects automatically generates an AssemblyInfo file under the Properties folder for the developer to fill in values for assembly attributes such as AssemblyTitle, AssemblyCopyright and AssemblyVersion.  That is until you build a website application.  For whatever reason the Visual Studio templates for building a website application don’t generate an AssemblyInfo file.

If you search the internet you will find all kinds of suggestions for versioning your website.  Ideas like putting version information in the web.config file, creating a class file with static property methods that return the version and the list goes on.  Why reinvent the wheel when the .NET framework already provides a standardized method for providing this information with an AssemblyInfo file.  The great thing about this is the process is simple, the .NET framework provides the necessary classes to access this information and developers are already familiar with the use of the AssemblyInfo file.
 
The first thing you need to do is create an AssemblyInfo.cs (or .vb) file in the App_Code folder.  Once created you can replaces the generated class code with the contents of and existing projects AssemblyInfo file and modify the information for each of the assembly attributes.

{% highlight csharp %}
using System.Reflection;
using System.Runtime.InteropServices;

// General Information about an assembly is controlled through the following 
// set of attributes. Change these attribute values to modify the information
// associated with an assembly.
[assembly: AssemblyTitle("")]
[assembly: AssemblyDescription("")]
[assembly: AssemblyConfiguration("")]
[assembly: AssemblyCompany("")]
[assembly: AssemblyProduct("")]
[assembly: AssemblyCopyright("")]
[assembly: AssemblyTrademark("")]
[assembly: AssemblyCulture("")]

// Setting ComVisible to false makes the types in this assembly not visible 
// to COM components.  If you need to access a type in this assembly from 
// COM, set the ComVisible attribute to true on that type.
[assembly: ComVisible(false)]

// The following GUID is for the ID of the typelib if this project is exposed to COM
[assembly: Guid("4ac9e932-3530-4eb6-a038-c3323aa36cb8")]

// Version information for an assembly consists of the following four values:
//
//      Major Version
//      Minor Version 
//      Build Number
//      Revision
//
// You can specify all the values or you can default the Build and Revision Numbers 
// by using the '*' as shown below:
// [assembly: AssemblyVersion("1.0.*")]
[assembly: AssemblyVersion("1.0.0.0")]
[assembly: AssemblyFileVersion("1.0.0.0")] 
{% endhighlight %}

Next you need to access this information. This becomes a bit tricky as ASP.NET runs web code inside an existing application assembly. So in say a Windows application you would use the following code to get the version information.

{% highlight csharp %}
using System.Reflection;
Version version = Assembly.GetEntryAssembly().GetName().Version;
{% endhighlight %}

But using this code in an ASP.NET application would actually pull the version information from the ASP.NET assembly spun up by IIS in the application pool and return version 0.0.0.0.  The trick is to go after the correct assembly through the current HttpContext.  By using the current HttpContext you can access the websites assembly information in any page, user control, internal class (\App_Code) and external compiled class.  The following is a code example you can paste in to a web page to display the information provided in the AssemblyInfo file.

{% highlight csharp %}
AssemblyTitle: <%= ((System.Reflection.AssemblyTitleAttribute)Attribute.GetCustomAttribute(HttpContext.Current.ApplicationInstance.GetType().BaseType.Assembly, typeof(System.Reflection.AssemblyTitleAttribute), false)).Title %><br />
AssemblyDescription:<%= ((System.Reflection.AssemblyDescriptionAttribute)Attribute.GetCustomAttribute(HttpContext.Current.ApplicationInstance.GetType().BaseType.Assembly, typeof(System.Reflection.AssemblyDescriptionAttribute), false)).Description %><br />
AssemblyConfiguration:<%= ((System.Reflection.AssemblyConfigurationAttribute)Attribute.GetCustomAttribute(HttpContext.Current.ApplicationInstance.GetType().BaseType.Assembly, typeof(System.Reflection.AssemblyConfigurationAttribute), false)).Configuration%><br />
AssemblyCompany: <%= ((System.Reflection.AssemblyCompanyAttribute)Attribute.GetCustomAttribute(HttpContext.Current.ApplicationInstance.GetType().BaseType.Assembly, typeof(System.Reflection.AssemblyCompanyAttribute), false)).Company %><br />
AssemblyProduct: <%= ((System.Reflection.AssemblyProductAttribute)Attribute.GetCustomAttribute(HttpContext.Current.ApplicationInstance.GetType().BaseType.Assembly, typeof(System.Reflection.AssemblyProductAttribute), false)).Product %><br />
AssemblyCopyright: <%= ((System.Reflection.AssemblyCopyrightAttribute)Attribute.GetCustomAttribute(HttpContext.Current.ApplicationInstance.GetType().BaseType.Assembly, typeof(System.Reflection.AssemblyCopyrightAttribute), false)).Copyright%><br />
AssemblyTrademark: <%= ((System.Reflection.AssemblyTrademarkAttribute)Attribute.GetCustomAttribute(HttpContext.Current.ApplicationInstance.GetType().BaseType.Assembly, typeof(System.Reflection.AssemblyTrademarkAttribute), false)).Trademark %><br />
AssemblyCulture: <%= (((System.Reflection.AssemblyCultureAttribute)Attribute.GetCustomAttribute(HttpContext.Current.ApplicationInstance.GetType().BaseType.Assembly, typeof(System.Reflection.AssemblyCultureAttribute), false)) ?? new System.Reflection.AssemblyCultureAttribute("")).Culture ?? "" %><br />
<br />
ComVisible: <%= ((System.Runtime.InteropServices.ComVisibleAttribute)Attribute.GetCustomAttribute(HttpContext.Current.ApplicationInstance.GetType().BaseType.Assembly, typeof(System.Runtime.InteropServices.ComVisibleAttribute), false)).Value.ToString() %><br />
Guid: <%= ((System.Runtime.InteropServices.GuidAttribute)Attribute.GetCustomAttribute(HttpContext.Current.ApplicationInstance.GetType().BaseType.Assembly, typeof(System.Runtime.InteropServices.GuidAttribute), false)).Value.ToString() %><br />
<br />
AssemblyVersion: <%= HttpContext.Current.ApplicationInstance.GetType().BaseType.Assembly.GetName().Version %><br /> 
AssemblyFileVersion: <%= ((System.Reflection.AssemblyFileVersionAttribute)Attribute.GetCustomAttribute(HttpContext.Current.ApplicationInstance.GetType().BaseType.Assembly, typeof(System.Reflection.AssemblyFileVersionAttribute), false)).Version %><br />
<br />
Available Attributes: <br />
<%
foreach(Attribute attribute in Attribute.GetCustomAttributes(HttpContext.Current.ApplicationInstance.GetType().BaseType.Assembly, false))
    {
        Response.Write(attribute.ToString()); %> <br />
<%  } %> 
{% endhighlight %}

Update: Check out ['Website Versioning Revisited'](/.net/2016/01/29/website-versioning-revisited.html).