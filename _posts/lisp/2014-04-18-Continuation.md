---
layout:	post
title:	DrRacket In Action
description: DrRacket In Action
categories:
- Programming Languages
tags:
- Languages
---

##Continuation

###0x01 Continuation是什么
我们先看[wiki](2)对continuation的描述:

>In computer science and computer programming, a continuation is an abstract representation of the control state of a computer program. A continuation reifies the program control state, i.e. the continuation is a data structure that represents the computational process at a given point in the process' execution; the created data structure can be accessed by the programming language, instead of being hidden in the runtime environment. Continuations are useful for encoding other control mechanisms in programming languages such as exceptions, generators, coroutines, and so on.

>The "current continuation" or "continuation of the computation step" is the continuation that, from the perspective of running code, would be derived from the current point in a program's execution. The term continuations can also be used to refer to first-class continuations, which are constructs that give a programming language the ability to save the execution state at any point and return to that point at a later point in the program.


Contination 最早出现在Scheme中,是一种流程控制的机制,实现诸如 return、break、goto、协程等功能。Continuation被翻译成“延续”,像它的名字一样,一个延续中包含程序将要进行的计算，即函数调用。

一些命令式语言中常常提供了continue、break、return等语法结构来支持执行流的转移.如果把这类语法结构看作异常处理的话，那么它们都需要一个外部的包围块来“捕获”， 在编译时即可确定。这些语法结构使程序的执行得以由内向外地传递。 而真正的异常可以在任何地方抛出，不一定要有一个包围块， 而且异常抛出后被哪一部分代码捕获也无法在编译时确定。 异常从被调用者传递给调用者，使执行流得以在调用栈上由下向上地传递。

另外，C也提供了setjmp/longjmp来允许程序进行非局部的跳转。 setjmp/longjmp的限制在于，longjmp跳转对应的setjmp所在的函数不能退出， 也就是说在调用longjmp时必须保证之前的栈信息不被修改。

call/cc全称是`call with current continuation`，在Scheme中是一个函数`call-with-current-continuation`， 用全名和call/cc均可以调用。call/cc可以看作一个控制流的操作符，用来实现程序执行的跳转。 所谓一个continuation是指程序运行中的某一个snapshot，告诉我们程序未来将要进行的所有操作。 例如在(sqr (+ 6 7))中，(+ 6 7)所处的continuation便是，得到(+ 6 7)这个值，然后将其平方。

在Scheme中，函数call/cc接收另一个函数f作为参数，并将f作用在当前continuation上， 因此call/cc常用来抽取当前的continuation。这个continuation保存了当前程序执行的上下文， 它（指当前continuation）唯一支持的操作是函数调用。 调用一个continuation时，现有的执行流被挂起， 被调用的continuation所对应的执行流被恢复。 调用这个continuation时传入的参数则作为创建它的call/cc的返回值（执行流回到call/cc创建这个continuation的地方）。 这样通过call/cc创建出来的continuation可以反复在其他地方调用， 而不仅仅限制于包围call/cc的作用域。 像这种将程序的隐含状态提取出来的过程称为`reification`（具体化）。

传给call/cc的函数f，将会以当前continuation为参数被调用。 在其中，若这个作为参数的continuation被调用， 则控制流返回其对应的call/cc处，并将调用continuation时传入的参数作为这个call/cc的返回值。 否则f正常退出，执行继续。


call/cc 为一个带参数的过程，参数为“当前延续”（current-continuation），current-continuation在任何一点都是rest of the program的抽象。

Example 01:

	#lang racket
	(+ 2 
   		(call/cc
    	(lambda (k)
      		(+ 2 (k 3)))))


分析：
代码可以看成(+ 2 [ ]),k就是current-continuation代表rest of the program的抽象。当程序进入(k 3) 的时候，就跳到(+ 2 [ ]),结果为5
我们可以用一个函数来表示：
	
	(lambda (val)
		(+ 2 val))
