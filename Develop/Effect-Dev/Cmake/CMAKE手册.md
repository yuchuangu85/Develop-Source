# CMAKE手册

## CMake 用法导览

Preface : 本文是CMake官方文档CMake Tutorial ([http://www.cmake.org/cmake/help/cmake_tutorial.html](http://www.cmake.org/cmake/help/cmake_tutorial.html)) 的翻译。通过一个样例工程从简单到复杂的完善过程，文档介绍了CMake主要模块（cmake, ctest, cpack）的功能和使用环境；从中可以一窥cmake的大体形貌。正文如下：

本文下述内容是一个手把手的使用指南；它涵盖了CMake需要解决的公共构建系统的一些问题。这些主题中的许多主题已经在`Mastering CMake一书中以单独的章节被介绍过，但是通过一个样例工程看一看它们如何工作也是非常有帮助的。本指南可以在CMake源码树的Tests/Tutorial路径下找到。每一步都有它自己的子路径，其中包含该步骤的一个完整的指南。

### 1.作为基础的起始点（步骤1）

最基本的工程是一个从源代码文件中构建可执行文件的例子。对于简单工程，只要一个两行的CMakeLists文件就足够了。这将会作为我们指南的起点。这份CMakeLists文件看起来像是这样：

```
cmake_minimum_required (VERSION 2.6)
project (Tutorial)
add_executable(Tutorial tutorial.cxx)
```

注意到这个例子在CMakeLists文件中使用了小写。CMake支持大写、小写、混合大小写的命令。tutorial.cxx中的源代码用来计算一个数的平方根，并且它的第一版非常简单，如下所示：

```
// A simple program that computes the square root of a number
// 计算一个数的平方根的简单程序
#include <stdio.h>
#include <stdlib.h>
#include <math.h>
int main (int argc, char *argv[]) {
  if (argc < 2) {
    fprintf(stdout,"Usage: %s number\n",argv[0]);
    return 1;
  }
  double inputValue = atof(argv[1]);
  double outputValue = sqrt(inputValue);
  fprintf(stdout,"The square root of %g is %g\n", inputValue, outputValue);
  return 0;
}
```

我们添加的第一个特性用来为工程和可执行文件指定一个版本号。虽然你可以在源代码中唯一指定它，但是你在CMakeLists文件中指定它可以提供更好的灵活性。如下所示，我么可以通过添加一个版本号来修改CMakeLists文件：

```
cmake_minimum_required (VERSION 2.6)
project (Tutorial)
# 版本号
set (Tutorial_VERSION_MAJOR 1)
set (Tutorial_VERSION_MINOR 0)
# 配置一个头文件，通过它向源代码中传递一些CMake设置。
configure_file (
  "${PROJECT_SOURCE_DIR}/TutorialConfig.h.in"
  "${PROJECT_BINARY_DIR}/TutorialConfig.h"
  )
# 将二进制文件树添加到包含文件的搜索路径中，这样我们可以找到TutorialConfig.h
include_directories("${PROJECT_BINARY_DIR}")
# 添加可执行文件
add_executable(Tutorial tutorial.cxx)
```

由于配置过的文件将会被写到二进制文件目录下，我们必须把该目录添加到包含文件的搜索路径清单中。然后，以下的代码就可以在源目录下创建一份TotorialConfig.h.in文件：

```
// 与tutorial相关的配置好的选项与设置；
#define Tutorial_VERSION_MAJOR @Tutorial_VERSION_MAJOR@
#define Tutorial_VERSION_MINOR @Tutorial_VERSION_MINOR@
```

当CMake配置这份头文件时，@Tutorial_VERSION_MAJOR@和@Tutorial_VERSION_MINOR@的值将会被从CMakeLists文件中传递过来的值替代。下一步，我们要修改tutorial.cxx来包含configured头文件然后使用其中的版本号。修改过的源代码展列于下：

```
// 计算平方根的简单程序。
#include <stdio.h>
#include <stdlib.h>
#include <math.h>
#include "TutorialConfig.h"
int main (int argc, char *argv[]) {
  if (argc < 2) {
    fprintf(stdout,"%s Version %d.%d\n",
            argv[0],
            Tutorial_VERSION_MAJOR,
            Tutorial_VERSION_MINOR);
    fprintf(stdout,"Usage: %s number\n",argv[0]);
    return 1;
  }
  double inputValue = atof(argv[1]);
  double outputValue = sqrt(inputValue);
  fprintf(stdout,"The square root of %g is %g\n", inputValue, outputValue);
  return 0;
}
```

### 2.引入库（步骤2）

现在我们将会在我们的工程中引入一个库。这个库会包含我们自己实现的计算一个数的平方根的函数。可执行文件随后可以使用这个库文件而不是编译器提供的标准开平方函数。在本指南中，我们将会把库文件放到一个子目录MathFunctions中。它包含下述的单行CMakeLists文件：

```
add_library(MathFunctions mysqrt.cxx)
```

源文件mysqrt.cxx有一个叫做mysqrt的函数，它提供了与编译器的sqrt函数类似的功能。为了使用新的库，我们在顶层的CMakeLists中增加一个add_subdirectory调用，这样这个库也会被构建。我们也要向可执行文件中增加另一个头文件路径，这样就可以从MathFunctions/mysqrt.h头文件中找到函数的原型。最后的一点更改是在向可执行文件中引入新的库。顶层CMakeLists文件的最后几行现在看起来像是这样：

```
include_directories ("${PROJECT_SOURCE_DIR}/MathFunctions")
add_subdirectory (MathFunctions) 
# 引入可执行文件
add_executable (Tutorial tutorial.cxx)
target_link_libraries (Tutorial MathFunctions)
```

现在，让我们考虑下让MathFunctions库变为可选的。在本指南中，确实没有必要这样画蛇添足；但是对于更大型的库或者依赖于第三方代码的库，你可能需要这种可选择性。第一步是为顶层的CMakeLists文件添加一个选项：

```
# 我们应该使用我们自己的数学函数吗？
option (USE_MYMATH "Use tutorial provided math implementation" ON) 
```

这将会在CMake的GUI中显示一个默认的ON值，并且用户可以随需改变这个设置。这个设置会被存储在cache中，那么用户将不需要在cmake该工程时，每次都设置这个选项。第二处改变是，让链接MathFunctions库变为可选的。要实现这一点，我们修改顶层CMakeLists文件的结尾部分：

```
# 添加MathFunctions库吗？
if (USE_MYMATH)
  include_directories ("${PROJECT_SOURCE_DIR}/MathFunctions")
  add_subdirectory (MathFunctions)
  set (EXTRA_LIBS ${EXTRA_LIBS} MathFunctions)
endif (USE_MYMATH)
# 添加可执行文件
add_executable (Tutorial tutorial.cxx)
target_link_libraries (Tutorial  ${EXTRA_LIBS})
```

这里用USE_MYMATH设置来决定是否MathFunctions应该被编译和执行。注意到，要用一个变量（在这里是EXTRA_LIBS）来收集所有以后会被连接到可执行文件中的可选的库。这是保持带有许多可选部件的较大型工程干净清爽的一种通用的方法。源代码对应的改变相当直白，如下所示：

```
// 计算一个数平方根的简单程序
#include <stdio.h>
#include <stdlib.h>
#include <math.h>
#include "TutorialConfig.h"
#ifdef USE_MYMATH
#include "MathFunctions.h"
#endif
int main (int argc, char *argv[]) {
    if (argc < 2) {
        fprintf(stdout,"%s Version %d.%d\n", argv[0],
            Tutorial_VERSION_MAJOR,
            Tutorial_VERSION_MINOR);
        fprintf(stdout,"Usage: %s number\n",argv[0]);
        return 1;
    }
    double inputValue = atof(argv[1]);
#ifdef USE_MYMATH
    double outputValue = mysqrt(inputValue);
#else
    double outputValue = sqrt(inputValue);
#endif
    fprintf(stdout,"The square root of %g is %g\n", inputValue, outputValue);
    return 0;
}
```

在源代码中，我们也使用了USE_MYMATH。这个宏是由CMake通过TutorialConfig.h.in配置文件中的下述语句行提供给源代码的：

```
#cmakedefine USE_MYMATH
```

### 3.安装与测试（步骤3）

下一步我们会为我们的工程引入安装规则以及测试支持。安装规则相当直白，对于MathFunctions库，我们通过向MathFunctions的CMakeLists文件添加如下两条语句来设置要安装的库以及头文件：

```
install (TARGETS MathFunctions DESTINATION bin)
install (FILES MathFunctions.h DESTINATION include)
```

对于应用程序，在顶层CMakeLists文件中添加下面几行，它们用来安装可执行文件以及配置头文件：

```
# 添加安装目标
install (TARGETS Tutorial DESTINATION bin)
install (FILES "${PROJECT_BINARY_DIR}/TutorialConfig.h" 
DESTINATION include)
```

这就是要做的全部；现在你应该可以构建tutorial工程了。然后，敲入命令make install（或者从IDE中构建INSTALL目标）然后它就会安装需要的头文件，库以及可执行文件CMake的变量CMAKE_INSTALL_PREFIX用来确定这些文件被安装的根目录。添加测试同样也只需要相当浅显的过程。在顶层CMakeLists文件的的尾部补充许多基本的测试代码来确认应用程序可以正确工作。

```
# 应用程序是否运行?
add_test (TutorialRuns Tutorial 25)
# 它是否对25做了开平方运算
add_test (TutorialComp25 Tutorial 25)
set_tests_properties (TutorialComp25 
  PROPERTIES PASS_REGULAR_EXPRESSION "25 is 5")
# 它是否能处理是负数作为输入的情况
add_test (TutorialNegative Tutorial -25)
set_tests_properties (TutorialNegative
  PROPERTIES PASS_REGULAR_EXPRESSION "-25 is 0")
# 它是否可以处理较小的数字。
add_test (TutorialSmall Tutorial 0.0001)
set_tests_properties (TutorialSmall
  PROPERTIES PASS_REGULAR_EXPRESSION "0.0001 is 0.01")
# 用法信息是否可用？
add_test (TutorialUsage Tutorial)
set_tests_properties (TutorialUsage
  PROPERTIES 
  PASS_REGULAR_EXPRESSION "Usage:.*number")
```
  
第一个测试用例仅仅用来验证程序可以运行，没有出现段错误或其他的崩溃，并且返回值必须是0。这是CTest所做测试的基本格式。余下的几个测试都是用PASS_REGULAR_EXPRESSION 测试属性来验证测试代码的输出是否包含有特定的字符串。在本例中，测试样例用来验证计算得出的平方根与预定值一样；当指定错误的输入数据时，要打印用法信息。如果你想要添加许多测试不同输入值的样例，你应该考虑创建如下所示的宏：

```
#定义一个宏来简化添加测试的过程，然后使用它
macro (do_test arg result)
  add_test (TutorialComp${arg} Tutorial ${arg})
  set_tests_properties (TutorialComp${arg}
    PROPERTIES PASS_REGULAR_EXPRESSION ${result})
endmacro (do_test)
# 做一系列基于结果的测试
do_test (25 "25 is 5")
do_test (-25 "-25 is 0")
```

对于每个do_test宏调用，都会向工程中添加一个新的测试用例；宏参数是测试名、函数的输入以及期望结果。

### 4.增加系统内省（步骤4）

下一步，让我们考虑向我们的工程中引入一些依赖于目标平台上可能不具备的特性的代码。在本例中，我们会增加一些依赖于目标平台是否有log或exp函数的代码。当然，几乎每个平台都有这些函数；但是对于tutorial工程，我们假设它们并非如此普遍。如果该平台有log函数，那么我们会在mysqrt函数中使用它去计算平方根。我们首先在顶层CMakeLists文件中使用宏CheckFunctionExists.cmake测试这些函数的可用性：

```
# 该系统提供log和exp函数吗？
include (CheckFunctionExists.cmake)
check_function_exists (log HAVE_LOG)
check_function_exists (exp HAVE_EXP)
```

下一步，如果CMake在对应平台上找到了它们，我们修改TutorialConfig.h.in来定义这些值；如下：

```
// 该平台提供exp和log函数吗？
#cmakedefine HAVE_LOG
#cmakedefine HAVE_EXP
```

这些log和exp函数的测试要在TutorialConfig.h的configure_file命令之前被处理，这一点很重要。最后，在mysqrt函数中，如果log和exp在当前系统上可用的话，我们可以提供一个基于它们的可选的实现：

```
// 如果我们有log和exp两个函数，那么使用它们
#if defined (HAVE_LOG) && defined (HAVE_EXP)
    result = exp(log(x)*0.5);
#else // 否则使用替代方法
```

### 5.添加一个生成文件以及生成器（步骤5）

在本节，我们会展示你应该怎样向一个应用程序的构建过程中添加一个生成的源文件。在本范例中，我们会创建一个预先计算出的平方根表作为构建过程的一部分。MathFunctions子路径下，一个新的MakeTable.cxx源文件来做这件事。

```
// 一个简单的用于构建平方根表的程序
#include <stdio.h>
#include <stdlib.h>
#include <math.h>
int main (int argc, char *argv[]) {
    int i;
    double result;
    // 确保有足够多的参数
    if (argc < 2) {
        return 1;
    }
    // 打开输出文件
    FILE *fout = fopen(argv[1],"w");
    if (!fout) {
        return 1;
    }
    // 创建一个带有平方根表的源文件
    fprintf(fout,"double sqrtTable[] = {\n");
    for (i = 0; i < 10; ++i) {
        result = sqrt(static_cast<double>(i));
        fprintf(fout,"%g,\n",result);
    }
    // 该表以0结尾
    fprintf(fout,"0};\n");
    fclose(fout);
    return 0;
}
```

注意到这个表是由合法的C++代码生成的，并且被写入的输出文件的名字是作为一个参数输入的。下一步是将合适的命令添加到MathFunction的CMakeLists文件中，来构建MakeTable可执行文件，然后运行它，作为构建过程的一部分。完成这几步，需要少数的几个命令，如下所示：

```
# 首先，我们添加生成该表的可执行文件
add_executable(MakeTable MakeTable.cxx) 
# 然后添加该命令来生成源文件
add_custom_command (
  OUTPUT ${CMAKE_CURRENT_BINARY_DIR}/Table.h
  COMMAND MakeTable ${CMAKE_CURRENT_BINARY_DIR}/Table.h
  DEPENDS MakeTable
  )
# 为包含文件，向搜索路径中添加二进制树路径
include_directories( ${CMAKE_CURRENT_BINARY_DIR} )
# 添加main库
add_library(MathFunctions mysqrt.cxx ${CMAKE_CURRENT_BINARY_DIR}/Table.h  )
```

首先，MakeTable的可执行文件也和其他被加入的文件一样被加入。然后，我们添加一个自定义命令来指定如何通过运行MakeTable来生成Table.h。这是通过将生成Table.h增加到MathFunctions库的源文件列表中来实现的。我们还必须增加当前的二进制路径到包含路径的清单中，这样Table.h可以被找到并且可以被mysqrt.cxx所包含。当该工程被构建后，它首先会构建MakeTable可执行文件。然后它会运行MakeTable来生成Table.h文件。最后，它会编译mysqrt.cxx（其中包含Table.h）来生成MathFunctions库。到目前为止，拥有我们添加的完整特性的顶层CMakeLists文件看起来像是这样：

```
cmake_minimum_required (VERSION 2.6)
project (Tutorial)
# 版本号
set (Tutorial_VERSION_MAJOR 1)
set (Tutorial_VERSION_MINOR 0)
# 本系统是否提供log和exp函数?
include (${CMAKE_ROOT}/Modules/CheckFunctionExists.cmake)
check_function_exists (log HAVE_LOG)
check_function_exists (exp HAVE_EXP)
# 我们应该使用自己的math函数吗?
option(USE_MYMATH 
  "Use tutorial provided math implementation" ON)
# 配置一个头文件来向源代码传递一些CMake设置。
configure_file (
  "${PROJECT_SOURCE_DIR}/TutorialConfig.h.in"
  "${PROJECT_BINARY_DIR}/TutorialConfig.h"
  )
# 为包含文件的搜索路径添加二进制树，这样才能发现TutorialConfig.h头文件。
include_directories ("${PROJECT_BINARY_DIR}")
# 添加MathFunctions库吗?
if (USE_MYMATH)
  include_directories ("${PROJECT_SOURCE_DIR}/MathFunctions")
  add_subdirectory (MathFunctions)
  set (EXTRA_LIBS ${EXTRA_LIBS} MathFunctions)
endif (USE_MYMATH)
# 添加可执行文件
add_executable (Tutorial tutorial.cxx)
target_link_libraries (Tutorial  ${EXTRA_LIBS})
# 添加安装的目标
install (TARGETS Tutorial DESTINATION bin)
install (FILES "${PROJECT_BINARY_DIR}/TutorialConfig.h"        
         DESTINATION include)
# 测试1 ：应用程序可以运行吗?
add_test (TutorialRuns Tutorial 25)
# 测试2 ： 使用信息可用吗?
add_test (TutorialUsage Tutorial)
set_tests_properties (TutorialUsage
  PROPERTIES 
  PASS_REGULAR_EXPRESSION "Usage:.*number"
  )
# 定义一个可以简化引入测试过程的宏
macro (do_test arg result)
  add_test (TutorialComp${arg} Tutorial ${arg})
  set_tests_properties (TutorialComp${arg}
    PROPERTIES PASS_REGULAR_EXPRESSION ${result}
    )
endmacro (do_test)
# do a bunch of result based tests
# 执行一系列基于结果的测试
do_test (4 "4 is 2")
do_test (9 "9 is 3")
do_test (5 "5 is 2.236")
do_test (7 "7 is 2.645")
do_test (25 "25 is 5")
do_test (-25 "-25 is 0")
do_test (0.0001 "0.0001 is 0.01")
```

TutorialConfig.h文件看起来像是这样：

```
// Tutorial的配置选项与设置如下
#define Tutorial_VERSION_MAJOR @Tutorial_VERSION_MAJOR@
#define Tutorial_VERSION_MINOR @Tutorial_VERSION_MINOR@
#cmakedefine USE_MYMATH
// 该平台提供exp和log函数吗?
#cmakedefine HAVE_LOG
#cmakedefine HAVE_EXP
```

然后，MathFunctions工程的CMakeLists文件看起来像是这样：

```
# 首先，我们添加生成这个表的可执行文件
add_executable(MakeTable MakeTable.cxx)
# 添加生成源代码的命令
add_custom_command (
  OUTPUT ${CMAKE_CURRENT_BINARY_DIR}/Table.h
  DEPENDS MakeTable
  COMMAND MakeTable ${CMAKE_CURRENT_BINARY_DIR}/Table.h
  )
# 为包含文件向搜索路径中添加二进制树目录
include_directories( ${CMAKE_CURRENT_BINARY_DIR} )
# 添加main库
add_library(MathFunctions mysqrt.cxx ${CMAKE_CURRENT_BINARY_DIR}/Table.h) 
install (TARGETS MathFunctions DESTINATION bin)
install (FILES MathFunctions.h DESTINATION include)
```

### 6.构建一个安装器（步骤6）

下一步假设我们想要向其他人分发我们的工程，这样他们就可以使用它。我们想同时提供在许多不同平台上的源代码和二进制文档发行版。这与之前我们在“安装与测试（步骤3）”做过的安装有一点不同，那里我们仅仅安装我们从源码中构建出来的二进制文件。在本例子中，我们会构建支持二进制安装以及类似于cygwin，debian，RPM等具有包管理特性的安装包。为了完成这个目标，我们会使用CPack来创建Packaging with CPack一章中描述的特定平台的安装器。

```
# 构建一个CPack驱动的安装包
include (InstallRequiredSystemLibraries)
set (CPACK_RESOURCE_FILE_LICENSE
     "${CMAKE_CURRENT_SOURCE_DIR}/License.txt")
set (CPACK_PACKAGE_VERSION_MAJOR "${Tutorial_VERSION_MAJOR}")
set (CPACK_PACKAGE_VERSION_MINOR "${Tutorial_VERSION_MINOR}")
include (CPack)
```

需要做的全部事情就这些。我们以包含InstallRequiredSystemLibraries开始。这个模块将会包含许多在当前平台上，当前工程需要的运行时库。第一步我们将一些CPack变量设置为保存本工程的许可证和版本信息的位置。版本信息使用了我们在本指南中先前设置的变量。最后，我们要包含CPack模块，它会使用这些变量以及你所处的系统的一些别的属性，然后来设置一个安装器。下一步是以通常的方式构建该工程然后随后运行CPack。如果要构建一个二进制发行包，你应该运行：

```
cpack -C CPackConfig.cmake
```

为了创建一个源代码发行版，你应该键入：

```
cpack -C CPackSourceConfig.cmake
```

### 7.增加对Dashboard的支持（步骤7）

增加对向一个dashboard提交我们的测试结果的功能的支持非常简单。我们在本指南的先前步骤中已经定义了我们工程中的许多测试样例。我们仅仅需要运行这些测试样例然后将它们提交到dashboard即可。为了包含对dashboards的支持，我们需要在顶层CMakeLists文件中包含CTest模块。

```
# 支持dashboard脚本
include (CTest)
```

我们也可以创建一个CTestConfig.cmake文件，在其中来指定该dashboard的工程名。

```
set (CTEST_PROJECT_NAME "Tutorial")
```

CTest 将会在运行期间读取这个文件。为了创建一个简单的dashboard，你可以在你的工程下运行CMake，然后切换到二进制树，然后运行ctest -DExperimental. 你的dashboard将会被更新到Kitware的公共dashboard.

## cmake指令

```
  公司的一个项目使用CMake作为跨平台构建工具；业务有需求，当然要好好研读一下官方的技术手册。目前的计划是先把官方手册翻译一下，了解清楚CMake中的各种命令、属性和变量的用法。同时在工作中也会阅读CMake的真实源码，后续会基于此陆续写一些工程中使用CMake的心得。CMake的版本也在不停更新，有些新的命令和变量会随着版本更新添加进来，这是后事了，暂且不管；现在锁定CMake 2.8.3作为手册翻译的版本。
```

### 命令名称

```
cmake - 跨平台Makefile生成工具。
```

### 用法

```
　　cmake [选项] <源码路径>
　　cmake [选项] <现有构建路径>
```

### 描述

cmake可执行程序是CMake的命令行界面。它可以用脚本对工程进行配置。工程配置设置可以在命令行中使用-D选项指定。使用-i选项，cmake将通过提示交互式地完成该设置。

CMake是一个跨平台的构建系统生成工具。它使用平台无关的CMake清单文件CMakeLists.txt，指定工程的构建过程；源码树的每个路径下都有这个文件。CMake产生一个适用于具体平台的构建系统，用户使用这个系统构建自己的工程。

### 选项
---

```
-C <initial-cache>: 预加载一个脚本填充缓存文件。
　　当cmake在一个空的构建树上第一次运行时，它会创建一个CMakeCache.txt文件，然后向其中写入可定制的项目设置数据。-C选项可以用来指定一个文件，在第一次解析这个工程的cmake清单文件时，从这个文件加载缓存的条目(cache entries)信息。被加载的缓存条目比项目默认的值有更高的优先权。参数中给定的那个文件应该是一个CMake脚本，其中包含有使用CACHE选项的SET命令；而不是一个缓存格式的文件。
-D <var>:<type>=<value>: 创建一个CMake的缓存条目。
　　当cmake第一次运行于一个空的构建数时，它会创建一个CMakeCache.txt文件，并且使用可定制的工程设置来填充这个文件。这个选项可以用来指定优先级高于工程的默认值的工程设置值。这个参数可以被重复多次，用来填充所需要数量的缓存条目(cache entries)。
-U <globbing_expr>: 从CMake的缓存文件中删除一条匹配的条目。
　　该选项可以用来删除CMakeCache.txt文件中的一或多个变量。文件名匹配表达式(globbing expression)支持通配符*和?的使用。该选项可以重复多次以删除期望数量的缓存条目。使用它时要小心，你可能因此让自己的CMakeCache.txt罢工。
-G <generator-name>: 指定一个makefile生成工具。
　　在具体的平台上，CMake可以支持多个原生的构建系统。makefile生成工具的职责是生成特定的构建系统。可能的生成工具的名称将在生成工具一节给出。
-Wno-dev: 抑制开发者警告。
　　抑制那些为CMakeLists.txt文件的作者准备的警告信息。
-Wdev: 使能开发者警告信息输出功能。
　　允许那些为CMakeLists.txt文件的作者准备的警告信息。
-E: CMake命令行模式。
　　为了真正做到与平台无关，CMake提供了一系列可以用于所有系统上的的命令。以-E参数运行CMake会帮助你获得这些命令的用法。可以使用的命令有：chdir, copy, copy_if_different copy_directory, compare_files, echo, echo_append, environment, make_directory, md5sum, remove_directory, remove, tar, time, touch, touch_nocreate, write_regv, delete_regv, comspec, create_symlink。
-i: 以向导模式运行CMake。
　　向导模式是在没有GUI时，交互式地运行cmake的模式。cmake会弹出一系列的提示，要求用户回答关于工程配置的一行问题。这些答复会被用来设置cmake的缓存值。
-L[A][H]: 列出缓存的变量中的非高级的变量。
　　-L选项会列出缓存变量会运行CMake，并列出所有CMake的内有被标记为INTERNAL或者ADVANCED的缓存变量。这会显示当前的CMake配置信息，然后你可以用-D选项改变这些选项。修改一些变量可能会引起更多的变量被创建出来。如果指定了A选项，那么命令也会显示高级变量。如果指定了H选项，那么命令会显示每个变量的帮助信息。
　　
--build <dir>: 构建由CMake生成的工程的二进制树。（这个选项的含义我不是很清楚—译注）
    该选项用以下的选项概括了内置构建工具的命令行界面
      <dir>          = 待创建的工程二进制路径。
      --target <tgt> = 构建<tgt>，而不是默认目标。
      --config <cfg> = 对于多重配置工具，选择配置<cfg>。
      --clean-first  = 首先构建目标的clean伪目标，然后再构建。
                       （如果仅仅要clean掉，使用--target 'clean'选项。）
      --             = 向内置工具（native tools）传递剩余的选项。
    运行不带选项的cmake --build来获取快速帮助信息。
-N: 查看模式。
    仅仅加载缓存信息，并不实际运行配置和生成步骤。
-P <file>: 处理脚本模式。
    将给定的cmake文件按照CMake语言编写的脚本进行处理。不执行配置和生成步骤，不修改缓存信息。如果要使用-D选项定义变量，-D选项必须在-P选项之前。
--graphviz=[file]: 生成依赖的graphviz图。
    生成一个graphviz软件的输入文件，其中包括了项目中所有库和可执行文件之间的依赖关系。
--system-information [file]: 输出与该系统相关的信息。
    输出范围比较广的、与当前使用的系统有关的信息。如果在一个CMake工程的二进制构建树的顶端运行该命令，它还会打印一些附加信息，例如缓存，日志文件等等。
--debug-trycompile: 不删除“尝试编译”路径。
    不删除那些为try_compile调用生成的路径。这在调试失败的try_compile文件时比较有用。不过，因为上一次“尝试编译”生成的旧的垃圾输出文件也许会导致一次不正确通过/不通过，且该结果与上次测试的结果不同，所以该选项可能会改变“尝试编译”的结果。对于某一次“尝试编译”，该选项最好只用一次；并且仅仅在调试时使用。
--debug-output: 将cmake设置为调试模式。
    在cmake运行时，打印额外的信息；比如使用message(send_error)调用得到的栈跟踪信息。
--trace: 将cmake设置为跟踪模式。
    用message(send_error)调用，打印所有调用生成的跟踪信息，以及这些调用发生的位置。（这句话含义不是很确定—译注。）
--help-command cmd [file]: 打印单个命令cmd的帮助信息，然后退出。
    显示给定的命令的完整的文档。如果指定了[file]参数，该文档会写入该文件，其输出格式由该文件的后缀名确定。支持的文件类型有：man page，HTML，DocBook以及纯文本。
--help-command-list [file]: 列出所有可用命令的清单，然后退出。
    该选项列出的信息含有所有命令的名字；其中，每个命令的帮助信息可以使用--help-command选项后跟一个命令名字得到。如果指定了[file]参数，帮助信息会写到file中，输出格式依赖于文件名后缀。支持的文件格式包括：man page，HTML，DocBook以及纯文本。
--help-commands [file]: 打印所有命令的帮助文件，然后退出。
    显示所有当前版本的命令的完整文档。如果指定了[file]参数，帮助信息会写到file中，输出格式依赖于文件名后缀。支持的文件格式包括：man page，HTML，DocBook以及纯文本。
--help-compatcommands [file]: 打印兼容性命令（过时的命令—译注）的帮助信息。
    显示所有关于兼容性命令的完整文档。如果指定了[file]参数，帮助信息会写到file中，输出格式依赖于文件名后缀。支持的文件格式包括：man page，HTML，DocBook以及纯文本。
--help-module module [file]: 打印某单一模块的帮助信息，然后退出。
    打印关于给定模块的完整信息。如果指定了[file]参数，帮助信息会写到file中，且输出格式依赖于文件名后缀。支持的文件格式包括：man page，HTML，DocBook以及纯文本。
--help-module-list [file]: 列出所有可用模块名，然后退出。
    列出的清单包括所有模块的名字；其中，每个模块的帮助信息可以使用--help-module选项，后跟模块名的方式得到。如果指定了[file]参数，帮助信息会写到file中，且输出格式依赖于文件名后缀。支持的文件格式包括：man page，HTML，DocBook以及纯文本。
--help-modules [file]: 打印所有模块的帮助信息，然后退出。
    显示关于所有模块的完整文档。如果指定了[file]参数，帮助信息会写到file中，且输出格式依赖于文件名后缀。支持的文件格式包括：man page，HTML，DocBook以及纯文本。
--help-custom-modules [file]: 打印所有自定义模块名，然后退出。
    显示所有自定义模块的完整文档。如果指定了[file]参数，帮助信息会写到file中，且输出格式依赖于文件名后缀。支持的文件格式包括：man page，HTML，DocBook以及纯文本。
--help-policy cmp [file]: 打印单个策略的帮助信息，然后退出。
    显示给定的策略的完整文档。如果指定了[file]参数，帮助信息会写到file中，且输出格式依赖于文件名后缀。支持的文件格式包括：man page，HTML，DocBook以及纯文本。
--help-policies [file]: 打印所有策略的帮助信息，然后退出。
    显示所有策略的完整文档。如果指定了[file]参数，帮助信息会写到file中，且输出格式依赖于文件名后缀。支持的文件格式包括：man page，HTML，DocBook以及纯文本。
--help-property prop [file]: 打印单个属性的帮助信息，然后退出。
    显示指定属性的完整文档。如果指定了[file]参数，帮助信息会写到file中，且输出格式依赖于文件名后缀。支持的文件格式包括：man page，HTML，DocBook以及纯文本。
--help-property-list [file]: 列出所有可用的属性，然后退出。
    该命令列出的清单包括所有属性的名字；其中，每个属性的帮助信息都可以通过--help-property选项后跟一个属性名的方式获得。如果指定了[file]参数，帮助信息会写到file中，且输出格式依赖于文件名后缀。支持的文件格式包括：man page，HTML，DocBook以及纯文本。
--help-properties [file]: 打印所有属性的帮助信息，然后退出。
    显示所有属性的完整文档。如果指定了[file]参数，帮助信息会写到file中，且输出格式依赖于文件名后缀。支持的文件格式包括：man page，HTML，DocBook以及纯文本。
--help-variable var [file]: 打印单个变量的帮助信息，然后退出。
    显示指定变量的完整文档。如果指定了[file]参数，帮助信息会写到file中，且输出格式依赖于文件名后缀。支持的文件格式包括：man page，HTML，DocBook以及纯文本。
--help-variable-list [file]: 列出文档中有记录的变量，然后退出。
    该命令列出的清单包括所有变量的名字；其中，每个变量的帮助信息都可以通过--help-variable选项后跟一个变量名的方式获得。如果指定了[file]参数，帮助信息会写到file中，且输出格式依赖于文件名后缀。支持的文件格式包括：man page，HTML，DocBook以及纯文本。
--help-variables [file]: 打印所有变量的帮助信息，然后退出。
    显示所有变量的完整帮助文档。如果指定了[file]参数，帮助信息会写到file中，且输出格式依赖于文件名后缀。支持的文件格式包括：man page，HTML，DocBook以及纯文本。
--copyright [file]: 打印CMake的版权信息，然后退出。
    如果指定了[file]参数，版权信息会写到这个文件中。
--help: 打印用法信息，然后退出。
    用法信息描述了基本的命令行界面及其选项。
--help-full [file]: 打印完整的帮助信息，然后退出。
    显示大多数UNIX man page提供的帮助信息。该选项是为非UNIX平台提供的；但是如果man手册页没有安装，它也能提供便利。如果制定了[file]参数，帮助信息会写到这个文件中。
--help-html [file]: 以HTML格式打印完整的帮助信息，然后退出。
    CMake的作者使用该选来帮助生成web页面。如果指定了[file]参数，帮助信息会写到这个文件中。
--help-man [file]: 以UNIX的man手册页格式打印完整的帮助信息，然后退出。
    cmake使用该选生成UNIX的man手册页。如果指定了[file]参数，帮助信息会写到这个文件中。
--version [file]: 显示程序名/版本信息行，然后退出。
    如果指定了[file]参数，版本信息会写到这个文件中。
```

---

对于CMake的语言要素，比如命令，属性和变量，帮助命令选项也是很有规律的，一般是用--help-xxx-list查看所有值的名字，找出感兴趣的项，然后用--help-xxx name查看该名字的详细信息；也可以用--help-xxxs获得相关语言要素的完整帮助信息。

### 生成器

在CMake 2.8.3平台上，CMake支持下列生成器：

---

* Borland Makefiles: 生成Borland makefile。

* MSYS Makefiles: 生成MSYS makefile。 
    生成的makefile用use /bin/sh作为它的shell。在运行CMake的机器上需要安装msys。

* MinGW Makefiles: 生成供mingw32-make使用的make file。 
    生成的makefile使用cmd.exe作为它的shell。生成它们不需要msys或者unix shell。

* NMake Makefiles: 生成NMake makefile。

* NMake Makefiles JOM: 生成JOM makefile。

* Unix Makefiles: 生成标准的UNIX makefile。 
    在构建树上生成分层的UNIX makefile。任何标准的UNIX风格的make程序都可以通过默认的make目标构建工程。生成的makefile也提供了install目标。

* Visual Studio 10: 生成Visual Studio 10 工程文件。

* Visual Studio 10 Win64: 生成Visual Studio 10 Win64 工程文件。

* Visual Studio 6: 生成Visual Studio 6 工程文件。

* Visual Studio 7: 生成Visual Studio .NET 2002 工程文件。

* Visual Studio 7 .NET 2003: 生成Visual Studio .NET 2003工程文件。

* Visual Studio 8 2005: 生成Visual Studio .NET 2005 工程文件。

* Visual Studio 8 2005 Win64: 生成Visual Studio .NET 2005 Win64工程文件。

* Visual Studio 9 2008: 生成Visual Studio 9 2008 工程文件。

* Visual Studio 9 2008 Win64: 生成Visual Studio 9 2008 Win64工程文件。

* Watcom WMake: 生成Watcom WMake makefiles。

* CodeBlocks - MinGW Makefiles: 生成CodeBlock工程文件。 
    在顶层目录以及每层子目录下为CodeBlocks生成工程文件，生成的CMakeList.txt的特点是都包含一个PROJECT()调用。除此之外还会在构建树上生成一套层次性的makefile。通过默认的make目标，正确的make程序可以构建这个工程。makefile还提供了install目标。

* CodeBlocks - NMake Makefiles: 生成CodeBlocks工程文件。 
    在顶层目录以及每层子目录下为CodeBlocks生成工程文件，生成的CMakeList.txt的特点是都包含一个PROJECT()调用。除此之外还会在构建树上生成一套层次性的makefile。通过默认的make目标，正确的make程序可以构建这个工程。makefile还提供了install目标。

* CodeBlocks - Unix Makefiles: 生成CodeBlocks工程文件。 
    在顶层目录以及每层子目录下为CodeBlocks生成工程文件，生成的CMakeList.txt的特点是都包含一个PROJECT()调用。除此之外还会在构建树上生成一套层次性的makefile。通过默认的make目标，正确的make程序可以构建这个工程。makefile还提供了install目标。

* Eclipse CDT4 - MinGW Makefiles: 生成Eclipse CDT 4.0 工程文件。 
    在顶层目录下为Eclipse生成工程文件。在运行源码外构建时，一个连接到顶层源码路径的资源文件会被创建。除此之外还会在构建树上生成一套层次性的makefile。通过默认的make目标，正确的make程序可以构建这个工程。makefile还提供了install目标。

* Eclipse CDT4 - NMake Makefiles: 生成Eclipse CDT 4.0 工程文件。 
    在顶层目录下为Eclipse生成工程文件。在运行源码外构建时，一个连接到顶层源码路径的资源文件会被创建。除此之外还会在构建树上生成一套层次性的makefile。通过默认的make目标，正确的make程序可以构建这个工程。makefile还提供了install目标。

* Eclipse CDT4 - Unix Makefiles: 生成Eclipse CDT 4.0 工程文件。 
    在顶层目录下为Eclipse生成工程文件。在运行源码外构建时，一个连接到顶层源码路径的资源文件会被创建。除此之外还会在构建树上生成一套层次性的makefile。通过默认的make目标，正确的make程序可以构建这个工程。makefile还提供了install目标。
    
---

## CMake函数

CMake手册的客套话总算说完了，开始进入正题。第一部分是CMake命令。命令就相当于命令行下操作系统提供的各种命令，重要性不言而喻；可以说，这些命令是CMake构建系统的骨架。CMake 2.8.3共有80条命令，分别是：add_custom_command, add_custom_target, add_definitions, add_dependencies, add_executable, add_library, add_subdirectory, add_test, aux_source_directory, break, build_command, cmake_minimum_required, cmake_policy, configure_file, create_test_sourcelist, define_property, else, elseif, enable_language, enable_testing, endforeach, endfunction, endif, endmacro, endwhile, execute_process, export, file, find_file, find_library, find_package, find_path, find_program, fltk_wrap_ui, foreach, function, get_cmake_property, get_directory_property, get_filename_component, get_property, get_source_file_property, get_target_property, get_test_property, if, include, include_directories, include_external_msproject, include_regular_expression, install, link_directories, list, load_cache, load_command, macro, mark_as_advanced, math, message, option, output_required_files, project, qt_wrap_cpp, qt_wrap_ui, remove_definitions, return, separate_arguments, set, set_directory_properties, set_property, set_source_files_properties, set_target_properties, set_tests_properties, site_name, source_group, string, target_link_libraries, try_compile, try_run, unset, variable_watch, while。这些命令在手册中是字典序排列的；为了便于查找，翻译也按照字典序来组织。但是在翻译结束后，会对命令进行小结，与大家讨论一下这些命令的使用方法和使用时机。

## CMD#1 add_custom_command

为生成的构建系统添加一条自定义的构建规则。 
add_custom_command命令有两种主要的功能；第一种是为了生成输出文件，添加一条自定义命令。

```
  add_custom_command(OUTPUT output1 [output2 ...]
                     COMMAND command1 [ARGS] [args1...]
                     [COMMAND command2 [ARGS] [args2...] ...]
                     [MAIN_DEPENDENCY depend]
                     [DEPENDS [depends...]]
                     [IMPLICIT_DEPENDS <lang1> depend1 ...]
                     [WORKING_DIRECTORY dir]
                     [COMMENT comment] [VERBATIM] [APPEND])
```

　　这种命令格式定义了一条生成指定的文件（文件组）的生成命令。在相同路径下创建的目标（CMakeLists.txt文件）——任何自定义命令的输出都作为它的源文件——被设置了一条规则：在构建的时候，使用指定的命令来生成这些文件。如果一个输出文件名是相对路径，它将被解释成相对于构建树路径的相对路径，并且与当前源码路径是对应的。注意，MAIN_DEPENDENCY完全是可选的，它用来向visual studio建议在何处停止自定义命令。对于各种类型的makefile而言，这条命令创建了一个格式如下的新目标：

```
  OUTPUT: MAIN_DEPENDENCY DEPENDS
　　　　COMMAND
```

　　如果指定了多于一条的命令，它们会按顺序执行。ARGS参数是可选的，它的存在是为了保持向后兼容，以后会被忽略掉。

　　第二种格式为一个目标——比如一个库文件或者可执行文件——添加一条自定义命令。这种格式可以用于目标构建前或构建后的一些操作。这条命令会成为目标的一部分，并且只有目标被构建时才会执行。如果目标已经构建了，该目标将不会执行。

```
  add_custom_command(TARGET target
                     PRE_BUILD | PRE_LINK | POST_BUILD
                     COMMAND command1 [ARGS] [args1...]
                     [COMMAND command2 [ARGS] [args2...] ...]
                     [WORKING_DIRECTORY dir]
                     [COMMENT comment] [VERBATIM])
```

　　这条命令定义了一个与指定目标的构建过程相关的新命令。新命令在何时执行，由下述的选项决定：

```
PRE_BUILD  - 在所有其它的依赖之前执行；
PRE_LINK   - 在所有其它的依赖之后执行；
POST_BUILD - 在目标被构建之后执行；
```

　　注意，只有Visual Studio 7或更高的版本才支持PRE_BUILD。对于其他的生成器，PRE_BUILD会被当做PRE_LINK来对待。

　　如果指定了WORKING_DIRECTORY选项，这条命令会在给定的路径下执行。如果设置了COMMENT选项，后跟的参数会在构建时、以构建信息的形式、在命令执行之前显示出来。如果指定了APPEND选项，COMMAND以及DEPENDS选项的值会附加到第一个输出文件的自定义命令上。在此之前，必须有一次以相同的输出文件作为参数的对该命令的调用。在当前版本下，如果指定了APPEND选项，COMMENT, WORKING_DIRECTORY和MAIN_DEPENDENCY选项会被忽略掉，不过未来有可能会用到。

　　如果指定了VERBATIM选项，所有该命令的参数将会合适地被转义，以便构建工具能够以原汁原味的参数去调用那些构建命令。注意，在add_custom_command能看到这些参数之前，CMake语言处理器会对这些参数做一层转义处理。推荐使用VERBATIM参数，因为它能够保证正确的行为。当VERBATIM未指定时，CMake的行为依赖于平台，因为CMake没有针对某一种工具的特殊字符采取保护措施。

　　如果自定义命令的输出并不是实际的磁盘文件，应该使用SET_SOURCE_FILES_PROPERTIES命令将该输出的属性标记为SYMBOLIC。

　　IMPLICIT_DEPENDS选项请求扫描一个输入文件的隐含依赖关系。给定的语言参数（文中的lang1—译注）指定了应该使用哪种编程语言的依赖扫描器。目前为止，仅支持C和CXX语言扫描器。扫描中发现的依赖文件将会在编译时添加到自定义命令中。注意，IMPLICIT_DEPENDS选项目前仅仅直至Makefile生成器，其它的生成器会忽略之。

　　如果COMMAND选项指定了一个可执行目标（由ADD_EXECUTABLE命令创建的目标），在构建时，它会自动被可执行文件的位置所替换。而且，一个目标级的依赖性将会被添加进去，这样这个可执行目标将会在所有依赖于该自定义命令的结果的目标之前被构建。不过，任何时候重编译这个可执行文件，这种特性并不会引入一个会引起自定义命令重新运行的文件级依赖。

　　DEPENDS选项指定了该命令依赖的文件。如果依赖的对象是同一目录（CMakeLists.txt文件）下另外一个自定义命令的输出，CMake会自动将其它自定义命令带到这个命令中来。如果DEPENDS指定了任何类型的目标（由ADD_*命令创建），一个目标级的依赖性将会被创建，以保证该目标在任何其它目标使用这个自定义命令的输出之前，该目标已经被创建了。而且，如果该目标是可执行文件或库文件，一个文件级依赖将会被创建，用来引发自定义命令在目标被重编译时的重新运行。

在Unix Makefile中，这条命令相当于增加了一个依赖关系和一条显式生成命令。

## CMD#2 add_custom_target

add_custom_target 添加一个目标，它没有输出；这样它就总是会被构建。

```
  add_custom_target(Name [ALL] [command1 [args1...]]
                    [COMMAND command2 [args2...] ...]
                    [DEPENDS depend depend depend ... ]
                    [WORKING_DIRECTORY dir]
                    [COMMENT comment] [VERBATIM]
                    [SOURCES src1 [src2...]])
```

　　用Name选项给定的名字添加一个目标，这个目标会引发给定的那些命令。这个目标没有输出文件，并且总是被认为是过时的，即使那些命令试图去创建一个与该目标同名的文件。使用ADD_CUSTOM_COMMAND命令可以生成一个带有依赖性的文件。默认情况下，没有目标会依赖于自定义目标。使用ADD_DEPENDENCIES命令可以添加依赖于该目标或者被该目标依赖的目标。如果指定了ALL选项，这表明这个目标应该被添加到默认的构建目标中，这样它每次都会被构建（命令的名字不能是ALL）。命令和选项是可选的；如果它们没有被指定，将会产生一个空目标。如果设定了WORKING_DIRECTORY参数，该命令会在它指定的路径下执行。如果指定了COMMENT选项，后跟的参数将会在构件的时候，在命令执行之前，被显示出来。DEPENDS选项后面列出来的依赖目标可以引用add_custom_command命令在相同路径下（CMakeLists.txt）生成的输出和文件。

　　如果指定了VERBATIM选项，所有传递到该命令的选项将会被合适地转义；这样，该命令调用的构建工具会接收到未经改变的参数。注意，CMake语言处理器会在add_custom_target命令在看到这些参数之前对它们进行一层转义。推荐使用该参数，因为它保证了正确的行为。当未指定该参数时，转义的行为依赖于平台，因为CMake没有针对于特定工具中特殊字符的保护措施。

　　SOURCES选项指定了会被包含到自定义目标中的附加的源文件。指定的源文件将会被添加到IDE的工程文件中，方便在没有构建规则的情况下能够编辑。

## CMD#3 add_definitions

为源文件的编译添加由-D引入的define flag。

```
  add_definitions(-DFOO -DBAR ...)
```

　　在编译器的命令行上，为当前路径以及下层路径的源文件加入一些define flag。这个命令可以用来引入任何flag，但是它的原意是用来引入预处理器的定义。那些以-D或/D开头的、看起来像预处理器定义的flag，会被自动加到当前路径的COMPILE_DEFINITIONS属性中。为了后向兼容，非简单值（non-trival，指的是什么？——译注）的定义会被留在flags组（flags set）里，而不会被转换。关于在特定的域以及配置中增加预处理器的定义，参考路径、目标以及源文件的COMPILE_DEFINITIONS属性来获取更多的细节。

## CMD#4 add_dependencies

为顶层目标引入一个依赖关系。

```
  add_dependencies(target-name depend-target1
                   depend-target2 ...)
```

　　让一个顶层目标依赖于其他的顶层目标。一个顶层目标是由命令ADD_EXECUTABLE，ADD_LIBRARY，或者ADD_CUSTOM_TARGET产生的目标。为这些命令的输出引入依赖性可以保证某个目标在其他的目标之前被构建。查看ADD_CUSTOM_TARGET和ADD_CUSTOM_COMMAND命令的DEPENDS选项，可以了解如何根据自定义规则引入文件级的依赖性。查看SET_SOURCE_FILES_PROPERTIES命令的OBJECT_DEPENDS选项，可以了解如何为目标文件引入文件级的依赖性。

## CMD#5 add_executable

使用给定的源文件，为工程引入一个可执行文件。

```
  add_executable(<name> [WIN32] [MACOSX_BUNDLE]
                 [EXCLUDE_FROM_ALL]
                 source1 source2 ... sourceN)
```

　　引入一个名为<name>的可执行目标，该目标会由调用该命令时在源文件列表中指定的源文件来构建。<name>对应于逻辑目标名字，并且在工程范围内必须是全局唯一的。被构建的可执行目标的实际文件名将根据具体的本地平台创建出来（比如<name>.exe或者仅仅是<name>）。

　　默认情况下，可执行文件将会在构建树的路径下被创建，对应于该命令被调用的源文件树的路径。如果要改变这个位置，查看RUNTIME_OUTPUT_DIRECTORY目标属性的相关文档。如果要改变最终文件名的<name>部分，查看OUTPUT_NAME目标属性的相关文档。

　　如果指定了MACOSX_BUNDLE选项，对应的属性会附加在创建的目标上。查看MACOSX_BUNDLE目标属性的文档可以找到更多的细节。

　　如果指定了EXCLUDE_FROM_ALL选项，对应的属性将会设置在被创建的目标上。查看EXCLUDE_FROM_ALL目标属性的文档可以找到更多的细节。

　　使用下述格式，add_executable命令也可以用来创建导入的（IMPORTED）可执行目标：

```
　　add_executable(<name> IMPORTED)
```

　　一个导入的可执行目标引用了一个位于工程之外的可执行文件。该格式不会生成构建这个目标的规则。该目标名字的作用域在它被创建的路径以及底层路径有效。它可以像在该工程内的其他任意目标一样被引用。导入可执行文件为类似于add_custom_command之类的命令引用它提供了便利。

　　关于导入的可执行文件的细节可以通过设置以IMPORTED_开头的属性来指定。这类属性中最重要的是IMPORTED_LOCATION（以及它对应于具体配置的版本IMPORTED_LOCATION_<CONFIG>）；该属性指定了执行文件主文件在磁盘上的位置。查看IMPORTED_*属性的文档来获得更多信息。 
　

## CMD#6 add_library

使用指定的源文件向工程中添加一个库。

```
  add_library(<name> [STATIC | SHARED | MODULE]
              [EXCLUDE_FROM_ALL]
              source1 source2 ... sourceN)
```

　　添加一个名为<name>的库文件，该库文件将会根据调用的命令里列出的源文件来创建。<name>对应于逻辑目标名称，而且在一个工程的全局域内必须是唯一的。待构建的库文件的实际文件名根据对应平台的命名约定来构造（比如lib<name>.a或者<name>.lib）。指定STATIC，SHARED，或者MODULE参数用来指定要创建的库的类型。STATIC库是目标文件的归档文件，在链接其它目标的时候使用。SHARED库会被动态链接，在运行时被加载。MODULE库是不会被链接到其它目标中的插件，但是可能会在运行时使用dlopen-系列的函数动态链接。如果没有类型被显式指定，这个选项将会根据变量BUILD_SHARED_LIBS的当前值是否为真决定是STATIC还是SHARED。

　　默认状态下，库文件将会在于源文件目录树的构建目录树的位置被创建，该命令也会在这里被调用。查阅ARCHIVE_OUTPUT_DIRECTORY，LIBRARY_OUTPUT_DIRECTORY，和RUNTIME_OUTPUT_DIRECTORY这三个目标属性的文档来改变这一位置。查阅OUTPUT_NAME目标属性的文档来改变最终文件名的<name>部分。

　　如果指定了EXCLUDE_FROM_ALL属性，对应的一些属性会在目标被创建时被设置。查阅EXCLUDE_FROM_ALL的文档来获取该属性的细节。

　　使用下述格式，add_library命令也可以用来创建导入的库目标：

```
  add_library(<name> <SHARED|STATIC|MODULE|UNKNOWN> IMPORTED)
```

　　导入的库目标是引用了在工程外的一个库文件的目标。没有生成构建这个库的规则。这个目标名字的作用域在它被创建的路径及以下有效。他可以向任何在该工程内构建的目标一样被引用。导入库为类似于target_link_libraries命令中引用它提供了便利。关于导入库细节可以通过指定那些以IMPORTED_的属性设置来指定。其中最重要的属性是IMPORTED_LOCATION（以及它的具体配置版本，IMPORTED_LOCATION_<CONFIG>），它指定了主库文件在磁盘上的位置。查阅IMPORTED_*属性的文档获取更多的信息。

## CMD#7 add_subdirectory 为构建添加一个子路径。

```
  add_subdirectory(source_dir [binary_dir] 
                   [EXCLUDE_FROM_ALL])
```

　　这条命令的作用是为构建添加一个子路径。source_dir选项指定了CMakeLists.txt源文件和代码文件的位置。如果source_dir是一个相对路径，那么source_dir选项会被解释为相对于当前的目录，但是它也可以是一个绝对路径。binary_dir选项指定了输出文件的路径。如果binary_dir是相对路径，它将会被解释为相对于当前输出路径，但是它也可以是一个绝对路径。如果没有指定binary_dir，binary_dir的值将会是没有做任何相对路径展开的source_dir，这也是通常的用法。在source_dir指定路径下的CMakeLists.txt将会在当前输入文件的处理过程执行到该命令之前，立即被CMake处理。

　　如果指定了EXCLUDE_FROM_ALL选项，在子路径下的目标默认不会被包含到父路径的ALL目标里，并且也会被排除在IDE工程文件之外。用户必须显式构建在子路径下的目标，比如一些示范性的例子工程就是这样。典型地，子路径应该包含它自己的project()命令调用，这样会在子路径下产生一份完整的构建系统（比如VS IDE的solution文件）。注意，目标间的依赖性要高于这种排除行为。如果一个被父工程构建的目标依赖于在这个子路径下的目标，被依赖的目标会被包含到父工程的构建系统中，以满足依赖性的要求。

## CMD#8 add_test

以指定的参数为工程添加一个测试。

```
  add_test(testname Exename arg1 arg2 ... )
```

　　如果已经运行过了ENABLE_TESTING命令，这个命令将为当前路径添加一个测试目标。如果ENABLE_TESTING还没有运行过，该命令啥事都不做。测试是由测试子系统运行的，它会以指定的参数执行Exename文件。Exename或者是由该工程构建的可执行文件，也可以是系统上自带的任意可执行文件（比如tclsh）。该测试会在CMakeList.txt文件的当前工作路径下运行，这个路径与二进制树上的路相对应。

```
  add_test(NAME <name> [CONFIGURATIONS [Debug|Release|...]]
           COMMAND <command> [arg1 [arg2 ...]])
```
　　如果COMMAND选项指定了一个可执行目标（用add_executable创建），它会自动被在构建时创建的可执行文件所替换。如果指定了CONFIGURATIONS选项，那么该测试只有在列出的某一个配置下才会运行。

　　在COMMAND选项后的参数可以使用“生成器表达式”，它的语法是$<...>。这些表达式会在构建系统生成期间，以及构建配置的专有信息的产生期间被评估。合法的表达式是：

```
    $<CONFIGURATION>          = 配置名称
    $<TARGET_FILE:tgt>        = 主要的二进制文件(.exe, .so.1.2, .a)
    $<TARGET_LINKER_FILE:tgt> = 用于链接的文件(.a, .lib, .so)
    $<TARGET_SONAME_FILE:tgt> = 带有.so.的文件(.so.3)
```

　　其中，"tgt"是目标的名称。目标文件表达式TARGET_FILE生成了一个完整的路径，但是它的_DIR和_NAME版本可以生成目录以及文件名部分：

```
$<TARGET_FILE_DIR:tgt>/$<TARGET_FILE_NAME:tgt>
$<TARGET_LINKER_FILE_DIR:tgt>/$<TARGET_LINKER_FILE_NAME:tgt>
$<TARGET_SONAME_FILE_DIR:tgt>/$<TARGET_SONAME_FILE_NAME:tgt>
```

　　用例：

```
  add_test(NAME mytest
            COMMAND testDriver --config $<CONFIGURATION>
                               --exe $<TARGET_FILE:myexe>)
```

　　这段代码创建了一个名为mytest的测试，它执行的命令是testDriver工具，传递的参数包括配置名，以及由目标生成的可执行文件myexe的完整路径。

## CMD#9 aux_source_directory

查找在某个路径下的所有源文件。

```
  aux_source_directory(<dir> <variable>)
```

　　搜集所有在指定路径下的源文件的文件名，将输出结果列表储存在指定的<variable>变量中。该命令主要用在那些使用显式模板实例化的工程上。模板实例化文件可以存储在Templates子目录下，然后可以使用这条命令自动收集起来；这样可以避免手工罗列所有的实例。

　　使用该命令来避免为一个库或可执行目标写源文件的清单，是非常具有吸引力的。但是如果该命令貌似可以发挥作用，那么CMake就不需要生成一个感知新的源文件何时被加进来的构建系统了（也就是说，新文件的加入，并不会导致CMakeLists.txt过时，从而不能引起CMake重新运行。——译注）。正常情况下，生成的构建系统能够感知它何时需要重新运行CMake，因为需要修改CMakeLists.txt来引入一个新的源文件。当源文件仅仅是加到了该路径下，但是没有修改这个CMakeLists.txt文件，使用者只能手动重新运行CMake来产生一个包含这个新文件的构建系统。

## CMD#10 break

从一个包围该命令的foreach或或while循环中跳出。

```
  break()
```

　　从包围它的foreach循环或while循环中跳出。 
　　 
　　

## CMD#11 build_command

获取构建该工程的命令行。

```
  build_command(<variable>
                [CONFIGURATION <config>]
                [PROJECT_NAME <projname>]
                [TARGET <target>])
```

　　把给定的变量<variable>设置成一个字符串，其中包含使用由变量CMAKE_GENERATOR确定的项目构建工具，去构建某一个工程的某一个目标配置的命令行。

　　对于多配置生成器，如果忽略CONFIGURATION选项，CMake将会选择一个合理的默认值；而对于单配置生成器，该选项会被忽略。

　　如果PROJECT_NAME选项被忽略，得到的命令行用来构建当前构建树上的顶层工程。

　　如果TARGET选项被忽略，得到的命令行可以用来构建所有目标，比较高效的用法是构建目标all或者ALL_BUILD。

```
  build_command(<cachevariable> <makecommand>)
```

　　不推荐使用以上的这种格式，但对于后相兼容还是有用的。只要可以，就要使用第一种格式。

　　这种格式将变量<cachevariable>设置为一个字符串，其中包含从构建树的根目录，用<makecommand>指定的构建工具构建这个工程的命令。<makecommand>应该是指向msdev，devenv，nmake，make或者是一种最终用户指定的构建工具的完整路径。

## CMD#12 cmake_minimum_required

设置一个工程所需要的最低CMake版本。

```
  cmake_minimum_required(VERSION major[.minor[.patch[.tweak]]]
                         [FATAL_ERROR])
```

　　如果CMake的当前版本低于指定的版本，它会停止处理工程文件，并报告错误。当指定的版本高于2.4时，它会隐含调用：

```
  cmake_policy(VERSION major[.minor[.patch[.tweak]]])
```

　　从而将cmale的策略版本级别设置为指定的版本。当指定的版本是2.4或更低时，这条命令隐含调用：

```
  cmake_policy(VERSION 2.4)
```

　　这将会启用对于CMake 2.4及更低版本的兼容性。

　　FATAL_ERROR选项是可以接受的，但是CMake 2.6及更高的版本会忽略它。如果它被指定，那么CMake 2.4及更低版本将会以错误告终而非仅仅给出个警告。

## CMD#13 cmake_policy

管理CMake的策略设置。 
　　随着CMake的演变，有时为了搞定bug或改善现有特色的实现方法，改变现有的行为是必须的。CMake的策略机制是在新的CMake版本带来行为上的改变时，用来帮助保持现有项目的构建的一种设计。每个新的策略（行为改变）被赋予一个CMP<NNNN>格式的识别符，其中<NNNN>是一个整数索引。每个策略相关的文档都会描述“旧行为”和“新行为”，以及引入该策略的原因。工程可以设置各种策略来选择期望的行为。当CMake需要了解要用哪种行为的时候，它会检查由工程指定的一种设置。如果没有可用的设置，工程假定使用“旧行为”，并且会给出警告要求你设置工程的策略。

　　cmake_policy是用来设置“新行为”或“旧行为”的命令。如果支持单独设置策略，我们鼓励各项目根据CMake的版本来设置策略。

```
  cmake_policy(VERSION major.minor[.patch[.tweak]])
```

　　上述命令指定当前的CMakeLists.txt是为给定版本的CMake书写的。所有在指定的版本或更早的版本中引入的策略会被设置为使用“新行为”。所有在指定的版本之后引入的策略将会变为无效（unset）。该命令有效地为一个指定的CMake版本请求优先采用的行为，并且告知更新的CMake版本给出关于它们新策略的警告。命令中指定的策略版本必须至少是2.4，否则命令会报告一个错误。为了得到支持早于2.4版本的兼容性特性，查阅策略CMP0001的相关文档。

```
　　cmake_policy(SET CMP<NNNN> NEW)
　　cmake_policy(SET CMP<NNNN> OLD)
```

　　对于某种给定的策略，该命令要求CMake使用新的或者旧的行为。对于一个指定的策略，那些依赖于旧行为的工程，通过设置策略的状态为OLD，可以禁止策略的警告。或者，用户可以让工程采用新行为，并且设置策略的状态为NEW。

```
  cmake_policy(GET CMP<NNNN> <variable>)
```

　　该命令检查一个给定的策略是否设置为旧行为或新行为。如果策略被设置，输出的变量值会是“OLD”或“NEW”，否则为空。

　　CMake将策略设置保存在一个栈结构中，因此，cmake_policy命令产生的改变仅仅影响在栈顶端的元素。在策略栈中的一个新条目由各子路径自动管理，以此保护它的父路径及同层路径的策略设置。CMake也管理通过include()和find_package()命令加载的脚本中新加入的条目，除非调用时指定了NO_POLICY_SCOPE选项（另外可参考CMP0011）。cmake_policy命令提供了一种管理策略栈中自定义条目的接口： 
　　
```
       cmake_policy(PUSH)
       cmake_policy(POP)
```

　　每个PUSH必须有一个配对的POP来去掉撤销改变。这对于临时改变策略设置比较有用。

　　函数和宏会在它们被创建的时候记录策略设置，并且在它们被调用的时候使用记录前的策略。如果函数或者宏实现设置了策略，这个变化会通过调用者(caller)一直上传，自动传递到嵌套的最近的策略栈条目。 
　　

## CMD#14 configure_file

将一份文件拷贝到另一个位置并修改它的内容。

```
  configure_file(<input> <output>
                 [COPYONLY] [ESCAPE_QUOTES] [@ONLY])
```

　　将文件<input>拷贝到<output>然后替换文件内容中引用到的变量值。如果<input>是相对路径，它被评估的基础路径是当前源码路径。<input>必须是一个文件，而不是个路径。如果<output>是一个相对路径，它被评估的基础路径是当前二进制文件路径。如果<output>是一个已有的路径，那么输入文件将会以它原来的名字放到那个路径下。

　　该命令替换掉在输入文件中，以${VAR}格式或@VAR@格式引用的任意变量，如同它们的值是由CMake确定的一样。 如果一个变量还未定义，它会被替换为空。如果指定了COPYONLY选项，那么变量就不会展开。如果指定了ESCAPE_QUOTES选项，那么所有被替换的变量将会按照C语言的规则被转义。该文件将会以CMake变量的当前值被配置。如果指定了@ONLY选项，只有@VAR@格式的变量会被替换而${VAR}格式的变量则会被忽略。这对于配置使用${VAR}格式的脚本文件比较有用。任何类似于#cmakedefine VAR的定义语句将会被替换为#define VAR或者/* #undef VAR */，视CMake中对VAR变量的设置而定。任何类似于#cmakedefine 01 VAR的定义语句将会被替换为#define VAR 1或#define VAR 0，视VAR被评估为TRUE或FALSE而定。

（configure_file的作用是让普通文件也能使用CMake中的变量。——译注）

## CMD#15 create_test_sourcelist

为构建测试程序创建一个测试驱动器和源码列表。

```
  create_test_sourcelist(sourceListName driverName
                         test1 test2 test3
                         EXTRA_INCLUDE include.h
                         FUNCTION function)
```

　　测试驱动器是一个将很多小的测试代码连接为一个单一的可执行文件的程序。这在为了缩减总的需用空间而用很多大的库文件去构建静态可执行文件的时候，特别有用。构建测试驱动所需要的源文件列表会在变量sourceListName中。DriverName变量是测试驱动器的名字。其它的参数还包括一个测试源代码文件的清单，中间可以用分号隔开。每个测试源码文件中应该有一个与去掉扩展名的文件名同名的函数（比如foo.cxx 文件里应该有int foo(int, char*[]);）（和main的函数签名一样——译注）。DriverName可以在命令行中按名字调用这些测试中的每一个。如果指定了EXTRA_INCLUDE，那么它后面的参数（即include.h——译注）会被包含到生成的文件里。如果指定了FUNCTION选项，那么它后面的参数（即function——译注）会被认为是一个函数名，传递给它的参数是一个指向argc的指针和argv。这个选项可以用来为每个测试函数添加额外的命令行参数处理过程。CMake变量CMAKE_TESTDRIVER_BEFORE_TESTMAIN用来设置在调用测试的main函数之前调用的代码。

## CMD#16 define_property

定义并描述（Document）自定义属性。

```
 　　define_property(<GLOBAL | DIRECTORY | TARGET | SOURCE |
                    TEST | VARIABLE | CACHED_VARIABLE>
                    PROPERTY <name> [INHERITED]
                    BRIEF_DOCS <brief-doc> [docs...]
                    FULL_DOCS <full-doc> [docs...])
```

　　在一个域(域（scope)）中定义一个可以用set_property和get_property命令访问的属性。这个命令对于把文档和可以通过get_property命令得到的属性名称关联起来非常有用。第一个参数确定了这个属性可以使用的范围。它必须是下列值中的一个：

```
GLOBAL    = 与全局命名空间相关联
DIRECTORY = 与某一个目录相关联
TARGET    = 与一个目标相关联
SOURCE    = 与一个源文件相关联
TEST      = 与一个以add_test命名的测试相关联
VARIABLE  = 描述（document）一个CMake语言变量
CACHED_VARIABLE = 描述（document）一个CMake语言缓存变量
```

　　注意，与set_property和get_property不相同，不需要给出实际的作用域；只有作用域的类型才是重要的。PROPERTY选项必须有，它后面紧跟要定义的属性名。如果指定了INHERITED选项，那么如果get_property命令所请求的属性在该作用域中未设置，它会沿着链条向更高的作用域去搜索。DIRECTORY域向上是GLOBAL。TARGET，SOURCE和TEST向上是DIRECTORY。

　　BRIEF_DOCS和FULL_DOCS选项后面的参数是和属性相关联的字符串，分别作为变量的简单描述和完整描述。在使用get_property命令时，对应的选项可以获取这些描述信息。 
　 
　 
## CMD#17 else

开始一个if语句块的else部分。

```
  else(expression)
```

参见if命令。 
　 
　 
　

## CMD#18 elseif

开始 if 块的 elseif 部分。

```
  elseif(expression)
```

参见if命令。 

## CMD#19 enable_language

支持某种语言（CXX/C/Fortran/等）

```
  enable_language(languageName [OPTIONAL] )
```

　　该命令打开了CMake对参数中指定的语言的支持。这与project命令相同，但是不会创建任何project命令会产生的额外变量。可以选用的语言的类型有CXX，C，Fortran等。如果指定了OPTIONAL选项，用CMAKE_<languageName>_COMPILER_WORKS变量来判断该语言是否被成功支持。

## CMD#20 enable_testing

打开当前及以下目录中的测试功能。

```
　　enable_testing()
```

　　为当前及其下级目录打开测试功能。也可参见add_test命令。注意，ctest需要在构建跟目录下找到一个测试文件。因此，这个命令应该在源文件目录的根目录下。

## CMD#21 endforeach

结束foreach语句块中的一系列命令。

```
  endforeach(expression)
```
　　参见FOREACH命令。 
　　

## CMD#22 endfunction

结束一个function语句块中的一系列命令。

```
  endfunction(expression)
```

　　参见function命令。 
　　

## CMD#23 endif

结束一个if语句块中的一系列命令。

```
  endif(expression)
```

　　参见if命令。 
　　

## CMD#24 endmacro

结束一个macro语句块中的一系列命令。

```
  endmacro(expression)
```

　　参见macro命令。 
　　

## CMD#25: endwhile 
结束一个while语句块中的一系列命令。

```
  endwhile(expression)
```

　　参见while命令。 
　　

## CMD#26 execute_process

执行一个或更多个子进程。

```
  execute_process(COMMAND <cmd1> [args1...]]
                  [COMMAND <cmd2> [args2...] [...]]
                  [WORKING_DIRECTORY <directory>]
                  [TIMEOUT <seconds>]
                  [RESULT_VARIABLE <variable>]
                  [OUTPUT_VARIABLE <variable>]
                  [ERROR_VARIABLE <variable>]
                  [INPUT_FILE <file>]
                  [OUTPUT_FILE <file>]
                  [ERROR_FILE <file>]
                  [OUTPUT_QUIET]
                  [ERROR_QUIET]
                  [OUTPUT_STRIP_TRAILING_WHITESPACE]
                  [ERROR_STRIP_TRAILING_WHITESPACE])
```

　　运行一条或多条命令，使得前一条命令的标准输出以管道的方式成为下一条命令的标准输入。所有进程公用一个单独的标准错误管道。如果指定了WORKING_DIRECTORY选项，后面的路径选项将会设置为子进程的当前工作路径。如果指定了TIMEOUT选项，如果子进程没有在指定的秒数（允许分数）里完成，子进程会自动终止。如果指定了RESULT_VARIABLE选项，该变量将保存为正在运行的进程的结果；它可以是最后一个子进程的整数返回代码，也可以是一个描述错误状态的字符串。如果指定了OUTPUT_VARIABLE或者ERROR_VARIABLE，后面的变量将会被分别设置为标准输出和标准错误管道的值。如果两个管道都是用了相同的变量，它们的输出将会按产生的顺序被合并。如果指定了INPUT_FILE，OUTPUT_FILE 或 ERROR_FILE选项，其后的文件将会分别被附加到第一个进程的标准输入、最后一个进程的标准输出，或者所有进程的标准错误管道上。如果指定了OUTPUT_QUIET后者ERROR_QUIET选项，那么标准输出或标准错误的结果将会被静静的忽略掉。如果为同一个管道指定了多于一个的OUTPUT_*或ERROR_* 选项，优先级是没有指定的。如果没有指定OUTPUT_*或者ERROR_*选项，输出将会与CMake进程自身对应的管道共享。

　　execute_process命令是exec_program命令的一个较新的功能更加强大的版本。但是为了兼容性的原因，旧的exec_program命令还会继续保留。

## CMD#27 export

从构建树中导出目标供外部使用。

```
  export(TARGETS [target1 [target2 [...]]] [NAMESPACE <namespace>]
         [APPEND] FILE <filename>)
```

　　创建一个名为<filename>的文件，它可以被外部工程包含进去，从而外部工程可以从当前工程的构建树中导入目标。这对于交叉编译那些可以运行在宿主平台的的utility可执行文件，然后将它们导入到另外一个编译成目标平台代码的工程中的情形，特别有用。如果指定了NAMESPACE选项，<namespace>字符串将会被扩展到输出文件中的所有目标的名字中。如果指定了APPEND选项，生成的代码将会续接在文件之后，而不是覆盖它。如果一个库目标被包含在export中，但是连接成它的目标没有被包含，行为没有指定。

　　由该命令创建的文件是与指定的构建树一致的，并且绝对不应该被安装。要从一个安装树上导出目标，参见install(EXPORT)命令。

```
  export(PACKAGE <name>)
```

　　在CMake的用户包注册表中，为<name>包(package)存储当前的构建目录。这将有助于依赖于它的工程从当前工程的构建树中查找并使用包而不需要用户的介入。注意，该命令在包注册表中创建的条目，仅仅在与跟构建树一起运行的包配置文件(<name>Config.cmake)一起使用时才会起作用。 
　　

## CMD#28 file

文件操作命令

```
  file(WRITE filename "message to write"... )
  file(APPEND filename "message to write"... )
  file(READ filename variable [LIMIT numBytes] [OFFSET offset] [HEX])
  file(STRINGS filename variable [LIMIT_COUNT num]
       [LIMIT_INPUT numBytes] [LIMIT_OUTPUT numBytes]
       [LENGTH_MINIMUM numBytes] [LENGTH_MAXIMUM numBytes]
       [NEWLINE_CONSUME] [REGEX regex]
       [NO_HEX_CONVERSION])
  file(GLOB variable [RELATIVE path] [globbing expressions]...)
  file(GLOB_RECURSE variable [RELATIVE path] 
       [FOLLOW_SYMLINKS] [globbing expressions]...)
  file(RENAME <oldname> <newname>)
  file(REMOVE [file1 ...])
  file(REMOVE_RECURSE [file1 ...])
  file(MAKE_DIRECTORY [directory1 directory2 ...])
  file(RELATIVE_PATH variable directory file)
  file(TO_CMAKE_PATH path result)
  file(TO_NATIVE_PATH path result)
  file(DOWNLOAD url file [TIMEOUT timeout] [STATUS status] [LOG log]
       [EXPECTED_MD5 sum] [SHOW_PROGRESS])
```

　　WRITE选项将会写一条消息到名为filename的文件中。如果文件已经存在，该命令会覆盖已有的文件；如果文件不存在，它将创建该文件。

　　APPEND选项和WRITE选项一样，将会写一条消息到名为filename的文件中，只是该消息会附加到文件末尾。

　　READ选项将会读一个文件中的内容并将其存储在变量里。读文件的位置从offset开始，最多读numBytes个字节。如果指定了HEX参数，二进制代码将会转换为十六进制表达方式，并存储在变量里。

　　STRINGS将会从一个文件中将一个ASCII字符串的list解析出来，然后存储在variable变量中。文件中的二进制数据会被忽略。回车换行符会被忽略。它也可以用在Intel的Hex和Motorola的S-记录文件；读取它们时，它们会被自动转换为二进制格式。可以使用NO_HEX_CONVERSION选项禁止这项功能。LIMIT_COUNT选项设定了返回的字符串的最大数量。LIMIT_INPUT设置了从输入文件中读取的最大字节数。LIMIT_OUTPUT设置了在输出变量中存储的最大字节数。LENGTH_MINIMUM设置了要返回的字符串的最小长度；小于该长度的字符串会被忽略。LENGTH_MAXIMUM设置了返回字符串的最大长度；更长的字符串会被分割成不长于最大长度的字符串。NEWLINE_CONSUME选项允许新行被包含到字符串中，而不是终止它们。REGEX选项指定了一个待返回的字符串必须满足的正则表达式。典型的使用方式是：

```
  file(STRINGS myfile.txt myfile)
```

该命令在变量myfile中存储了一个list，该list中每个项是输入文件中的一行文本。 
　　GLOB选项将会为所有匹配查询表达式的文件生成一个文件list，并将该list存储进变量variable里。文件名查询表达式与正则表达式类似，只不过更加简单。如果为一个表达式指定了RELATIVE标志，返回的结果将会是相对于给定路径的相对路径。文件名查询表达式的例子有：

```
*.cxx      - 匹配所有扩展名为cxx的文件。
*.vt?      - 匹配所有扩展名是vta,...,vtz的文件。
f[3-5].txt - 匹配文件f3.txt, f4.txt, f5.txt。
```

　　GLOB_RECURSE选项将会生成一个类似于通常的GLOB选项的list，只是它会寻访所有那些匹配目录的子路径并同时匹配查询表达式的文件。作为符号链接的子路径只有在给定FOLLOW_SYMLINKS选项或者cmake策略CMP0009被设置为NEW时，才会被寻访到。参见cmake --help-policy CMP0009 查询跟多有用的信息。

使用递归查询的例子有：

```
/dir/*.py  - 匹配所有在/dir及其子目录下的python文件。

    MAKE_DIRECTORY选项将会创建指定的目录，如果它们的父目录不存在时，同样也会创建。（类似于mkdir命令——译注）

    RENAME选项对同一个文件系统下的一个文件或目录重命名。（类似于mv命令——译注）

    REMOVE选项将会删除指定的文件，包括在子路径下的文件。（类似于rm命令——译注）

    REMOVE_RECURSE选项会删除给定的文件以及目录，包括非空目录。（类似于rm -r 命令——译注）

    RELATIVE_PATH选项会确定从direcroty参数到指定文件的相对路径。

    TO_CMAKE_PATH选项会把path转换为一个以unix的 / 开头的cmake风格的路径。输入可以是一个单一的路径，也可以是一个系统路径，比如"$ENV{PATH}"。注意，在调用TO_CMAKE_PATH的ENV周围的双引号只能有一个参数(Note the double quotes around the ENV call TO_CMAKE_PATH only takes one argument. 原文如此。quotes和后面的takes让人后纠结，这句话翻译可能有误。欢迎指正——译注)。

    TO_NATIVE_PATH选项与TO_CMAKE_PATH选项很相似，但是它会把cmake风格的路径转换为本地路径风格：windows下用\，而unix下用/。

    DOWNLOAD 将给定的URL下载到指定的文件中。如果指定了LOG var选项，下载日志将会被输出到var中。如果指定了STATUS var选项，下载操作的状态会被输出到var中。该状态返回值是一个长度为2的list。list的第一个元素是操作的数字返回值，第二个返回值是错误的字符串值。错误信息如果是数字0，操作中没有发生错误。如果指定了TIMEOUT time选项，在time秒之后，操作会超时退出；time应该是整数。如果指定了EXPECTED_MD5 sum选项，下载操作会认证下载的文件的实际MD5和是否与期望值匹配。如果不匹配，操作将返回一个错误。如果指定了SHOW_PROGRESS选项，进度信息会以状态信息的形式被打印出来，直到操作完成。
```

　　file命令还提供了COPY和INSTALL两种格式：

```
  file(<COPY|INSTALL> files... DESTINATION <dir>
       [FILE_PERMISSIONS permissions...]
       [DIRECTORY_PERMISSIONS permissions...]
       [NO_SOURCE_PERMISSIONS] [USE_SOURCE_PERMISSIONS]
       [FILES_MATCHING]
       [[PATTERN <pattern> | REGEX <regex>]
        [EXCLUDE] [PERMISSIONS permissions...]] [...])
```

　　COPY版本把文件、目录以及符号连接拷贝到一个目标文件夹。相对输入路径的评估是基于当前的源代码目录进行的，相对目标路径的评估是基于当前的构建目录进行的。复制过程将保留输入文件的时间戳；并且如果目标路径处存在同名同时间戳的文件，复制命令会把它优化掉。赋值过程将保留输入文件的访问权限，除非显式指定权限或指定NO_SOURCE_PERMISSIONS选项（默认是USE_SOURCE_PERMISSIONS）。参见install(DIRECTORY)命令中关于权限(permissions)，PATTERN，REGEX和EXCLUDE选项的文档。

　　INSTALL版本与COPY版本只有十分微小的差别：它会打印状态信息，并且默认使用NO_SOURCE_PERMISSIONS选项。install命令生成的安装脚本使用这个版本（它会使用一些没有在文档中涉及的内部使用的选项。） 
　　 
　　

## CMD#29 find_file

查找一个文件的完整路径。

```
   find_file(<VAR> name1 [path1 path2 ...])
```

　　这是该命令的精简格式，对于大多数场合它都足够了。它与命令find_file(<VAR> name1 [PATHS path1 path2 ...])是等价的。

```
   find_file(
             <VAR>
             name | NAMES name1 [name2 ...]
             [HINTS path1 [path2 ... ENV var]]
             [PATHS path1 [path2 ... ENV var]]
             [PATH_SUFFIXES suffix1 [suffix2 ...]]
             [DOC "cache documentation string"]
             [NO_DEFAULT_PATH]
             [NO_CMAKE_ENVIRONMENT_PATH]
             [NO_CMAKE_PATH]
             [NO_SYSTEM_ENVIRONMENT_PATH]
             [NO_CMAKE_SYSTEM_PATH]
             [CMAKE_FIND_ROOT_PATH_BOTH |
              ONLY_CMAKE_FIND_ROOT_PATH |
              NO_CMAKE_FIND_ROOT_PATH]
            )
```

　　这条命令用来查找指定文件的完整路径。一个名字是<VAR>的缓存条目（参见CMakeCache.txt的介绍——译注）变量会被创建，用来存储该命令的结果。如果发现了文件的一个完整路径，该结果会被存储到该变量里并且搜索过程不会再重复，除非该变量被清除。如果什么都没发现，搜索的结果将会是<VAR>-NOTFOUND；并且在下一次以相同的变量调用find_file时，该搜索会重新尝试。被搜索的文件的文件名由NAMES选项后的名字列表指定。附加的其他搜索位置可以在PATHS选项之后指定。如果ENV var在HINTS或PATHS段中出现，环境变量var将会被读取然后被转换为一个系统级环境变量，并存储在一个cmake风格的路径list中。比如，使用ENV PATH将会将系统的path变量列出来。在DOC之后的变量将会用于cache中的文档字符串(documentation string)。PATH_SUFFIXES指定了在每个搜索路径下的需要搜索的子路径。

　　如果指定了NO_DEFAULT_PATH选项，那么在搜索时不会附加其它路径。如果没有指定NO_DEFAULT_PATH选项，搜索过程如下：

　　1、在cmake特有的cache变量中指定的搜索路径搜索。这些路径用于在命令行里用-DVAR=value被设置。如果使用了NO_CMAKE_PATH选项，该路径会被跳过。（此句翻译可能有误——译注。）搜索路径还包括：

```
对于每个在CMAKE_PREFIX_PATH中的路径<prefix>，<prefix>/include  
变量：CMAKE_INCLUDE_PATH
变量：CMAKE_FRAMEWORK_PATH
```

　　2、在cmake特定的环境变量中指定的搜索路径搜索。该路径会在用户的shell配置中被设置。如果指定了NO_CMAKE_ENVIRONMENT_PATH选项，该路径会被跳过。搜索路径还包括：

```
对于每个在CMAKE_PREFIX_PATH中的路径<prefix>，<prefix>/include  
变量：CMAKE_INCLUDE_PATH
变量：CMAKE_FRAMEWORK_PATH
```

　　3、由HINTS选项指定的搜索路径。这些路径是由系统内省(introspection)时计算出来的路径，比如已经发现的其他项的位置所提供的痕迹。硬编码的参考路径应该使用PATHS选项指定。（HINTS与PATHS有何不同？比后者的优先级高？有疑问。——译注）

　　4、搜索标准的系统环境变量。如果指定NO_SYSTEM_ENVIRONMENT_PATH选项，搜索路径将跳过其后的参数。搜索路径包括环境变量PATH个INCLUDE。

　　5、查找在当前系统的平台文件中定义的cmake变量。如果指定了NO_CMAKE_SYSTEM_PATH选项，该路径会被跳过。其他的搜索路径还包括：

```
对于每个在CMAKE_PREFIX_PATH中的路径<prefix>，<prefix>/include  
变量：CMAKE_SYSTEM_INCLUDE_PATH
变量：CMAKE_SYSTEM_FRAMEWORK_PATH
```

　　6、搜索由PATHS选项指定的路径或者在命令的简写版本中指定的路径。这一般是一些硬编码的参考路径。在Darwin后者支持OS X框架的系统上，cmake变量CMAKE_FIND_FRAMWORK可以设置为空或者下述值之一：

```
"FIRST"  - 在标准库或者头文件之前先查找框架。对于Darwin系统，这是默认的。
"LAST"   - 在标准库或头文件之后再查找框架。
"ONLY"   - 只查找框架。
"NEVER" - 从不查找框架。
```

　　在Darwin或者支持OS X Application Bundles的系统上，cmake变量CMAKE_FIND_APPBUNDLE可以被设置为空，或者下列值之一：

```
"FIRST"  - 在标准程序之前查找application bundles，这也是Darwin系统的默认选项。
"LAST"   - 在标准程序之后查找application bundlesTry。
"ONLY"   - 只查找application bundles。
"NEVER" - 从不查找application bundles。
```

　　CMake的变量CMAKE_FIND_ROOT_PATH指定了一个或多个在所有其它搜索路径之前的搜索路径。该选项很有效地将给定位置下的整个搜索路径的最优先路径进行了重新指定。默认情况下，它是空的。当交叉编译一个指向目标环境下的根目录中的目标时，CMake也会搜索那些路径；该变量这时显得非常有用。默认情况下，首先会搜索在CMAKE_FIND_ROOT_PATH变量中列出的路径，然后才是非根路径。设置CMAKE_FIND_ROOT_PATH_MODE_INCLUDE变量可以调整该默认行为。该行为可以在每次调用时被手动覆盖。通过使用CMAKE_FIND_ROOT_PATH_BOTH变量，搜索顺序将会是上述的那样。如果使用了NO_CMAKE_FIND_ROOT_PATH变量，那么CMAKE_FIND_ROOT_PATH将不会被用到。如果使用了ONLY_CMAKE_FIND_ROOT_PATH变量，那么只有CMAKE_FIND_ROOT_PATH中的路径（即re-rooted目录——译注）会被搜索。

　　一般情况下，默认的搜索顺序是从最具体的路径到最不具体的路径。只要用NO_*选项多次调用该命令，工程就可以覆盖该顺序。

```
     find_file(<VAR> NAMES name PATHS paths... NO_DEFAULT_PATH)
     find_file(<VAR> NAMES name)
```

　　只要这些调用中的一个成功了，返回变量就会被设置并存储在cache中；然后该命令就不会再继续查找了。 
　　

## CMD#30 find_library

查找一个库文件

```
   find_library(<VAR> name1 [path1 path2 ...])
```

　　这是该命令的简写版本，在大多数场合下都已经够用了。它与命令find_library(<VAR> name1 [PATHS path1 path2 ...])等价。

```
   find_library(
             <VAR>
             name | NAMES name1 [name2 ...]
             [HINTS path1 [path2 ... ENV var]]
             [PATHS path1 [path2 ... ENV var]]
             [PATH_SUFFIXES suffix1 [suffix2 ...]]
             [DOC "cache documentation string"]
             [NO_DEFAULT_PATH]
             [NO_CMAKE_ENVIRONMENT_PATH]
             [NO_CMAKE_PATH]
             [NO_SYSTEM_ENVIRONMENT_PATH]
             [NO_CMAKE_SYSTEM_PATH]
             [CMAKE_FIND_ROOT_PATH_BOTH |
              ONLY_CMAKE_FIND_ROOT_PATH |
              NO_CMAKE_FIND_ROOT_PATH]
            )
```

　　该命令用来查找一个库文件。一个名为<VAR>的cache条目会被创建来存储该命令的结果。如果找到了该库文件，那么结果会存储在该变量里，并且搜索过程将不再重复，除非该变量被清空。如果没有找到，结果变量将会是<VAR>-NOTFOUND，并且在下次使用相同变量调用find_library命令时，搜索过程会再次尝试。在NAMES参数后列出的文件名是要被搜索的库名。附加的搜索位置在PATHS参数后指定。如果再HINTS或者PATHS字段中设置了ENV变量var，环境变量var将会被读取并从系统环境变量转换为一个cmake风格的路径list。例如，指定ENV PATH是获取系统path变量并将其转换为cmake的list的一种方式。在DOC之后的参数用来作为cache中的注释字符串。PATH_SUFFIXES选项指定了每个搜索路径下待搜索的子路径。

　　如果指定了NO_DEFAULT_PATH选项，那么搜索的过程中不会有其他的附加路径。如果没有指定该选项，搜索过程如下：

　　1、搜索cmake特有的cache变量指定的路径。这些变量是在用cmake命令行时，通过-DVAR=value指定的变量。如果指定了NO_CMAKE_PATH选项，这些路径会被跳过。搜索的路径还包括：

```
对于每个在CMAKE_PREFIX_PATH中的<prefix>，路径<prefix>/lib 
CMAKE_LIBRARY_PATH
CMAKE_FRAMEWORK_PATH
```

　　2、搜索cmake特有的环境变量指定的路径。这些变量是用户的shell配置中设置的变量。如果指定了NO_CMAKE_ENVIRONMENT_PATH选项，这些路径会被跳过。搜索的路径还包括：

```
对于每个在CMAKE_PREFIX_PATH中的<prefix>，路径<prefix>/lib 
CMAKE_LIBRARY_PATH
CMAKE_FRAMEWORK_PATH
```

　　3、搜索由HINTS选项指定的路径。这些路径是系统内省(introspection)估算出的路径，比如由另一个已经发现的库文件的地址提供的参考信息。硬编码的推荐路径应该通过PATHS选项指定。

　　4、查找标准的系统环境变量。如果指定了NO_SYSTEM_ENVIRONMENT_PATH选项，这些路径会被跳过。搜索的路径还包括：

```
PATH
LIB
```

　　5、查找在为当前系统的平台文件中定义的cmake变量。如果指定了NO_CMAKE_SYSTEM_PATH选项，该路径会被跳过。搜索的路径还包括：

```
对于每个在CMAKE_SYSTEM_PREFIX_PATH中的<prefix>，路径<prefix>/lib 
CMAKE_SYSTEM_LIBRARY_PATH
CMAKE_SYSTEM_FRAMEWORK_PATH
```

　　6、搜索PATHS选项或者精简版命令指定的路径。这些通常是硬编码的推荐搜索路径。

　　在Darwin或者支持OS X 框架的系统上，cmake变量CMAKE_FIND_FRAMEWORK可以用来设置为空，或者下述值之一：
　　
```
FIRST"  - 在标准库或头文件之前查找框架。在Darwin系统上这是默认选项。
"LAST"   - 在标准库或头文件之后查找框架。
"ONLY"   - 仅仅查找框架。
NEVER" - 从不查找框架。
```

　　在Darwin或者支持OS X Application Bundles的系统，cmake变量CMAKE_FIND_APPBUNDLE可以被设置为空或者下面这些值中的一个：
　　
```
"FIRST"  - 在标准库或头文件之前查找application bundles。在Darwin系统上这是默认选项。
"LAST"   - 在标准库或头文件之后查找application bundles。
"ONLY"   - 仅仅查找application bundles。
"NEVER" - 从不查找application bundles。
```

　　CMake变量CMAKE_FIND_ROOT_PATH指定了一个或者多个优先于其他搜索路径的搜索路径。该变量能够有效地重新定位在给定位置下进行搜索的根路径。该变量默认为空。当使用交叉编译时，该变量十分有用：用该变量指向目标环境的根目录，然后CMake将会在那里查找。默认情况下，在CMAKE_FIND_ROOT_PATH中列出的路径会首先被搜索，然后是“非根”路径。该默认规则可以通过设置CMAKE_FIND_ROOT_PATH_MODE_LIBRARY做出调整。在每次调用该命令之前，都可以通过设置这个变量来手动覆盖默认行为。如果使用了NO_CMAKE_FIND_ROOT_PATH变量，那么只有重定位的路径会被搜索。

　　默认的搜索顺序的设计逻辑是按照使用时从最具体到最不具体。通过多次调用find_library命令以及NO_*选项，可以覆盖工程的这个默认顺序：

```
    find_library(<VAR> NAMES name PATHS paths... NO_DEFAULT_PATH)
    find_library(<VAR> NAMES name)
```

　　只要这些调用中的一个成功返回，结果变量就会被设置并且被存储到cache中；这样随后的调用都不会再行搜索。如果那找到的库是一个框架，VAR将会被设置为指向框架<完整路径>/A.framework 的完整路径。当一个指向框架的完整路径被用作一个库文件，CMake将使用-framework A，以及-F<完整路径>这两个选项将框架连接到目标上。 
　　 
　　 
　　

## CMD#31 find_package

为外部工程加载设置。

```
  find_package(<package> [version] [EXACT] [QUIET]
               [[REQUIRED|COMPONENTS] [components...]]
               [NO_POLICY_SCOPE])
```

　　查找并加载外来工程的设置。该命令会设置<package>_FOUND变量，用来指示要找的包是否被找到了。如果这个包被找到了，与它相关的信息可以通过包自身记载的变量中得到。QUIET选项将会禁掉包没有被发现时的警告信息。REQUIRED选项表示如果报没有找到的话，cmake的过程会终止，并输出警告信息。在REQUIRED选项之后，或者如果没有指定REQUIRED选项但是指定了COMPONENTS选项，在它们的后面可以列出一些与包相关的部件清单(components list)。[version]参数需要一个版本号，它是正在查找的包应该兼容的版本号（格式是major[.minor[.patch[.tweak]]]）。EXACT选项要求该版本号必须精确匹配。如果在find-module内部对该命令的递归调用没有给定[version]参数，那么[version]和EXACT选项会自动地从外部调用前向继承。对版本的支持目前只存在于包和包之间（详见下文）。

　　用户代码总体上应该使用上述的简单调用格式查询需要的包。本命令文档的剩余部分则详述了find_package的完整命令格式以及具体的查询过程。期望通过该命令查找并提供包的项目维护人员，我们鼓励你能继续读下去。

　　该命令在搜索包时有两种模式：“模块”模式和“配置”模式。当该命令是通过上述的精简格式调用的时候，合用的就是模块模式。在该模式下，CMake搜索所有名为Find<package>.cmake的文件，这些文件的路径由变量由安装CMake时指定的CMAKE_MODULE_PATH变量指定。如果查找到了该文件，它会被CMake读取并被处理。该模式对查找包，检查版本以及生成任何别的必须信息负责。许多查找模块（find-module）仅仅提供了有限的，甚至根本就没有对版本化的支持；具体信息查看该模块的文档。如果没有找到任何模块，该命令会进入配置模式继续执行。

　　完整的配置模式下的命令格式是：

```
  find_package(<package> [version] [EXACT] [QUIET]
               [[REQUIRED|COMPONENTS] [components...]] [NO_MODULE]
               [NO_POLICY_SCOPE]
               [NAMES name1 [name2 ...]]
               [CONFIGS config1 [config2 ...]]
               [HINTS path1 [path2 ... ]]
               [PATHS path1 [path2 ... ]]
               [PATH_SUFFIXES suffix1 [suffix2 ...]]
               [NO_DEFAULT_PATH]
               [NO_CMAKE_ENVIRONMENT_PATH]
               [NO_CMAKE_PATH]
               [NO_SYSTEM_ENVIRONMENT_PATH]
               [NO_CMAKE_PACKAGE_REGISTRY]
               [NO_CMAKE_BUILDS_PATH]
               [NO_CMAKE_SYSTEM_PATH]
               [CMAKE_FIND_ROOT_PATH_BOTH |
                ONLY_CMAKE_FIND_ROOT_PATH |
                NO_CMAKE_FIND_ROOT_PATH])
```

　　NO_MODULE可以用来明确地跳过模块模式。它也隐含指定了不使用在精简格式中使用的那些选项。

　　配置模式试图查找一个由待查找的包提供的配置文件的位置。包含该文件的路径会被存储在一个名为<package>_DIR的cache条目里。默认情况下，该命令搜索名为<package>的包。如果指定了NAMES选项，那么其后的names参数会取代<package>的角色。该命令会为每个在names中的name搜索名为<name>Config.cmake或者<name全小写>-config.cmake的文件。通过使用CONFIGS选项可以改变可能的配置文件的名字。以下描述搜索的过程。如果找到了配置文件，它将会被CMake读取并处理。由于该文件是由包自身提供的，它已经知道包中内容的位置。配置文件的完整地址存储在cmake的变量<package>_CONFIG中。

　　所有CMake要处理的配置文件将会搜索该包的安装信息，并且将该安装匹配的适当版本号(appropriate version)存储在cmake变量<package>_CONSIDERED_CONFIGS中，与之相关的版本号(associated version)将被存储在<package>_CONSIDERED_VERSIONS中。

　　如果没有找到包配置文件，CMake将会生成一个错误描述文件，用来描述该问题——除非指定了QUIET选项。如果指定了REQUIRED选项，并且没有找到该包，将会报致命错误，然后配置步骤终止执行。如果设置了<package>_DIR变量被设置了，但是它没有包含配置文件信息，那么CMake将会直接无视它，然后重新开始查找。

　　如果给定了[version]参数，那么配置模式仅仅会查找那些在命令中请求的版本（格式是major[.minor[.patch[.tweak]]]）与包请求的版本互相兼容的那些版本的包。如果指定了EXACT选项，一个包只有在它请求的版本与[version]提供的版本精确匹配时才能被找到。CMake不会对版本数的含义做任何的转换。包版本号由包自带的版本文件来检查。对于一个备选的包配置文件<config-file>.cmake，对应的版本文件的位置紧挨着它，并且名字或者是<config-file>-version.cmake或者是<config-file>Version.cmake。如果没有这个版本文件，那么配置文件就会认为不兼容任何请求的版本。当找到一个版本文件之后，它会被加载然后用来检查（find_package）请求的版本号。版本文件在一个下述变量被定义的嵌套域中被加载：

```
PACKAGE_FIND_NAME          = <package>名字。
PACKAGE_FIND_VERSION       = 请求的完整版本字符串
PACKAGE_FIND_VERSION_MAJOR = 如果被请求了，那么它是major版本号，否则是0。
PACKAGE_FIND_VERSION_MINOR = 如果被请求了，那么它是minor版本号，否则是0。
PACKAGE_FIND_VERSION_PATCH = 如果被请求了，那么它是patch版本号，否则是0。
PACKAGE_FIND_VERSION_TWEAK = 如果被请求了，那么它是tweak版本号，否则是0。
PACKAGE_FIND_VERSION_COUNT = 版本号包含几部分，0到4。
```

　　版本文件会检查自身是否满足请求的版本号，然后设置了下面这些变量：

```
PACKAGE_VERSION            = 提供的完整的版本字符串。
PACKAGE_VERSION_EXACT      = 如果版本号精确匹配，返回true。
PACKAGE_VERSION_COMPATIBLE = 如果版本号相兼容，返回true。
PACKAGE_VERSION_UNSUITABLE = 如果不适合任何版本，返回true。
```

　　下面这些变量将会被find_package命令检查，用以确定配置文件是否提供了可接受的版本。在find_package命令返回后，这些变量就不可用了。如果版本可接受，下述的变量会被设置：

```
<package>_VERSION       = 提供的完整的版本字符串。
<package>_VERSION_MAJOR = 如果被请求了，那么它是major版本号，否则是0。
<package>_VERSION_MINOR = 如果被请求了，那么它是minor版本号，否则是0。
<package>_VERSION_PATCH = 如果被请求了，那么它是patch版本号，否则是0。
<package>_VERSION_TWEAK = 如果被请求了，那么它是tweak版本号，否则是0。
<package>_VERSION_COUNT = 版本号包含几部分，0到4。
```

然后，对应的包配置文件才会被加载。当多个包配置文件都可用时，并且这些包的版本文件都与请求的版本兼容，选择哪个包将会是不确定的。不应该假设cmake会选择最高版本或者是最低版本。（以上的若干段是对find_package中版本匹配步骤的描述，并不需要用户干预——译注。）

　　配置模式提供了一种高级接口和搜索步骤的接口。这些被提供的接口的大部分是为了完整性的要求，以及在模块模式下，包被find-module加载时供内部使用。大多数用户仅仅应该调用：

```
  find_package(<package> [major[.minor]] [EXACT] [REQUIRED|QUIET])
```

来查找包。鼓励那些需要提供CMake包配置文件的包维护人员应该命名这些文件并安装它们，这样下述的整个过程将会找到它们而不需要使用附加的选项。

　　CMake为包构造了一组可能的安装前缀。在每个前缀下，若干个目录会被搜索，用来查找配置文件。下述的表格展示了待搜索的路径。每个条目都是专门为Windows(W)，UNIX(U)或者Apple(A)约定的安装树指定的。

```
<prefix>/                                               (W)
<prefix>/(cmake|CMake)/                                 (W)
<prefix>/<name>*/                                       (W)
<prefix>/<name>*/(cmake|CMake)/                         (W)
<prefix>/(share|lib)/cmake/<name>*/                     (U)
<prefix>/(share|lib)/<name>*/                           (U)
<prefix>/(share|lib)/<name>*/(cmake|CMake)/             (U)
```

　　在支持OS X平台和Application Bundles的系统上，包含配置文件的框架或者bundles会在下述的路径中被搜索：

```
<prefix>/<name>.framework/Resources/                    (A)
<prefix>/<name>.framework/Resources/CMake/              (A)
<prefix>/<name>.framework/Versions/*/Resources/         (A)
<prefix>/<name>.framework/Versions/*/Resources/CMake/   (A)
<prefix>/<name>.app/Contents/Resources/                 (A)
<prefix>/<name>.app/Contents/Resources/CMake/           (A)
```

　　在所有上述情况下，<name>是区分大小写的，并且对应于在<package>或者由NAMES给定的任何一个名字。

　　这些路径集用来与那些在各自的安装树上提供了配置文件的工程协作。上述路径中被标记为(W)的是专门为Windows上的安装设置的，其中的<prefix>部分可能是一个应用程序的顶层安装路径。那些被标记为(U)的是专门为UNIX平台上的安装设置的，其中的<prefix>被多个包共用。这仅仅是个约定，因此，所有(W)和(U)路径在所有平台上都仍然会被搜索。那些被标记为(A)的路径是专门为Apple平台上的安装设置的。CMake变量CMAKE_FIND_FRAMEWORK和CMAKE_FIND_APPBUNDLE确定了偏好的顺序，如下所示：

　　安装前缀是通过以下步骤被构建出来的。如果指定了NO_DEFAULT_PATH选项，所有NO_*选项都会被激活。

　　1、搜索在cmake特有的cache变量中指定的搜索路径。这些变量是为了在命令行中用-DVAR=value选项指定而设计的。通过指定NO_CMAKE_PATH选项可以跳过该搜索路径。搜索路径还包括：

```
CMAKE_PREFIX_PATH
CMAKE_FRAMEWORK_PATH
CMAKE_APPBUNDLE_PATH
```

　　2、搜索cmake特有的环境变量。这些变量是为了在用户的shell配置中进行配置而设计的。通过指定NO_CMAKE_ENVIRONMENT_PATH选项可以跳过该路径。搜索的路径包括：

```
<package>_DIR
CMAKE_PREFIX_PATH
CMAKE_FRAMEWORK_PATH
CMAKE_APPBUNDLE_PATH
```

　　3、搜索HINTS选项指定的路径。这些路径应该是由操作系统内省时计算产生的，比如由其它已经找到的项的位置而提供的线索。硬编码的参考路径应该在PATHS选项中指定。

　　4、搜索标准的系统环境变量。如果指定了NO_SYSTEM_ENVIRONMENT_PATH选项，这些路径会被跳过。以/bin或/sbin结尾的路径条目会被自动转换为它们的父路径。搜索的路径包括：

PATH
　　5、搜索在CMake GUI中最新配置过的工程的构建树。可以通过设置NO_CMAKE_BUILDS_PATH选项来跳过这些路径。这是为了在用户正在依次构建多个相互依赖的工程时而准备的。

　　6、搜索存储在CMake用户包注册表中的路径。通过设置NO_CMAKE_PACKAGE_REGISTRY选项可以跳过这些路径。当CMake用export(PACKAGE<name>)配置一个工程时，这些路径会被存储在注册表中。参见export(PACKAGE)命令的文档阅读更多细节。

　　7、搜索在当前系统的平台文件中定义的cmake变量。可以用NO_CMAKE_SYSTEM_PATH选项跳过这些路径。

```
CMAKE_SYSTEM_PREFIX_PATH
CMAKE_SYSTEM_FRAMEWORK_PATH
CMAKE_SYSTEM_APPBUNDLE_PATH
```

　　8、搜索由PATHS选项指定的路径。这些路径一般是硬编码的参考路径。

　　在Darwin或者支持OS X 框架的系统上，cmake变量CMAKE_FIND_FRAMEWORK可以用来设置为空，或者下述值之一：

```
"FIRST"  - 在标准库或头文件之前查找框架。在Darwin系统上这是默认选项。
"LAST"   - 在标准库或头文件之后查找框架。
"ONLY"   - 仅仅查找框架。
"NEVER" - 从不查找框架。
```

　　在Darwin或者支持OS X Application Bundles的系统，cmake变量CMAKE_FIND_APPBUNDLE可以被设置为空或者下面这些值中的一个：

```
"FIRST"  - 在标准库或头文件之前查找application bundles。在Darwin系统上这是默认选项。
"LAST"   - 在标准库或头文件之后查找application bundles。
"ONLY"   - 仅仅查找application bundles。
"NEVER" - 从不查找application bundles。
```

　　CMake变量CMAKE_FIND_ROOT_PATH指定了一个或者多个优先于其他搜索路径的搜索路径。该变量能够有效地重新定位在给定位置下进行搜索的根路径。该变量默认为空。当使用交叉编译时，该变量十分有用：用该变量指向目标环境的根目录，然后CMake将会在那里查找。默认情况下，在CMAKE_FIND_ROOT_PATH中列出的路径会首先被搜索，然后是“非根”路径。该默认规则可以通过设置CMAKE_FIND_ROOT_PATH_MODE_LIBRARY做出调整。在每次调用该命令之前，都可以通过设置这个变量来手动覆盖默认行为。如果使用了NO_CMAKE_FIND_ROOT_PATH变量，那么只有重定位的路径会被搜索。

　　默认的搜索顺序的设计逻辑是按照使用时从最具体到最不具体。通过多次调用find_library命令以及NO_*选项，可以覆盖工程的这个默认顺序：

```
    find_library(<VAR> NAMES name PATHS paths... NO_DEFAULT_PATH)
    find_library(<VAR> NAMES name)
```

　　只要这些调用中的一个成功返回，结果变量就会被设置并且被存储到cache中；这样随后的调用都不会再行搜索。如果那找到的库是一个框架，VAR将会被设置为指向框架<完整路径>/A.framework 的完整路径。当一个指向框架的完整路径被用作一个库文件，CMake将使用-framework A，以及-F<完整路径>这两个选项将框架连接到目标上。

　　参见cmake_policy()命令的文档中关于NO_POLICY_SCOPE选项讨论。 
　　 
　　 
　　

## CMD#32 find_path

搜索包含某个文件的路径

```
   find_path(<VAR> name1 [path1 path2 ...])
```

　　在多数情况下，使用上述的精简命令格式就足够了。它与命令find_path(<VAR> name1 [PATHS path1 path2 ...])等价。

```
   find_path(
             <VAR>
             name | NAMES name1 [name2 ...]
             [HINTS path1 [path2 ... ENV var]]
             [PATHS path1 [path2 ... ENV var]]
             [PATH_SUFFIXES suffix1 [suffix2 ...]]
             [DOC "cache documentation string"]
             [NO_DEFAULT_PATH]
             [NO_CMAKE_ENVIRONMENT_PATH]
             [NO_CMAKE_PATH]
             [NO_SYSTEM_ENVIRONMENT_PATH]
             [NO_CMAKE_SYSTEM_PATH]
             [CMAKE_FIND_ROOT_PATH_BOTH |
              ONLY_CMAKE_FIND_ROOT_PATH |
              NO_CMAKE_FIND_ROOT_PATH]
            )
 ```
 
　　该命令用于给定名字文件所在的路径。一条名为<VAR>的cache条目会被创建，并存储该命令的执行结果。如果在某个路径下发现了该文件，该结果会被存储到该变量中；除非该变量被清除，该次搜索不会继续进行。如果没有找到，存储的结果将会是<VAR>-NOTFOUND，并且当下一次以相同的变量名调用find_path命令时，该命令会再一次尝试搜索该文件。需要搜索的文件名通过在NAMES选项后面的列出来的参数来确定。附加的搜索位置可以在PATHS选项之后指定。如果在PATHS或者HINTS命令中还指定了ENV var选项，环境变量var将会被读取并从一个系统环境变量转换为一个cmake风格的路径list。比如，ENV PATH是列出系统path变量的一种方法。参数DOC将用来作为该变量在cache中的注释。PATH_SUFFIXES指定了在每个搜索路径下的附加子路径。

　　如果指定了NO_DEFAULT_PATH选项，那么没有其它附加的路径会被加到搜索过程中。如果并未指定NO_DEFAULT_PATH选项，搜索的过程如下：

　　1、搜索cmake专有的cache变量中的路径。这种用法是为了在命令行中用选项-DVAR=value指定搜索路径。如果指定了NO_CMAKE_PATH选项，该路径会被跳过。搜索路径还包括：

```
对于每个在CMAKE_PREFIX_PATH中的<prefix>/，路径<prefix>/include
CMAKE_INCLUDE_PATH
CMAKE_FRAMEWORK_PATH
```

　　2、搜索cmake专有的环境变量中指定的路径。这种用法是为了在用户的shell配置中设置指定的搜索路径。如果指定了NO_CMAKE_ENVIRONMENT_PATH选项，该路径会被跳过。搜索路径还包括：

```
对于每个在CMAKE_PREFIX_PATH中的<prefix>/，路径<prefix>/include
CMAKE_INCLUDE_PATH
CMAKE_FRAMEWORK_PATH
```

　　3、搜索由HINTS选项指定的路径。这些路径应该是由系统内省时计算得出的路径，比如由其它已经发现的项目提供的线索。硬编码的参考路径应该在PATHS选项中指定。

　　4、搜索标准的系统环境变量。通过指定选项NO_SYSTEM_ENVIRONMENT_PATH可以跳过搜索环境变量。搜索的路径还包括：

```
PATH
INCLUDE
```

　　5、查找在为当前系统的平台文件中定义的cmake变量。如果指定了NO_CMAKE_SYSTEM_PATH选项，该路径会被跳过。搜索的路径还包括：

```
对于每个在CMAKE_SYSTEM_PREFIX_PATH中的<prefix>，路径<prefix>/include 
CMAKE_SYSTEM_LIBRARY_PATH
CMAKE_SYSTEM_FRAMEWORK_PATH
```

　　6、搜索PATHS选项或者精简版命令指定的路径。这些通常是硬编码的推荐搜索路径。

　　在Darwin或者支持OS X 框架的系统上，cmake变量CMAKE_FIND_FRAMEWORK可以用来设置为空，或者下述值之一：

```
"FIRST"  - 在标准库或头文件之前查找框架。在Darwin系统上这是默认选项。
"LAST"   - 在标准库或头文件之后查找框架。
"ONLY"   - 仅仅查找框架。
"NEVER" - 从不查找框架。
```

　　在Darwin或者支持OS X Application Bundles的系统，cmake变量CMAKE_FIND_APPBUNDLE可以被设置为空或者下面这些值中的一个：

```
"FIRST"  - 在标准库或头文件之前查找application bundles。在Darwin系统上这是默认选项。
"LAST"   - 在标准库或头文件之后查找application bundles。
"ONLY"   - 仅仅查找application bundles。
"NEVER" - 从不查找application bundles。
```

　　CMake变量CMAKE_FIND_ROOT_PATH指定了一个或者多个优先于其他搜索路径的搜索路径。该变量能够有效地重新定位在给定位置下进行搜索的根路径。该变量默认为空。当使用交叉编译时，该变量十分有用：用该变量指向目标环境的根目录，然后CMake将会在那里查找。默认情况下，在CMAKE_FIND_ROOT_PATH中列出的路径会首先被搜索，然后是“非根”路径。该默认规则可以通过设置CMAKE_FIND_ROOT_PATH_MODE_LIBRARY做出调整。在每次调用该命令之前，都可以通过设置这个变量来手动覆盖默认行为。如果使用了NO_CMAKE_FIND_ROOT_PATH变量，那么只有重定位的路径会被搜索。

　　默认的搜索顺序的设计逻辑是按照使用时从最具体到最不具体的路径。通过多次调用find_path命令以及NO_*选项，可以覆盖工程的这个默认顺序：

```
    find_path(<VAR> NAMES name PATHS paths... NO_DEFAULT_PATH)
    find_path(<VAR> NAMES name)
```

　　只要这些调用中的一个成功返回，结果变量就会被设置并且被存储到cache中；这样随后的调用都不会再行搜索。在搜索框架时，如果以A/b.h的格式指定文件，那么该框架搜索过程会搜索A.framework/Headers/b.h。如果找到了该路径，它将会被设置为框架的路径。CMake将把它转换为正确的-F选项来包含该文件。

# CMD#33 find_program

查找可执行程序

```
   find_program(<VAR> name1 [path1 path2 ...])
```

　　这是该命令的精简格式，它在大多数场合下都够用了。命令find_program(<VAR> name1 [PATHS path1 path2 ...])是它的等价形式。

```
   find_program(
             <VAR>
             name | NAMES name1 [name2 ...]
             [HINTS path1 [path2 ... ENV var]]
             [PATHS path1 [path2 ... ENV var]]
             [PATH_SUFFIXES suffix1 [suffix2 ...]]
             [DOC "cache documentation string"]
             [NO_DEFAULT_PATH]
             [NO_CMAKE_ENVIRONMENT_PATH]
             [NO_CMAKE_PATH]
             [NO_SYSTEM_ENVIRONMENT_PATH]
             [NO_CMAKE_SYSTEM_PATH]
             [CMAKE_FIND_ROOT_PATH_BOTH |
              ONLY_CMAKE_FIND_ROOT_PATH |
              NO_CMAKE_FIND_ROOT_PATH]
            )
```

　　该命令用于查找程序。一个名为<VAR>的cache条目会被创建用来存储该命令的结果。如果该程序被找到了，结果会存储在该变量中，搜索过程将不会再重复，除非该变量被清除。如果没有找到，结果将会是<VAR>-NOTFOUND，并且下次以相同的变量调用该命令时，还会做搜索的尝试。被搜索的程序的名字由NAMES选项后列出的参数指定。附加的搜索位置可以在PATHS参数后指定。如果在HINTS或者PATHS选项后有ENV var参数，环境变量var将会被读取并从系统环境变量转换为cmake风格的路径list。比如ENV PATH是一种列出所有系统path变量的方法。DOC后的参数将会被用作cache中的注释字符串。PATH_SUFFIXES指定了在每个搜索路径下要检查的附加子路径。

　　如果指定了NO_DEFAULT_PATH选项，那么搜索的过程中不会有其他的附加路径。如果没有指定该选项，搜索过程如下：

　　1、搜索cmake特有的cache变量指定的路径。这些变量是在用cmake命令行时，通过-DVAR=value指定的变量。如果指定了NO_CMAKE_PATH选项，这些路径会被跳过。搜索的路径还包括：

```
对于每个在CMAKE_PREFIX_PATH中的<prefix>，路径<prefix>/[s]bin 
CMAKE_PROGRAM_PATH
CMAKE_APPBUNDLE_PATH
```

　　2、搜索cmake特有的环境变量指定的路径。这些变量是用户的shell配置中设置的变量。如果指定了NO_CMAKE_ENVIRONMENT_PATH选项，这些路径会被跳过。搜索的路径还包括：

```
对于每个在CMAKE_PREFIX_PATH中的<prefix>，路径<prefix>/[s]bin 
CMAKE_PROGRAM_PATH
CMAKE_APPBUNDLE_PATH
```

　　3、搜索由HINTS选项指定的路径。这些路径是系统内省(introspection)估算出的路径，比如由另一个已经发现的程序的地址提供的参考信息。硬编码的推荐路径应该通过PATHS选项指定。

　　4、查找标准的系统环境变量。如果指定了NO_SYSTEM_ENVIRONMENT_PATH选项，这些路径会被跳过。搜索的路径还包括：

```
PATH
```

　　5、查找在为当前系统的平台文件中定义的cmake变量。如果指定了NO_CMAKE_SYSTEM_PATH选项，该路径会被跳过。搜索的路径还包括：

```
对于每个在CMAKE_SYSTEM_PREFIX_PATH中的<prefix>，路径<prefix>/[s]bin 
CMAKE_SYSTEM_PROGRAM_PATH
CMAKE_SYSTEM_APPBUNDLE_PATH
```

　　6、搜索PATHS选项或者精简版命令指定的路径。这些通常是硬编码的推荐搜索路径。

　　在Darwin或者支持OS X 框架的系统上，cmake变量CMAKE_FIND_FRAMEWORK可以设置为空，或者下述值之一：

```
"FIRST"  - 在标准库或头文件之前查找框架。在Darwin系统上这是默认选项。
"LAST"   - 在标准库或头文件之后查找框架。
"ONLY"   - 仅仅查找框架。
"NEVER" - 从不查找框架。
```

　　在Darwin或者支持OS X Application Bundles的系统，cmake变量CMAKE_FIND_APPBUNDLE可以被设置为空或者下面这些值中的一个：

```
"FIRST"  - 在标准程序之前查找application bundles。在Darwin系统上这是默认选项。
"LAST"   - 在标准程序之后查找application bundles。
"ONLY"   - 仅仅查找application bundles。
"NEVER" - 从不查找application bundles。
```

　　CMake变量CMAKE_FIND_ROOT_PATH指定了一个或者多个优先于其他搜索路径的搜索路径。该变量能够有效地重新定位在给定位置下进行搜索的根路径。该变量默认为空。当使用交叉编译时，该变量十分有用：用该变量指向目标环境的根目录，然后CMake将会在那里查找。默认情况下，在CMAKE_FIND_ROOT_PATH中列出的路径会首先被搜索，然后是“非根”路径。该默认规则可以通过设置CMAKE_FIND_ROOT_PATH_MODE_LIBRARY做出调整。在每次调用该命令之前，都可以通过设置这个变量来手动覆盖默认行为。如果使用了NO_CMAKE_FIND_ROOT_PATH变量，那么只有重定位的路径会被搜索。

　　默认的搜索顺序的设计逻辑是按照使用时从最具体到最不具体。通过多次以NO_*选项调用find_program命令，可以覆盖工程的这个默认顺序：

```
    find_library(<VAR> NAMES name PATHS paths... NO_DEFAULT_PATH)
    find_library(<VAR> NAMES name)
```

　　只要这些调用中的一个成功返回，结果变量就会被设置并且被存储到cache中；这样随后的调用都不会再行搜索。

## CMD#34 fltk_wrap_ui

创建FLTK用户界面包装器。

```
  fltk_wrap_ui(resultingLibraryName source1
               source2 ... sourceN )
```

　　为所有列出的.fl和.fld文件生成.h和.cxx文件。这些生成的.h和.cxx文件将会加到变量resultingLibraryName_FLTK_UI_SRCS中，它也会加到你的库中。 
　　 
　　

## CMD#35 foreach

对一个list中的每一个变量执行一组命令。

```
  foreach(loop_var arg1 arg2 ...)
    COMMAND1(ARGS ...)
    COMMAND2(ARGS ...)
    ...
  endforeach(loop_var)
```

　　所有的foreach和与之匹配的endforeach命令之间的命令会被记录下来而不会被调用。等到遇到endforeach命令时，先前被记录下来的命令列表中的每条命令都会为list中的每个变量调用一遍。在每次迭代中，循环变量${loop_var}将会被设置为list中的当前变量值。

```
  foreach(loop_var RANGE total)
  foreach(loop_var RANGE start stop [step])
```

foreach命令也可以遍历一个人为生成的数据区间。遍历的方式有三种：

```
*如果指定了一个数字，区间是[0, total]。
*如果指定了两个数字，区间将会是第一个数字到第二个数字。
*第三个数字是从第一个数字遍历到第二个数字时的步长。
```

```
  foreach(loop_var IN [LISTS [list1 [...]]]
                      [ITEMS [item1 [...]]])
```

　　该命令的含义是：精确遍历一个项组成的list。LISTS选项后面是需要被遍历的list变量的名字，包括空元素（一个空字符串是一个零长度list）。ITEMS选项结束了list参数的解析，然后在迭代中引入所有在其后出现的项。（猜测是用list1中的项item1，依次类推，为循环变量赋值。——译注） 
　　 
　　

## CMD#36 : function

开始记录一个函数，为以后以命令的方式调用它做准备。

```
  function(<name> [arg1 [arg2 [arg3 ...]]])
    COMMAND1(ARGS ...)
    COMMAND2(ARGS ...)
    ...
  endfunction(<name>)
```

　　定义一个名为<name>的函数，它以arg1 arg2 arg3 (...)为参数。在function和endfunction之前列出的命令，在函数被调用之前是不会被调用的。当函数被调用时，在函数中记录的那些命令首先会用传进去的参数替换掉形参（${arg1}）；然后跟正常命令一样去调用这些命令。除了形参，你还可以引用这些变量：ARGC为传递给函数的变量个数，ARGV0 ARGV1 ARGV2 ...表示传到函数中的实参值。这些变量为编写可选参数函数提供了便利。此外，ARGV保留了一个该函数所有实参的list，ARGN保留了函数形参列表以后的所有参数列表。

　　参见cmake_policy()命令文档中function内部策略行为的相关行为。

## CMD#37 get_cmake_property

获取一个CMake实例的属性。

```
  get_cmake_property(VAR property)
```

　　从指定的CMake实例中获取属性。属性的值存储在变量VAR中。如果属性不存在，CMake会报错。一些会被支持的属性包括：VATIABLES，COMMANDS，MACROS以及COMPONENTS。

## CMD#38 get_directory_property

获取DIRECTORY域中的某种属性。

```
  get_directory_property(<variable> [DIRECTORY <dir>] <prop-name>)
```

　　在指定的变量中存储路径（directory）域中的某种属性。如果该属性没有被定义，将会返回空字符串。DIRECTORY参数指定了要取出的属性值的另一个路径。指定的路径必须已经被CMake遍历过了。

```
   get_directory_property(<variable> [DIRECTORY <dir>]
                         DEFINITION <var-name>)
```

　　该命令从一个路径中获取一个变量的定义。这种格式在从另一个路径中获取变量的定义时比较有用。

## CMD#39 get_filename_component

得到一个完整文件名中的特定部分。

```
  get_filename_component(<VAR> FileName
                         PATH|ABSOLUTE|NAME|EXT|NAME_WE|REALPATH
                         [CACHE])
```

　　将变量<VAR>设置为路径(PATH)，文件名(NAME)，文件扩展名(EXT)，去掉扩展名的文件名(NAME_WE)，完整路径(ABSOLUTE)，或者所有符号链接被解析出的完整路径(REALPATH)。注意，路径会被转换为Unix的反斜杠(/)，并且没有结尾的反斜杠。该命令已经考虑了最长的文件扩展名。如果指定了CACHE选项，得到的变量会被加到cache中。

```
  get_filename_component(<VAR> FileName
                         PROGRAM [PROGRAM_ARGS <ARG_VAR>]
                         [CACHE])
```

　　在FileName中的程序将会在系统搜索路径中被查找，或者是一个完整路径。如果与PRPGRAM一起给定了PROGRAM_ARGS选项，那么任何在Filename字符串中出现的命令行中选项将会从程序名中分割出来并存储在变量<ARG_VAR>中。这可以用来从一个命令行字符串中分离程序名及其选项。 
　　 
　　 
　　

## CMD#40 get_property

获取一个属性值

```
  get_property(<variable>
               <GLOBAL             |
                DIRECTORY [dir]    |
                TARGET    <target> |
                SOURCE    <source> |
                TEST      <test>   |
                CACHE     <entry>  |
                VARIABLE>
               PROPERTY <name>
               [SET | DEFINED | BRIEF_DOCS | FULL_DOCS])
```

　　获取在某个域中一个对象的某种属性值。第一个参数指定了存储属性值的变量。第二个参数确定了获取该属性的域。域的选项仅限于：

```
GLOBAL 域是唯一的，它不接受域名字。
DIRECTORY域默认为当前目录，但是其他的路径（已经被CMake处理过）可以以相对路径或完整路径的方式跟在该域后面。
TARGET域后面必须跟有一个已有的目标名。
SOURCE域后面必须跟有一个源文件名。
TEST域后面必须跟有一个已有的测试。
CACHE域后面必须跟有一个cache条目。
VARIABLE域是唯一的，它不接受域名字。
PROPERTY选项是必须的，它后面紧跟要获取的属性名。如果该属性没有被设置，该命令将返回空值。如果给定了SET选项，那么返回值会被设置为一个布尔值，用来指示该属性是否被设置过。如果给定了DEFINED选项，那么返回值会被设置为一个布尔值，用来指示该属性是否被类似于define_property的命令定义过。如果指定了BRIEF_DOCS或者FULL_DOCS选项，那么该变量将会被设置为被查询属性的文档的字符串。如果被请求的属性的文档没有被定义，将返回NOTFOUND。
```

## CMD#41 get_source_file_property

为一个源文件获取一种属性值。

```
  get_source_file_property(VAR file property)
```

　　从一个源文件中获取某种属性值。这个属性值存储在变量VAR中。如果该属性没有被找到，VAR会被设置为NOTFOUND。使用set_source_files_proterties命令来设置属性值。源文件属性通常用来控制文件如何被构建。一个必定存在的属性是LOCATION。

## CMD#42 get_target_property

从一个目标中获取一个属性值。

```
  get_target_property(VAR target property)
```

　　从一个目标中获取属性值。属性的值会被存储在变量VAR中。如果该属性没有被发现，VAR会被设置为NOTFOUND。使用set_target_properties命令来设置属性值。属性值一般用于控制如何去构建一个目标，但是有些属性用来查询目标的信息。该命令可以获取当前已经被构建好的任意目标的属性。该目标不一定存在于当前的CMakeLists.txt文件中。

## CMD#43 get_test_property

获取一个测试的属性。

```
  get_test_property(test VAR property)
```

　　从指定的测试中获取某种属性。属性值会被存储到变量VAR中。如果没有找到该属性，CMake将会报错。你可以使用命令cmake --help-property-list来获取标准属性的清单。

## CMD#44 if

条件执行一组命令。

```
  if(expression)
    # then section.
    COMMAND1(ARGS ...)
    COMMAND2(ARGS ...)
    ...
  elseif(expression2)
    # elseif section.
    COMMAND1(ARGS ...)
    COMMAND2(ARGS ...)
    ...
  else(expression)
    # else section.
    COMMAND1(ARGS ...)
    COMMAND2(ARGS ...)
    ...
  endif(expression)
```

　　评估给定的表达式。如果结果是true，在THEN段的命令就会被调用。否则，在ELSE区段的命令会被调用。ELSEIF和ELSE区段是可选的 。可以有多个ELSEIF子句。注意，在else和elseif子句中的表达式也是可选的。判断条件可以用长表达式，并且表达式有约定的优先级顺序。括号中的表达式会首先被调用；然后是一元运算符，比如EXISTS，COMMAND以及DEFINED；然后是EQUAL，LESS，GREATER，STRLESS，STRGREATER，STREQUAL，MATCHES；然后是NOT运算符，最后是AND，OR运算符。几种可能的表达式是： 
　　
```
  if(<常量>)
  #如果<常量>是1，ON，YES，TRUE，Y或者非0数值，那么表达式为真；如果<常量>是0，OFF，NO，FALSE，N，IGNORE，""，或者以'-NOTFOUND'为后缀，那么表达式为假。这些布尔常量值是大小写无关的。
  if(<变量>)
  #如果<变量>的值不是一个false常量，表达式为真。
  if(NOT <表达式>)
  #如果<表达式>的值是false的话，真个表达式为真。
  if(<表达式1> AND <表达式2>)
  #如果两个表达式都为真，整个表达式为真。
  if(<表达式1> OR <表达式2>)
  #只要有一个表达式为真，整个表达式为真。
  if(COMMAND command-name)
  #如果给出的名字是一个可以被调用的命令，宏，或者函数的话，整个表达式的值为真。
  if(POLICY policy-id)
  #如果给出的名字是一个已有的策略（格式是CMP<NNNN>），表达式为真。
  if(TARGET 目标名)
  #如果给出的名字是一个已有的构建目标或导入目标的话，表达式为真。
  if(EXISTS 文件名)
  if(EXISTS 路径名)
  #如果给出的文件名或路径名存在，表达式为真。该命令只对完整路径有效。
  if(file1 IS_NEWER_THAN file2)
  #如果file1比file2更新或者其中的一个文件不存在，那么表达式为真。该命令只对完整路径有效。
  if(IS_DIRECTORY directory-name)
  #如果给定的名字是一个路径，表达式返回真。该命令只对完整路径有效。
  if(IS_SYMLINK file-name)
  #如果给定的名字十一个符号链接的话，表达式返回真。该命令只对完整路径有效。
  if(IS_ABSOLUTE path)
  #如果给定的路径是一个绝对路径的话，表达式返回真。
  if(variable MATCHES regex)
  if(string MATCHES regex)
  #如果给定的字串或变量值域给定的正则表达式匹配的话，表达式返回真。
  if(variable LESS number)
  if(string LESS number)
  if(variable GREATER number)
  if(string GREATER number)
  if(variable EQUAL number)
  if(string EQUAL number)
  #如果给定的字串或变量值是一个有效的数字并且不等号或等号满足的话，表达式返回真。
  if(variable STRLESS string)
  if(string STRLESS string)
  if(variable STRGREATER string)
  if(string STRGREATER string)
  if(variable STREQUAL string)
  if(string STREQUAL string)
  #如果给定的字串或变量值依字典序小于（或者大于，或者等于）右边给出的字串或变量值的话，表达式返回真。
  if(version1 VERSION_LESS version2)
  if(version1 VERSION_EQUAL version2)
  if(version1 VERSION_GREATER version2)
  #对版本号的各部分依次比较（版本号格式是major[.minor[.patch[.tweak]]]）version1和version2的大小。
  if(DEFINED variable)
  #如果给定的变量被定义了的话，该表达式为真。如果变量被设置了，它的值是真是假都无所谓。
  if((expression) AND (expression OR (expression)))
  #在小括号内的表达式会首先被计算，然后才按照先前介绍的运算来计算。有内嵌的括号时，最里的括号会作为包含它们的表达式的计算过程的一部分。IF语句在CMake的历史上出现的相当早，它拥有一些需要特殊介绍的便捷特性。IF表达式只有在其中有一个单一的保留值的时候，才会精简操作（即不做变量展开——译注）；这些保留值包括：如果是大小写无关的 ON，1， YES，TRUE，Y，它返回真；如果是OFF，0，NO，FALSE，N，NOTFOUND，*-NOTFOUND，IGNORE，它返回假。这种特性非常合理，它为新作者提供了一种不需要精确匹配true或者false的便利性。这些值会被当做变量处理，即使它们没有使用${}语法的时候，也会被解引用。
  #这意味着，如果你写下了这样的语句： 
  if (boobah)  
  #CMake将会把它当做你写了 
  if (${boobah})
  #来处理。类似地，如果你写了
  if (fubar AND sol) 
  #CMake将会便捷地把它解释为 
  if ("${fubar}" AND "${sol}")
  #上述两例的后者确实是正确的书写方式，但是前者也是可行的。if语句中只有某些操作有这种特殊的变量处理方式。这些特殊的语句包括：
  #    对于MATCHES运算符，待匹配的左边的参数首先被检查，用来确认它是否是一个已经定义的变量；如果是，该变量的值会被使用，否则就会用它的原始值。
  #    如果MATCHES运算符没有左边的参数，它返回false，但不产生错误。 
  #    LESS，GREATER，EQUAL运算符的左边的参数和右边的参数会被独立测试，用来确认它们是否是被定义的变量；如果是，使用它们被定义的值，否则使用它们的原始值。
  #    STRLESS，STRGREATER，STREQUAL运算符的左边的参数和右边的参数会被独立测试，用来确认它们是否是被定义的变量；如果是，使用它们被定义的值，否则使用它们的原始值。
  #    VERSIONLESS，VERSIONGREATER，VERSIONEQUAL运算符的左边的参数和右边的参数会被独立测试，用来确认它们是否是被定义的变量；如果是，使用它们被定义的值，否则使用它们的原始值。
  #    NOT运算符右边的参数会被测试用来确定它是否是布尔常量，如果是，就用这个常量；否则它会被当做一个变量然后被解引用。
  #    AND和OR运算符的左边的参数和右边的参数会被独立测试，用来确认它们是否是布尔常量；如果是，就用这个常量，否则它们会被当做变量然后被解引用。
```

## CMD#45 include

从给定的文件中读取CMake的列表文件。

```
  include(<file|module> [OPTIONAL] [RESULT_VARIABLE <VAR>]
                        [NO_POLICY_SCOPE])
```

　　从给定的文件中读取CMake的清单文件代码。在清单文件中的命令会被立即处理，就像它们是写在这条include命令展开的地方一样。如果指定了OPTIONAL选项，那么如果被包含文件不存在的话，不会报错。如果指定了RESULT_VARIABLE选项，那么var或者会被设置为被包含文件的完整路径，或者是NOTFOUND，表示没有找到该文件。

　　如果指定的是一个模块(module)而不是一个文件，查找的对象会变成路径CMAKE_MODULE_PATH下的文件<modulename>.camke。

　　参考cmake_policy()命令文档中关于NO_POLICY_SCOPE选项的讨论。 
　　 
　　

## CMD#46 include_directories

为构建树添加包含路径。

```
  include_directories([AFTER|BEFORE] [SYSTEM] dir1 dir2 ...)
```

　　将给定的路径添加到编译器搜索包含文件（.h文件）的路径列表中。缺省情况下，该路径会被附加在当前路径列表的后面。这种缺省行为可以通过设置CMAKE_include_directories_BEFORE变量为ON被改变。通过将该变量改变为BEFORE或AFTER，你可以在追加和附加在前端这两种方式中选择，而不用理会缺省设置。如果指定了SYSTEM选项，编译器将会认为该路径是某种平台上的系统包含路径。 
　　 
　　

## CMD#47 include_external_msproject

在一个workspace中包含一个外部的Microsoft工程。　　　　　　

```
　　include_external_msproject(projectname location dep1 dep2 ...)
```

　　在生成的workspace文件中包含一个外部的Microsoft工程。它会创建一个名为[projectname]的目标。这个目标可以用在add_dependencies命令中让其他工程依赖于这个外部工程。当前版本下，该命令在UNIX平台上不会做任何事情。

## CMD#48 include_regular_expression

设置用于依赖性检查的正则表达式。

```
  include_regular_expression(regex_match [regex_complain])
```

　　设置依赖性检查的正则表达式。这有匹配正则表达式regex_match的文件会成为依赖性跟踪的对象。只有匹配regex_complain的文件，在找不到它们的时候才会给出警告（标准头文件不会被搜索）。正则表达式的默认值是：

```
  　　regex_match    = "^.*$" (匹配所有文件)
　　　regex_complain = "^$" (仅匹配空字符串)
```
　　　
## CMD#49 install

指定在安装时要运行的规则。 
　　该命令为一个工程生成安装规则。在某一源文件路径中，调用这条命令所指定的规则会在安装时按顺序执行。在不同路径之间的顺序未定义。

　　该命令有诸多版本。其中的一些版本定义了文件以及目标的安装属性。这多个版本的公共属性都有所涉及，但是只有在指定它们的版本中，这些属性才是合法的（下面的DESTIONATION到OPTIONAL的选项列表是公共属性。——译注）。

　　DESTINATION选项指定了一个文件会安装到磁盘的哪个路径下。若果给出的是全路径（以反斜杠或者驱动器名开头），它会被直接使用。如果给出的是相对路径，它会被解释为相对于CMAKE_INSTALL_PREFIX的值的相对路径。

　　PERMISSIONS选项制定了安装文件需要的权限。合法的权限有：OWNER_READ，OWNER_WRITE，OWNER_EXECUTE，GROUP_READ，GROUP_WRITE，GROUP_EXECUTE，WORLD_READ，WORLD_WRITE，WORLD_EXECUTE，SETUID和SETGID。对于在某些特定的平台上没有意义的权限，在这些平台上会忽略这些选项。

　　CONFIGURATIONS选项指定了该安装规则将会加诸之上的一系列的构建配置（Debug，Release，等等）。

　　COMPONENT选项指定了该安装规则相关的一个安装部件的名字，比如runtime或development。对于那些指定安装部件的安装过程来说，在安装时只有与给定的部件名相关的安装规则会被执行。对于完整安装，所有部件都会被安装。

　　RENAME选项为一个可能不同于原始文件的已经安装的文件指定另一个名字。重命名只有在该命令正在安装一个单一文件时才被允许（猜测是为了防止文件名冲突时覆盖掉旧文件。——译注）。

　　OPTIONAL选项表示要安装的文件不存在不会导致错误。

　　TARGETS版本的install命令

```
  install(TARGETS targets... [EXPORT <export-name>]
          [[ARCHIVE|LIBRARY|RUNTIME|FRAMEWORK|BUNDLE|
            PRIVATE_HEADER|PUBLIC_HEADER|RESOURCE]
           [DESTINATION <dir>]
           [PERMISSIONS permissions...]
           [CONFIGURATIONS [Debug|Release|...]]
           [COMPONENT <component>]
           [OPTIONAL] [NAMELINK_ONLY|NAMELINK_SKIP]
          ] [...])
```

　　TARGETS格式的install命令规定了安装工程中的目标（targets）的规则。有5中可以被安装的目标文件：ARCHIVE，LIBRARY，RUNTIME，FRAMEWORK，和BUNDLE。除了被标记为MACOSX_BUNDLE属性的可执行文件被当做OS X上的BUNDLE目标外，其他的可执行文件都被当做RUNTIME目标。静态链接的库文件总是被当做ARCHIVE目标。模块库总是被当做LIBRARY目标。对于动态库不是DLL格式的平台来说，动态库会被当做LIBRARY目标来对待，被标记为FRAMEWORK的动态库是例外，它们被当做OS X上的FRAMEWORK目标。对于DLL平台而言，动态库的DLL部分被当做一个RUNTIME目标而对应的导出库被当做是一个ARCHIVE目标。所有基于Windows的系统，包括Cygwin，都是DLL平台。ARCHIVE，LIBRARY，RUNTIME和FRAMEWORK参数改变了后续属性会加诸之上的目标的类型。如果只给出了一种类型，那么只有那种类型的目标会被安装（这样通常只会安装一个DLL或者一个导出库。）

　　PRIVATE_HEADER，PUBLIC_HEADER，和RESOURCE选项的功能是，在非苹果平台上，将后续的属性应用在待安装的一个FRAMEWORK共享库目标的相关文件上。这些选项定义的规则在苹果系统上会被忽略掉，因为相关的文件将会被安装到framework文件夹内的合适位置。参见PRIVATE_HEADER，PUBLIC_HEADER和RESOURCE目标属性中更为详细的解释。

　　可以指定NAMELINK_ONLY或者NAMELINK_SKIP选项作为LIBRARY选项。在一些平台上，版本化的共享库有一个符号链接，比如lib<name>.so -> lib<name>.so.1，其中“lib<name>.so.1”是so库文件名（soname）而“lib<name>.so”是一个符号链接，当指定“-l<name>”选项时，链接器将会查找这个符号链接。如果一个库目标已经被安装，NAMELINK_ONLY选项表示仅仅安装符号链接；而NAME_SKIP选项则表示仅仅安装库文件而不是符号链接。当两种选项都没有给出时，动态库的两个部分都会被安装。在那些版本化的共享库没有符号链接或者库没有被版本化的平台，选项NAMELINK_SKIP安装这个库，而NAMELINK_ONLY选项什么都不会安装。参见VERSION和SOVERSION目标属性，获取关于创建版本化共享库的更多细节。

　　在该命令的TARGETS版本的一次调用中，可以一次性指定一个或多个属性组。一个目标也可以被多次安装到不同的位置。假设有三个目标myExe，mySharedLib和myStaticLib，下面的代码

```
    install(TARGETS myExe mySharedLib myStaticLib
            RUNTIME DESTINATION bin
            LIBRARY DESTINATION lib
            ARCHIVE DESTINATION lib/static)
    install(TARGETS mySharedLib DESTINATION /some/full/path)
```

将会把myExe安装到<prefix>/bin目录下，把myStaticLib安装到<prefix>/lib/static目录下。在非-DLL平台上，mySharedLib将会被安装到<prefix>/lib和/some/full/path下。在DLL平台上，mySharedLib DLL将会被安装到<prefix>/bin和/some/full/path路径下，它的导出库会被安装到<prefix>/lib/static和/some/full/path路径下。

　　EXPORT选项将已经安装的目标文件和一个名为<export-name>的导出文件关联起来。它必须出现在所有RUNTIME，LIBRARY或者ARCHIVE选项之前。为了实际安装导出文件(export file)本身，调用install(EXPORT)。参见下述install命令EXPORT版本的文档获取更多的细节。

　　将EXCLUDE_FROM_ALL设置为true时，安装一个目标会造成未定义的行为。

　　FILES版本的install命令

```
  install(FILES files... DESTINATION <dir>
          [PERMISSIONS permissions...]
          [CONFIGURATIONS [Debug|Release|...]]
          [COMPONENT <component>]
          [RENAME <name>] [OPTIONAL])
```

　　FILES版本的install命令指定了为一个工程安装文件的规则。在命令中，以相对路径方式给出的文件名是相对于当前源代码路径而言的。以这个版本安装的文件，如果没有指定PERMISSIONS选项，默认会具有OWNER_WRITE，OWNER_READ，GROUP_READ，和WORLD_READ的权限。

　　PROGRAMS版本的install命令

```
  install(PROGRAMS files... DESTINATION <dir>
          [PERMISSIONS permissions...]
          [CONFIGURATIONS [Debug|Release|...]]
          [COMPONENT <component>]
          [RENAME <name>] [OPTIONAL])
```

　　PROGRAMS版本与FILES版本一样，只在默认权限上有所不同：它还包括了OWNER_EXECUTE，GROUP_EXECUTE和`WORLD_EXECUTE选项。INSTALL的这个版本用来安装不是目标的程序，比如shell脚本。使用TARGETS格式安装该工程内部构建的目标。

　　DIRECTORY版本的install命令

```
  install(DIRECTORY dirs... DESTINATION <dir>
          [FILE_PERMISSIONS permissions...]
          [DIRECTORY_PERMISSIONS permissions...]
          [USE_SOURCE_PERMISSIONS] [OPTIONAL]
          [CONFIGURATIONS [Debug|Release|...]]
          [COMPONENT <component>] [FILES_MATCHING]
          [[PATTERN <pattern> | REGEX <regex>]
           [EXCLUDE] [PERMISSIONS permissions...]] [...])
```

　　INSTALL的DIRECTORY版本将一个或者多个路径下的内容安装到指定的目标地址下。目录结构会原封不动地（verbatim）拷贝到目标地址。每个路径名的最后一部分会追加到目标路径下，但是结尾反斜杠(trailing slash)可以用来避免这一点，因为这样最后一部分就是空的。给定的相对路径名被解释成相对于当前源路径的路径。如果没有指定输入目录名字，目标目录会被创建，但是不会安装任何东西。FILE_PERMISSIONS和DIRECTORY_PERMISSIONS选项指定了赋予目标路径和目标文件的权限。如果指定了USE_SOURCE_PERMISSIONS选项，但没有指定FILE_PERMISSIONS选项，文件权限将沿袭源目录结构的权限，而且这个路径会被赋予PAROGRAMS版本中指定的默认权限。

　　通过使用PATTERN或REGEX选项可以对路径安装做出细粒度的控制。这些用于匹配的选项指定了一个查询模式或正则表达式来匹配输入路径内的路径或文件。它们可以用来将特定的选项（见下文）加诸于遇到的文件和路径的一个子集上。每个输入文件或路径的完整路径（反斜杠/开头的路径）将用来匹配该表达式。PATTERN仅仅用来匹配完全文件名：匹配该模式的全路径的那部分必须出现在文件名的结尾，并且必须以一个反斜杠开始。

　　正则表达式会用来匹配一个完全路径的任何部分，但是它也可以使用'/'和'$'模仿PATTERN的行为。默认情况下，所有文件和路径不管是否匹配都会被安装。可以在第一个匹配选项之前指定FILE_MATCHING选项，这样就能禁止安装那些不与任何表达式匹配的文件。比如，代码

```
  install(DIRECTORY src/ DESTINATION include/myproj
          FILES_MATCHING PATTERN "*.h")
```

将会精确匹配并安装从源码树上得到的头文件。

　　有些选项后面可以跟在PATTERN或者REGEX表达式的后面，这样这些选项只能加诸于匹配PATTERN/REGEX的文件或路径上。EXCLUDE选项将会指示安装过程跳过那些匹配的文件或者路径。PERMISSIONS选项可以覆盖那些匹配PATTERN/REGEX的文件的权限设定。例如，代码

```
  install(DIRECTORY icons scripts/ DESTINATION share/myproj
          PATTERN "CVS" EXCLUDE
          PATTERN "scripts/*"
          PERMISSIONS OWNER_EXECUTE OWNER_WRITE OWNER_READ
                      GROUP_EXECUTE GROUP_READ)
```

会将icons路径安装到share/myproject/icons下，同时把scripts目录安装到share/myproj路径下。icons将具备默认的文件权限，scripts将会被给与指定的权限，但是所有CVS路径排除在外。

　　SCRIPT和CODE版本的install命令

```
  install([[SCRIPT <file>] [CODE <code>]] [...])
```

　　SCRIPT格式将会在安装期调用给定的脚本文件。如果脚本文件名是一个相对路径，它会被解释为相对于当前的源路径。CODE格式将会在安装期调用给定的CMake代码。code被指定为一个双引号括起来的单独的参数。例如，代码

  install(CODE "MESSAGE(\"Sample install message.\")")
会在安装时打印一条消息。

　　EXPORT版本的install命令

```
  install(EXPORT <export-name> DESTINATION <dir>
          [NAMESPACE <namespace>] [FILE <name>.cmake]
          [PERMISSIONS permissions...]
          [CONFIGURATIONS [Debug|Release|...]]
          [COMPONENT <component>])
```

　　EXPORT格式的install命令生成并安装一个包含将安装过程的安装树导入到另一个工程中的CMake文件。Target格式的安装过程与上文提及的使用EXPORT选项的install(TARGET ...)格式的命令中的EXPORT <export-name>选项是相关的。NAMESPACE选项会在它们被写入到导入文件时加到目标名字之前。缺省时，生成的文件就是<export-name>.cmake；但是FILE选项可以用来指定不同于次的文件名。FILE选项后面的参数必须是以“.cmake”为扩展名的文件。如果指定了CONFIGURATIONS选项，那么只有那些具名的配置中的一个被安装时，这个文件才会被安装。而且，生成的导入文件只能涉及到匹配的目标配置版本。如果指定了一个COMPONENT选项，并且<component>与那个<export-name>相关的目标指定的部件不匹配，那么行为是未定义的。如果一个库目标被包含在export之中，但是与之关联的库却没有背包含，那么结果是未指定的。

　　EXPORT格式可以协助外部工程使用当前工程构建出来并安装的目标。例如，代码

```
  install(TARGETS myexe EXPORT myproj DESTINATION bin)
  install(EXPORT myproj NAMESPACE mp_ DESTINATION lib/myproj)
```

将会把可执行文件myexe安装到<prefix>/bin下，并且将导入它的代码写到文件"<prefix>/lib/myproj/myproj.cmake"中。一个外部工程可以用include命令加载这个文件，并且可以在安装树上使用导入的目标名mp_myexe（前缀_目标名——译注）引用myexe可执行文件，如同这个目标是它自身的构建树的内置目标一样。

　　注意：这个命令会取代INSTALL_TARGETS命令以及PRE_INSTALL_SCRIPT和POST_INSTALL_SCRIPT两个目标属性。它也可以取代FILES格式的INSTALL_FILES命令和INSTALL_PROGRAMS命令。由INSTALL命令生成的安装规则相对于那些由INSTALL_TARGETS，INSTALL_FILES和INSTALL_PROGRAMS命令生成的安装规则处理顺序是未定义的。 
　　 
　　

## CMD#50 link_directories

指定连接器查找库的路径。

```
  link_directories(directory1 directory2 ...)
```

　　指定连接器搜索库文件时的路径。该命令仅仅能用在那些在它被调用后才生成的目标上。由于历史上的原因，为该命令指定的相对路径将会不加改变地传递给连接器（不像许多其他CMake命令那样解释为相对于当前源路径的相对路径。） 
　　 
　　

## CMD#51 list

列表操作命令。

```
  　　list(LENGTH <list> <output variable>)
  　　list(GET <list> <element index> [<element index> ...] <output variable>)
  　　list(APPEND <list> <element> [<element> ...])
  　　list(FIND <list> <value> <output variable>)
  　　list(INSERT <list> <element_index> <element> [<element> ...])
  　　list(REMOVE_ITEM <list> <value> [<value> ...])
  　　list(REMOVE_AT <list> <index> [<index> ...])
  　　list(REMOVE_DUPLICATES <list>)
  　　list(REVERSE <list>)
  　　list(SORT <list>)
```
  　　
　　使用LENGTH选项时，该命令会返回给定list的长度。

　　使用GET选项时，该命令返回list中所有被index索引的元素构成的list。

　　使用APPEND选项时，该命令将会在该list之后追加若干元素。

　　使用FIND选项时，该命令将返回list中指定的元素的索引；若果未找到，返回-1。

　　使用INSERT选项时，该命令将在list中指定的位置插入若干元素。

　　使用REMOVE_AT和REMOVE_ITEM选项将会从list中删除一些元素。它们之间的区别是：REMOVE_ITEM删除的是指定的项，而REMOVE_AT删除的是在指定索引处的项。

　　使用REMOVE_DUPLICATES选项时，该命令将删除list中的重复项。

　　使用REVERSE选项时，该命令将把list的内容就地前后倒换。

　　使用SORT选项时，该命令将按字母序对list总的内容就地排序。

　　注意：在CMake中，一个list是一个由分号;分割的一组字符串。使用set命令可以创建一个list。例如，set(var a b c d e)命令将会创建一个list:a;b;c;d;e, 而set(var "a b c d e")命令创建的只是一个字符串, 或者说是只有一个项的list。

　　当使用指定索引的命令格式时，如果<element index>是大于等于0的数，<element index>是从list第一个项开始的序号，list的第一项的索引是0。如果<element index>小于等于-1，这个索引是从结尾开始的逆向索引，其中-1表示的是list的最后一项。当使用负数索引时，(注意它们不是从0开始!!!!), -0与0等价，它指向list的第一个成员。

## CMD#52 load_cache

从另一个工程的CMake cache中加载值。

```
  　　load_cache(pathToCacheFile READ_WITH_PREFIX
             prefix entry1...)
```

　　该命令读取指定的cache文件，并将以请求的前缀为其前缀的那些cache文件中的entry(ies)保存到变量中。这个格式仅仅读取值，但是不在本地工程的cache中创建entry(ies)。

```
  　　load_cache(pathToCacheFile [EXCLUDE entry1...]
             [INCLUDE_INTERNALS entry1...])
```

　　从另一个cache文件中加载值并像内部entry(ies)那样，将它们存储到本地工程的cache中。这条命令对于一个依赖于另一个不同构建树上的另一个工程的工程比较有用。EXCLUDE选项给出了那些需要排除在外的entry(ies)的一个list。INCLUDE_INTERNALS选项给出了需要包含的entry(ies)的内部entry(ies)的一个list。通常情况下，不需要引入内部entry(ies)。强烈不推荐使用该命令的这种格式，但是它可以被用来维持向后兼容性。

## CMD#53 load_command

将一条命令加载到一个运行中的CMake。

```
  　　load_command(COMMAND_NAME <loc1> [loc2 ...])
```

　　该命令将在给定的路径下查找名字为COMMAND_NAME的一个库。如果找到了，它将会以模块的方式被加载，然后该命令将会被添加到可用的CMake命令集中。通常，TRY_COMPILE选项被用在这个命令之前来编译这个模块。如果该命令被成功加载，一个名为CMAKE_LOADED_COMMAND_<COMMAND_NAME>的变量将会被设置为这个加载模块的完整路径。否则，这个变量就不会被设置。

## CMD#54 macro

为后续以命令方式调用而开始记录一组宏。

```
  　　macro(<name> [arg1 [arg2 [arg3 ...]]])
    　　COMMAND1(ARGS ...)
    　　COMMAND2(ARGS ...)
    　　...
  　　endmacro(<name>)
```

　　定义一个名为<name>的宏，它以arg1 arg2 arg3 (...)为参数。在macro命令之后，在与之配对的endmacro命令之前出现的命令，只有在宏被调用的时候才会被调用。当被调用的时候，这些被记录的命令首先以传进来的实参替换掉形参(如${arg1})，然后像正常的命令那样执行。除了形参之外，你还可以引用变量${ARGC}}，它表示传递到宏里的参数的数量；${ARG0}, ${ARG1}, ${ARG2} ...等等则是传进来的实参值。这些变量使得创建带可选参数的宏变得很便捷。此外，变量${ARGV}保留了所有传递到宏里的所有参数组成的一个list，变量${ARGN}保留了在最后一个形参之后的参数组成的一个list。注意：传递到宏内部的参数和值，比如ARGN不是CMake通常意义下的变量；它们只是字符串替换，这一点非常像C预处理器对C语言宏的处理过程。如果你想要用真正的CMake变量，你应该查看一下function命令的说明。

