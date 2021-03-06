title: "Python 中if \__name\__ == '\__main\__'：用法解析"
date: 2015-02-04 14:38:46
categories:
- Python
tags:
- Python
- main
---

>【转】原创作品，允许转载。转载时请务必以超链接形式标明文章原始出处、作者信息和本声明，否则将追究法律责任。
>[http://keliang.blog.51cto.com/3359430/649318](http://keliang.blog.51cto.com/3359430/649318 "http://keliang.blog.51cto.com/3359430/649318")

想必很多初次接触python的同学都会见到这样一个语句，if \_\_name\_\_ == "\_\_main\_\_"：

那么这个语句到底是做什么用的呢？在解释之前，首先要声明的是，不管你是多么小白，你一定要知道的是：

1. python文件的后缀为.py；
2. py文件既可以用来直接执行，就像一个小程序一样，也可以用来作为模块被导入（比如360安全卫士，就是依靠一个个功能模块来实现的，好比360安全卫士本身框架是一个桌面，而上面的图标就是快捷方式，这些快捷方式所指向的就是这一个个功能模块）
3. 在python中导入模块一般使用的是import

好了，在确定知道以上几点之后，就可以开始解释if \_\_name\_\_ == "\_\_main\_\_"：这个语句了。

首先解释一下if，顾名思义，if就是如果的意思，在句子开始处加上if，就说明，这个句子是一个条件语句。学习if语句的使用是很简单的，当然想要真正灵活运用还需大量的实践。

接着是 \_\_name\_\_，\_\_name\_\_作为模块的内置属性，简单点说呢，就是.py文件的调用方式。

最后是\_\_main\_\_，刚才我也提过，.py文件有两种使用方式：作为模块被调用和直接使用。如果它等于"\_\_main\_\_"就表示是直接执行。

总结：在if \_\_name\_\_ == "\_\_main\_\_"：之后的语句作为模块被调用的时候，语句之后的代码不执行；直接使用的时候，语句之后的代码执行。通常，此语句用于模块测试中使用。
