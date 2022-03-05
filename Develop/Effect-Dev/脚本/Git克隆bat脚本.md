# Git克隆bat脚本
```bash
/--------------------开始--------------------------/
@echo off
TITLE RongHeFangAn GIT
echo Start git clone
echo=

pause
echo=

rem 下载 funcion developer 分支
git clone git@gitlab.mfexcel.com:product-develop-express/common/fusion.git -b developer

rem 下载 midware2 developer 分支
git clone git@gitlab.mfexcel.com:product-develop-express/midWare/midware2.git -b developer

rem 下载 uiCoreAccessLib developer 分支
git clone git@gitlab.mfexcel.com:product-develop-express/commonlib/uiCoreAccessLib.git -b developer

rem 下载 smsCompileLib developer 分支
git clone git@gitlab.mfexcel.com:product-develop-express/commonlib/smsCompileLib.git -b developer

echo=
echo Finish git clone

echo=
pause
/-------------------------结束------------------------/
```

名词：
**echo** 表示显示此命令后的字符 
**echo on**  表示在此语句后所有运行的命令都显示命令行本身 
**echo off** 表示在此语句后所有运行的命令都不显示命令行本身
**@**与**echo off**相像，但它是加在每个命令行的最前面，表示运行时不显示这一行的命令行（只能影响当前行）。
**call** 调用另一个批处理文件（如果不用call而直接调用别的批处理文件，那么执行完那个批处理文件后将无法返回当前文件并执行当前文件的后续命令）。
**pause** 运行此句会暂停批处理的执行并在屏幕上显示Press any key to continue...的提示，等待用户按任意键后继续
**rem** 表示此命令后的字符为注释，不执行。
**title** BAT的标题
**cls** 清除屏幕