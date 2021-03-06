[[语法分析]]

------------------------------------------------------------------------------
call函数妙用，简化打印信息：
@file:Kbuild.include 定义call的表达式，和相应的参数：
```
    echo-cmd = $(if $($(quiet)cmd_$(1)),echo '  $($(quiet)cmd_$(1))';)
	cmd = @$(echo-cmd) $(cmd_$(1))
```
									}}}
@Makefile 定义命令信息变量和命令变量
{{{
include Kbuild.include
	quiet_cmd_sse ?= COMPILE        $@
	      cmd_sse ?= $(MAKE) -C $(SSE_SRC)
目标：
	sse:
		$(call cmd,sse)
									}}}
最终cmd就是
{{{
	$cmd = @echo 'COMPILE 	sse'; make -C /home/sqm/tmp/JusonFlow/sse 
									}}}
call函数回执行这两条命令，并且括号形成一个变量，echo $(call cmd,sse)会把cmd 信
息打印出来，但是也会继续被call函数执行。

include-flag: 把libexec中所有的子目录都加入了include变量中了，目标在被生成之前
先把变量补充（替换成）最后一次的赋值，不管目标是在此变量之前还是之后。
```
	LIB_SUBDIR := $(shell ls $(LIBEXEC_DIR) -FR | grep ":" | sed 's/://g')
	CFLAGS_SUBDIR := $(addprefix -I,$(LIB_SUBDIR))
	CFLAGS_GLOBAL += $(CFLAGS_SUBDIR)
	export CFLAGS_GLOBAL CFLAGS_LOCAL CFLAGS_ZXEXEC CC LD
```
# se编译过程
make -C sse; include zxmx.mk; make -f libexec/Makefile.build

make menuconfig 生成两个配置文件
	autoconf.h	c文件包含
	auto.conf 	给makefile包含，子目录也经过它进行配置。

sse<---cmd_sse
	mipsisa64-octeon-elf-gcc obj/zxmd_init.o obj/zxmd_main.o \
	obj/zxmd_mproc.o obj/libcvm-common.a obj/libzxexe.a \
	obj/libcvm-pci-drv.a obj/libcvmhfa.a obj/libocteon-hfa.a \
	/home/sqm/tmp/JusonFlow/sdk/OCTEON-SDK/components/hfa/lib-octeon \
	/pp/octeon/se/libpp.a obj/libcvmx.a obj/libfdt.a -mfix-cn63xxp1 \
	-march=octeon2 -o cn66hw1.bin
```
	$(TARGET): $(CVMX_CONFIG) $(OBJS) $(LIBS_LIST)
```
OBJS:
	obj/zxmd_init.o obj/zxmd_main.o obj/zxmd_mproc.o

CVMX_CONFIG:
	config/cvmx-config.h

LIBS_LIST:

    obj/libcvm-common.a obj/libzxexe.a obj/libcvm-pci-drv.a obj/libcvmhfa.a obj/libocteon-hfa.a /home/sqm/tmp/JusonFlow/sdk/OCTEON-SDK/components/hfa/lib-octeon/pp/octeon/se/libpp.a obj/libcvmx.a obj/libfdt.a

libexec: 

    在libexec/zxmx.mk中， libexec--->libzxexe.a注意库的排列顺序

libexec目录:
```
	obj/zxmx_*.o: zxmx_*.c
```
libexec子目录:

sse中make -f libexec/Makefile.build obj=...

然后Makefile.build自己也make -f Makefile.build obj=...

	eprm	csps	acl
	都编译成build-in.o,就是
	$(LIBEXEC_DIR)/eprm/build-in.o
	$(LIBEXEC_DIR)/csps/build-in.o
	$(LIBEXEC_DIR)/acl/build-in.o
