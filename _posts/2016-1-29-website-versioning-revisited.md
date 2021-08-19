---
layout: single
title:  "Website Versioning Revisited"
date:   2016-1-29 12:55:00 -0600
category: .NET
tags: ASP.NET AssemblyInfo .NET
---

![Image](/assets/images/posts/Website Versioning Revisited-Version 2.0.1.6.png "Version 2.0.1.6"){: .align-left} A while back I took a look at versioning web applications using an AssemblyInfo file (see [‘Adding Versioning Information to Your Website using AssemblyInfo’](/.net/adding-versioning-information-to-your-website-using-assemblyinfo "Adding Versioning Information to Your Website using AssemblyInfo")).  The article was short sited in one aspect…code reuse.  Because of the amount of research I had done at the time I wrote the article, I was more obsessed with the actual technical means of getting to the information in the AssemblyInfo file and not the bigger picture where no programmer wants to have to remember the long complex code line needed to get say the version number.

Since then I’ve had a chance to encapsulate these methods in a class loosely based on the AssemblyInfo class found in the Visual Basic Application Services namespace.  Note a similar class does not exist in C#.  For your enjoyment you can access this class through NuGet at [https://www.nuget.org/packages/Web.AssemblyInfo/](https://www.nuget.org/packages/Web.AssemblyInfo/ "Web.AssemblyInfo"){:target="_blank"}.

One last interesting thing to note is that this code for a reason I have yet to have time to research will not pull the information from the AssemblyInfo file in the App_Code folder without implementing a global.asax class inherited from System.Web.HttpApplication (NOT SCRIPTED).