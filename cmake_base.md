# CMake概述

## 程序如何执行？

>  源文件(.c / .cpp) --> 预处理 --> 编译 --> 汇编 --> 链接 --> 可执行程序(.exe)

缺点：如果项目结构复杂，使用的命令会变得长且繁琐

解决方法：makefile --> cmake

makefile：首先需要创建一个脚本文件，名为 makefile ，告诉编译器如何处理源文件。然后执行批处理命令 make 。

cmake：首先需要创建一个脚本文件，名为 ==CMakeLists.txt== ，告诉编译器如何处理源文件。然后执行 ==cmake== 命令，此时会生成一个 ==makefile== 文件，最后在执行 ==make== 指令，便会自发的执行程序运行操作。

cmake 不仅仅可以帮助生成可执行文件，也可以生成库文件，库文件分为静态库和动态库，生成的库文件可以引入到另外的第三方项目中。

为什么使用库，而不直接发源文件？

> 1. 首先就是保密，防止源文件代码的泄露。
>
> 2. 此外引入大量的代码时复杂的，如果一个项目存在几百个文件，如果要引入第三方库中时，需要大量的文件引入，是复杂也不利于维护的。

## 写一个CMake脚本

> Linux 知识补充：
>
> 1. 添加一个文件 ---> 	                   touch 文件
> 2. 添加一个文件夹--> 	                 mkdir 文件夹路径
> 3. 删除目录(空文件夹)->                     rmdir  文件夹路径
> 4. 删除文件(或者是非空的文件夹)-> rm 文件路径

> CMake 不区分大小写

### CMake注释

1. 注释行 ：`# 注释内容`
2. 注释块 ：`#[[ 注释块内容 ]]`

### CMake初始写点什么

1.  cmake_minimum_required
   指定cmake的最低版本，如果本机的cmake版本低于指定版本，无法构建

   ```cmake
   cmake_minimum_required(VERSION 3.xx.xx)
   ```

   可选，非必须，如果不加可能会有警告

2. project
   构建工程，指定工程信息，包括（工程名称，工程版本，...)

   ```cmake
   project(PROJECT-NAME, [VERSION], ...)
   ```

   

3. add_executable

   定义工程会生成一个可执行文件

   ```cmake
   add_executable(目标文件名称 [被链接文件] [被链接文件] [被链接文件] ...)
   add_executable(目标文件名称 [被链接文件];[被链接文件];[被链接文件];...)
   ```

   

### 编译生成可执行文件

> CMakeLists.txt --`cmake`--> Makefile --`make`--> .exe

1. ```cmake
   cmake CMakeLists.txt文件所在路径
   ```

   + **在当前目录下**生成一组对应的CMake文件，包括Makefile

2. ```
   make
   ```

   + **make指令在Makefile文件的同级目录下执行**
   + make编译Makefile文件，**在当前目录下**生成可执行文件





## 写一个更好的CMake

### SET命令

#### 变量

+ 定义变量
  ```cmake
  # SET 指令的语法是:
  # [] 是可选
  SET(VAR [VALUE])
  ```

  - `VAR`：变量名
  - `VALUE`：变量值
  - 使用SET命令声明变量，变量为一个字符串string，类似于
    VAR = string（VALUE）

+ 使用变量
  ```cmake
  SET(SRC1 add.c div.c main.c mult.c sub.c)
  # SET(SRC2 add.c div.c main.c mult.c sub.c)
  add_executable(app ${SRC1})
  ```

  + `${VAR}`来取出变量的值



#### 指定C++版本

> 在编写 c++ 程序的时候，可能会用到 c++11， c++14，c++17，c++20等新特性，那么就需要在编译的时候指出说使用的 c++ 版本

```cmd
$ g++ *.cpp -std=c++11 -o app
```

```cmd
$ cmake CMakeLists.txt文件路径 -DCMAKE_CXX_STANDARD=11
```

```cmake
# 在Cmake中添加SET命令
SET(CMAKE_CXX_STANDARD 11)
```



#### 指定生成的.exe文件路径

```cmake
SET(HOME /home)
SET(EXECUTABLE_OUTPUT_PATH ${HOME}/bin)
```

+ 第一行：定义一个绝对路径

+ 第二行：基于第一个绝对路径，将生成的.exe文件存放到指定路径下

  + 如果这个路径中包含没有存在的文件，会自动生成
  + 这个.exe文件是可执行文件，在make命令编译Makefile时生成得到的

  





### 搜索指令

> 如果一个项目中源文件很多，在编写CMakeLists.txt文件时一一罗列是一件很繁琐的事情，所以我们希望有个方法来更好的表示多个文件 ---》搜索指令