```
	MAKE_FUNC = libexec/Makefile.build
	%/build-in.o :  FORCE
        	@$(MAKE) -f $(MAKE_FUNC) obj=$(dir $@)
```
```
	all:$(obj)build-in.o	#obj等于make -f 传进来的obj

	$(obj)build-in.o : $(local-obj) $(sub-obj)#当前目录的.o,子目录的.o
		$(call cmd,ld)

	wildcard子目录的Kbuild，然后-include子目录的Kbuild，相当于obj-dir+=

	quiet_cmd_built ?= COMPILE      $@                                       
		cmd_built ?= $(MAKE) -f $(LIBEXEC_DIR)/Makefile.build obj=$(dir $@)
	$(obj)%/build-in.o : FORCE	#在递归调用，编译子目录
		$(call cmd,built)

	%.o : %.c
		$(call cmd,base)
```
------------------------------------------------------------------------------
目录作为目标名，如果想利用目录名作为目标名，然后进行MAKE -C 这时候目录名的时间
应该进行思考了，如果没有依赖那么，一直不会make -c 因为目标存在，而且没有依赖。
如果有依赖，但是make -c 没有影响到目录的时间戳，而且依赖比目录的时间戳旧那么一
直不会执行，如果新那么一直会执行。
------------------------------------------------------------------------------
如果依赖是个软连接文件，那么不管连接文件还是原文件被touch更新了， 都会重新执行

但是ln -sf选项，重新建立的连接文件就不会引起重新执行。

rm -f 依赖的连接，ln -s 新的连接，也不会重新执行。
------------------------------------------------------------------------------
call函数妙用，简化打印信息：
@file:Kbuild.include 定义call的表达式，和相应的参数：
{{{
	echo-cmd = $(if $($(quiet)cmd_$(1)),echo '  $($(quiet)cmd_$(1))';)
	cmd = @$(echo-cmd) $(cmd_$(1))
									}}}
@Makefile 定义命令信息变量和命令变量
{{{
include Kbuild.include
	quiet_cmd_sse ?= COMPILE        $@
	      cmd_sse ?= $(MAKE) -C $(SSE_SRC)
目标：
	sse:
		$(call cmd,sse)
									}}}
最终cmd就是
{{{
	$cmd = @echo 'COMPILE 	sse'; make -C /home/sqm/tmp/JusonFlow/sse 
									}}}
call函数回执行这两条命令，并且括号形成一个变量，echo $(call cmd,sse)会把cmd 信
息打印出来，但是也会继续被call函数执行。

include-flag: 把libexec中所有的子目录都加入了include变量中了，目标在被生成之前
先把变量补充（替换成）最后一次的赋值，不管目标是在此变量之前还是之后。
{{{
	LIB_SUBDIR := $(shell ls $(LIBEXEC_DIR) -FR | grep ":" | sed 's/://g')
	CFLAGS_SUBDIR := $(addprefix -I,$(LIB_SUBDIR))
	CFLAGS_GLOBAL += $(CFLAGS_SUBDIR)
	export CFLAGS_GLOBAL CFLAGS_LOCAL CFLAGS_ZXEXEC CC LD
									}}}

make menuconfig 生成两个配置文件
	autoconf.h	c文件包含
	auto.conf 	给makefile包含

sse<---cmd_sse
	mipsisa64-octeon-elf-gcc obj/zxmd_init.o obj/zxmd_main.o \
	obj/zxmd_mproc.o obj/libcvm-common.a obj/libzxexe.a \
	obj/libcvm-pci-drv.a obj/libcvmhfa.a obj/libocteon-hfa.a \
	/home/sqm/tmp/JusonFlow/sdk/OCTEON-SDK/components/hfa/lib-octeon \
	/pp/octeon/se/libpp.a obj/libcvmx.a obj/libfdt.a -mfix-cn63xxp1 \
	-march=octeon2 -o test
{{{
	$(TARGET): $(CVMX_CONFIG) $(OBJS) $(LIBS_LIST)
									}}}
OBJS:
	obj/zxmd_init.o obj/zxmd_main.o obj/zxmd_mproc.o

CVMX_CONFIG:
	config/cvmx-config.h

LIBS_LIST:
	obj/libcvm-common.a obj/libzxexe.a obj/libcvm-pci-drv.a obj/libcvmhfa.a 
	obj/libocteon-hfa.a 
	/home/sqm/tmp/JusonFlow/sdk/OCTEON-SDK/components/hfa/lib-octeon/pp\
	/octeon/se/libpp.a obj/libcvmx.a obj/libfdt.a

libexec: 在libexec/zxmx.mk中， libexec--->libzxexe.a注意库的排列顺序

libexec目录:
{{{
	obj/zxmx_*.o: zxmx_*.c
									}}}
