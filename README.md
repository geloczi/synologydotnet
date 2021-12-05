# SynologyDotNet

Solution for local development using project references instead of NuGet packages to help debugging.
This repository used internally to develop my SynologyDotNet packages.  

## Overview

My **client** implementations are **referencing** the **synologydotnet-core** package, which provides a small framework to send requests to the Synology API, and also handles authentication.  

```
synologydotnet-core
 │
 ├── synologydotnet-filestation
 │
 ├── synologydotnet-audiostation
 │   │
 │   └── synologydotnet-audiostation-wpf
 │
 └── ...
```

## The (NuGet) Package Reference vs Project Reference dilemma

Debugging a NuGet package is tricky. I have the source code for my solutions, so I could just add project references, right? But after I finished my development, I'd like to release a package... Then I got to remove the project references and re-add the NuGet packages references. Next time, repat from the beginning...  

The goal of this repository is to provide simple and easy-to-maintain solution to this problem:  

- Use project references only when I develop locally 
- No duplicated *.csproj* files
- Do not break the build in the individual repository for the default *Debug* and *Release* configs.

The last point is important. I want to be able to just clone one single repository which loads the NuGet packages by default and I can just hit F5 in Debug mode.  

## Local configuration

This solution file has a **Local** configuration apart from **Debug** and **Release**. This exists only here, no other solution file contains this entry. But the project files do.  

## Project file magic

I use a project file with both the **NuGet package reference** and the **project reference**.  
The project file literally contains a **switch .. case** statement to setup the project by the selected configuration.  
Here is a snippet from my *synologydotnet-audiostation* repository:  
```
<Project Sdk="Microsoft.NET.Sdk">
    
    <Choose>
        <When Condition=" '$(Configuration)'=='Local' ">
            <PropertyGroup>
                <DefineConstants>DEBUG;TRACE</DefineConstants>
            </PropertyGroup>
            <ItemGroup>
                <ProjectReference Include="..\..\synologydotnet-core\SynologyDotNet.Core\SynologyDotNet.Core.csproj" />
            </ItemGroup>
        </When>
        <Otherwise>
            <ItemGroup>
                <PackageReference Include="SynologyDotNet.Core" Version="0.3.0" />
            </ItemGroup>
        </Otherwise>
    </Choose>
    
</Project>
```

So this little snippet **switches** the project **to project referenes if** I set the configuration to **Local**.  
Otherwise everything defaults to packages references.  
The beautiful in this solution is that the **Local** entry exists only in this "master" solution file, so I do not disturb the build inside the individual repositories. They simply won't switch to **Local**, because their own solution file does not include the **Local** setting.  

## How to use
To debug everything locally, one can clone this repository along with all the others and just set the config to **Local** and hit **Build**.