　　关于在macro内部的策略的行为，参见cmake_policy()命令的相关文档。

## CMD#55 mark_as_advanced

将CMake 的缓存变量标记为高级。

```
  　　mark_as_advanced([CLEAR|FORCE] VAR VAR2 VAR...)
```

　　将缓存的变量标记为高级变量。其中，高级变量指的是那些在cmake GUI中，只有当“显示高级选项”被打开时才会被显示的变量。如果CLEAR是第一个选项，参数中的高级变量将变回非高级变量。如果FORCE是第一个选项，参数中的变量会被提升为高级变量。如果两者都未出现，新的变量会被标记为高级变量；如果这个变量已经是高级/非高级状态`的话，它将会维持原状。

　　该命令在脚本中无效。

## CMD#56 math

数学表达式。

  　　math(EXPR <output variable> <math expression>)
　　EXPR计算数学表达式然后通过output变量返回计算结果。数学表达式的一个例子是"5*(10+13)"。该命令支持的运算符包括：+ - * / % ^ ~ << >> ；它们的含义与C语言中的完全一致。

## CMD#57 message

为用户显示一条消息。

```
  　　message([STATUS|WARNING|AUTHOR_WARNING|FATAL_ERROR|SEND_ERROR]
          "message to display" ...)
```

　　可以用下述可选的关键字指定消息的类型：

```
(无)         　 = 重要消息；
 STATUS         = 非重要消息；
 WARNING        = CMake 警告, 会继续执行；
 AUTHOR_WARNING = CMake 警告 (dev), 会继续执行；
 SEND_ERROR     = CMake 错误, 继续执行，但是会跳过生成的步骤；
 FATAL_ERROR    = CMake 错误, 终止所有处理过程；
