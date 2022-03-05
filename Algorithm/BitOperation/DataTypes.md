<h1 align="center">Data Types</h1>

[toc]

## 位(bit)

来自英文 bit，音译为“比特”，表示二进制位。位是计算机内部数据储存的最小单位，11010100是一个 8 位二进制数。一个二进制位只可以表示 0 和 1 两种状态；两个二进制位可以表示 00、01、10、11 四种状态；三位二进制数可表示八种状态……。

## 字节(byte)

字节来自英文 Byte，音译为“拜特”，习惯上用大写的“B”表示。

字节是计算机中数据处理的基本单位。计算机中以字节为单位存储和解释信息，规定**一个字节由八个二进制位**构成，即 1 个字节等于 8 个比特**（1Byte=8bit）**。八位二进制数最小为 00000000，最大为 11111111；通常 1 个字节可以存入一个 ASCII 码，2 个字节可以存放一个汉字国标码。

在64位系统中Java基本类型占用的字节数：

| byte , boolean | 1字节 | 8位  |
| -------------- | ----- | ---- |
| short , char   | 2字节 | 16位 |
| int , float    | 4字节 | 32位 |
| long , double  | 8字节 | 64位 |

**编码与中文：**
Unicode/GBK：   中文2字节
UTF-8： 		中文通常3字节，在拓展B区之后的是4字节
综上，中文字符在编码中占用的字节数一般是2-4个字节。



## Integer Types

|      Type      |           Storage size            |                     Value range                      |
| :------------: | :-------------------------------: | :--------------------------------------------------: |
|      char      |              1 byte               |               -128 to 127 or 0 to 255                |
| unsigned char  |              1 byte               |                       0 to 255                       |
|  signed char   |              1 byte               |                     -128 to 127                      |
|      int       |           2 or 4 bytes            | -32,768 to 32,767 or -2,147,483,648 to 2,147,483,647 |
|  unsigned int  |           2 or 4 bytes            |          0 to 65,535 or 0 to 4,294,967,295           |
|     short      |              2 bytes              |                  -32,768 to 32,767                   |
| unsigned short |              2 bytes              |                     0 to 65,535                      |
|      long      | 8 bytes or (4bytes for 32 bit OS) |     -9223372036854775808 to 9223372036854775807      |
| unsigned long  |              8 bytes              |              0 to 18446744073709551615               |

## Floating-Point Types

|    Type     | Storage size |      Value range       |     Precision     |
| :---------: | :----------: | :--------------------: | :---------------: |
|    float    |    4 byte    |   1.2E-38 to 3.4E+38   | 6 decimal places  |
|   double    |    8 byte    |  2.3E-308 to 1.7E+308  | 15 decimal places |
| long double |   10 byte    | 3.4E-4932 to 1.1E+4932 | 19 decimal places |





