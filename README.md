
# python bytecode解析


# 前言


我们的电脑是怎么运行的呢？计算机内部的 CPU 处理器是个硅片，上面雕刻着精心布置的电路，输入特定的电流，就能得到另一种模式的电流，而且模式可以预测，给这些模式起上名字并赋予含义，我们就可以说这种电流模式代表加法，电脑的工作原理就是如此，我们起的这些名字叫做 CPU 指令，有时也被成为机器码。​\[引自:James Bennett] 我们的编程语言是怎么运行的呢？一些语言通过编译器，直接将源代码编译成机器码，这些语言就是编译语言，还有一些语言解除解释器，直接在运行时把源代码解释为机器码，这些就是解释型语言。不过还有第三种语言，介于源代码和机器码之间，一些语言编译得到的指令，但是这种指令不能被现有的CPU直接运行，而需要解释器去理解，并将这些指令翻译为真实的 CPU 接受的二进制码，这种中间指令就是我们今天要说的bytecode（字节码），有很多语言属于此类比如java，C\#，还有python。


Java编译的字节码运行在java虚拟机上，C\#编译的字节码运行在.Net 虚拟机上，而Python 编译的字节码运行在 Python 虚拟机上。


‍


# 工作原理


CPython解释器在内部会将Python源代码编译成[字节码](https://github.com)，并缓存在`.pyc`​文件中，目的是当再次执行该文件时，直接读取`.pyc`​文件会更快，这样可以避免从源码重新编译到字节码，当然，Python再找到符合文件后，检查此文件的时间戳，如果发现字节码文件（文件在导入时就被编译完成）比源代码文件时间戳早（比如你修改过原文件），那么就会重新生成字节码，否则就会跳过此步骤。如果，Python在搜索时只找到了字节码而没有找到源代码文件，那么就会直接执行字节码文件（如果没有印象，请回想在模块导入时发生了什么）。然后，Python虚拟机执行字节码编译器发出的字节码。


‍


# 面向栈


这个是在看**码农高天**（一个非常厉害的pytohn核心开发者）的视频里学到的概念，CPython使用一个基于栈的虚拟机，也就是说，它完全是面向栈，这种数据结构的。就是不断地push、pop。


CPython使用3种类型的栈：


* 调用栈（call stack）。这是运行Python程序的主要结构，它为每个当前活动的函数调用，使用了一个东西`帧（frame）`​，**栈底是程序的入口点，每个函数调用推送一个新的帧到调用栈，当函数调用返回后，这个帧被销毁**。
* 计算栈（evaluation stack，或称数据栈data stack）。在每个帧中，计算栈就是函数运行的地方，运行的代码大多数是由推入到这个栈中的东西组成的。在栈中操作它们，当函数被返回后，销毁它们。
* 块栈（block stack）。在每个帧中，块栈被Python用于跟踪某些类型的控制结构，如循环、`try/except`​块和`with ... as ...`​块 ，这些控制结构全部被推入到块栈中，当退出这些控制结构式，块栈被销毁，这将帮助Python了解任意给定时刻哪个块是活动的，比如一个continue或者break语句，这些可能影响结果的块。


大多数Python字节码指令操作的是当前调用栈的计算栈，虽然还有些指令可以做其他的事情，比如跳转到指定指令，或者操作块栈。


‍


# 字节码的阅读


其实还有 **代码对象** 和 **字节码的工作** 这两个概念没说，因为本文主要讲怎么阅读字节码，能够通过字节码手搓出py源码（是的，我是个CTFer），想了解更多的可以去本文末的推荐链接里看。


‍


## dis模块


因为python代码运行是到字节码再到机器码一气呵成的，我们想要看到中间指令，需要借助python的标准库dis模块，它可以将py代码翻译成字节码。如下：


​![image](https://img2023.cnblogs.com/blog/3400631/202411/3400631-20241102004730962-322730520.png)​


‍


## 案例


2024网鼎杯青龙组初赛的MISC02题目(1 解)，是一个linux内存镜像取证，前面繁琐的步骤略过，最后一步获得一个 flag.txt 文件，里面是python字节码，明显需要我们手搓还原py代码，如下：



```
 31         226 PUSH_NULL
            228 LOAD_NAME                8 (key_encode)
            230 LOAD_NAME                7 (key)
            232 PRECALL                  1
            236 CALL                     1
            246 STORE_NAME               7 (key)

 32         248 PUSH_NULL
            250 LOAD_NAME               10 (len)
            252 LOAD_NAME                7 (key)
            254 PRECALL                  1
            258 CALL                     1
            268 LOAD_CONST               7 (16)
            270 COMPARE_OP               2 (==)
            276 POP_JUMP_FORWARD_IF_FALSE    43 (to 364)

 33         278 PUSH_NULL
            280 LOAD_NAME                9 (sm4_encode)
            282 LOAD_NAME                7 (key)
            284 LOAD_NAME                5 (flag)
            286 PRECALL                  2
            290 CALL                     2
            300 LOAD_METHOD             11 (hex)
            322 PRECALL                  0
            326 CALL                     0
            336 STORE_NAME              12 (encrypted_data)

 34         338 PUSH_NULL
            340 LOAD_NAME                6 (print)
            342 LOAD_NAME               12 (encrypted_data)
            344 PRECALL                  1
            348 CALL                     1
            358 POP_TOP
            360 LOAD_CONST               2 (None)
            362 RETURN_VALUE

 32     >>  364 LOAD_CONST               2 (None)
            366 RETURN_VALUE

Disassembly of :
 10           0 RESUME                   0

 11           2 LOAD_GLOBAL              1 (NULL + list)
             14 LOAD_FAST                0 (key)
             16 PRECALL                  1
             20 CALL                     1
             30 STORE_FAST               1 (magic_key)

 12          32 LOAD_GLOBAL              3 (NULL + range)
             44 LOAD_CONST               1 (1)
             46 LOAD_GLOBAL              5 (NULL + len)
             58 LOAD_FAST                1 (magic_key)
             60 PRECALL                  1
             64 CALL                     1
             74 PRECALL                  2
             78 CALL                     2
             88 GET_ITER
        >>   90 FOR_ITER               105 (to 302)
             92 STORE_FAST               2 (i)

 13          94 LOAD_GLOBAL              7 (NULL + str)
            106 LOAD_GLOBAL              9 (NULL + hex)
            118 LOAD_GLOBAL             11 (NULL + int)
            130 LOAD_CONST               2 ('0x')
            132 LOAD_FAST                1 (magic_key)
            134 LOAD_FAST                2 (i)
            136 BINARY_SUBSCR
            146 BINARY_OP                0 (+)
            150 LOAD_CONST               3 (16)
            152 PRECALL                  2
            156 CALL                     2
            166 LOAD_GLOBAL             11 (NULL + int)
            178 LOAD_CONST               2 ('0x')
            180 LOAD_FAST                1 (magic_key)
            182 LOAD_FAST                2 (i)
            184 LOAD_CONST               1 (1)
            186 BINARY_OP               10 (-)
            190 BINARY_SUBSCR
            200 BINARY_OP                0 (+)
            204 LOAD_CONST               3 (16)
            206 PRECALL                  2
            210 CALL                     2
            220 BINARY_OP               12 (^)
            224 PRECALL                  1
            228 CALL                     1
            238 PRECALL                  1
            242 CALL                     1
            252 LOAD_METHOD              6 (replace)
            274 LOAD_CONST               2 ('0x')
            276 LOAD_CONST               4 ('')
            278 PRECALL                  2
            282 CALL                     2
            292 LOAD_FAST                1 (magic_key)
            294 LOAD_FAST                2 (i)
            296 STORE_SUBSCR
            300 JUMP_BACKWARD          106 (to 90)

 15     >>  302 LOAD_GLOBAL              3 (NULL + range)
            314 LOAD_CONST               5 (0)
            316 LOAD_GLOBAL              5 (NULL + len)
            328 LOAD_FAST                0 (key)
            330 PRECALL                  1
            334 CALL                     1
            344 LOAD_CONST               6 (2)
            346 PRECALL                  3
            350 CALL                     3
            360 GET_ITER
        >>  362 FOR_ITER               105 (to 574)
            364 STORE_FAST               2 (i)

 16         366 LOAD_GLOBAL              7 (NULL + str)
            378 LOAD_GLOBAL              9 (NULL + hex)
            390 LOAD_GLOBAL             11 (NULL + int)
            402 LOAD_CONST               2 ('0x')
            404 LOAD_FAST                1 (magic_key)
            406 LOAD_FAST                2 (i)
            408 BINARY_SUBSCR
            418 BINARY_OP                0 (+)
            422 LOAD_CONST               3 (16)
            424 PRECALL                  2
            428 CALL                     2
            438 LOAD_GLOBAL             11 (NULL + int)
            450 LOAD_CONST               2 ('0x')
            452 LOAD_FAST                1 (magic_key)
            454 LOAD_FAST                2 (i)
            456 LOAD_CONST               1 (1)
            458 BINARY_OP                0 (+)
            462 BINARY_SUBSCR
            472 BINARY_OP                0 (+)
            476 LOAD_CONST               3 (16)
            478 PRECALL                  2
            482 CALL                     2
            492 BINARY_OP               12 (^)
            496 PRECALL                  1
            500 CALL                     1
            510 PRECALL                  1
            514 CALL                     1
            524 LOAD_METHOD              6 (replace)
            546 LOAD_CONST               2 ('0x')
            548 LOAD_CONST               4 ('')
            550 PRECALL                  2
            554 CALL                     2
            564 LOAD_FAST                1 (magic_key)
            566 LOAD_FAST                2 (i)
            568 STORE_SUBSCR
            572 JUMP_BACKWARD          106 (to 362)

 18     >>  574 LOAD_CONST               4 ('')
            576 LOAD_METHOD              7 (join)
            598 LOAD_FAST                1 (magic_key)
            600 PRECALL                  1
            604 CALL                     1
            614 STORE_FAST               1 (magic_key)

 19         616 LOAD_GLOBAL             17 (NULL + print)
            628 LOAD_FAST                1 (magic_key)
            630 PRECALL                  1
            634 CALL                     1
            644 POP_TOP

 20         646 LOAD_GLOBAL              7 (NULL + str)
            658 LOAD_GLOBAL              9 (NULL + hex)
            670 LOAD_GLOBAL             11 (NULL + int)
            682 LOAD_CONST               2 ('0x')
            684 LOAD_FAST                1 (magic_key)
            686 BINARY_OP                0 (+)
            690 LOAD_CONST               3 (16)
            692 PRECALL                  2
            696 CALL                     2
            706 LOAD_GLOBAL             11 (NULL + int)
            718 LOAD_CONST               2 ('0x')
            720 LOAD_FAST                0 (key)
            722 BINARY_OP                0 (+)
            726 LOAD_CONST               3 (16)
            728 PRECALL                  2
            732 CALL                     2
            742 BINARY_OP               12 (^)
            746 PRECALL                  1
            750 CALL                     1
            760 PRECALL                  1
            764 CALL                     1
            774 LOAD_METHOD              6 (replace)
            796 LOAD_CONST               2 ('0x')
            798 LOAD_CONST               4 ('')
            800 PRECALL                  2
            804 CALL                     2
            814 STORE_FAST               3 (wdb_key)

 21         816 LOAD_GLOBAL             17 (NULL + print)
            828 LOAD_FAST                3 (wdb_key)
            830 PRECALL                  1
            834 CALL                     1
            844 POP_TOP

 22         846 LOAD_FAST                3 (wdb_key)
            848 RETURN_VALUE

magic_key:7a107ecf29325423
encrypted_data:f2c85bd042247896b43345e589e3ad025fba1770e4ac0d274c1f7c2a670830379195aa5547d78bcee7ae649bc3b914da

```

‍


我们从`key_encode`​函数源码第11行（字节码第一列是源码行号）开始看：



```
Disassembly of :
 10           0 RESUME                   0

 11           2 LOAD_GLOBAL              1 (NULL + list)
             14 LOAD_FAST                0 (key)
             16 PRECALL                  1
             20 CALL                     1
             30 STORE_FAST               1 (magic_key)

```

第十行是定义`key_encode`​函数，`LOAD_GLOBAL`​ 加载内置list函数，`LOAD_FAST`​ 加载key参数，`PRECALL`​准备调用list函数，参数数量为一，`CALL`​ 执行函数调用，


​`STORE_FAST`​ 将结果存储在局部变量magic\_key中。所以源码就是:



```
magic_key = list(key)

```

然后我们看源码第12行：



```
12          32 LOAD_GLOBAL              3 (NULL + range)
             44 LOAD_CONST               1 (1)
             46 LOAD_GLOBAL              5 (NULL + len)
             58 LOAD_FAST                1 (magic_key)
             60 PRECALL                  1
             64 CALL                     1
             74 PRECALL                  2
             78 CALL                     2
             88 GET_ITER
        >>   90 FOR_ITER               105 (to 302)
             92 STORE_FAST               2 (i)

```

前四行`LOAD_GLOBAL`​、`LOAD_CONST`​、`LOAD_FAST`​分别将range、1、len、magic\_key压入栈中，即加载，第一组`PRECALL`​和`CALL`​是调用len计算magic\_key的长度，第二组`PRECALL`​和`CALL`​是调用range，`GET_ITER、FOR_ITER`​开始循环，直到地址302结束，`STORE_FAST`​将栈顶弹出存入变量 i 中。所以源码为：



```
magic_key = list(key)
for i in range(1,len(magic_key)):

```

然后我们看源码第13行：



```
 13          94 LOAD_GLOBAL              7 (NULL + str)
            106 LOAD_GLOBAL              9 (NULL + hex)
            118 LOAD_GLOBAL             11 (NULL + int)
            130 LOAD_CONST               2 ('0x')
            132 LOAD_FAST                1 (magic_key)
            134 LOAD_FAST                2 (i)
            136 BINARY_SUBSCR
            146 BINARY_OP                0 (+)
            150 LOAD_CONST               3 (16)
            152 PRECALL                  2
            156 CALL                     2
            166 LOAD_GLOBAL             11 (NULL + int)
            178 LOAD_CONST               2 ('0x')
            180 LOAD_FAST                1 (magic_key)
            182 LOAD_FAST                2 (i)
            184 LOAD_CONST               1 (1)
            186 BINARY_OP               10 (-)
            190 BINARY_SUBSCR
            200 BINARY_OP                0 (+)
            204 LOAD_CONST               3 (16)
            206 PRECALL                  2
            210 CALL                     2
            220 BINARY_OP               12 (^)
            224 PRECALL                  1
            228 CALL                     1
            238 PRECALL                  1
            242 CALL                     1
            252 LOAD_METHOD              6 (replace)
            274 LOAD_CONST               2 ('0x')
            276 LOAD_CONST               4 ('')
            278 PRECALL                  2
            282 CALL                     2
            292 LOAD_FAST                1 (magic_key)
            294 LOAD_FAST                2 (i)
            296 STORE_SUBSCR
            300 JUMP_BACKWARD          106 (to 90)

```

​`LOAD_GLOBAL`​、`LOAD_CONST`​、`LOAD_FAST`​ 分别将全局常量str、hex、int压栈，常量'0x'压栈，变量magic\_key和 i 压栈，`BINARY_SUBSCR`​索引动作，即magic\_key\[i]，`BINARY_OP`​执行\+操作，`LOAD_CONST`​将常量16压栈，`PRECALL`​和`CALL`​执行函数调用，在这里我们先暂停一下，强调一下：**因为是不断的面向栈操作，我们还原源码时一定要和进栈的顺序对应上**，所以我们此时可以还原`int('0x'+magic_key[i],16)`​，继续往后看，将int、'0x'、magic\_key、i、1压栈，`BINARY_OP`​执行 \- 操作，`BINARY_SUBSCR`​索引，即magic\_key\[i\-1]，`LOAD_CONST`​将16压栈，`PRECALL`​和`CALL`​执行函数调用，此时可以还原 `int('0x'+magic_key[i-1],16)`​，然后`BINARY_OP`​执行 ^ 操作，两组`PRECALL`​和`CALL`​执行函数调用，此时还原到 `str(hex(int('0x'+magic_key[i],16) ^ int('0x'+magic_key[i-1],16)))`​，`LOAD_METHOD`​将方法replace压栈，后面将'0x'和''压栈，然后调用函数，即replace('0x','')，然后将magic\_key、i压栈，进行索引存储，所以源码为：



```
magic_key = list(key)
for i in range(1,len(magic_key)):
	magic_key[i] = str(hex(int('0x'+magic_key[i],16) ^ int('0x'+magic_key[i-1],16))).replace('0x','')

```

后面的同理，不再赘叙，第15行、16行 ：



```
magic_key = list(key)
for i in range(1,len(magic_key)):
	magic_key[i] = str(hex(int('0x'+magic_key[i],16) ^ int('0x'+magic_key[i-1],16))).replace('0x','')

for i in range(0,len(key),2):
	magic_key[i] = str(hex(int('0x'+magic_key[i],16) ^ int('0x'+magic_key[i+1],16))).replace('0x','')

```

第18行：



```
magic_key = list(key)
for i in range(1,len(magic_key)):
	magic_key[i] = str(hex(int('0x'+magic_key[i],16) ^ int('0x'+magic_key[i-1],16))).replace('0x','')

for i in range(0,len(key),2):
	magic_key[i] = str(hex(int('0x'+magic_key[i],16) ^ int('0x'+magic_key[i+1],16))).replace('0x','')

magic_key = ''.join(magic_key)

```

第19行：



```
magic_key = list(key)
for i in range(1,len(magic_key)):
	magic_key[i] = str(hex(int('0x'+magic_key[i],16) ^ int('0x'+magic_key[i-1],16))).replace('0x','')

for i in range(0,len(key),2):
	magic_key[i] = str(hex(int('0x'+magic_key[i],16) ^ int('0x'+magic_key[i+1],16))).replace('0x','')

magic_key = ''.join(magic_key)
print(magic_key)

```

第20行：



```
magic_key = list(key)
for i in range(1,len(magic_key)):
	magic_key[i] = str(hex(int('0x'+magic_key[i],16) ^ int('0x'+magic_key[i-1],16))).replace('0x','')

for i in range(0,len(key),2):
	magic_key[i] = str(hex(int('0x'+magic_key[i],16) ^ int('0x'+magic_key[i+1],16))).replace('0x','')

magic_key = ''.join(magic_key)
print(magic_key)
wdb_key = str(hex(int('0x'+magic_key) ^ int('0x'+key,16))).replace('0x','')

```

第21行、22行，此时`key_encode`​函数结束：



```
def key_encode(key):
	magic_key = list(key)
	for i in range(1,len(magic_key)):
		magic_key[i] = str(hex(int('0x'+magic_key[i],16) ^ int('0x'+magic_key[i-1],16))).replace('0x','')

	for i in range(0,len(key),2):
		magic_key[i] = str(hex(int('0x'+magic_key[i],16) ^ int('0x'+magic_key[i+1],16))).replace('0x','')

	magic_key = ''.join(magic_key)
	print(magic_key)
	wdb_key = str(hex(int('0x'+magic_key) ^ int('0x'+key,16))).replace('0x','')
	print(wdb_key)
	return wdb_key

```

然后我们看31行：



```
key = key_encode(key)

```

第32行：



```
key = key_encode(key)
if len(key) == 16:

```

第33行：



```
key = key_encode(key)
if len(key) == 16:
	encrypted_data = hex(sm4_encode(key,flag))

```

第34行：



```
def key_encode(key):
	magic_key = list(key)
	for i in range(1,len(magic_key)):
		magic_key[i] = str(hex(int('0x'+magic_key[i],16) ^ int('0x'+magic_key[i-1],16))).replace('0x','')

	for i in range(0,len(key),2):
		magic_key[i] = str(hex(int('0x'+magic_key[i],16) ^ int('0x'+magic_key[i+1],16))).replace('0x','')

	magic_key = ''.join(magic_key)
	print(magic_key)
	wdb_key = str(hex(int('0x'+magic_key) ^ int('0x'+key,16))).replace('0x','')
	print(wdb_key)
	return wdb_key

key = key_encode(key)
if len(key) == 16:
	encrypted_data = hex(sm4_encode(key,flag))
	print(encrypted_data)

```

至此我们的源码就搓出来了，题目还给了如下信息：



```
magic_key:7a107ecf29325423
encrypted_data:f2c85bd042247896b43345e589e3ad025fba1770e4ac0d274c1f7c2a670830379195aa5547d78bcee7ae649bc3b914da

```

分析可知，我们已知`magic_key`​和`encrypted_data`​，`encrypted_data`​是由`flag`​经过sm4加密得到的，密钥为`key`​，所以我们需要知道`key`​,就可以sm4解密得到`flag`​，所以我们需要对 key\_encode 函数的逻辑进行逆向得到`key`​，最后exp如下：



```
def key_encode(key):
	magic_key = list(key)
	for i in range(1,len(magic_key)):
		magic_key[i] = str(hex(int('0x'+magic_key[i],16) ^ int('0x'+magic_key[i-1],16))).replace('0x','')

	for i in range(0,len(key),2):
		magic_key[i] = str(hex(int('0x'+magic_key[i],16) ^ int('0x'+magic_key[i+1],16))).replace('0x','')

	magic_key = ''.join(magic_key)
	# print(magic_key)
	wdb_key = str(hex(int('0x'+magic_key,16) ^ int('0x'+key,16))).replace('0x','')
	# print(wdb_key)
	return wdb_key

magic_key = list("7a107ecf29325423")

for i in range(0,16,2):
	magic_key[i] = str(hex(int('0x'+magic_key[i],16) ^ int('0x'+magic_key[i+1],16))).replace('0x','')

for i in range(len(magic_key)-1,0,-1):
	magic_key[i] = str(hex(int('0x'+magic_key[i],16) ^ int('0x'+magic_key[i-1],16))).replace('0x','')

key = "".join(magic_key)
print(key_encode(key))

# 输出：ada1e9136bb16171

```

然后去赛博厨子解密即可拿到flag：wdflag{815ad4647b0b181b994eb4b731efa8a0}


​![image](https://img2023.cnblogs.com/blog/3400631/202411/3400631-20241102004731528-1443777706.png)​


‍


‍


**参考链接：**



> [PyCon 2018：James Bennett\-\-理解 Python 字节码 掘金翻译计划](https://github.com)
> 
> 
> [码农高天：字节码和虚拟机？python代码竟然是这么执行的！](https://github.com)
> 
> 
> [王战山的学习笔记：Python中的字节码](https://github.com)
> 
> 
> [python官方文档：dis — Disassembler for Python bytecode](https://github.com)


‍


  * [python bytecode解析](#python-bytecode%E8%A7%A3%E6%9E%90)
* [前言](#%E5%89%8D%E8%A8%80)
* [工作原理](#%E5%B7%A5%E4%BD%9C%E5%8E%9F%E7%90%86)
* [面向栈](#%E9%9D%A2%E5%90%91%E6%A0%88):[樱花宇宙官网](https://yzygzn.com)
* [字节码的阅读](#%E5%AD%97%E8%8A%82%E7%A0%81%E7%9A%84%E9%98%85%E8%AF%BB)
* [dis模块](#dis%E6%A8%A1%E5%9D%97)
* [案例](#%E6%A1%88%E4%BE%8B)

   ![](https://github.com/avatar/3400631/20240622171012.png)    - **本文作者：** [MiaCTFer](https://github.com)
 - **本文链接：** [https://github.com/MiaCTFer/p/18521526/python\-bytecode\-analysis\-fuese](https://github.com)
 - **关于博主：** 评论和私信会在第一时间回复。或者[直接私信](https://github.com)我。
 - **版权声明：** 本博客所有文章除特别声明外，均采用 [BY\-NC\-SA](https://github.com "BY-NC-SA") 许可协议。转载请注明出处！
 - **声援博主：** 如果您觉得文章对您有帮助，可以点击文章右下角**【[推荐](javascript:void(0);)】**一下。
     
