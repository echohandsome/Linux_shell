写本文的目的在于，学习杨璐所写的GBASE导数主程序的具体27个函数的作用。

具体需要对shell的基础知识一步一步来了解和掌握，以便后期提升自己的工程能力：

gbase_functions.sh请参见有道云笔记代码部分

- 【HP20190514】GBASE_functions.sh上传


## 需要用到的函数实例：
```bash
INPUT_DATE=201902
V_DATA_POSTFIX="${INPUT_DATE}"
V_TMP_TABLE_INFO="dy_lb.zy_acct_shoulditem_${V_DATA_POSTFIX}"
V_TMP_TABLE_NAME=${V_TMP_TABLE_INFO#*.}
V_TMP_TABLE_SCHEMA=${V_TMP_TABLE_INFO%%.*}

echo ${V_TMP_TABLE_INFO} #  dy_lb.zy_acct_shoulditem_201902
echo ${V_TMP_TABLE_NAME} # zy_acct_shoulditem_201902
echo ${V_TMP_TABLE_SCHEMA} # dy_lb

function test_table()
{
    local V_TMP_TABLE_INFO=$1
    local V_TMP_TABLE_NAME=${V_TMP_TABLE_INFO#*.}
    local V_TMP_TABLE_SCHEMA=${V_TMP_TABLE_INFO%%.*}
    echo -e "`date +%Y-%m-%d' '%T`    判断表 ${V_TMP_TABLE_SCHEMA}.${V_TMP_TABLE_NAME} 是否存在 ..." | tee -a ${V_LOG_FILE} 
    connect_to_database > /dev/null 2>&1
    local V_IS_TABLE_EXISTS_STRING=$(db2 "values $(whoami).is_table_exists_fun('${V_TMP_TABLE_NAME}','${V_TMP_TABLE_SCHEMA}')")
    local V_IS_TABLE_EXISTS=$(echo ${V_IS_TABLE_EXISTS_STRING} | awk '{print $3}')
    if [ "${V_IS_TABLE_EXISTS}" = "1" ]; then 
        echo "`date +%Y-%m-%d' '%T`    ${V_TMP_TABLE_SCHEMA}.${V_TMP_TABLE_NAME} 表存在。" | tee -a ${V_LOG_FILE} 
    else 
        echo "`date +%Y-%m-%d' '%T`    ${V_TMP_TABLE_SCHEMA}.${V_TMP_TABLE_NAME} 表不存在。退出。" | tee -a ${V_LOG_FILE} 
        exit 1 
    fi 
}
```

### 1. shell代码规范
函数名用小写，变量名用大写，方便调用的时候识别
###  2. whoami命令
在linux中输入whoami将展示当前用户名
```bash
[hp@hadoop16 ~]$ whoami
hp
```
### 3. local 变量类型
主要用于局部变量申明，凡是被local命令申明的变量类型，均只能本函数内有效，在其他函数中无法进行调用。
```bash
[hp@hadoop16 ~]$ local V_TMP_TABLE_INFO=$1
[hp@hadoop16 ~]$ local V_TMP_TABLE_NAME=${V_TMP_TABLE_INFO#*.}
[hp@hadoop16 ~]$ local V_TMP_TABLE_SCHEMA=${V_TMP_TABLE_INFO%%.*}
```
**在以上的实例中，$1定义的时候脚本稍后将要输入的参数名称。**

### 4. Linux awk 命令
AWK是一种处理文本文件的语言，是一个强大的文本分析工具。
```bash
local V_IS_TABLE_EXISTS=$(echo ${V_IS_TABLE_EXISTS_STRING} | awk '{print $3}')
```
此处的实例意思为，选择这个变量中的第三个顺位的文本字符串。


```bash
DATA_CNT=$(gbase -u${V_GDB_USER} -p${V_GDB_PASSWD} -h${V_GDB_HOST} -e "${SQL_STR}" 2>&1)
DATA_CNT=$(echo ${DATA_CNT} | awk -F ' ' '{print $2}') 
```
awk是一个强大的文本分析工具，相对于grep的查找，sed的编辑，awk在其对数据分析并生成报告时，显得尤为强大。

++简单来说awk就是把文件逐行的读入，以空格为默认分隔符将每行切片，切开的部分再进行各种分析处理。++


