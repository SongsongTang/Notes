---
date: 2024-07-20
aliases:
  - 数据处理软件
tags:
  - 数据处理软件
---
# Compile CERN ROOT from source on Windows
1. Clone ROOT's git repository and create a build directory
``` shell
git clone --branch latest-stable --depth=1 https://github.com/root-project/root.git root_src
mkdir root_build root_install && cd root_build
```
2. `root\_src\graf2d\win32gdk\gdk\src\glib\gunichartables.h` 文件编码有问题，需要修改。
	1. 用记事本打开文件
	2. 另存为
	3. Encoding 选择 *UTF-8 with BOM*
	4. 保存
3. 开始编译、安装
``` shell
cmake -G"Visual Studio 17 2022" -A x64 -Thost=x64 -DCMAKE_INSTALL_PREFIX=C:\Users\stang\root\root_install C:\Users\stang\root\root_src -Droot7=ON -DCMAKE_CXX_STANDARD=17 -Dwebgui=ON -Dqt6web=ON
cmake --build . --config Release
cmake --install . --config Release
```
# 将 CERN ROOT 随安装包一起发布
1. 将 CERNT ROOT 安装包全部复制到项目的 *Dependencies* 文件夹下。
2. 在 *find_package* 之前 加上
``` cmake
set(CMAKE_PREFIX_PATH ${CMAKE_PREFIX_PATH} 
	${PROJECT_SOURCE_DIR}/Dependencies/root/cmake)
```
    用于设置 cmake 文件路径。保证程序在本地环境中可运行。
3. 打开 *Qt Creator*，以 *release* 方式编译运行。
4. 将 _*.exe_ 文件复制到要打包的路径下。
5. 打开 *Qt 6 (MSVC 2019 64-bit)* 的命令行工具，进入到要打包的路径下。
6. 输入 `windeployqt *.exe`，将会自动将所需的 *dll* 文件复制到当前路径下。
7. 将 *Dependencies* 文件夹复制到当前路径下。
8. 打开 *Inno setup* 软件进行打包，需要将 *Dependencies* 文件夹添加到打包文件中，并添加到环境变量中。
	1. 在 **[Setup]** 下添加 `ChangesEnvironment=yes`
	2. 在 **[Files]** 和 **[Icons]** 之间添加以下代码（需替换 `{app}\bin` 路径）:
``` Inno
	[Registry]
	Root: HKLM; Subkey: "SYSTEM\CurrentControlSet\Control\Session Manager\Environment"; \
	    ValueType: expandsz; ValueName: "Path"; ValueData: "{olddata};{app}\Dependencies\root\bin"; \
	    Check: NeedsAddPath(ExpandConstant('{app}\Dependencies\root\bin'))
	
	[Code]
	const
	  EnvironmentKey = 'SYSTEM\CurrentControlSet\Control\Session Manager\Environment';
	  
	function NeedsAddPath(Param: string): boolean;
	var
	  OrigPath: string;
	begin
	  if not RegQueryStringValue(HKEY_LOCAL_MACHINE, EnvironmentKey, 'Path', OrigPath) then
	  begin
	    Result := True;
	    exit;
	  end;
	  // look for the path with leading and trailing semicolon
	  // Pos() returns 0 if not found
	  Result := Pos(';' + Param + ';', ';' + OrigPath + ';') = 0;
	end;
	
	procedure RemovePath(Path: string);
	var
	  Paths: string;
	  P: Integer;
	begin
	  if not RegQueryStringValue(HKEY_LOCAL_MACHINE, EnvironmentKey, 'Path', Paths) then
	  begin
	    Log('PATH not found');
	  end
	    else
	  begin
	    Log(Format('PATH is [%s]', [Paths]));
	    P := Pos(';' + Uppercase(Path) + ';', ';' + Uppercase(Paths) + ';');
	    if P = 0 then
	    begin
	      Log(Format('Path [%s] not found in PATH', [Path]));
	    end
	      else
	    begin
	      if P > 1 then P := P - 1;
	      Delete(Paths, P, Length(Path) + 1);
	      Log(Format('Path [%s] removed from PATH => [%s]', [Path, Paths]));
	      if RegWriteStringValue(HKEY_LOCAL_MACHINE, EnvironmentKey, 'Path', Paths) then
	      begin
	        Log('PATH written');
	      end
	        else
	      begin
	        Log('Error writing PATH');
	      end;
	    end;
	  end;
	end;
	
	procedure CurUninstallStepChanged(CurUninstallStep: TUninstallStep);
	begin
	  if CurUninstallStep = usUninstall then
	  begin
	    RemovePath(ExpandConstant('{app}\Dependencies\root\bin'));
	  end;
	end;
```
	若要正确解析类似 {app}\bin 的路径，需要将含有这类变量的字符串外套上 ExpandConstant() 函数。

# 软件功能