```

　　CMake的命令行工具会在stdout上显示STATUS消息，在stderr上显示其他所有消息。CMake的GUI会在它的log区域显示所有消息。交互式的对话框（ccmake和CMakeSetup）将会在状态行上一次显示一条STATUS消息，而其他格式的消息会出现在交互式的弹出式对话框中。

　　CMake警告和错误消息的文本显示使用的是一种简单的标记语言。文本没有缩进，超过长度的行会回卷，段落之间以新行做为分隔符。 
　　 
　　 
　　

## CMD#58 option

为用户提供一个可选项。

```
  option(<option_variable> "描述选项的帮助性文字" [initial value])
```

　　该命令为用户提供了一个在ON和OFF中做出选择的选项。如果没有指定初始值，将会使用OFF作为初值。如果有些选项依赖于其他选项的值，参见CMakeDependentOption模块的帮助文件。

## CMD#59: output_required_files

输出一个list，其中包含了一个给定源文件所需要的其他源文件。

```
  output_required_files(srcfile outputfile)
```

　　输出一个指定的源文件所需要的所有源文件的list。这个list会写到outputfile变量中。该命令的功能是将srcfile的依赖性写出到outputfile中，不过该命令将尽可能地跳过.h文件，搜索依赖中的.cxx，.c和.cpp文件。

## CMD#60 project

为整个工程设置一个工程名。

```
  project(<projectname> [languageName1 languageName2 ... ] )
