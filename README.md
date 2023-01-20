## Wix Tutorail

### Enviroment
* OS: Windows 10 X64
* IDE: Visual Studio 2022 Community
* Wix Tooset: 3.11

### Topic
Use Wix toolset \<HeatDirectory\> (heat.exe) to the publish file and build the installer for WebApi, WebMvcApp, WPF applications.

### WixProject Custimzation
This is the customized wix project file.
``` xml
<?xml version="1.0" encoding="utf-8"?>
<Project ToolsVersion="4.0" DefaultTargets="Build" InitialTargets="EnsureWixToolsetInstalled" xmlns="http://schemas.microsoft.com/developer/msbuild/2003">
  <PropertyGroup>
    <Configuration Condition=" '$(Configuration)' == '' ">Debug</Configuration>
    <Platform Condition=" '$(Platform)' == '' ">x86</Platform>
    <ProductVersion>3.10</ProductVersion>
    <ProjectGuid>2f2deb46-58ae-4981-8612-1188b654b32d</ProjectGuid>
    <SchemaVersion>2.0</SchemaVersion>
    <OutputName>WebMvcApp.Installer</OutputName>
    <OutputType>Package</OutputType>
  </PropertyGroup>
  <PropertyGroup Condition=" '$(Configuration)|$(Platform)' == 'Debug|x86' ">
    <OutputPath>bin\$(Configuration)\</OutputPath>
    <IntermediateOutputPath>obj\$(Configuration)\</IntermediateOutputPath>
    <DefineConstants>Debug</DefineConstants>
    <SuppressIces>ICE30</SuppressIces>
  </PropertyGroup>
  <PropertyGroup Condition=" '$(Configuration)|$(Platform)' == 'Release|x86' ">
    <OutputPath>bin\$(Configuration)\</OutputPath>
    <IntermediateOutputPath>obj\$(Configuration)\</IntermediateOutputPath>
  </PropertyGroup>
  <ItemGroup>
    <Compile Include="Product.wxs" />
    <Compile Include="WebMvcApp.wxs" />
  </ItemGroup>
  <ItemGroup>
    <ProjectReference Include="..\WebMvcApp\WebMvcApp.csproj">
      <Name>WebMvcApp</Name>
      <Project>{d0edb040-d8db-4468-b085-09084963006f}</Project>
      <Private>True</Private>
      <DoNotHarvest>True</DoNotHarvest>
      <RefProjectOutputGroups>Binaries;Content;Satellites</RefProjectOutputGroups>
      <RefTargetDir>INSTALLFOLDER</RefTargetDir>
    </ProjectReference>
  </ItemGroup>
  <Import Project="$(WixTargetsPath)" Condition=" '$(WixTargetsPath)' != '' " />
  <Import Project="$(MSBuildExtensionsPath32)\Microsoft\WiX\v3.x\Wix.targets" Condition=" '$(WixTargetsPath)' == '' AND Exists('$(MSBuildExtensionsPath32)\Microsoft\WiX\v3.x\Wix.targets') " />
  <Target Name="EnsureWixToolsetInstalled" Condition=" '$(WixTargetsImported)' != 'true' ">
    <Error Text="The WiX Toolset v3.11 build tools must be installed to build this project. To download the WiX Toolset, see https://wixtoolset.org/releases/v3.11/stable" />
  </Target>
  <!--
	To modify your build process, add your task inside one of the targets below and uncomment it.
	Other similar extension points exist, see Wix.targets.
	<Target Name="BeforeBuild">
	</Target>
	<Target Name="AfterBuild">
	</Target>
	-->
  <!-- Adding Heat copying -->
  <!-- WiX tools are 32bit EXEs, so run them out-of-proc when MSBuild is not 32bit. -->
  <PropertyGroup>
    <RunWixToolsOutOfProc Condition=" '$(PROCESSOR_ARCHITECTURE)'!='x86' ">true</RunWixToolsOutOfProc>
  </PropertyGroup>
  <Target Name="BeforeBuild">
    <Exec Command="dotnet publish ..\WebMvcApp\WebMvcApp.csproj -c $(Configuration) -r win10-x64 --self-contained" />
    <PropertyGroup>
      <LinkerBaseInputPaths>..\WebMvcApp\bin\$(Configuration)\net6.0\win10-x64\publish</LinkerBaseInputPaths>
      <DefineConstants>BasePath=..\WebMvcApp\bin\$(Configuration)\net6.0\win10-x64\publish</DefineConstants>
    </PropertyGroup>
    <HeatDirectory OutputFile="WebMvcApp.wxs" DirectoryRefId="INSTALLFOLDER" ComponentGroupName="WebMvcAppProject" SuppressCom="true" Directory="..\WebMvcApp\bin\$(Configuration)\net6.0\win10-x64\publish" SuppressFragments="true" SuppressRegistry="true" SuppressRootDirectory="true" AutoGenerateGuids="false" GenerateGuidsNow="true" ToolPath="$(WixToolPath)" PreprocessorVariable="var.BasePath" RunAsSeparateProcess="$(RunWixToolsOutOfProc)" />
  </Target>
</Project>
```

In this file, we need to modify 3 places:
* First, add RunWixToolsOutOfProc
```
<!-- WiX tools are 32bit EXEs, so run them out-of-proc when MSBuild is not 32bit. -->
  <PropertyGroup>
    <RunWixToolsOutOfProc Condition=" '$(PROCESSOR_ARCHITECTURE)'!='x86' ">true</RunWixToolsOutOfProc>
  </PropertyGroup>
```
* Second add the HeatDirectory
```
<!-- Adding Heat copying -->
  <Target Name="BeforeBuild">
    <!-- build the mvc project with self contained dependency -->
    <Exec Command="dotnet publish ..\WebMvcApp\WebMvcApp.csproj -c $(Configuration) -r win10-x64 --self-contained" />
    <PropertyGroup>
      <LinkerBaseInputPaths>..\WebMvcApp\bin\$(Configuration)\net6.0\win10-x64\publish</LinkerBaseInputPaths>
      <DefineConstants>BasePath=..\WebMvcApp\bin\$(Configuration)\net6.0\win10-x64\publish</DefineConstants>
    </PropertyGroup>
    <HeatDirectory OutputFile="WebMvcApp.wxs" DirectoryRefId="INSTALLFOLDER" ComponentGroupName="WebMvcAppProject" SuppressCom="true" Directory="..\WebMvcApp\bin\$(Configuration)\net6.0\win10-x64\publish" SuppressFragments="true" SuppressRegistry="true" SuppressRootDirectory="true" AutoGenerateGuids="false" GenerateGuidsNow="true" ToolPath="$(WixToolPath)" PreprocessorVariable="var.BasePath" RunAsSeparateProcess="$(RunWixToolsOutOfProc)" />
  </Target>

```
Third, Wix Tool Setting to suppress ICE30
```
<SuppressIces>ICE30</SuppressIces>

```