+ ```cmake
  aux_source_directory(<dir>  <variable>)
  ```

  + `<dir>`：要搜索的目录
  + `<variable>`：变量名，将`<dir>`目录下的文件存入变量

+ ```cmake
  file(GLOB/GLOB_RECURSE <variable> <dir>)
  ```

  + `GLOB`：将目录下搜索到的满足条件的所有文件名生成一个列表，并将其存入到变量中
  + `GLOB_RECURSE`：递归搜索指定目录，将目录下搜索到的满足条件的所有文件名生成一个列表，并将其存入到变量中
  + `<variable>`：变量名
  + `<dir>`：要搜索的文件，要指定文件格式，`${XXX}/ --> ${XXX}/*.cpp`

1. 举个例子--`aux_source_director()`

   ```cmake
   aux_source_director(${PRROJECT_SOURCE_DIR} SRC)
   add_executable(run ${SRC})
   ```

   + `${PRROJECT_SOURCE_DIR}`指的是执行cmake指令时，`cmake XX`后面的文件`XX`

2. 举个例子--`file()`

   ```cmake
   file(GLOB SRC ${CMAKE_CURRENT_SOURCE_DIR}/*.cpp)
   add_executable(run ${SRC})
   ```

   + `${CMAKE_CURRENT_SOURCE_DIR}`指的是这个`CMakeLists.txt`文件所在目录





### 包含头文件

> 在项目开发中，如果头文件和源文件分别放在了两个文件夹下，这样我们如何在源文件中指定头文件呢？
> ```cmd
> .
> ├── CMakeLists.txt
> ├── build
> ├── include
> │   └── head.h
> └── src
>     ├── add.cpp
>     ├── div.cpp
>     ├── main.cpp
>     ├── mult.cpp
>     └── sub.cpp
> ```
>
> + 如图所示，在main.cpp中如何使用`#include "head.h"`包含头文件呢？

+ 在`#include "head.h"`使用头文件时，就指明头文件所在路径

+ 在CMakeLists.txt中添加 `include_directories(<dir>)` 代码

  ```cmake
  include_directories(<dir>)
  ```

  + `<dir>`指出了使用头文件时要去查找的目录

  + `include_directories(${PROJECT_SOURCE_DIR}/include)`





### 生成库文件

> 什么是库文件？为什么要使用库文件？？
>
> + 库文件是由一个或多个.cpp文件链接生成的二进制码文件，这些.cpp文件都是由一些不包含main函数的其他功能函数构成的。
> + 发行时通常为了保证源码的安全，发行只开放 ==头文件 + 库文件==

#### 制作库文件

```cmake
add_library(库名称 STATIC/SHARED 源文件1 源文件2 ...)
```

+ `static`：表示静态库  ---> .a
+ `shared`：表示动态库   ---> .so
+ 库名称：通常生成的库文件名称为`lib+库名称`

#### 指定库文件存放目录

> 默认执行make后生成的库文件的目录是和Makefile在相同的目录下
> ```cmd
> .
> ├── CMakeCache.txt
> ├── CMakeFiles
> ├── Makefile
> ├── cmake_install.cmake
> └── liblibtest.a
> ```



```cmake
set(LIBRARY_OUTPUT_PATH 目录)
```

+ `LIBRARY_OUTPUT_PATH`：是生成的库文件存放的地址，可以补全不存在的目录





### 连接静态库

> 如果我们得到了发行的==头文件 + 库文件==我们应该如何去使用这些库文件呢？
>
> 连接库，将库文件中的函数连接到我们的项目中

```cmake
link_libraries(<static lib> [<static lib>] [<static lib>] ...)
```

+ 连接库文件
+ `<static lib>`：表示一个库文件的名称，通常掐头去尾`lib，.a`

```cmake
# 对于系统中定义的库，我们直接连接即可
# 对于自定义的库，我们需要使用 link_directories 来指明库的路径
link_directories(<lib path> [<lib path>] [<lib path>] ...)
```

> 我们也可以使用如下的方式连接静态库
> ```cmake
> target_link_libraries(
> 	<target>
> 	<PRIVATE|PUBLIC|INTERFACE> <item> ...
> 	[<PRIVATE|PUBLIC|INTERFACE> <item> ...]
> )
> ```
>
> 放到生成库文件/可执行文件==之后==



### 连接动态库

> 静态库和动态库的区别？
>
> + 静态库：在编译阶段时，将库文件的内容加载到了内存中，等待调用
> + 动态库：在编译阶段时，不讲库文件的内容加载到内存，直到执行文件调用库中的内容时，才会将动态库中对应的部分存入内存中

```cmake
target_link_libraries(
	<target>
	<PRIVATE|PUBLIC|INTERFACE> <item> ...
	[<PRIVATE|PUBLIC|INTERFACE> <item> ...]
)
```