```

　　为本工程设置一个工程名。而且，该命令还将变量<projectName>_BINARY_DIR和<projectName>_SOURCE_DIR设置为对应值。后面的可选项还可以让你指定你的工程可以支持的语言。比如CXX(即C++)，C，Fortran，等等。在默认条件下，支持C和CXX语言。例如，如果你没有C++编译器，你可以通过列出你想要支持的语言，例如C，来明确地禁止对它的检查。使用特殊语言"NONE"，针对任何语言的检查都会被禁止。

## CMD#61 qt_wrap_cpp

创建Qt包裹器。

```
  qt_wrap_cpp(resultingLibraryName DestName SourceLists ...)
```
  
　　为所有在SourceLists中列出的.h文件生成moc文件。这些moc文件将会被添加到那些使用DestName源文件列表的库文件中。

Produce moc files for all the .h files listed in the SourceLists. The moc files will be added to the library using the DestName source list.

## CMD#62 qt_wrap_ui

创建Qt的UI包裹器。

```
  qt_wrap_ui(resultingLibraryName HeadersDestName SourcesDestName SourceLists ...)
```

　　为所有在SourceLists中列出的.ui文件生成.h和.cxx文件。这些.h文件会被添加到使用HeadersDestNamesource列表的库中。这些.cxx文件会被添加到使用SourcesDestNamesource列表的库中。

## CMD#63 remove_definitions

取消由add_definitions命令添加的-D定义标志。

```
  remove_definitions(-DFOO -DBAR ...)
