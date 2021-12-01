## 背景
有A、B两个用户id文件，希望筛选出只存在于B文件中的所有行。

## 解决方案

### comm
`comm`是对两个已经`有序`的文件进行比较，可以比较输出：仅在A中出现的、仅在B中出现的、在两个文件中都存在的。
直接使用comm的话会输出三列，第一列为A独有的、第二列为B独有的、第三列为C独有的，如果在comm后面加数字，则指明不展示这一列。

```shell
comm -1 A B 不显示在A文件中独有内容(显示B文件独有内容+两个文件共有)
comm -2 A B 不显示在B文件中独有内容
comm -3 A B 不显示同时在两个文件中都存在的内容
comm -12 A B 显示A与B公共的部分
comm -23 A B 显示A独有的
comm -13 A B 显示B独有的
```

comm --help

```shell
用法：comm [选项]... 文件1 文件2
逐行比较已排序的文件文件1 和文件2。

如果不附带选项，程序会生成三列输出。第一列包含文件1特有的行，第二列包含文件2特有的行，而第三列包含两个文件共有的行。

-1 不输出文件1 特有的行
-2 不输出文件2 特有的行
-3 不输出两个文件共有的行

--check-order 检查输入是否被正确排序，即使所有输入行均成对
--nocheck-order 不检查输入是否被正确排序
--output-delimiter=STR 依照STR 分列
--help 显示此帮助信息并退出
--version 显示版本信息并退出

Note, comparisons honor the rules specified by 'LC_COLLATE'.

示例：
comm -12 文件1 文件2 只打印在文件1 和文件2 中都有的行
comm -3 文件1 文件2 打印在文件1 中有，而文件2 中没有的行。反之亦然。

```
### grep

grep一般用来查找筛选，这次意外发现它也可用通过-f参数来进行文件比较
-f参数用法：
>-f, --file=FILE           take PATTERNS from FILE

示例：
```shell
grep -f A B  #筛选出B文件中与A相同的行

grep -v -f A B  #筛选出B文件中独有的行
```

## 总结
要解决开篇提到的问题，从A、B文件中筛选出B文件中独有的行，只需：
```shell
grep -v -f A B

```
或者
```shell
# 使用commn之前先对文件进行排序
 cat A | sort | uniq > A_sort.txt

 cat B | sort | uniq > B_sort.txt

 comm -13 A_sort.txt B_sort.txt 
```