libexec子目录:
	eprm	csps	acl
	都编译成build-in.o,就是
	$(LIBEXEC_DIR)/eprm/build-in.o
	$(LIBEXEC_DIR)/csps/build-in.o
	$(LIBEXEC_DIR)/acl/build-in.o
{{{
	MAKE_FUNC = libexec/Makefile.build
	%/build-in.o :  FORCE
        	@$(MAKE) -f $(MAKE_FUNC) obj=$(dir $@)
									}}}
{{{
	all:$(obj)build-in.o	#obj等于make -f 传进来的obj

	$(obj)build-in.o : $(local-obj) $(sub-obj)#当前目录的.o,子目录的.o
		$(call cmd,ld)

	wildcard子目录的Kbuild，然后-include子目录的Kbuild，相当于obj-dir+=
	quiet_cmd_built ?= COMPILE      $@                                       
		cmd_built ?= $(MAKE) -f $(LIBEXEC_DIR)/Makefile.build obj=$(dir $@)
	$(obj)%/build-in.o : FORCE	#在递归调用，编译子目录
		$(call cmd,built)

	%.o : %.c
		$(call cmd,base)
									}}}
------------------------------------------------------------------------------
目录作为目标名，如果想利用目录名作为目标名，然后进行MAKE -C 这时候目录名的时间
应该进行思考了，如果没有依赖那么，一直不会make -c 因为目标存在，而且没有依赖。
如果有依赖，但是make -c 没有影响到目录的时间戳，而且依赖比目录的时间戳旧那么一
直不会执行，如果新那么一直会执行。
------------------------------------------------------------------------------
$@变量使用的重要性：
{{{
	LINUX_OBJS_TMP=$(MDUAPI_LINUX:.c=.o)
	$(LINUX_OBJS_TMP):%.o:%.c FORCE
		$(CC) $(CFLAGS_PCI) $(CFLAGS) -c -o $@ $<
		mv $@ $(OBJS_DIR)/
上面是把一个变量的.c后缀变成.o后缀,然后目标是所有的.o，依赖是对应的.c然后$@ $<
的使用就把下面的执行语句，按照变量中的所有成员按序展开执行。

第一次失误,
	mv $@ $(OBJS_DIR)/
这一句写成了
	mv $(LINUX_OBJS_TMP) $(OBJS_DIR)/

这样的话，就是把$(LINUX_OBJS_TMP)中的成员一次性在这条mv 语句中展开去执行了，然
后就报错了。因为相应的.o文件还没有全部生成。
									}}}
------------------------------------------------------------------------------
变量 在语句中，作为目标，作为依赖

变量在语句中的值是是在整个makefile文件最后的被赋值确定的值，语句出现在前，也是
被最终的赋值。
{{{
	var = string
	all:
		@echo $(var)
	var = data
									}}}
	$ make
	$ data
如果var=* ，那么，将显示目录下所有的文件和目录，因为$(var）被makefile解释成“*”
送给echo，echo在shell中执行就是：echo *，从而会显示所有的文件。如果想显示“*”号
可以写成：
{{{	var = string-tmp
	all:
		@echo "$(var)"
	var = *
									}}}

作为目标和依赖时候，就会跟定义的位置和使用位置有关系了。
{{{
	target = main.c
	all:$(target)		#这里all的依赖是main.c

	target = dddd.o
	$(target):dddd.c	#这里目标就又成了dddd.c了
									}}}
------------------------------------------------------------------------------
变量：
  变量名 = main.o command.o
  $(变量名)	#引用

1.   =	递归赋值，右边的变量不一定要先赋值
2.  :=	递归展开赋值，但是右边的变量一定要先赋值
{{{
	eq = $a
	a = $b
	b = c

	f = g
	e = $f
	dot := $e

	all:
		@echo $(eq)
	dd:
		@echo $(dot)
									}}}
3.  ?=	如果左边变量已经定义，则什么都不做
{{{	
	FOO = bar
	等价于：
	ifeq ($(origin FOO), undefined)
		FOO = bar
	endif
									}}}
4.  += 	给变量追加值



自动推导：
  main.o:def.h <--> main.o:def.h main.c
			cc -c main.c

.PHONY:clean 
clean:
	-rm edit $(objects)
.PHONY显式的指明一个目标是"伪目标",避免和文件重名的情况。而且不依赖任何文件，总是被执行。
如果把伪目标放在第一目标作为默认目标，然后在它后面写上依赖文件，那么根据伪目标的特性，总是被执行，所以其依赖的那三个目标总是被决议。
make推导第一个遇到的目标(默认目标)。所以clean从来都是放在文件的最后。
表示clean是一个伪目标。"-"表示某些文件出现问题，不管，继续做后面的事。

Makefile主要包含了5样东西:显式规则、隐晦规则、变量定义、文件指示和注释。
1.显示规则：生成文件，依赖文件，生成命令
2.隐晦规则：自动推导
3.变量定义：像C的宏
4.文件指示：1)引用另外一个makefile，像C的include 2)指定有效部分，像C中的预编译
	    #if 3)定义个一个多行的命令