+ `<target>`：表示要与库连接的文件
  + 该文件可以是一个源文件
  + 该文件可以是一个动态库文件
  + 该文件可以是一个可执行文件
+ `<PRIVATE|PUBLIC|INTERFACE>`：规定动态库连接的权限，默认`PUBLIC`
  + `PRIVATE` 这里的连接可以传递，例如`<target>`a-->b-->c，如果有一个动态库d连接了a（d --> a），那么在d中可以使用b，c中的方法。
  + `PUBLIC` 这里的连接不具有传递像，只能使用连接库中的方法，a --> b --> c，a只能使用b中的方法，不能使用c中的方法。
  + `INTERFACE` 这里的连接后，a --> b，我们并不能区分方法是来源于a还是来源于b，同时也不可能传递。本质上a --> b 时，并没有把b的方法加载到a中，而只是使用了接口的方式，接口在传递过程中会消失，所以也不具有传递性。
+ `<item>`：连接的库的名字

#### 注意

```cmake
# 对于系统中定义的库，我们直接连接即可
# 对于自定义的库，我们需要使用 link_directories 来指明库的路径
link_directories(<lib path> [<lib path>] [<lib path>] ...)
```

```cmake
# 动态库在使用时，连接在使用之后
add_executable(run ${SRC})
target_link_libraries(
	run
	lib_shared1
	PRIVATE lib_shared2
)
```





## 调试一下CMake

### 输出日志信息

```cmake
message([STATUS|WARNING|AUTHOR_WARNING|FATAL_ERROR|SEND_ERROR] "message to display" ...)
```

+ `（无）`：重要信息
+ `STATUS`：非重要信息
+ `WARNING`：CMake警告，会继续执行
+ `AUTHOR_WARNING`：CMake警告，比较严重，但是会继续执行
+ `SEND_ERROR`：CMake错误，继续执行，但是会跳过生成的步骤
+ `FATAL_ERROR`：CMake错误，终止所有处理过程



```cmake
# 输出一般日志信息
message（STATUS "source path: ${PROJECT_SOURCE_DIR}")
```

+ 可以使用`${}`打印一些变量的值来观察代码执行情况





## CMake的字符串

### 字符串拼接

```cmake
# 方法1
set(变量名 ${变量名1} ${变量名2} ...)
```

```cmake
# 方法2
list(APPEND <list> [<element> ...])
```

+ APPEND：关键字，表示对list执行拼接操作
+ `<list>`：这里就是变量名
+ `<element>`：拼接的字符串

> 拼接后，自动会补充`";"`进行分隔



### 字符串移除list

```cmake
list(REMOVE_ITEM <list> <value> [<value> ...])
```

+ `REMOVE_ITEM`：移除指令
+ `<value>`：表示移除的字符串内容，如果是路径，应该是 完整路径



### list的其他指令

#### list 的长度

```cmake
list(LENGT <list> <output variable>)
```

+ `<list>`：表示变量
+ `<output variable>`：将长度存储在该变量中

#### list 索引

> + 索引从零开始，索引0表示第一个元素
> + 索引可以是负数，-1表示最后一个元素，-2表示倒数第二个元素
> + 索引无论正负，表示的大小不可以超过list的length长度

#### list 查询

```cmake
list(GET <list> <element index> [<element index> ...] <output variable>)
```

+ `<element index>`：表示索引值
+ `<output variable>`：将索引对应的值存储在该变量中

#### list 插入

```cmake
list(INSERT <list> <element index> <element> [<element> ...])
```

+ 将`<element1> <element2> <element3> ...`插入到`list`中，然后`<element1>`索引为`<element index>`

#### list 头插

```cmake
list(PREPEND <list> [<element> ...])
```

+ 和`list插入`指令相同，其中插入到索引0中

#### list 删去尾部元素

```cmake
list(POP_BACK <list> <out-variable>)
```

+ `<out-variable>`存放弹出的变量

#### list 删除首部元素

```cmake
list(POP_FRONT <list> <out-variable>)
```

+ `<out-variable>`存放弹出变量

#### list 删除指定元素

```cmake
list(REMOVE_ITEM <list> <value> [<value>...])
```

+ `<value>`指定元素

#### list 删除指定索引对应的元素

```cmake
list(REMOVE_AT <list> <Index> [<Index> ...])
```

+ `<Index>`指定索引

#### list 去重

```cmake
list(REMOVE_OUPLICATES <list>)
```

#### list 排序

```cmake
list(SORT <list> [COMPARE <compare>] [CASE <case>] [ORDER <order>])
```

+ `COMPARE` 排序方法，参数`<compare>`有以下几种选择：
  + `SIRING` 按照字母顺序排序，默认
  + `FILE_BASENAME` 如果是一系列路径，使用`basename`排序
  + `NATURAL` 自然数排序