```

　　在当前及以下的路径，从编译命令行中取消（由add_definitions命令添加的）标志。

## CMD#64 return

从一个文件，路径或函数内返回。

```
  return()
```

　　从一个文件，路径或函数中返回。若出现在一个include文件里（经由include()或find_package()命令），该命令会导致当前文件的处理过程停止，并且将控制权转移到试图包含它的文件中。若出现在一个不被任何文件包含的文件中，例如，一个CMakeLists.txt中，那么该命令将控制权转移到父目录下，如果存在这样的父目录的话。如果在一个函数中调用return函数，控制权会返回到该函数的调用函数那里。注意，宏不是函数，它不会像函数那样去处理return命令。 
　　 
　　 
　　

## CMD#65 separate_arguments

将空格分隔的参数解析为一个分号分隔的list。

```
  separate_arguments(<var> <UNIX|WINDOWS>_COMMAND "<args>")
```

　　解析一个unix或者windows风格的命令行字符串"<args>"，并将结果以分号分隔的list的形式存储到<var>中。整个命令行都必须从这个"<args>"参数中给出。

　　UNIX_COMMAND模式以没有被括起来的白字符为参数的分隔符。它可以识别单引号和双引号的引号对。反斜杠可以对下一个字符的字面值转义（\"，就是"）;没有其他特殊的转义字符（例如\n就是n）。

　　WINDOWS_COMMAND模式按照与运行时库相同的语法解析一个windows命令行，在启动(starrtup)时构造argv。它使用没有被双引号括起来的白字符来分隔参数。反斜杠维持其字面含义，除非它们在双引号之前。更多细节，参见MSDN的文章："Parsing C Command-Line Arguments"。

  separate_arguments(VARIABLE)
　　将VARIABLE的值转换为一个分号分隔的list。所有的空格会被替换为';;'。该命令可以用来辅助生成命令行。

## CMD#66 set

将一个CMAKE变量设置为给定值。

```
  set(<variable> <value> [[CACHE <type> <docstring> [FORCE]] | PARENT_SCOPE])
