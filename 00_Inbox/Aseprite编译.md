1. 获取Aseprite源代码
	 - git
	 - realease下载
2. 下载CMake，Ninja，skia
4. Visual Studio 2022安装C++桌面开发组件，Win10 SDK
5. 控制台执行命令：
```
call "C:\Program Files\Microsoft Visual Studio\2022\Community\Common7\Tools\VsDevCmd.bat" -arch=x64

cd aseprite

mkdir build

cd build

cmake -DCMAKE_BUILD_TYPE=RelWithDebInfo -DLAF_BACKEND=skia -DSKIA_DIR=C:\deps\skia -DSKIA_LIBRARY_DIR=C:\deps\skia\out\Release-x64 -DSKIA_LIBRARY=C:\deps\skia\out\Release-x64\skia.lib -G Ninja ..

ninja aseprite
```