5.注释：Makefile中只有行注释，"#"。 反斜杠转义，"\#".

Makefile中的命令，必须要以[Tab]键开始。

include前面可以有一些空字符，但是绝不能是TAB键开始
include和<filename>可以用一个或多个空格隔开。
1.make执行时，有"-I"或"--include-dir"参数，make就会在这个参数所指定的目录去寻找。
2.如果目录<prefix>/include(一般是：/usr/local/bin或/usr/include)存在的话，make也会找。
-include <filename> <==> sinclude <filename>

小心环境变量中的MAKEFILES

targets:prerequistites;command
	command
targets是文件名，以空格分开，可以使用通配符。command是命令行，如果不与"targets:prerequisites"在一行，那么必须以TAB键开头，如果和prerequisites在一行，那么可以用分号做分隔。

objects = *.o
这里*.o不会展开。
objects := $(wildcard *.o)
这里就展开了，wildcard是关键字。

VPATH = src:../headers
如果没有指明这个变量，make只会在当前的目录去寻找依赖文件和目标文件。如果定义了这个变量，那么，make就会在当前目录找不到的情况下，到所指定的目录中去找寻文件。
上面的定义指定了两个目录，"src"和"../headers“,当前目录永远是最高优先搜索的地方。

vpath关键字：
1.vpath <pattern> <directories> 为符合模式<pattern>的文件指定搜索目录
2.vpath <pattern> 清除符合模式<pattern>的文件的搜索目录
3.vpath	清除所有已被设置好了的文件的搜索目录。
<pattern>需要包含"%"字符。"%"的意思是匹配零或若干字符，"%.h"表示所有以".h"结尾的文件。

bigoutput littleoutput:text.g
	generate text.g -$(subst output,,$@)>$@
||
bigoutput:text.g
	generate text.g -big >bigoutput
littleoutput:text.g
	generate text.g -little >littleouput
-$(subst output,,$@)中的"$"表示执行一个Makefile函数，函数名位subst，后面的为参数。$@ 表示目标的集合，就像一个数组，"$@"依次取出目标，并执行命令。

objects = foo.o bar.o
all:$(objects)
$(objects):%.o:%.c 	#<targets ...>:<targets-pattern>:<prereq-patterns ...>
	$(CC) -c $(CFLAHGS) $< -o $@	#<commands>
上面，指明我们的目标从$objects中获取，"%.o"表明要所有以".o"结尾的目标。也就是"foo.o,bar.o"，而依赖模式"%.c"则取模式"%.o"的"%",也就是foo bar，并为其加上.c
"$<"表示所有的依赖目标集;
"$@"表示所有的目标集。

files = foo.elc bar.o lose.o
{{{
	$(filter %.o,$(files)):%.o:%.c
		$(CC) -c $(CFLAGS) $< -o $@
}}}

include $(source:.c=.d)
把变量所有[.c]的字串都替换成[.d]

@命令
就不会把执行的命令显示出来了

make -n 或 --just-print只是显示命令，但不会执行命令，用于调试Makefile
make -s 或 --slien则是全面禁止命令的显示
make -i 或 --ignore-errors 所有命令都会忽略错误
make -k 或 --keep-going如果某规则的命令出错了，那么就终止该规则的执行，但继续执行其它的规则。

gcc -M[M] *.c生成 .d的依赖

如果让上一条命令的结果应用在下一条命令时，应该使用分号分隔这两条命令。而不能把这两条命令写在两行上。
subsystem:
	cd subdir && $(MAKE)
