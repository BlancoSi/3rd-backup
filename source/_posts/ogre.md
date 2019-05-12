### 使用ogre的游戏编程（1）：通过cmake构建工程文件并编译
### 用到：vs2019 cmake ogre源码

#### 一、编译ogre源码

1. 官网下载安装vs2019和cmake

2. 下载源码，160m绝大部分是sample（ogrev1在github上直接下载zip；v2在SourceTree，依赖库需要自己在上面找）

3. cmake生成vs2019工程文件，open project全部生成

#### 二、编译基于ogre游戏

生成工程文件然后编译的过程中最好使用cmake：

	1. 把你写的·cpp文件·hpp文件放一个文件夹里
	2. 写个CMakeLists.txt格式如下：
```
		cmake_minimum_required (VERSION 2.8)
		project(XXX)

	## [discover_ogre]

		find_package(OGRE 1.11 REQUIRED COMPONENTS Bites RTShaderSystem)
		file(COPY ${OGRE_CONFIG_DIR}/resources.cfg DESTINATION ${CMAKE_BINARY_DIR})

	## [discover_ogre]

		add_executable(oneexe X.cpp)
		target_link_libraries(oneexe OgreBites OgreRTShaderSystem)
		#both ok
		add_executable(anotherexe Y.cpp)
		target_link_libraries(anotherexe ${OGRE_LIBRARIES})
```
	3. cmake把这个文件夹设为源文件夹，然后随便找个文件夹生成，open project，用vs全部生成
	4. ok了，制作完成

过程可以参考：https://blog.csdn.net/zb1165048017/article/details/78673742