```

　　将变量<variable>的值设置为<value>。在<variable>被设置之前，<value>会被展开。如果有CACHE选项，那么<variable>就会添加到cache中；这时<type>和<docstring>是必需的。<type>被CMake GUI用来选择一个窗口，让用户设置值。<type>可以是下述值中的一个：

```
FILEPATH = 文件选择对话框。
PATH     = 路径选择对话框。
STRING   = 任意的字符串。
BOOL     = 布尔值选择复选框。
INTERNAL = 不需要GUI输入端。(适用于永久保存的变量)。
```

　　如果<type>是内部(INTERNAL)的，那么<value>总是会被写入到cache中，并替换任何已经存在于cache中的值。如果它不是一个cache变量，那么这个变量总是会写入到当前的makefile中。FORCE选项将覆盖cache值，从而去掉任何用户带来的改变。

　　如果指定了PARENT_SCOPE选项，变量<variable>将会被设置为当前作用域之上的作用域中。每一个新的路径或者函数都可以创建一个新作用域。该命令将会把一个变量的值设置到父路径或者调用函数中（或者任何类似的可用的情形中。）

　　如果没有指定<value>，那么这个变量就会被撤销而不是被设置。另见：unset()命令。

```
  set(<variable> <value1> ... <valueN>)
```

　　在这种情形下，<variable>被设置为一个各个值之间由分号分隔的list。

　　<variable>可以是环境变量，比如：

```
  set( ENV{PATH} /home/martink )