实际上，延续本质上就是一个函数，或者说闭包。



在sheme中，continuation也可以被用来多次跳转到程序已经执行过的地方

Example 02:

	#lang racket
	(define f '())
	(+ 2 
   		(call/cc
    		(lambda (k)
      			(set! f k)
      			(+ 2 (k 3)))))

	(f 5)
	(+ 2 (f 5))
	; => 5
	; => 7
	; => 7
--
分析:

(f 5)
这时候结果为7.
也就是(f 5)解释为(+ 2 [])，然后[]被替换成了5.

再看代码：(+ 2 (f 5))
返回结果也为7
解释：f 调用会将程序控制流直接跳转到 "the continuation" (+ 2 []) 处，再用 5 替换 []，执行那里的代码。

###0x02 Continuation的特性

1. Scheme 中使用 call-with-current-continuation （可以简写成 call/cc）来抽取当前程序中的延续。

2. call/cc 的参数是一个 lambda 函数，lambda 的参数即为当前延续。和函数一样，延续也是 first-class，并且接受一个参数的调用

3. call/cc 的返回值遵守这样的规则：如果当前延续被调用，则 call/cc 立即返回，返回值为调用当前延续的参数。否则程序正常执行并返回

4. 延续保存了程序执行时的所有堆栈状态。当调用一个延续时，当前的执行状态被挂起，不再执行，转而跳到延续所在堆栈继续执行并返回

###0x03 Continuation 作用
####实现 return
作为一门函数型语言，Scheme 是没有 return 语句的，每一个函数都会隐式地 "return" 一个值，这个值又传递给上一级函数调用，这样逐级返回，直到 top-level。这就是说，显示的 return 对于 Scheme 实际上是没多大用处的，但是为了展示延续的功用，我还是来实现一个 return。

	#lang racket

	(define (member? lst element)
  	(define (member lst element return)
    	(for-each (lambda (l)
                (if (equal? l element)
                    (return l)
                    '())) lst)
    '())
  		(call/cc (lambda(c)
             (member lst element c))))
	(member? '(5 4 3 2 1) 4)
	(member? '(5 4 3 2 1) 6)

	;=> 4
	;=> '()

####实现生成器


	#lang racket
	(define (make-next lst)
  		(define (next-i lst return)
    		(for-each (lambda (l)
                (call/cc (lambda (c)
                           (set! next-i c)
                           (return l))))
              lst) "no element")

	(define (next)
    	(call/cc (lambda (c)
               (next-i lst c))))
  		next)
	(define f (make-next '(1 2 3 4)))
	(f)
	(f)
	(f)
	(f)

	; => 1
	; => 2
	; => 3
	; => 4

这个例子比 return 要复杂一些，它有两个 call/cc，(next) 中使用 call/cc 的是为了实现 return。而 (next-i) 中的 call/cc 要稍稍麻烦一点，当第一次调用 (f)，(f) 内部会调用 (next-i)，这时的 next-i 是 (make-next) 中定义的普通函数，函数中 (for-each) 首先迭代 lst 的第一个元素，然后抽取当前延续，保存到变量 c 中，之后再将 c（延续）赋值给 next-i，最后返回 lst 的第一个元素。把 c 赋值给 next-i 之后，next-i 就不再是之前 (make-next) 中定义的那个函数了，而是当前的延续。所以，当 (next-i) 第二次被调用，程序回到第一次调用 (next-i) 时的运行堆栈上，接着当时返回第一个元素地方继续执行，即 (for-each) 迭代第二个元素，然后再次保存当前延续到 next-i，最后返回第二个元素。类似的，直到第五次调用 (next-i) 返回 "no element" 。

###协程




--


###其他

[![给我捐款](http://c000005.qiniudn.com/donate_me.png "给我捐款")](http://me.alipay.com/0xc000005)

[回到主页][1]

                                       
[1]: http://0xc000005.github.io/
[2]: http://en.wikipedia.org/wiki/Continuation