+ `CASE` 大小写敏感，参数`<case>`有以下几种选项：
  + `SENSITIVE` 大小写敏感排序，默认
  + `INSENSITIVE` 大小写不敏感排序
+ `ORDER` 排序顺序，参数`<order>`由以下集中选择：
  + `ASCENDING` 升序，默认
  + `DESCENDING` 降序



## CMake的宏定义

```cmake
#include <stdio.h>
#define NUMBER 3

int main(){
	int a = 10;
#ifdef DEBUG
	printf("我是一个程序猿，只会code，不会爬树\n");
#endif
	for(int i = 0; i < NUMBER; ++i){
		printf("hello,GCC\n");
	}

	return 0;
}
```

我们可以通过宏定义的方式，来控制这段代码的分支语句

> 当DEBUG被定义的时候，我们就会执行`#ifdef`，相反的我们不执行

```cmake
# CMakeLists.txt 中定义 DEBUG
# add_definitions(-D宏名称)
add_definitions(-DDEBUG)
# 此时DEBUG被定义，所以执行#ifdef
```





## CMake高级使用

### 子CMakeLists.txt

```cmd
.
├── CMakeLists.txt
├── calc
│   ├── CMakeLists.txt
│   ├── add.cpp
│   ├── div.cpp
│   ├── mult.cpp
│   └── sub.cpp
├── include
│   ├── calc.h
│   └── sort.h
├── sort
│   ├── CMakeLists.txt
│   ├── insert.cpp
│   └── select.cpp
├── test1
│   ├── CMakeLists.txt
│   └── calc.cpp
└── test2
    ├── CMakeLists.txt
    └── sort.cpp
```

+ 父节点中定义的变量，在它的子节点中可以直接使用

+ 父节点添加命令，帮助父节点可以找到子节点的目录
  ```cmake
  add_subdirectory(source_dir)
  ```

  + `source_dic` 表示子节点所在目录

### 举个例子

#### 顶层CMakeLists.txt

```cmake
# 顶层CMakeLists.txt
cmake_minimum_required(VERSION 3.15)
project(test)

# 定义变量
# 静态库生成的路径
set(LIBPATH ${PROJECT_SOURCE_DIR}/lib)
# 可执行程序的存储目录
set(EXECPATH ${PROJECT_SOURCE_DIR}/bin)
# 头文件路径
set(HEADPATH ${PROJECT_SOURCE_DIR}/include)
# 库文件的名字
set(CALLIB calc)
set(SORTLIB sort)
# 可执行程序的名字
set(APPNAME1 app1)
set(APPNAME2 app2)

# 给当前节点添加子目录
add_subdirectory(calc)
add_subdirectory(sort)
add_subdirectory(test1)
add_subdirectory(test2)
```

#### calc 目录下CMakeLists.txt

```cmake
cmake_minimum_required(VERSION 3.15)
project(calc)

# 搜索源文件
aux_source_directory(./ SRC)
include_directories(${HEADPATH})
set(LIBRARY_OUTPUT_PATH ${LIBPATH})
add_library(${CALCLIB} STATIC ${SRC})
```



#### sort 目录下CMakeLists.txt

```cmake
cmake_minimum_required(VERSION 3.15)
project(sort)

# 搜索源文件
aux_source_directory(./ SRC)
include_directories(${HEADPATH})
set(LIBRARY_OUTPUT_PATH ${LIBPATH})
add_library(${SORTLIB} STATIC ${SRC})
```



#### test1 目录下CMakeLists.txt

```cmake
cmake_minimum_required(VERSION 3.15)
project(test1)

aux_source_directory(./ SRC)
include_directories(${HEADPATH})
link_directories(${LIBPATH})
link_libraries(${CALCLIB})
set(EXECUTABLE_OUTPUT_PATH ${EXECPATH})
add_executable(${APPNAME1} ${SRC})
```



#### test2 目录下CMakeLists.txt

```cmake
cmake_minimum_required(VERSION 3.15)
project(test2)

aux_source_directory(./ SRC)
include_directories(${HEADPATH})
link_directories(${LIBPATH})
link_libraries(${SORTLIB})
set(EXECUTABLE_OUTPUT_PATH ${EXECPATH})
add_executable(${APPNAME2} ${SRC})
```



## 大小写如何区分

```markdown
1. 命令和函数名 不区分
2. 宏名称 		不区分
3. 关键字和选项 不区分
4. 参数和路径   区分
5. 变量名称	    区分
```

```markdown
1. CMAKE_CURRENT_SOURCE_DIR 是CMake的一个内置变量，而不是宏
2. STATIC和static等效，关键字和选项不区分	
```