```

　　在这种情形下，环境变量将会被设置。 
　　 
　　

## CMD#67 set_directory_properties

设置某个路径的一种属性。

```
  set_directory_properties(PROPERTIES prop1 value1 prop2 value2)
```

　　为当前的路径及其子路径设置一种属性。如果该属性不存在，CMake将会报告一个错误。属性包括：INCLUDE_DIRECTORIES, LINK_DIRECTORIES, INCLUDE_REGULAR_EXPRESSION, 以及ADDITIONAL_MAKE_CLEAN_FILES共四种。ADDITIONAL_MAKE_CLEAN_FILES是一个文件名的list，其中包含有"make clean"阶段会被清除掉的文件。

## CMD#68 set_property

在给定的作用域内设置一个命名的属性。

```
  set_property(<GLOBAL                            |
                DIRECTORY [dir]                   |
                TARGET    [target1 [target2 ...]] |
                SOURCE    [src1 [src2 ...]]       |
                TEST      [test1 [test2 ...]]     |
                CACHE     [entry1 [entry2 ...]]>
               [APPEND]
               PROPERTY <name> [value1 [value2 ...]])
```

　　为作用域里的0个或多个对象设置一种属性。第一个参数决定了属性可以影响到的作用域。他必须是下述值之一：GLOBAL，全局作用域，唯一，并且不接受名字。DIRECTORY，路径作用域，默认为当前路径，但是也可以用全路径或相对路径指定其他值。TARGET，目标作用域，可以命名0个或多个已有的目标。SOURCE，源作用域，可以命名0个或多个源文件。注意，源文件属性只对加到相同路径（CMakeLists.txt）中的目标是可见的。TEST 测试作用域可以命名0个或多个已有的测试。CACHE作用域必须指定0个或多个cache中已有的条目。

　　PROPERTY选项是必须的，并且要紧跟在待设置的属性的后面。剩余的参数用来组成属性值，该属性值是一个以分号分隔的list。如果指定了APPEND选项，该list将会附加在已有的属性值之后。

## CMD#69 set_source_files_properties

源文件有一些属性来可以改变它们构建的方式。

```
  set_source_files_properties([file1 [file2 [...]]]
                              PROPERTIES prop1 value1
                              [prop2 value2 [...]])
```

　　以键/值对的方式设置与源文件相关的那些属性值。那些CMake中的源文件属性，参见关于属性的相关文档。不能被识别的属性将会被忽略。源文件属性只对同一路径（CMakeLists.txt）中添加的目标可见。

## CMD#70 set_target_properties

设置目标的一些属性来改变它们构建的方式。

```
  set_target_properties(target1 target2 ...
                        PROPERTIES prop1 value1
                        prop2 value2 ...)
```

　　为一个目标设置属性。该命令的语法是列出所有你想要变更的文件，然后提供你想要设置的值。你能够使用任何你想要的属性/值对，并且在随后的代码中调用GET_TARGET_PROPERTY命令取出属性的值。

　　影响一个目标输出文件的名字的属性详述如下。PREFIX和SUFFIX属性覆盖了默认的目标名前缀（比如lib）和后缀（比如.so）。IMPORT_PREFIX和IMPORT_SUFFIX是与之等价的属性，不过针对的是DLL（(共享库目标）)的导入库。在构建目标时，OUTPUT_NAME属性设置目标的真实名字，并且可以用来辅助创建两个具有相同名字的目标，即使CMake需要唯一的逻辑目标名。<CONFIG>_OUTPUT_NAME可以为不同的配置设置输出的目标名字。当目标在指定的配置名<CONFIG>（全部大写，例如DEBUG_POSTFIX）下被构建时，<CONFIG>_POSTFIX为目标的真实名字设置一个后缀。该属性的值在目标创建时被初始化为CMAKE_<CONFIG>_POSTFIX的值（可执行目标除外，因为较早的CMake版本不会为可执行文件使用这个属性。）

　　LINK_FLAGS属性可以用来为一个目标的链接阶段添加额外的标志。LINK_FLAGS_<CONFIG>将为配置<CONFIG>添加链接标志，例如DEBUG，RELEASE，MINSIZEREL，RELWITHDEBINFO。DEFINE_SYMBOL属性设置了编译一个共享库中的源文件时才会被定义的预处理器符号名。如果这个值没有被设置的话，那么它会被设置为默认值target_EXPORTS（如果目标不是一个合法的C标示符的话可以用一些替代标志）。这对于检测头文件是包含在它们的库以内还是以外很有帮助，从而可以合理设置dllexport/dllimport修饰符（注意，只有在编译到的时候，这个符号才会被定义；因此猜测在代码中，判断预处理符号是否被定义可以知道依赖库是导入的还是导出的——译注）。COMPILE_FLAGS属性可以设置附加的编译器标志，它们会在构建目标内的源文件时被用到。它也可以用来传递附加的预处理器定义。

　　LINKER_LANGUAGE属性用来改变链接可执行文件或共享库的工具。默认的值是设置与库中的文件相匹配的语言。CXX和C是这个属性的公共值。

　　对于共享库，VERSION和SOVERSION属性分别可以用来指定构建的版本号以及API版本号。当构建或者安装时，如果平台支持符号链接并且链接器支持so名字，那么恰当的符号链接会被创建。如果只指定两者中的一个，缺失的另一个假定为具有相同的版本号。对于可执行文件，VERSION可以被用来指定构建版本号。当构建或者安装时，如果该平台支持符号链接，那么合适的符号链接会被创建。对于在Windows系统而言，共享库和可执行文件的VERSION属性被解析成为一个"major.minor"的版本号。这些版本号被用做该二进制文件的镜像版本。

　　还有一些属性用来指定RPATH规则。INSTALL_RPATH是一个分号分隔的list，它指定了在安装目标时使用的rpath（针对支持rpath的平台而言）（-rpath在gcc中用于在编译时指定加载动态库的路径；优先级较系统库路径要高。详情参见http://www.cmake.org/Wiki/CMake_RPATH_handling#What_is_RPATH_.3F——译注）。INSTALL_RPATH_USE_LINK_PATH是一个布尔值属性，如果它被设置为真，那么在链接器的搜索路径中以及工程之外的目录会被附加到INSTALL_RPATH之后。SKIP_BUILD_RPATH是一个布尔值属性，它指定了是否跳过一个rpath的自动生成过程，从而可以从构建树开始运行。BUILD_WITH_INSTALL_RPATH是一个布尔值属性，它指定了是否将在构建树上的目标与INSTALL_RPATH链接。该属性要优先于SKIP_BUILD_RPATH，因此避免了安装之前的重新链接。INSTALL_NAME_DIR是一个字符串属性，它用于在Mac OSX系统上，指定了被安装的目标中使用的共享库的"install_name"域的目录部分。如果目标已经被创建，变量CMAKE_INSTALL_RPATH, CMAKE_INSTALL_RPATH_USE_LINK_PATH, CMAKE_SKIP_BUILD_RPATH, CMAKE_BUILD_WITH_INSTALL_RPATH和CMAKE_INSTALL_NAME_DIR的值会被用来初始化这个属性。

　　PROJECT_LABEL属性可以用来在IDE环境，比如visual studio，中改变目标的名字。 VS_KEYWORD可以用来改变visual studio的关键字，例如如果该选项被设置为Qt4VSv1.0的话，QT集成将会运行得更好。

　　VS_SCC_PROJECTNAME, VS_SCC_LOCALPATH, VS_SCC_PROVIDER可以被设置，从而增加在一个VS工程文件中对源码控制绑定的支持。

　　PRE_INSTALL_SCRIPT和POST_INSTALL_SCRIPT属性是在安装一个目标之前及之后指定运行CMake脚本的旧格式。只有当使用旧式的INSTALL_TARGETS来安装目标时，才能使用这两个属性。使用INSTALL命令代替这种用法。

　　EXCLUDE_FROM_DEFAULT_BUILD属性被visual studio生成器使用。如果属性值设置为1，那么当你选择"构建解决方案"时，目标将不会成为默认构建的一部分。 
　　 
　　

## CMD#71 set_tests_properties

设置若干个测试的属性值。

```
  set_tests_properties(test1 [test2...] PROPERTIES prop1 value1 prop2 value2)
