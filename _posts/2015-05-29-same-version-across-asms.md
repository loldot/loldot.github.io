---
title: Maintain the same version no. across multiple .net assemblies
author: Lorentz Vedeler
date: 2015-05-29
tags: Code
---
If you want several projects to always have the same version number, this is a pretty neat trick:

1. Remove AssemblyVerison and AssemblyFileVersion from existing AssemblyInfo.cs files
2. Add a CommonAssemblyInfo.cs file to one of the projects containing just:

	```csharp
	using System.Reflection;

	[assembly: AssemblyVersion("2.3.1.0")]
	[assembly: AssemblyFileVersion("2.3.1.0")]
	```

3. In the other projects add existing item choose the CommonAssemblyInfo.cs, but remember to add it as a link (click on the arrow on the add button)
