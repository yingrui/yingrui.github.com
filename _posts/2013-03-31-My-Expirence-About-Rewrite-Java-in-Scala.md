---
layout: post
category : programming
tags : [Scala]
---

#一次用Scala重写Java的经验
##目标
目前因为要将英文词性识别整合到中文分词中，同时希望能让开发更有效率一些，最好能支持Windows开发，于是决定使用Scala重写Java代码。

###之前的经验教训
这是我第二次重写Java代码，上一次是重写一段拼音识别的代码！当时因为代码量比较少，于是就直接对着原来的Java代码，硬着头皮写了。但是效率很差，这次我算是学乖了些，使用了一些技巧，在这里分享一下，希望能给大家一些参考？

##方法
###正则表达式
因为Java的语法比较简单，而且有一种说法，就是你能以写Java的方式来写Scala。所以大多数Java语法Scala都是支持的，我想到的一个点子就是通过正则替换的方式来实现。

###增加测试
在开始之前，我又为原来的项目增加了几个测试，这样可以稍微增加一点我的信心。主要是因为Scala的基本控制结构不支持continue和break，这样在重写的过程中很可能会产生逻辑错误，其他的地方倒还好。事后证明，确实在控制结构相关的重写中，容易出错。

###在开始正则替换之前
首先我实现了一个脚本，可以将我原来的Java代码包括目录结构拷贝至另外一个目录，因为是一个Maven项目，我在新的项目目录下重写了POM文件，增加Scala相关的Dependencies和Plugins。

接下来使用一个脚本将所有的Java文件更名为Scala文件，这样我就准备好了。主要是通过find和xargs两个命令完成的以上步骤。

	每次我都运行以下命令：
	clean.sh; copy.sh; translate.sh
这几个文件就存在：<https://github.com/yingrui/max-prob-segment/tree/master/lib-segment-scala>

我编写的脚本每次会重新拷贝Java代码，所以我可以随意修改脚本然后重新运行。但我后来想，如果我一开始就将拷贝过来的代码加入到代码库，每次想到新的命令的时候，就不再重新删除再修改。这样可以节省不少程序运行时间的，但是每次都删除的话也有其灵活的地方，例如有两个替换命令有执行顺序的依赖关系的时候，就还是需要重新开始。

##自动代码替换
现在我能做的是，将一些Java语法替换成Scala语法。当然我没指望所有的Java程序都可以替换成Scala代码，虽然这并不是不可能的。

我使用如下Perl命令帮助我将原文件中的字符串直接进行替换，来达到我的目的。后来我发现直接写程序更好，因为简单通过正则表达式替换，有时候能够做的还是有限。
	
	find src -name "*.scala" -exec perl -p -i -e "s/\b([A-z0-9]+) ([a-z][A-z0-9<>,]+)\[\](\)| =|;)/\1\[\] \2\3/g" {} \;

###1、规范化代码
我所做的是规范化原来的Java代码，因为我以前使用的IDE和现在的不一样，我担心代码格式会有差别。
	
	1 替换 type variable[] 成 type[] variable
	2 删除变量的final关键字

###2、基本类型
接下来将基本类型的声明变更成Scala的，包括Int、Boolean等。
	
	3 基本类型替换，例如：find src -name "*.scala" -exec perl -p -i -e "s/\b(int|Integer)\b/Int/g" {} \;

###3、数组声明
因为在Scala中对数组的声明和访问，与Java相比都有较大的变化，于是我的脚本由以下几个步骤构成。

	4 将所有type[]变成Array[type]
	5 将所有new type[num]替换为new Array[type](num)
	6 将所有array[i][j]替换为array(i)(j)
	7 将所有array[i]替换为array(i)
这里就包含了一个顺序依赖，必须先运行第6步，才能运行第7步

###4、Override方法
首先我处理override关键字。在Scala中必须声明override，在Java中则不是强制的。
	
	8 将所有'@Override\n'替换成'override'
###5、接口
在Scala中interface关键字不再出现，trait可以替代，但trait和interface还是有很大不同的。但从Java到Scala，我直接替换就好了。

	9 将interface关键字替换为trait
	10 将implements关键字替换为extends
	11 将method throws Exception;中的throws声明删除
	12 将public class声明替换为class
###6、方法声明
方法声明是任何程序的基本部分，在替换的时候，有很多步和方法声明有关
	
	13 将public void method替换成def method
	14 将public type method替换成def method(): type =
	15 将接口声明中的方法声明替换成public type method();替换成def method():type
	16 对私有方法重复13 14的步骤，替换成private def method(): type=
	17 对不起观众的是，因为随意，在这里我顺便将private type variable替换成private var variable: type了

###7、Java Generics
因为我的代码中没有出现&lt;A,B,C&gt;的情况，所以我直接写了两个命令分别处理&lt;K&gt;和&lt;A,B&gt;的情况。

	18 替换泛型&lt;A&gt;为[A]，&lt;A, B&gt;为[A,B]

###8、异常捕获
	19 替换catch(ex: ExceptionType)为catch
这里我简单的将catch(ex: ExceptionType)替换为catch，是个偷懒的决定，当时认为代码中try catch不是很多，但是后来发现还是有浪费时间。

###9、打印输出
	20 将System.out.println替换成println

###10、方法声明中的参数
	21 将def method(type parameter)替换成def method(parameter:type)
我最多只处理一个函数中有4个参数的情况，因为我的代码中没有再多的情况。

###11、变量声明
将所有变量声明改成Scala形式的，但都是var，而不是val，因为Java代码里面大多没有声明成final。
	
	22 替换type variable = value;成var variable = value
事后我发现还需要将type variable = null;替换成var variable:type = null。

###12、for循环
	23 替换for(type variable: list)成for(variable <- list)
因为懒没有将for(int i = 0; i < max; i++)这种形式的循环进行替换，后面有些后悔，后来花了很多精力在这上面，如果有人参考这个方法重写Java，一定记得这个痛啊！！！

###13、将所有代码comment
最后我将所有代码使用//符号comment了。这样就可以从单元测试开始，一个一个的uncomment。所有的代码在重写的过程中，有测试覆盖是很好的体验。

##结果
我花了一个星期的业余时间重写了分词项目，每天大概两个多小时，Java代码一共不到10000行。最痛苦的是没有把for循环给替换好，后来很后悔。

在重写的过程中，不要太多想着使用Scala的特性改写原来的代码，很有可能会出现逻辑错误。尤其是break和continue出现的地方，一定尽可能有测试覆盖。

虽然最后代码可能很难看，但是个人觉得首先要确保代码正确，最后当功能测试都全部重写完成之后，再想着如何重构会比较好。因为如果不是TDD实现的代码，仅靠后来补充的单元测试，是不一定能完全覆盖所有的功能的。

吐槽的一点是，Scala的性能不如我原来的Java实现，尤其是HashMap的性能，差了一倍以上，原来的分词是1秒钟50万词，现在只有30万词了。

参考[Yammer从Scala转向Java][yammer-scala]后，不得不在一些关键地方，放弃scala.collection，减少我的性能损失。最后程序的性能损失减少至20%左右！

[yammer-scala]:http://www.infoq.com/cn/news/2012/02/yammer-scala