```

　　为若干个测试设置一组属性。若属性未被发现，CMake将会报告一个错误。这组属性包括：WILL_FAIL， 如果设置它为true，那将会把这个测试的“通过测试/测试失败”标志反转。PASS_REGULAR_EXPRESSION，如果它被设置，这个测试的输出将会被检测是否违背指定的正则表达式，并且至少要有一个正则表达式要匹配；否则测试将会失败。

例子:

  PASS_REGULAR_EXPRESSION "TestPassed;All ok"
　　FAIL_REGULAR_EXPRESSION: 如果该属性被设置，那么只要输出匹配给定的正则表达式中的一个，那么测试失败。

例子:

```
  PASS_REGULAR_EXPRESSION "[^a-z]Error;ERROR;Failed"
```

　　PASS_REGULAR_EXPRESSION和和FAIL_REGULAR_EXPRESSION属性都期望一个正则表达式列表（list）作为其参数。

　　TIMEOUT: 设置该属性将会限制测试的运行时长不超过指定的秒数。 
　　 
　　

## CMD#72: site_name

将给定的变量设定为计算机名。

```
  site_name(variable)
```

## #73: source_group

为Makefile中的源文件定义一个分组。

```
  source_group(name [REGULAR_EXPRESSION regex] [FILES src1 src2 ...])
```

　　为工程中的源文件中定义一个分组。这主要用来在Visual Studio中建立文件组按钮(file tabs)。所有列出来的文件或者匹配正则表达式的文件都会被放到这个文件组中。如果一个文件匹配多个组，那么最后明确地列出这个文件的组将会包含这个文件，如果有这样的组的话。如果没有任何组明确地列出这个文件，那么最后那个其正则表达式与该文件名匹配的组，将会成为最终候选者。

　　组名中可以包含反斜杠，以指定子文件组：source_group(outer\\inner ...)

　　为了保持后向兼容性，这个命令也支持这种格式：source_group(name regex)

## CMD#74: string

字符串操作函数。

```
  string(REGEX MATCH <regular_expression> <output variable> <input> [<input>...])
  string(REGEX MATCHALL <regular_expression> <output variable> <input> [<input>...])
  string(REGEX REPLACE <regular_expression> <replace_expression> <output variable> <input> [<input>...])
  string(REPLACE <match_string> <replace_string> <output variable> <input> [<input>...])
  string(COMPARE EQUAL <string1> <string2> <output variable>)
  string(COMPARE NOTEQUAL <string1> <string2> <output variable>)
  string(COMPARE LESS <string1> <string2> <output variable>)
  string(COMPARE GREATER <string1> <string2> <output variable>)
  string(ASCII <number> [<number> ...] <output variable>)
  string(CONFIGURE <string1> <output variable> [@ONLY] [ESCAPE_QUOTES])
  string(TOUPPER <string1> <output variable>)
  string(TOLOWER <string1> <output variable>)
  string(LENGTH <string> <output variable>)
  string(SUBSTRING <string> <begin> <length> <output variable>)
  string(STRIP <string> <output variable>)
  string(RANDOM [LENGTH <length>] [ALPHABET <alphabet>] [RANDOM_SEED <seed>] <output variable>)
```

　　REGEX MATCH : 匹配正则表达式一次，然后将匹配的值存储到输出变量中。

　　REGEX MATCHALL : 尽可能多次地匹配正则表达式，然后将匹配的值以list的形势存储到输出变量中。

　　REGEX REPLACE : 尽可能多次地匹配正则表达式，并且将匹配的值用replacement expression 替换掉，然后存储到输出变量中。这个replace expression 可以引用包含匹配字符串的子表达式，这些匹配的字符串用圆括号隔开的\1，\2，...，\9等加以引用。注意：在CMake代码里，如果要使用一个反斜杠，必须要用两个反斜杠(\1)转义，才能通过参数解析。

　　REPLACE : 将输入字符串内所有出现match_string的地方都用replace_string代替，然后将结果存储到输出变量中。

　　COMPARE EQUAL/NOTEQUAL/LESS/GREATER : 将会比较两个字符串，然后将比较的结果（true/false）存储到输出变量中。

　　ASCII : 将会把所有数字转换为对应的ASCII字符。

　　CONFIGURE : 将一个字符串进行变换，这种变换与将一个FILE变换为CONFIGURE_FILE相似。

　　TOUPPER/TOLOWER : 将字符串转换为大写/小写字符。

　　LENGTH : 返回给定字符串的长度。

　　SUBSTRING : 返回给定字符串的子串。

　　STRIP : 返回一个给定字符串的子串，它会去掉原先字符串开始和结尾的空格。

　　RANDOM : 将会返回一个给定长度的随机字符串，它由给定的字母表中的字母组成。默认的长度是5个字符，默认的字母表是全部的大小写字母以及数字。如果指定了一个整数RANDOM_SEED，它的值将会被用做随机数发生器的种子。

　　在正则表达式中，下述字符有特殊含义：

```
^         在行首匹配。
$         在行尾匹配。
.         匹配任意单个字符。
[ ]       匹配在中括号中的任意字符。
[^ ]      匹配不在中括号中的任意字符。
-         匹配任意在短横线两端字符闭区间中间的任意一个字符。
*         匹配先前模式零次或多次。
+         匹配先前模式一次或多次。
?         匹配先前模式零次或一次。
|         匹配|两侧的任意一种模式。
()        保存一个匹配的子表达式，这个子表达式后续可以在REGEX REPLACE操作中以\n的方式引用。 它也会被
所有正则表达式相关的命令所保存；包括，比如，
```

　　　　　　　 如果用到if( MATCHES )命令的话，这些匹配的值被保存在变量CMAKE_MATCH_(0..9)中。

## CMD#75: target_link_libraries

将给定的库链接到一个目标上。

```
  target_link_libraries(<target> [item1 [item2 [...]]] [[debug|optimized|general] <item>] ...)
```

　　为给定的目标设置连接时使用的库或者标志(flags)。如果一个库名字与工程中的另外一个目标相匹配，一个依赖关系会自动添加到构建系统中来，这样就可以在链接目标之前，保证正在被链接的库是最新的。以-“-”开始，但不是“-l或”或“-framework”的那些项，将会被当作链接器标志来处理。

　　关键字“debug，”，“optimized”或者“general” 表示紧随关键字之后的库仅仅会被用到相应的构建配置上。“debug”关键字对应于调试配置（或者，如果全局属性DEBUG_CONFIGURATIONS被设置的话，就是DEBUG_CONFIGURATIONS中的名字所指定的配置）。“optimized”关键字对应于所有其他的配置类型。“general”关键字对应于所有的配置，并且纯粹是可选的（它是默认配置，可以省略）。通过创建并链接到导入库目标，可以对每种配置规则进行更细致的粒度控制。更多内容参见add_library命令的IMPORTED模式。

　　默认时，库之间的依赖性是可传递的。当这个目标被链接到其他目标上时，那么链接到这个目标上的库也会出现在其他目标的链接依赖上。参见LINK_INTERFACE_LIBRARIES属性的相关文档，其中有关于如何覆盖一个目标的链接依赖性传递设置的介绍。

```
  target_link_libraries(<target> LINK_INTERFACE_LIBRARIES [[debug|optimized|general] <lib>] ...)
```

　　对于LINK_INTERFACE_LIBRARIES模式，它将会把库附加在LINK_INTERFACE_LIBRARIES以及LINK_INTERFACE_LIBRARIES在不同配置下的等价目标属性，而不是用这些库去链接。指定为“debug”的库将会被附加到LINK_INTERFACE_LIBRARIES_DEBUG属性（或者是在DEBUG_CONFIGURATIONS全局属性中列出的配置，如果DEBUG_CONFIGURATIONS被设置的话）。指定为“optimized”库将会被附加到LINK_INTERFACE_LIBRARIES属性上。指定为“general”的库（或者没有任何关键字的库），将会被当做即被指定为“debug”又被指定为“optimized”对待。

　　库之间的依赖图通常是非循环图(（DAG)），但是如果出现互相依赖的静态库，CMake会允许依赖图中包含循环依赖（强连通分支）。当其它目标链接到这些库中的一个时，CMake会重复整个连通分支。例如，代码：

```
add_library(A STATIC a.c)
add_library(B STATIC b.c)
target_link_libraries(A B)
target_link_libraries(B A)
add_executable(main main.c)
target_link_libraries(main A)
```

将“main”链接到了“A B A B”。（虽然通常一次重复就足够了，但是病态对象文件以及符号排布可能需要多次重复。你可以通过在上一次target_link_libraries调用中手动重复该分支来处理这种情况。不过，如果两个归档文件确实是如此紧密的相互关联，它们可能会被合并为一个单一的归档文件。）

## CMD#76: try_compile

尝试编译一些代码。

```
  try_compile(RESULT_VAR bindir srcdir
              projectName <targetname> [CMAKE_FLAGS <Flags>]
              [OUTPUT_VARIABLE var])
```

　　尝试编译一个程序。在这种格式时，srcdir路径下应该包含一个完整的CMake工程，包括CMakeLists.txt文件以及所有的源文件。在该命令运行完之后，路径bindir和和srcdir不会被删除。如果指定了<target name>，那么CMake将只构建那个目标；否则，目标all或或ALL_BUILD将会被构建。

```
  try_compile(RESULT_VAR bindir srcfile
              [CMAKE_FLAGS <Flags>]
              [COMPILE_DEFINITIONS <flags> ...]
              [OUTPUT_VARIABLE var]
              [COPY_FILE <filename> )
```

　　尝试编译一个srcfile。在这种情况下，用户仅仅需要提供源文件。CMake会创建合适的CMakeLists.txt文件来构建源文件。如果使用了COPY_FILE选项，编译出的文件将会被拷贝到给定的文件那里。

　　在这个版本里，所有在bindir//CMakeFiles//CMakeTmp文件夹下的文件，将会被自动清除；通过向CMake传递调试选项--debug-trycompile可以避免这个清除步骤。另外一些可以包含的额外标志有：INCLUDE_DIRECTORIES, LINK_DIRECTORIES, 和LINK_LIBRARIES。。COMPILE_DEFINITIONS是通过-Ddefinations选项设置的预定义符号，这会传递到编译器命令行。try_compile命令在构建过程中伴随创建出的CMakeLists.txt看起来像这样：

```
  add_definitions( <expanded COMPILE_DEFINITIONS from calling cmake>)
  include_directories(${INCLUDE_DIRECTORIES})
  link_directories(${LINK_DIRECTORIES})
  add_executable(cmTryCompileExec sources)
  target_link_libraries(cmTryCompileExec ${LINK_LIBRARIES})
```

　　在该命令的这两种版本里，如果指定了OUTPUT_VARIABLE，那么构建过程的输出会存储到给定的变量里。编译成功或失败的结果，会通过RESULT_VAR返回。CMAKE_FLAGS可以用来向正在构建的CMake传递-DVAR:TYPE = VALUE 符号。

## CMD#77: try_run

尝试编译并运行某些代码。

```
  try_run(RUN_RESULT_VAR COMPILE_RESULT_VAR
          bindir srcfile [CMAKE_FLAGS <Flags>]
          [COMPILE_DEFINITIONS <flags>]
          [COMPILE_OUTPUT_VARIABLE comp]
          [RUN_OUTPUT_VARIABLE run]
          [OUTPUT_VARIABLE var]
          [ARGS <arg1> <arg2>...])
```

　　尝试编译一个源文件srcfile。通过变量COMPILE_RESULT_VAR返回TRUE或者FALSE来反应编译是否失败。如果构建出了可执行文件，但是不能运行，那么RUN_RESULT_VAR会被设置为FAILED_TO_RUN。。COMPILE_OUTPUT_VARIABLE变量指定了一个变量，这个变量存储了构建步骤输出的信息。RUN_OUTPUT_VARIABLE指定了一个变量，这个变量存储了运行可执行文件时的输出。出于兼容性的考虑，OUTPUT_VARIABLE还会被支持，它包含了包含编译和运行阶段的输出信息。

交叉编译相关问题

　　当运行交叉编译时，第一步中编译出的可执行文件通常不能在编译宿主机上直接运行。try_run()函数会检查CMAKE_CROSSCOMPILING变量来检测CMake是否是交叉编译模式。如果是的话，CMake还是会尝试编译可执行文件，但是它不会尝试运行可执行文件。相反，他会创建一些cache变量，这些变量必须由用户填充，或者在某个CMake脚本中预先设置为那些在真实目标机平台上执行的结果。这些变量有：RUN_RESULT_VAR (解释参见上文)，以及如果使用了RUN_OUTPUT_VARIABLE (或者OUTPUT_VARIABLE) ，还有一个附加的cache变量RUN_RESULT_VAR__COMPILE_RESULT_VAR__TRYRUN_OUTPUT。该变量是为了保存执行过程中stdout和stderr的输出。

　　为了让交叉编译更加容易些，必要时再使用try_run命令。如果你使用了try_run命令，那么只有必要时才使用RUN_OUTPUT_VARIABLE（或者OUTPUT_VARIABLE）变量。在交叉编译时，使用这些变量需要cache变量必须被手动设置为可执行文件的输出。你也可以用if(CMAKE_CROSSCOMPILING)将将try_run的调用“保护”起来，同时还要为这种情形给定一个易于预先设置的备选方案。

## CMD#78 unset

撤销对一个变量，cache变量或者环境变量的设置。

```
  unset(<variable> [CACHE])
```

　　删除一个指定的变量，让它变成未定义的。如果指定了CACHE选项，那么这个变量将会从cache中删除而不是当前作用域。<variable>可以是一个环境变量，比如：

```
  unset(ENV{LD_LIBRARY_PATH})
```

在这个例子中，这个变量将会从当前的环境中被删除。

## CMD#79 : variable_watch

监视CMake变量的改变。

```
  variable_watch(<variable name> [<command to execute>])
```

　　如果给定的变量发生了变化，关于正在被改写的变脸的消息会被打印出来。如果指定了command选项，这条命令会被执行。这条命令会接受这样的参数：COMMAND(<variable> <access> <value> <current list file> <stack>)

## CMD#80: while

当条件为真时，评估（执行）一组命令。

```
  while(condition)
    COMMAND1(ARGS ...)
    COMMAND2(ARGS ...)
    ...
  endwhile(condition)
```

　　所有在while和与之配对的endwhile之间的命令将会被记录，但并不会执行。只有当endwhile被评估，并且条件为真时，这个命令列表的记录才会被调用。条件值的评估与if命令使用相同的逻辑。 
　　

## CMake变量

CMake变量按功能分有主要有四种不同的类型： 
1.) 提供信息的变量[共53个]； 
2.) 改变行为的变量[共23个]； 
3.) 描述系统的变量[共24个]； 
4.)控制构建过程的变量[共22个]。 
此外还有一些变量因编译使用的语言不同而不同，将它们归为第五类[共29个]。

由于变量比较多，这里只给出变量的大概描述；具体作用可使用cmake --help-variable variable_name命令查看。

#### 1、提供信息的变量

　　VAR#1-1 : CMAKE_AR 静态库的归档工具名字。

　　VAR#1-2 : CMAKE_BINARY_DIR 构建树的顶层路径。

　　VAR#1-3 : CMAKE_BUILD_TOOL 实际构建过程中使用的工具。

　　VAR#1-4 : CMAKE_CACHEFILE_DIR 文件CMakeCache.txt所在的路径。

　　VAR#1-5 : CMAKE_CACHE_MAJOR_VERSION 用于创建CMakeCache.txt文件的CMake的主版本号。

　　VAR#1-6 : VCMAKE_CACHE_MINOR_VERSION 用于创建CMakeCache.txt文件的CMake的子版本号。

　　VAR#1-7 : CMAKE_CACHE_PATCH_VERSION 用于创建CMakeCache.txt文件的CMake的补丁号。

　　VAR#1-8 : CMAKE_CFG_INTDIR 构建时，与构建配置相对应的输出子路径（只读）。

　　VAR#1-9 : CMAKE_COMMAND 指向CMake可执行文件的完整路径。

　　VAR#1-10 : CMAKE_CROSSCOMPILING 指出CMake是否正在交叉编译。

　　VAR#1-11 : CMAKE_CTEST_COMMAND 与cmake一起安装的ctest命令的完整路径。

　　VAR#1-12 : CMAKE_CURRENT_BINARY_DIR 当前正在被处理的二进制目录的路径。

　　VAR#1-13 : CMAKE_CURRENT_LIST_DIR 当前正在处理的listfile的完整目录。

　　VAR#1-14 : CMAKE_CURRENT_LIST_FILE 当前正在处理的listfile的完整路径。

　　VAR#1-15 : CMAKE_CURRENT_LIST_LINE 当前正在处理的listfile的行号。

　　VAR#1-16 : CMAKE_CURRENT_SOURCE_DIR 指向正在被处理的源码目录的路径。

　　VAR#1-17 : CMAKE_DL_LIBS 包含dlopen和dlclose函数的库的名称。

　　VAR#1-18 : CMAKE_EDIT_COMMAND 指向cmake-gui或ccmake的完整路径。

　　VAR#1-19 : CMAKE_EXECUTABLE_SUFFIX(_<LANG>) 本平台上可执行文件的后缀。

　　VAR#1-20 : CMAKE_EXTRA_GENERATOR 构建本工程所需要的额外生成器。

　　VAR#1-21 : CMAKE_EXTRA_SHARED_LIBRARY_SUFFIXES 附加的共享库后缀（除CMAKE_SHARED_LIBRARY_SUFFIX以外，其他可以识别的共享库的后缀名。）

　　VAR#1-22 : CMAKE_GENERATOR 用于构建该工程的生成器。

　　VAR#1-23 : CMAKE_HOME_DIRECTORY 指向源码树顶层的路径。

　　VAR#1-24 : CMAKE_IMPORT_LIBRARY_PREFIX(_<LANG>) 需要链接的导入库的前缀。

　　VAR#1-25 : CMAKE_IMPORT_LIBRARY_SUFFIX(_<LANG>) 需要链接的导入库的后缀。

　　VAR#1-26 : CMAKE_LINK_LIBRARY_SUFFIX 需要链接的库的后缀。

　　VAR#1-27 : CMAKE_MAJOR_VERSION cmake的主版本号（例如2.X.X中的2）。

　　VAR#1-28 : CMAKE_MAKE_PROGRAM 参见CMAKE_BUILD_TOOL。

　　VAR#1-29 : CMAKE_MINOR_VERSION cmake的次版本号（例如X.4.X中的4）。

　　VAR#1-30 : CMAKE_PARENT_LIST_FILE 当前正在被处理listfile的父listfile的全路径。

　　VAR#1-31 : CMAKE_PATCH_VERSION cmake的补丁号(例如X.X.3中的3)。

　　VAR#1-32 : CMAKE_PROJECT_NAME 当前工程的工程名。

　　VAR#1-33 : CMAKE_RANLIB 静态库的随机化工具的名字（比如linux下的ranlib）。

　　VAR#1-34 : CMAKE_ROOT CMake的安装路径。

　　VAR#1-35 : CMAKE_SHARED_LIBRARY_PREFIX(_<LANG>) 被链接的共享库的前缀。

　　VAR#1-36 : CMAKE_SHARED_LIBRARY_SUFFIX(_<LANG>) 被链接的共享库的后缀。

　　VAR#1-37 : CMAKE_SHARED_MODULE_PREFIX(_<LANG>) 被链接的可加载模块的前缀。

　　VAR#1-38 : CMAKE_SHARED_MODULE_SUFFIX(_<LANG>) 被链接的共享库的后缀。

　　VAR#1-39 : CMAKE_SIZEOF_VOID_P void指针的长度。

　　VAR#1-40 : CMAKE_SKIP_RPATH 如果变量为真，不为编译出的可执行文件添加运行时的路径信息。默认添加。

　　VAR#1-41 : CMAKE_SOURCE_DIR 源码树的顶层路径。

　　VAR#1-42 : CMAKE_STANDARD_LIBRARIES 链接到所有可执行文件和共享库上的库。这是一个list。

　　VAR#1-43 : CMAKE_STATIC_LIBRARY_PREFIX(_<LANG>) 被链接的静态库的前缀。

　　VAR#1-44 : CMAKE_STATIC_LIBRARY_SUFFIX(_<LANG>) 被链接的静态库的后缀。

　　VAR#1-45 : CMAKE_TWEAK_VERSION cmake的tweak版本号(例如X.X.X.1中的1)。

　　VAR#1-46 : CMAKE_USING_VC_FREE_TOOLS 如果用到了免费的visual studio工具，设置为真。

　　VAR#1-47 : CMAKE_VERBOSE_MAKEFILE 设置该变量为真将创建完整版本的makefile。

　　VAR#1-48 : CMAKE_VERSION cmake的完整版本号；格式为major.minor.patch[.tweak[-id]]。

　　VAR#1-49 : PROJECT_BINARY_DIR 指向工程构建目录的全路径。

　　VAR#1-50 : PROJECT_NAME 向project命令传递的工程名参数。

　　VAR#1-51 : PROJECT_SOURCE_DIR 当前工程的源码路径。

　　VAR#1-52 : [Project name]_BINARY_DIR 给定工程的二进制文件顶层路径。

　　VAR#1-53 : [Project name]_SOURCE_DIR 给定工程的源码顶层路径。