awk中结合{print $1}使用的具体案例如下：
```bash
[hp@hadoop16 ~]$ echo ${a}
aa bb cc
[hp@hadoop16 ~]$ echo "aa bb cc" | awk -F ' ' '{print $1}' 
aa

# {print $1}就是将某一行（一条记录）中以空格为分割符的第一个字段打印出来
```


### 5. shell中变量名后添加#*,##*,#*,##*,% *,%% *的含义及用法

可以参阅

https://blog.csdn.net/jiezi2016/article/details/79649382

https://blog.csdn.net/wangzhaotongalex/article/details/73321766

假设定义了一个变量为：
代码如下:

file=/dir1/dir2/dir3/my.file.txt

可以用${ }分别替换得到不同的值：

${file#*/}：删掉第一个 / 及其左边的字符串：dir1/dir2/dir3/my.file.txt

${file##*/}：删掉最后一个 /  及其左边的字符串：my.file.txt

${file#*.}：删掉第一个 .  及其左边的字符串：file.txt

${file##*.}：删掉最后一个 .  及其左边的字符串：txt

${file%/*}：删掉最后一个  /  及其右边的字符串：/dir1/dir2/dir3

${file%%/*}：删掉第一个 /  及其右边的字符串：(空值)

${file%.*}：删掉最后一个  .  及其右边的字符串：/dir1/dir2/dir3/my.file

${file%%.*}：删掉第一个  .   及其右边的字符串：/dir1/dir2/dir3/my

记忆的方法为：

     # 是去掉左边（键盘上#在 $ 的左边）
 
     % 是去掉右边（键盘上% 在$ 的右边）

**单一符号是最小匹配；两个符号是最大匹配**

本实例中的代码部分

```bash
local V_TMP_TABLE_NAME=${V_TMP_TABLE_INFO#*.}
# 删掉V_TMP_TABLE_INFO第一个.及其左边的字符串
local V_TMP_TABLE_SCHEMA=${V_TMP_TABLE_INFO%%.*}
# 删掉V_TMP_TABLE_SCHEMA第一个.及其右边的字符串
```

###  6. tee 命令
Linux tee命令用于读取标准输入的数据，并将其内容输出成文件。

**参数：**

-a或--append 　附加到既有文件的后面，而非覆盖它。

-i或--ignore-interrupts 　忽略中断信号。

--help 　在线帮助。

--version 　显示版本信息。

以上案例中的写法，代表着将日志记录输入附加到日志文件变量中去。



### 7. shell $()命令替换

在bash中，$( )与` `（反引号）都是用来作命令替换的。

命令替换与变量替换差不多，都是用来重组命令行的，先完成引号里的命令行，然后将其结果替换出来，再重组成新的命令行。

```bash
local V_IS_TABLE_EXISTS_STRING=$(db2 "values $(whoami).is_table_exists_fun('${V_TMP_TABLE_NAME}','${V_TMP_TABLE_SCHEMA}')")
```
在这个实例中，使用了命令替换。
```bash
db2 "values $(whoami).is_table_exists_fun('${V_TMP_TABLE_NAME}','${V_TMP_TABLE_SCHEMA}')"
```
这一部分实际上是一个命令，当命令的结果出来以后，再赋值给V_IS_TABLE_EXISTS_STRING这个变量。

```bash
[root@localhost ~]# echo today is $(date "+%Y-%m-%d")
today is 2017-11-07
[root@localhost ~]# echo today is `date "+%Y-%m-%d"`
today is 2017-11-07
```

$( )与｀｀
在操作上，这两者都是达到相应的效果，但是建议使用$( )，理由如下：

｀｀很容易与''搞混乱，尤其对初学者来说，而$( )比较直观。
最后，$( )的弊端是，并不是所有的类unix系统都支持这种方式，但反引号是肯定支持的。

### 8. && 运算符(and)

```bash
command1 && command2
```
&&左边的命令（命令1）返回真(即返回0，成功被执行）后，&&右边的命令（命令2）才能够被执行；换句话说，“如果这个命令执行成功&&那么执行这个命令”。
### 9. || 运算符(or) 和 | 运算符

**|| 运算符**
```bash
command1 || command2
```
||则与&&相反。如果||左边的命令（command1）未执行成功，那么就执行||右边的命令（command2）；或者换句话说，“如果这个命令执行失败了||那么就执行这个命令。

**|运算符**

管道符号，是unix一个很强大的功能,符号为一条竖线:"|"

用法:

```bash
command1 | command2
```
他的功能是把第一个命令command 1执行的结果作为command2的输入传给command 2。


### 10. touch修改文件时间属性

Linux touch命令用于修改文件或者目录的时间属性，包括存取时间和更改时间。若文件不存在，系统会建立一个新的文件。

```bash
touch -f  # -f 是为了确保和其他unix系统的兼容性而添加的参数
```


### 11. cat命令

其实就是将文件内容读取并输出到屏幕显示出来，例如如下：
```bash 
cat ${FILE_PATH_HP}/data_finish_check.log
#显示结果
201902_dwd_cust_unit_info.finish
201903_base_item_code_fenlei.finish
201903_dw_5_4_danger.finish
201903_dw_group_meb.finish
201903_kd_user_t.finish
201903_ods_demand_tv_pay.finish
201903_user_contact.finish
201903_user_dualcard.finish

```

### 12. > 和 >> 输出符号
    > 是指将输入的内容运行结果从符号左边输入，符号右边输出，输出新的内容将替换原来的内容
    >> 同样是输出，区别在于追加的方式输出。
    
    
### 13. 变量使用单引号‘’、双引号“”、反引号``的不同区别

**1. 单引号**

使用单引号的情况下，不管里面的是否有变量或者其他的表达是都是原样子输出

**2. 双引号**

如果其定义变量的时候使用双引号的话，则里面的变量或者函数会通过解析，解析完成后再输出内容，而不是把双引号中的变量名以及命令原样子输出。

==这样做，可以防止变量没有被完整的解析出来命令。==

**3. 不使用引号**

用于一些简单字符数字的定义与双引号类似

### 14. PID文件的作用

PID文件主要表达为进程编号，用于识别进程是否正在运行的唯一性，确保该进行当前不会重复执行

PID文件会根据程序的不同存储在固定的一个PID文件夹内，至少在杨璐的GBASE程序内是这样的，那么这样就会用于识别不同程序之间的进程的唯一性。

```bash
#关于PID进程编号的原理指导：

PID进程定义：一个shell程序执行的时候，那么shell程序会根据程序名称定义一个进程编号，这个进程编号在这个程序运行状态下是唯一的。当程序运行完毕退出后，那么程序的进程编号也同步会消失。当第二个启动同名称的同一个程序的时候，那么实际上进程编号也是不一致的。


PID进程的特例情况：当一个程序定义了多个参数可选项目的时候，实际上进程编号默认情况下是同一个，这个时候，为了确保识别不同的参数选项，就需要人为去定义同一个程序下的不同参数的进程编号，确保是不一致的，方便其他调用的程序能够准确的获取返回的被引用进程的不同参数的返回结果。


因此，在识别某一个程序是否正在运行的时候，实际上可以先定义一个文件用来存储当前正在运行的进程的编号。这样，当进程正在运行时，如果启动了同文件的新进程，那么可以先比对新进程是否和存储的进程编号一致，如果有一致的编号，就说明正在运行，退出，避免重复执行。
```


### 15. grep命令 查找文件里符合条件的字符串

grep指令用于查找内容包含指定的范本样式的文件，如果发现某文件的内容符合所指定的范本样式，预设grep指令会把含有范本样式的那一列显示出来。若不指定任何文件名称，或是所给予的文件名为"-"，则grep指令会从标准输入设备读取数据。


### 16. cut命令 显示每行从开头算起 num1 到 num2 的文字

cut 命令从文件的每一行剪切字节、字符和字段并将这些字节、字符和字段写至标准输出。

**参数设置如下：**

如果不指定 File 参数，cut 命令将读取标准输入。必须指定 -b、-c 或 -f 标志之一。

-b ：以字节为单位进行分割。这些字节位置将忽略多字节字符边界，除非也指定了 -n 标志。

-c ：以字符为单位进行分割。

-d ：自定义分隔符，默认为制表符。

-f ：与-d一起使用，指定显示哪个区域。

-n ：取消分割多字节字符。仅和 -b
标志一起使用。如果字符的最后一个字节落在由 -b 标志的 List 参数指示的
范围之内，该字符将被写出；否则，该字符将被排除

```bash 
# 截取文件中的每一行字符串的第二个字符
[hp@hadoop16 bin]$ who
hp       pts/0        2019-05-14 08:57 (192.168.0.60)
hp       pts/4        2019-05-09 01:38 (192.168.0.60)
[hp@hadoop16 bin]$ who|cut -b 2
p
p
```


### 17. [ ]符号表示if-test结构，左中括号调用test命令标识，右中括号关闭条件判断

这个命令把它的参数作为比较表达式或者作为文件测试，并且根据比较的结果来返回一个退出状态码。

```bash
# 当我们要获取一个[ ]结构的返回值是多少的时候可以使用以下命令
echo $?
```

**if-test结构中并不是必须右中括号，但是新版的Bash中要求必须这样。**

Test和[]中可用的比较运算符只有==和!=，两者都是用于字符串比较的，不可用于整数比较，整数比较只能使用-eq，-gt这种形式。

- **linux 下shell中if的“-e，-d，-f”是什么意思**


文件表达式

- -e filename 如果 filename存在，则为真

- -d filename 如果 filename为目录，则为真 

- -f filename 如果 filename为常规文件，则为真

- -L filename 如果 filename为符号链接，则为真

- -r filename 如果 filename可读，则为真 

- -w filename 如果 filename可写，则为真 

- -x filename 如果 filename可执行，则为真

- -s filename 如果文件长度不为0，则为真

- -h filename 如果文件是软链接，则为真

filename1 -nt filename2 如果 filename1比 filename2新，则为真。

filename1 -ot filename2 如果 filename1比 filename2旧，则为真。


参见：

https://www.jb51.net/article/123081.htm

http://www.zsythink.net/archives/2252/


### 18. [[ ]] 符号
- [[是 bash 程序语言的关键字。并不是一个命令
- [[ ]] 结构比[ ]结构更加通用。在[[和]]之间所有的字符都不会发生文件名扩展或者单词分割，但是会发生参数扩展和命令替换
- 支持字符串的模式匹配，使用=~操作符时甚至支持shell的正则表达式
- 使用[[ ... ]]条件判断结构，而不是[ ... ]，能够防止脚本中的许多逻辑错误
- 比如，&&、||、<和> 操作符能够正常存在于[[ ]]条件判断结构中，但是如果出现在[ ]结构中的话，会报错
- **bash把双中括号中的表达式看作一个单独的元素，并返回一个退出状态码**



### 19. echo 命令 + 参数

```bash
[hp@hadoop16 201905]$ echo -e "\n---表示转义字符不再是单独的字符串，变得有意义并执行---"

---表示转义字符不再是单独的字符串，变得有意义并执行---

```

### 20. let 命令

let 命令是 BASH 中用于计算的工具，用于执行一个或多个表达式，变量计算中不需要加上 $ 来表示变量。如果表达式中包含了空格或其他特殊字符，则必须引起来。

```bash
# 脱离let命令下的计算，最终会得不到真正的计算结果
[hp@hadoop16 201905]$ a=3
[hp@hadoop16 201905]$ b=2
[hp@hadoop16 201905]$ c=a-b
[hp@hadoop16 201905]$ echo ${c}
a-b
[hp@hadoop16 201905]$ let c=a-b
[hp@hadoop16 201905]$ echo ${c}
1
```



### 21.rm 删除命令

**rm 删除文件、文件夹**

- -r Recurve的首字母，递归的意思，删除此文件夹下所有文件 
- -f不提醒，直接删除 
- -i交互删除，会提醒 



### 22. Shell特殊变量： $0, $#, $*, $@, $?, $$和命令行参数

变量和含义
- $0	当前脚本的文件名
- $n	传递给脚本或函数的参数。n 是一个数字，表示第几个参数。例如，第一个参数是$1，第二个参数是$2。
- $#	传递给脚本或函数的参数个数。
- $*	传递给脚本或函数的所有参数。
- $@	传递给脚本或函数的所有参数。被双引号(" ")包含时，与 $* 稍有不同，下面将会讲到。
- $?	**上个命令的退出状态，或函数的返回值。0表示没有错误，其他任何值表明有错误**
- $$	当前Shell进程ID。对于 Shell 脚本，就是这些脚本所在的进程ID。


**$?使用方法举例如下：**
```bash
if [ -f ${FILE} ]; then  # if -f 是评估变量的文件是不是常规文件
        echo -e "\n`date +%Y-%m-%d' '%T`    文件 ${FILE} 存在, 删除..." | tee -a ${V_LOG_FILE} 
        rm -f ${FILE}
        if [ $? -ne 0 ];  # $?是指上个删除命令的退出结果，0表示没有错误，其他任何值表明有错误
        then
            echo "`date +%Y-%m-%d' '%T`    文件 ${FILE} 删除 失败(ERROR)。退出!" | tee -a ${V_LOG_FILE} 
            [ "${RETURN_FLAG}" = "return" ] && return 1 
            calculate_time 
            exit 1
```


**命令行参数**

运行脚本时传递给脚本的参数称为命令行参数。命令行参数用 $n 表示，例如，$1 表示第一个参数，$2 表示第二个参数，依次类推。

### 23. return 退出状态码 表示在调用本函数时，如果程序运行结束并退出，返回一个退出状态码

参考理解代码如下

```bash
# 检测文件, 如果存在则删除
# 输入参数
#   1   文件名称
#   2   返回标志 'return'表示返回, 不退出
# 使用变量
#   V_LOG_FILE   日志文件
function test_file_to_del()
{
    local FILE=$1 RETURN_FLAG=$2
    if [ -f ${FILE} ]; then  # if -f 是评估变量的文件是不是常规文件
        echo -e "\n`date +%Y-%m-%d' '%T`    文件 ${FILE} 存在, 删除..." | tee -a ${V_LOG_FILE} 
        rm -f ${FILE}
        if [ $? -ne 0 ];  # 显示最后命令的退出状态。0表示没有错误，其他任何值表明有错误。 -ne 不等于
        then
            echo "`date +%Y-%m-%d' '%T`    文件 ${FILE} 删除 失败(ERROR)。退出!" | tee -a ${V_LOG_FILE} 
            [ "${RETURN_FLAG}" = "return" ] && return 1 # 判断第二个参数是否是 return 此处返回退出状态码1
            calculate_time # 此处已经开始引用了第一个函数，计算程序运行时间
            exit 1
        else
            echo "`date +%Y-%m-%d' '%T`    文件 ${FILE} 删除 成功。" | tee -a ${V_LOG_FILE} 
        fi 
    fi 
}
```


### 24. exit 0 和 exit 1的区别

接23案例的例子可以更好的理解exit返回值的作用

```bash
exit 0
# 正常运行程序并退出程序；
exit 1
# 非正常运行导致退出程序；
```
- exit 0 可以告知你的程序的使用者：你的程序是正常结束的。如果 exit 非 0 值，那么你的程序的使用者通常会认为你的程序产生了一个错误。
- 在 shell 中调用完你的程序之后，用 echo $? 命令就可以看到你的程序的 exit 值。在 shell 脚本中，通常会根据上一个命令的 $? 值来进行一些流程控制。
- 当你 exit 0 的时候,在调用环境 echo $? 就返回0，也就是说调用环境就认为你的这个程序执行正确
- 当你 exit 1 的时候,一般是出错定义这个1，也可以是其他数字，很多系统程序这个错误编号是有约定的含义的。 
- 但不为0就表示程序运行出错。调用环境就可以根据这个返回值判断 你这个程序运行是否ok。
- 如果你用 脚本 a 调用 脚本b ，要在a中判断b是否正常返回，就是根据 exit 0 or 1 来识别。
- 执行完b后， 判断 $? 就是返回值



### 25. [-z ${a}] 和 [-z "${a}"] 的区别，什么时候加上双引号

- Shell 变量用双引号引起来，双引号就是表示这个双引号内为一个字符串。

- 对于 if 条件语句里所有的字符串的比较时，最好是在变量的外面加上双引号。特别是 if -n 判断字符串是否为null时候(null意思就是字符串长度为0)，一定要加上双引号。否则，像下面的case就会出错。
```bash
a=""

if [ -n $a ] 等价于 if[ -n ]

# 对于字符串长度为0时，相当于没有参数，这句总返回为真。明明a的为空串，长度为空，但是却判断出来为非空字符串。

改为if [ -n "$a" ]就没有此问题，可以判断出来为此为空字符串。
```


### 26. [-z ] 和 [-n ] 的区别

```bash
-z 判断 变量的值，是否为空； zero = 0

- 变量的值，为空，返回0，为true
- 变量的值，非空，返回1，为false
-n 判断变量的值，是否为空   name = 名字
- 变量的值，为空，返回1，为false
- 变量的值，非空，返回0，为true
pid="123"
  [ -z "$pid" ]  单对中括号变量必须要加双引号
 [[ -z $pid ]]   双对括号，变量不用加双引号

 [ -n "$pid" ]  单对中括号，变量必须要加双引号
 [[ -z  $pid ]]  双对中括号，变量不用加双引号
```


### 27. basename 去掉目录只要文件名 和 dirname 去掉文件名只要目录

示例如下：

```bash
# basename 取文件名

shell>temp=/home/temp/1.test
shell>base=`basename ${temp}`
shell>echo ${base}
结果为：1.test


# dirname 是取目录

shell>temp=/home/temp/1.test
shell>dir=`dirname ${temp}`
shell>echo ${dir}
结果为：/home/temp

另一种实现的方法：

${var##*/} 就是把变量var最后一个/以及左边的内容去掉

${var%/*} 就是把变量var最后一个/以及右边的内容去掉

```


### 28. sed命令 用于对文本文件内容进行编辑

Linux sed命令是利用脚本来处理文本文件。

sed可依照script的指令，来处理、编辑文本文件。

Sed主要用来自动编辑一个或多个文件；简化对文件的反复操作；编写转换程序等。


### 29.   typeset -l 将字符串全部变为小写

### 30.  时间后面跟上  -d 表示将-d前面的时间与-d后面的内容连接起来，是一个扩展功能

案例如下
```bash
[ -z "${DATE_CHECK}" ]; then 
        DATA_DATE=$(date +%Y%m -d "${DATA_DATE}01" 2>&1)
        ERR_FLAG=$?
        DATA_DATE_=$(date +%Y-%m -d "${DATA_DATE}01" 2>&1)
```
这里就是将前面 date +%Y%m -d 输入到 ${DATA_DATE} 这个变量中去

同时还有另外一个功能就是用于计算date 和后面的时间差，案例如下：

```bash
[hp@hadoop16 ~]$ date +%Y%m%d
20190522
[hp@hadoop16 ~]$ date +%Y%m%d -d "-3 days"
20190519

```



### 31. local 在函数中定义为局部变量，仅在当前函数内调用有效

```bash
function drop_table()
{
    typeset -l TABLE_NAME
    local TABLE_NAME=$1 
    local ERR_MSG SQL_STR 
    local VAR_UNSET_FLAG=0
}
```
在上面的案例中，local 定义的变量，均只在本函数内进行调用的意思。


### 32. df 命令，检查磁盘空间情况命令

**必要参数：**

- -a 全部文件系统列表
- -h 方便阅读方式显示
- -H 等于“-h”，但是计算式，1K=1000，而不是1K=1024
- -i 显示inode信息
- -k 区块为1024字节
- -l 只显示本地文件系统
- -m 区块为1048576字节
- --no-sync 忽略 sync 命令
- -P 输出格式为POSIX
- --sync 在取得磁盘信息前，先执行sync命令
- -T 文件系统类型

示例如下：

```bash
[hp@hadoop16 ~]$ df -h
Filesystem      Size  Used Avail Use% Mounted on
/dev/sda2        50G  3.4G   44G   8% /
tmpfs           7.8G     0  7.8G   0% /dev/shm
/dev/sda1       190M   40M  141M  23% /boot
/dev/sda7       607G   29G  548G   5% /home
/dev/sda6        50G  2.3G   45G   5% /usr
/dev/sda3       148G   30G  111G  21% /var
/dev/sdb1       931G  134G  797G  15% /mnt/disk1
cm_processes    7.8G   30M  7.8G   1% /var/run/cloudera-scm-agent/process
```


### 33.[[ =~ ]] 表示在if中使用正则匹配


### 34. Linux eval命令读取一连串的参数，然后按照参数特性来执行

eval会对后面的命令进行两遍扫描，如果第一遍扫描后，命令是个普通命令，则执行此命令