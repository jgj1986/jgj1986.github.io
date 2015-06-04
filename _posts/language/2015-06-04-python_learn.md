---
title: python 学习笔记
date: 2015-06-04 14:12:28
tags: language
- a
- b
---

* toc 
{:toc}

##python字符串操作
[python字符串操作和string模块代码分析](http://blog.chinaunix.net/uid-25992400-id-3283846.html){:target="_blank"}

    str.zfill(20)       #str右对齐，左边填充0
    
set判断元素是否存在 

    if key in set:
        xxx
        
json解析后变为dict，判断是否有

    if dict.has_key(key)：
        xxxx
        
##python简洁的表达式：

    print [x for x in foo if x % 3 == 0]

python 中lambda用来创建匿名函数，参考[Python中lambda表达式](http://blog.csdn.net/imzoer/article/details/8667176){:target="_blank"}:

    fs = [(lambda n, i=i : i + n) for i in range(10)]   
    

n的阶乘

    n = 5
    reduce(lambd x,y:x*y,range(1, n+1))

lambda用户函数表达式

    def action(x)
    return lambda y:x+y

    a = action(2)
    a(22)
    #24

    b = lambda x:lambda y:x+y
    a = b(2)
    a(22)
    #24

##特殊语法

转载一篇博客 [Python特殊语法--filter、map、reduce、lambda](http://www.cnblogs.com/linjiqin/p/4222160.html){:target="_blank"}

###filter(function, sequence)

对sequence中的item依次执行function(item)，将执行结果为True的item组成一个List/String/Tuple(取决于sequence的类型)返回：

    def f(x): 
    return x % 2 != 0 and x % 3 != 0 

    print filter(f, range(2, 25)) 
    #[5, 7, 11, 13, 17, 19, 23]

    def f1(x): 
    return x != 'a' 

    print filter(f1, "abcdef") 
    #bcdef 

###map(function, sequence) 

对sequence中的item依次执行function(item)，见执行结果组成一个List返回：

    def cube(x): 
    return x*x*x 
    print map(cube, range(1, 11)) 
    #[1, 8, 27, 64, 125, 216, 343, 512, 729, 1000]

    def cube1(x): 
    return x + x 
    print map(cube1 , "abcde") 
    #['aa', 'bb', 'cc', 'dd', 'ee']
 
另外map也支持多个sequence，这就要求function也支持相应数量的参数输入：

    def add(x, y): 
    return x+y 
    print map(add, range(8), range(8)) 
    #[0, 2, 4, 6, 8, 10, 12, 14]

    status_ids = map(lambda x:x.get('status__id'), snaps)

### reduce(function, sequence, starting_value)

对sequence中的item顺序迭代调用function，如果有starting_value，还可以作为初始值调用，例如可以用来对List求和：

    def add(x,y):
    return x + y 
    print reduce(add, range(1, 11)) 
    注：1+2+3+4+5+6+7+8+9+10

    reduce(add, range(1, 11), 20) 
    注：1+2+3+4+5+6+7+8+9+10+20


### lambda

这是Python支持一种有趣的语法，它允许你快速定义单行的最小函数，类似与C语言中的宏，这些叫做lambda的函数，是从LISP借用来的，可以用在任何需要函数的地方： 

    g = lambda x: x * 2 
    print g(3) 
    #6 

    print (lambda x: x * 2)(3) 
    #6

### sorted

    aa = [
        {'name':'zhangsan', 'price':20.01, 'date':'2015-01-09T01:00:00Z'},  
        {'name':'lisi', 'price':10.01, 'date':'2013-01-09T01:00:00Z'},  
        {'name':'wangwu', 'price':0.01, 'date':'2012-01-09T01:00:00Z'}  
    ]  
    sorted(aa, key=lambda s:s.amount) #对list进行排序
    sorted(aa, key=lambda s:s.amount, reverse=True)

    aa = [<Symbol: Symbol object>, <Symbol: Symbol object>, <Symbol: Symbol object>] 
    sorted(aa, key=lambda s:s["date"]) #对Symbol对象进行排序，date为Symbol属性
    sorted(aa, key=lambda s:s["date"], reverse=True)

###for特殊用法
    for i in range(4):
    print i

    se=[x**2 for x in range(4)]
    print se 
    #[0, 1, 4, 9]

    se=[x**2 for x in range(10) if not x%2]
    print se 
    #[0, 4, 16, 36, 64]

###zip
zip函数接受任意多个（包括0个和1个）序列作为参数，返回一个tuple列表。

    x = [1, 2, 3]
    y = [a, b, c]
    z = [A, B, C]
    xyz = zip(x, y, z)
    print xyz   
    #[(1,a,A),(2,b,B),(3,c,C)]
    
## 一个例子

    test=[[['db','table','index'],'key','val'], [['db1','table1','index1'],'key1','val1']]
    tables = [reduce(lambda a,b: a+b,[(y if type(y) == type([]) else [y]) for y in x ]) for x in test ]
    #[['db', 'table', 'index', 'key', 'val'], ['db1', 'table1', 'index1', 'key1', 'val1']]

