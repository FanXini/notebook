---
typora-copy-images-to: ..\..\img
---

## 1. CMD下面登陆mysql

**进入cmd 输入 mysql -u admin-p password **

![1550798063425](..\..\img\1550798063425.png)

## 2. 怎么查看帮助文档

```cmd
mysql> ? contents
You asked for help about help category: "Contents"
For more information, type 'help <item>', where <item> is one of the following
categories:
   Account Management
   Administration
   Compound Statements
   Data Definition
   Data Manipulation
   Data Types
   Functions
   Functions and Modifiers for Use with GROUP BY
   Geographic Features
   Help Metadata
   Language Structure
   Plugins
   Procedures
   Storage Engines
   Table Maintenance
   Transactions
   User-Defined Functions
   Utility

mysql>
```

```cmd
mysql> ? functions
You asked for help about help category: "Functions"
For more information, type 'help <item>', where <item> is one of the following
categories:
   Bit Functions
   Comparison operators
   Control flow functions
   Date and Time Functions
   Encryption Functions
   Information Functions
   Logical operators
   Miscellaneous Functions
   Numeric Functions
   String Functions
```

```c
mysql> ? String Functions
You asked for help about help category: "String Functions"
For more information, type 'help <item>', where <item> is one of the following
topics:
   ASCII
   BIN
   BINARY OPERATOR
   BIT_LENGTH
   CAST
   CHAR FUNCTION
   CHARACTER_LENGTH
   CHAR_LENGTH
   CONCAT
   CONCAT_WS
   CONVERT
   ELT
   EXPORT_SET
   EXTRACTVALUE
   FIELD
   FIND_IN_SET
   FORMAT
   HEX
   INSERT FUNCTION
   INSTR
   LCASE
   LEFT
   LENGTH
   LIKE
   LOAD_FILE
   LOCATE
   LOWER
   LPAD
   LTRIM
   MAKE_SET
   MATCH AGAINST
   MID
   NOT LIKE
   NOT REGEXP
   OCT
   OCTET_LENGTH
   ORD
   POSITION
   QUOTE
   REGEXP
   REPEAT FUNCTION
   REPLACE FUNCTION
   REVERSE
   RIGHT
   RPAD
   RTRIM
   SOUNDEX
   SOUNDS LIKE
   SPACE
   STRCMP
   SUBSTR
   SUBSTRING
   SUBSTRING_INDEX
   TRIM
   UCASE
   UNHEX
   UPDATEXML
   UPPER

```

```cmd
mysql> ? concat
Name: 'CONCAT'
Description:
Syntax:
CONCAT(str1,str2,...)

Returns the string that results from concatenating the arguments. May
have one or more arguments. If all arguments are nonbinary strings, the
result is a nonbinary string. If the arguments include any binary
strings, the result is a binary string. A numeric argument is converted
to its equivalent string form. This is a nonbinary string as of MySQL
5.5.3. Before 5.5.3, it is a binary string to avoid that and produce a
nonbinary string, you can use an explicit type cast, as in this
example:

SELECT CONCAT(CAST(int_col AS CHAR), char_col)

CONCAT() returns NULL if any argument is NULL.

URL: http://dev.mysql.com/doc/refman/5.5/en/string-functions.html

Examples:
mysql> SELECT CONCAT('My', 'S', 'QL')
        -> 'MySQL'
mysql> SELECT CONCAT('My', NULL, 'QL')
        -> NULL
mysql> SELECT CONCAT(14.3)
        -> '14.3'

```

## 3. [Navicat]常用快捷键

1. ctrl + q: 打开新查询窗口

2. ctrl + r: 运行当前窗口内的所有语句

3. ctrl + w: 关闭当前窗口

4. F6: 打开一个mysql命令行窗口

---------------------------以下是然并卵的快捷键----------------------------------------

5. ctrl + n: 打开新查询窗口

6. ctrl + shit + r: 只运行选中的语句

7. ctrl + /: 注释

8. ctrl + shift + /: 取消注释

9. ctrl + l: 删除一行

10. F7: 运行从光标当前位置开始的一条完整sql语句

11. ctrl + d: 在表数据窗口上查看表定义