||
subsystem:
	$(MAKE) -C subdir 先进入subdir，然后执行make命令
这样不会覆盖下层的Makefile中定义的变量，除非指定了"-e"参数
(-e参数，系统环境变量将覆盖Makefile中定义的变量)
如果要传递变量到下级Makefile，那么可以使用这样的声明:
export <variable ...>
不想
unexport <variable ...>

要使用真实的"$"字符，需要用"$$"来表示

:=   前面的变量不能使用后面的变量，只能使用前面已定义好了的变量。
用变量存储一个空格：
nullstring :=
space := $(nullstring) #end of the line

dir :=/foo/bar   #directory to put the frobs in
$(dir)/file  <==>/foo/bar   /file出错了


FOO ?= bar
||
ifeq ($(origin FOO), undefined)
	FOO = bar
endif

$(var:a=b) 
${var:a=b} 把var变量中所有以"a"字串结尾的"a"替换成"b"字串。

x = variable1
variable2:=Hello
y = $(subst 1,2,$(x))
#subst把ariable1中的所有1替换成2
z = y
a := $($($(z)))

#命令包
define run-yacc
yacc $(firstword $^)
mv y.tab.c $@
endef
	$(run-yacc)#TAB激活命令包
#多行变量、变量值可以有换行
define two-lines
echo foo
echo $(bar)
endef
$(two-lines)#和命令包定义一样但是不要加TAB使用

override指示符:
如果有变量是make的命令行参数设置的，那么Makefile中对这个变量的赋值会被忽略。如果想在Makefile中设置这类参数的值可以使用override指示符。
override <variable> = <value>
override <variable> := <value>
override <variable> += <value>
对于多行的变量定义，在define指示符前，也同样可以使用override指示符.

目标变量:
<target...>:<variable-assignment>
<target...>:override <variable-assignment>
prog:CFLAGS = -g
prog:prog.o foo.o bar.o
	$(CC) $(CFLAGS) prog.o foo.o bar.o
prog.o:prog.c
	$(CC) $(CFLAGS) prog.c
... 由这个目标引发的所有的规则都会引用这个变量，名字可以和全局的重复，因为作用域不同。
模式变量:
%.o:CFLAGS = -O

条件判断:
ifeq ($(1),$(2))
	...
else
	...
endif
ifeq (<arg1>, <arg2>)
ifeq '<arg1>' '<arg2>'
ifeq "<arg1>" "<arg2>"
ifeq "<arg1>" '<arg2>'
ifeq '<arg1>' "<arg2>"

ifeq ($(strip $(foo)),)
	...
endif#如果strip函数的返回值是空，那么就是生效
ifneq,ifdef else endif,ifndef

函数调用:
$(<function> <arguments>)
bar:=$(subst arg1,arg2,arg2))#第一个参数是被替换字串，第二个参数是替换字串，第三个参数替换作用的字串。
1.$(subst <from>,<to>,<text>)#把字串<text>中的<form>字符串替换成<to>
2.$(patsubst <pattern>,<replacement>,<text>) -- $(var:<pattern>=<replacement>)匹配替换
||$(patsubst %.c,%.o,x.c.c bar.c) --> x.c.o bar.o 
  $(patsubst %<suffix>,%<replacement>,<text>) -- $(var:<suffix>=<replacement>)
3.$(strip <string>)#去空格，去掉字串中开头和结尾的空字符
4.$(findstring <find>,<in>)#查找字符串函数，找到返回find，否则返回空字符串
5.$(filter <pattern...>,<text>)#过滤返回符合匹配模式的字串
6.$(filter-out <pattern...>,<text>)#反过滤，以<pattern>模式过滤<text>字串，去除符合的单词
7.$(sort <list>) #排序函数,回去掉相同的单词

if函数
$(if <condition>,<then-part>,<else-part>)

wildcard:扩展通配符
$(wildcard *.o /src/*.o)#就是把通匹配符匹配的展开
% *通配符的不同。

## 变量传递的方法
* include一个文件
* export变量名


## 清除连接文件
> 后来使用objs/%.o:source-dir/%.c，避免使用连接
clear:
	$(Q)find . -type l -exec rm -rf {} \;
	
## 变量作为依赖或目标，间接作为依赖或目标
