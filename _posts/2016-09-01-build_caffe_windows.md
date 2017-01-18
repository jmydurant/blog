---
layout: post
title: 在 windows 上编译 caffe
category: tech
comments: false
---

## 起因

在看ssd代码的时候，就对caffe的源码很感兴趣。想要借此机会稍微学习一下。
但是Ubuntu下看代码速度和效率比较慢，想到了如果能在vs上面读代码那么效率肯定会大很多。
因此，编译了一下windows版本的caffe。其中有些暗坑还需要多注意。

## caffe windows 源

我们可以选用[这个源](https://github.com/Microsoft/caffe)，并且我只推荐这个源。选用的原因也很简单，基本算是比较官方官方的源了。
这个源的md文件上面讲述了编译方法，当然也可以参考我安装的过程。

## 环境配置

首先，非常重要的一点是，我们需要安装一个visual studio 2013。最好就是这个版本，更新的2015我是编译失败了（我没有具体去查原因）。
并且由于cuda8.0暂时不支持包括2015 update 2 以及以上的版本(改变了c++的一些语法特性)，所以选择2013可以免去非常多蛋疼的问题（我甚至没有安装update包）。

其次，安装一个比较新的cuda。windows上面我仍然安装了cuda8.0，安装过程比Ubuntu上面要简单很多。安装完成之后就可以在上面建立简单的cuda工程了。
如果要安装cudnn，我的建议是解压到一个自己规定的文件夹比较好。在这里我选择5.1版本的cudnn。

最后，因为我需要pycaffe，所以我还安装了miniconda。这个在官网上找到64位安装包就可以安装好了，非常简单。目录理论上就在`C:\Miniconda2`。
顺便执行如下命令安装基本python的库：

``` bash
conda install --yes numpy scipy matplotlib scikit-image pip
pip install protobuf
```

建议上面除了`protobuf`都用`conda`来安装，我自己用`pip`装的时候出现过很多莫名的问题（各种编译器问题），
**并且需要管理员权限**。

## 配置文件

我们打开下载的caffe源，进入windows文件夹。看到一个CommonSettings.props.example文件，将其复制一份并且命名为
CommonSettings.props。打开之后，我们会看到如下内容：

``` xml
<?xml version="1.0" encoding="utf-8"?>
<Project ToolsVersion="4.0" xmlns="http://schemas.microsoft.com/developer/msbuild/2003">
    <ImportGroup Label="PropertySheets" />
    <PropertyGroup Label="UserMacros">
        <BuildDir>$(SolutionDir)..\Build</BuildDir>
        <!--NOTE: CpuOnlyBuild and UseCuDNN flags can't be set at the same time.-->
        <CpuOnlyBuild>false</CpuOnlyBuild>
        <UseCuDNN>true</UseCuDNN>
        <CudaVersion>7.5</CudaVersion>
        <!-- NOTE: If Python support is enabled, PythonDir (below) needs to be
         set to the root of your Python installation. If your Python installation
         does not contain debug libraries, debug build will not work. -->
        <PythonSupport>false</PythonSupport>
        <!-- NOTE: If Matlab support is enabled, MatlabDir (below) needs to be
         set to the root of your Matlab installation. -->
        <MatlabSupport>false</MatlabSupport>
        <CudaDependencies></CudaDependencies>

        <!-- Set CUDA architecture suitable for your GPU.
         Setting proper architecture is important to mimize your run and compile time. -->
        <CudaArchitecture>compute_35,sm_35;compute_52,sm_52</CudaArchitecture>

        <!-- CuDNN 3 and 4 are supported -->
        <CuDnnPath></CuDnnPath>
        <ScriptsDir>$(SolutionDir)\scripts</ScriptsDir>
    </PropertyGroup>
    <PropertyGroup Condition="'$(CpuOnlyBuild)'=='false'">
        <CudaDependencies>cublas.lib;cuda.lib;curand.lib;cudart.lib</CudaDependencies>
    </PropertyGroup>

    <PropertyGroup Condition="'$(UseCuDNN)'=='true'">
        <CudaDependencies>cudnn.lib;$(CudaDependencies)</CudaDependencies>
    </PropertyGroup>
    <PropertyGroup Condition="'$(UseCuDNN)'=='true' And $(CuDnnPath)!=''">
        <LibraryPath>$(CuDnnPath)\cuda\lib\x64;$(LibraryPath)</LibraryPath>
        <IncludePath>$(CuDnnPath)\cuda\include;$(IncludePath)</IncludePath>
    </PropertyGroup>

    <PropertyGroup>
        <OutDir>$(BuildDir)\$(Platform)\$(Configuration)\</OutDir>
        <IntDir>$(BuildDir)\Int\$(ProjectName)\$(Platform)\$(Configuration)\</IntDir>
    </PropertyGroup>
    <PropertyGroup>
        <LibraryPath>$(OutDir);$(CUDA_PATH)\lib\$(Platform);$(LibraryPath)</LibraryPath>
        <IncludePath>$(SolutionDir)..\include;$(SolutionDir)..\include\caffe\proto;$(CUDA_PATH)\include;$(IncludePath)</IncludePath>
    </PropertyGroup>
    <PropertyGroup Condition="'$(PythonSupport)'=='true'">
        <PythonDir>C:\Miniconda2\</PythonDir>
        <LibraryPath>$(PythonDir)\libs;$(LibraryPath)</LibraryPath>
        <IncludePath>$(PythonDir)\include;$(IncludePath)</IncludePath>
    </PropertyGroup>
    <PropertyGroup Condition="'$(MatlabSupport)'=='true'">
        <MatlabDir>C:\Program Files\MATLAB\R2014b</MatlabDir>
        <LibraryPath>$(MatlabDir)\extern\lib\win64\microsoft;$(LibraryPath)</LibraryPath>
        <IncludePath>$(MatlabDir)\extern\include;$(IncludePath)</IncludePath>
    </PropertyGroup>
    <ItemDefinitionGroup Condition="'$(CpuOnlyBuild)'=='true'">
        <ClCompile>
            <PreprocessorDefinitions>CPU_ONLY;%(PreprocessorDefinitions)</PreprocessorDefinitions>
        </ClCompile>
    </ItemDefinitionGroup>
    <ItemDefinitionGroup Condition="'$(UseCuDNN)'=='true'">
        <ClCompile>
            <PreprocessorDefinitions>USE_CUDNN;%(PreprocessorDefinitions)</PreprocessorDefinitions>
        </ClCompile>
        <CudaCompile>
            <Defines>USE_CUDNN</Defines>
        </CudaCompile>
    </ItemDefinitionGroup>
    <ItemDefinitionGroup Condition="'$(PythonSupport)'=='true'">
        <ClCompile>
            <PreprocessorDefinitions>WITH_PYTHON_LAYER;BOOST_PYTHON_STATIC_LIB;%(PreprocessorDefinitions)</PreprocessorDefinitions>
        </ClCompile>
    </ItemDefinitionGroup>
    <ItemDefinitionGroup Condition="'$(MatlabSupport)'=='true'">
        <ClCompile>
            <PreprocessorDefinitions>MATLAB_MEX_FILE;%(PreprocessorDefinitions)</PreprocessorDefinitions>
        </ClCompile>
    </ItemDefinitionGroup>
    <ItemDefinitionGroup>
        <ClCompile>
            <MinimalRebuild>false</MinimalRebuild>
            <MultiProcessorCompilation>true</MultiProcessorCompilation>
            <PreprocessorDefinitions>_SCL_SECURE_NO_WARNINGS;USE_OPENCV;USE_LEVELDB;USE_LMDB;%(PreprocessorDefinitions)</PreprocessorDefinitions>
            <TreatWarningAsError>true</TreatWarningAsError>
        </ClCompile>
    </ItemDefinitionGroup>
    <ItemDefinitionGroup Condition="'$(Configuration)|$(Platform)'=='Release|x64'">
        <ClCompile>
            <Optimization>Full</Optimization>
            <PreprocessorDefinitions>NDEBUG;%(PreprocessorDefinitions)</PreprocessorDefinitions>
            <RuntimeLibrary>MultiThreadedDLL</RuntimeLibrary>
            <FunctionLevelLinking>true</FunctionLevelLinking>
        </ClCompile>
        <Link>
            <EnableCOMDATFolding>true</EnableCOMDATFolding>
            <GenerateDebugInformation>true</GenerateDebugInformation>
            <LinkTimeCodeGeneration>UseLinkTimeCodeGeneration</LinkTimeCodeGeneration>
            <OptimizeReferences>true</OptimizeReferences>
        </Link>
    </ItemDefinitionGroup>
    <ItemDefinitionGroup Condition="'$(Configuration)|$(Platform)'=='Debug|x64'">
        <ClCompile>
            <Optimization>Disabled</Optimization>
            <PreprocessorDefinitions>_DEBUG;%(PreprocessorDefinitions)</PreprocessorDefinitions>
            <RuntimeLibrary>MultiThreadedDebugDLL</RuntimeLibrary>
        </ClCompile>
        <Link>
            <GenerateDebugInformation>true</GenerateDebugInformation>
        </Link>
    </ItemDefinitionGroup>
</Project>
```

对于这个xml文件，我们只需要修改几个内容：

- (1). 对于`CudaVersion`，我们需要修改成我们现在使用的cuda版本。

- (2). 如果需要编译pycaffe，则需要修改`PythonSupport`为`true`，并且确认PythonDir是否为`miniconda`的安装地址。

- (3). 如果cudnn是和我一样解压到一个文件夹里面，而不是和cuda的头文件库文件混在一起的话，修改`CuDnnPath`项，
并且注意，路径填写到cuda文件夹之前，比如如果你的cudnn是在`c:\xxx\cuda`里面的话，这一项只需要填写`c:\xxx\`即可。

- (4). 关于matlab的支持，参考源文档。

## 打开工程，并且配置编译。

利用visual studio 2013 打开Caffe.sln，将项目设置为Release版本x64。

不仅如此，你会看到这个解决方案有很多的项目，我们设置libcaffe为默认启动项目(右键此项目，选择设置为启动项目)。

这个项目有很多库都还暂时不存在，不过不要紧，像boost，OpenCV，glog之类的都会由NuGet来自动下载，非常酷炫！
我们从菜单栏里选择`工具(tools) -> NuGet 程序包管理器 -> 管理解决方案的NuGet程序包`，
就可以下载好这个项目的所有依赖。

下载好了之后，就可以痛快的编译了，因为warning比较多，我们可以选择忽略warning。

完成之后，我们打开Build文件夹，依次进入x64，Release，就可以看到caffe的一系列程序已经编译完成啦。Please enjoy it！！