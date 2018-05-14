# 控制复杂度的方法
* 黑盒抽象方法
    * --> |找到一个数的平方根| -->
    黑盒抽象的基本原则
        * 将黑盒中的细节隐藏，这样你能够抽身去做更大的黑盒
        * 黑盒中的规则实现，寻找通用方法 * x--> F|黑盒| -->y
            * 我们现在有x, 怎么设计黑盒规则，找到y
                - 任意猜测一个z, 去验证你的描述F(z, x), 如果不符合预期，
                — 不断应用现有规则F作用于improve(x, z),直到y不变,不再改变。
* 按照约定来实现相应的接口
    F(A)->B ,将任意类型的A都可以应用于F，这种对A的约束就是接口 * 定义新的语言
    * 新语言的设计是为了强调系统的某个方面,它隐藏了部分细节，另一个方面强调了一些细节

# 如何开始一门的新的语言?
    * 语言的基本元素有哪些？
    * 语言是如何将这些元素结合在一起的？ 或者说是组合的方法? 组合成更大的对象是什么?
    * 抽象方法是什么？(如何利用这些元素并把他们封装成黑盒？)

(+ 2 17.4 5) -> 14.4
 操作符 操作对象
组合式

# Lisp

## 定义一个过程
~~~
    * (define (square x) 
            ( * x x))
    * (square 100)

    * (define square (lamda(x)(* x x))

    if 是cond的语法糖
    * (define (abs x)
        (cond 
            (( < x 0) (-x))
            (( = x 0) 0)
            (( > x 0) x))
    * (define (abs x)
        (if (< x 0) 
            (-x)
             x))
~~~

## 软件的功能从总体来看，将一项功能封装在一个黑盒里面
## 子功能也一样
## 这种把东西打包大定义内部的一种方法叫块结构
## 快结构展示

~~~
(define (sqrt x)
    (define (improve guess)
        (average guess (/ x guess)))
    (define (good-enough? guess)
        (< (abs (- (square(guess)), x))
            .001))
    (define (try guess)
        (if (good-enough? guess) 
            guess
            (try (improve guess))))
~~~

## 表达式的种类

* 数字
* 符号
* 组合
* Y型表达式
* 定义的块
    (define A (* 4 5))
* 条件表达式
    (define (B x) (if (< x 0) 0 x))
前三个是通用表达式，后三个是特殊表达式

理解复杂事物的关键就是避免不必要的观察计算和思考
这其实也是presentation的关键

应用替代模型并不能真实描述计算机的工作过程
但是可以把问题描述的很清楚
求值顺序是从右到左
掌握一个过程的命名很重要，因为当你可以准备表达它的时候，
你就准确的使用它

另一种理解迭代和递归的过程

迭代
    * 把计算过程想象成一个计算机，参数变量想象成持久存储。如果停机了，还可以接着之前的事务状态继续计算
    * 因为继续计算所需要的信息都保存在参数变量中
    * 如果是递归，计算过程需要存储中间数据，如果停机重启了，这个计算过程就无法恢复了

把相似的过程抽象提取出来，这样你只需要调试一次，理解一次
高阶函数就是这么被提取出来的东西

求和操作的抽象

~~~
(define (<name> a b)
    (if (> a b)
        0
        (+ (<term> a)
            (<name> (<next> a) b))))  

(define (sum term a next b)
    (if (> a b)
        0
        ( + (term a)
            (sum
                term 
                (next a)
                next
                b))))

迭代方法，相当于尾递归
(define (sum term a next b)
    (define (iter j ans)
        (if (> j b)
            ans
            (iter (next j)
            ( + (term j) ans))))

    (iter a 0))
~~~

将求不动点的过程抽象出来
~~~
(define (fixed-pointed f start)
    (define tolerance 0.00001)
    (define (closed-enough? old new)
        ( < (abs (- old new)) tolerance ))
    (define (iter old new)
        (if (closed-enough? old new)
            new
           (iter new, f(new))))
    (iter start f(start)))

再用sqrt对fixed-pointed进行封装
(define average-dump
    (lambad(f), (lambda(x)(average(f x) x))))
(define (sqrt x)
    (fixed-pointed (average-dump(lambda(y)(/ x y)))
        1)
~~~
to find a y such that 

f(y) = 0

start with a guess, yn
yN+1 = yN - f(yN) / df/dy|y=yN

假设我们已经实现了牛顿法

~~~
fix-point的f表示传入一个旧的值怎么得到新的值
(define (sqrt x)
    ;存在一个函数让y^2 = x
    (newton (lambda(y)(- x (square y))
    1))

(define (newton f guess)
    (define df (deriv f))
    (fix-point (lambda(x)(- x (/ (f x)(df x))))
    guess)

求导函数
(define deriv
    (lamda (f)
        (lamda(x)
            (/ (- (f (+ x dx))
                    (f x))  
                  dx))))

(define dx 0.00001)
~~~

# 数据类型抽象

在我们构造一个任务的时候
最重要的是让构造任务的抽象和实现分离开来。

在大型系统中我们就需要许多这种一层层的的抽象屏障。
这种思想需要不断的贯彻在我们编码的过程之中。

# 数据抽象的目的也是为了构建层次化的抽象系统
建立抽象屏障把实现细节隔离在底层。

抽象的关键就是订立契约(接口)，让其他人去根据契约去具体实现

~~~
层B
(define (make-rat n d)
    ..
(define (numer x)
    n
(define (denom y)
    d
层A
(define (+rat x y)
    (make-rat (+ (*(numer x)(demon y))
       (*(numer y)(demon x)))
    (* (demon x)(demon y))))
(define (*rat x y)
   (make-rat (* (numer x)(numer y)) 
             (* (demon x)(demon y)))

~~~
### 上面的例子为什么把n，d组成一个数，然后多次一举后面又抽出来呢？

这样是为了把n,d总是绑成一个整体对象，+rat 和 *rat过程中对外暴露只有有理数

而不需要暴露有理数里面的细节，我们这一层的抽象总是以有理数这个对象来操作的

而且通过numer和demon函数的操作对象也是有理数对象，返回值只是一个通用的对象
也意思是number x->d 解绑了x和d的实际关系了

注：我们在一些面向对象语言中有一些访问器，这些访问器实际上也是隔离了抽象对象
内部的实际实现细节

我们上面实际把这个问题隔离成了两部分，第一部分是让另一个去实现如何制造有理数

他向我们提供了构造有理数的构造函数，还有一部分是用于提取分子和分母的选择函数

为什么层A和层B这样分呢？或者为什么层A在层B之上?

因为在层B里面，这三个过程都暴露了有理数的内部数据结构
而在层A里面，我们是完全看不到有理数内部结构的。
想象一下，如果demon过程放到层A里面，A的用户完全可以看到对象的内部结构了
这违反了封装的原则。

层B的实现问题

如何将n，d绑定在一起形成一个新的对象呢？
Lisp 提供这种粘合数据的胶水,叫表结构(构造有序对的方法)

(cons x y)
    constructs a pair whose first part is x and whose second
    part is y

(car p)
    selects the first part of the pair p
(cdr p)
    selects the second part of the pair p

~~~
 层B的实现

(define (make-rate n d)
        (cons n d ))
(define (numer x)
    (car x))
(define (demon x)
    (cdr x))
~~~

~~~
+rat *rat           Use
----------抽象屏障
-----------
make-rat numer demon 抽象层
----------
使用了pairs实现了抽象层   Represention
~~~
抽象层就是构造函数和选择函数
这是一种方法学,该方法分离了对象的表示和使用方法
抽象层有个名字叫数据抽象
数据抽象是一种通过假定的构造函数和选择函数将数据对象与它表示分离开来的
编程方法学

这种抽象的好处是什么？和直接实现有什么根本的分别吗？
~~~
(define (+rat x y)
    (cons (+ (* (car x)(cdr y))
             (* (car y)(cdr x)))
    (* (cdr x)(cdr y))))
~~~
和之前的实现对比一下，这里的实现我们根本无法找到有理数加法的定义，
也就是说，下面这个没办法反应有理数的加法规则, 这里没有一个有理数的
概念实体。

通过下面练习我们来深刻理解一下
~~~
(define A (make-rate 1 2)
(define B (make-rate 1 4)

(define C (+rate A B))

(numer C)->6
(demon C)->8
这里我们期望的输出是这样的
(numer C)->3
(demon C)->4

这里明显缺少了一个话公因式的方法，但是我们可以在两个地方来实现这个方法
放在表示层来实现，或者放在使用层来实现？

表示层
(define (make-rate n d)
    (let ((g (gcd n d ))))
    (cons (/ n g)
           (/ d g)))
~~~
这里明显是要放在表示层，因为使用层根本不需要知道最小公约数这种东西
还有一种重要的原因就是，把化简这种东西发在表现层，可以更好延时操作，
减少上层抽象的复杂性.

通过抽象方法，我们可以假设我们已经得到了结果了
然后在讨论哪个是对的，或者你应该怎么得到结果

在编码之前就能把系统设计完美，是没有设计大型系统的。

所以通过抽象层做出决定非常重要。

这些东西真正的厉害之处不是体现在实现本身,系统本身的实现构建没什么了不起

而是你可以将这些东西用于构建更复杂的东西

~~~
(define (make-vector x y)
    (cons x y))
(define (x-cor v)
    (car v))
(define (y-cor v)
    (cdr v))

(define (make-seg x y)
    (cons x y))
(define (seg-start seg)
    (car seg))
(define (seg-end seg)
    (cdr seg))
(define (seg-length seg)
    (let ((dx (- (x-cor (seg-end seg))
            (x-cor (seg-start seg))))
          (dy (- (y-cor (seg-end seg))
            (ycor (seg-end seg)))))
    (sqrt (+ (square dx)
              (square dy)))))

~~~
我这里还在思考一个问题，是不是最好上一层的抽象使用下一层抽象的实现呢？
还是说这种抽象方法的调用可以任意跨层的?

~~~
segments
----------
make-seg seg-start seg-end 抽象层
----------
vectors
-----------
make-vector xcor ycor  抽象层
----------
parts and number
---------
~~~

抽象层的方法，应该能很好的代表这个对象的概念
比如

IF x = (make-rat n d)
THEN
    (numer x)/(demon x) = n/d

这个就是有理数的契约，别人只要实现这个契约就行了.

这个规则能很好的表示一个有理数，所以当我们抽象一个事物的时候
我需要弄清楚这个被抽象的事物是什么，被抽象之后，能否使用抽象接口来
表示这个被抽象的事物。

换个说法被抽象之后，还能根据事物的定义或者定理来表示这个事物

抽象的讲，我们可以认为有理数实际上就是这些公理

我们可以延伸一下这个概念，计算机很多协议，这些协议就是对事物的抽象，实现
方式却是千变万化的

让我们看看有序对的抽象

什么才是有序对呢？
这里其实也是一个抽象

你会发现有序对应该具有这些属性

for any x and y
    (car (cons x y)) is x
    (crd (cons x y)) is y

这里cons car crd就是有序对的抽象层
下面是有序对的一种实现
~~~
(define (cons a b)
(lambda (pick)
    (cond ((= pick 1) a)
          ((= pick 2) b))))

(define (car x) (x 1))
(define (crd x) (x 2))
这里好神奇，cons居然返回一个函数，然后如果要去里面的元素就要使用car或者crd
而car或者crd就调用这个函数.
这里更神奇的时候模糊了数据和代码之间的界限，我们完全可以把cons后的东西当成一个
表数据结构，但是这里神奇用函数解决了。

这里需要明确一个概念，过程不是一个动作的结合，而是一个对象或者实体，是可以带有数据的.

所有可以这么理解(cons 3 4)和(cons 5 6)完全是两个不一样的过程实体。里面包含的
数据是不一样的。
~~~

我们看看字符串，也是一种抽象，就是对+，strlen这些方法进行抽象
整形我们要实现这种对象的四则运算

所有说每一种事务都可以抽象，关键我们要把这些事务的抽象法则抽取出来


序对是具有闭包性质的，也就是说可以用这个完成更复杂的东西

使用闭包可以创建更多的带数据的过程。
LIST
列表是对序对一种特殊的约束形式

(cons 1(cons 2(cons 3(cons 4 nil))))
这种方式写起来太麻烦了，lisp提供了一个语法糖
(list 1 2 3 4)

发现闭包的伟大
闭包是元语言抽象的基础
闭包可以轻易让过程+数据=新的过程
其实初始过程 = 过程+初始数据

(define p 
    (beside g
        (rot-180 (flip g)) 0.5))
(define q
    (above p (flip p) 0.5))

(define (make-rect horiz vert origin)

(define (coord-map rect)
    (lambda (point)
        (+ vect
            (+ vect (scale (xcor point)
                        (horiz rect))
                    (scale (ycor point)
                            (vert rect)))
        (origin rect))))
如何表示一个图像呢？
我们可以用线段来表示

加入了新的数据和代码
(define (make-picture seglist)
    (lambda (rect)
        (for-each
            (lambda(s)
                (drawline
                    ((coord-map rect)(seg-start s))
                    ((coord-map rect)(seg-end s))))
            seglist)))
(define R (make-rect ...))
(define G (make-picture ...))
(G R)
(define (beside p1 p2 a)
    (lambda (rect)
        (p1 (make-rect
               (origin rect)
               (scale a (horiz rect))
               (vert rect)))
        (p2 (make-rect
                (+vect (origin rect)
                       (scale a (horiz rect)))
                (scale (-1 a)(horiz rect))
                (vert rect)))))

# 这里我渐渐发现所有操作返回都是闭包
## beside 返回的也是一个闭包过程
                
元语言抽象和对象抽象有什么关系？
对象抽象出来的都是固定的数据,不能对这些数据再进行调用

元语言抽象出来的都是闭包过程(包含数据和代码)
可以发现没做一次封装我们都添加了新的代码和新的数据

对一个矩形旋转90度，原点也变了
(define (rotate90 pict)
    (lambda (rect)
    (G (make-rect
        (+ vect (origin rect)
                (horiz rect))
        (vert rect)
        (scale -1 (horiz rect))))))
这里关键使用过程来表示图像，使其自动地有闭包的性质
n表示要画多少次
a是缩放因子
(define (right-push p n a)
    (if (= 0 n)
        p
        (beside p (right-push p (- n 1)  a)
        a)))

(define (push comb)
    (lambda(pict n a)
        ((repeated
            (lambda(p)(comb pict p a))
            n)
        pict)))

软件工程又叫神话学
任务->很多子任务->再又分成很多子任务

元语言抽象呢

schemem 组合语言
-----------
language of
几何位置
--------
language of
primitive pictures

上一层永远是基于下一层之上的

软件工程的方法，是每次分解的子任务都是确认的任务
而元语言每一层都是抽象的。不是确切的任务
这就导致后一种方法设计软件更健壮
这里的健壮指的是当你在描述上做出改变时，我们的实现可以
立马做出改变。

这里分解有本质上的不同，是按层次分还是按继承分解
最主要的是元语言抽象在每一层都有确切的抽象概念
这种不同还在于元语言每一层都是抽象的，可以迅速改变，而不影响上下层

与其在问题分解出的子问题求解问题，不如解决一类问题
在这类问题之上构建一门新的语言
当你在语言层次做出小的改变是，这会很简单，因为语言层次就是解决某一类
问题的各种操作

法则从左到右的方向是归约规则
reduction 归约

归约会把一个复杂的问题转化成了简单的问题

(define (deriv exp var)
exp是一个常量表达式
导数的表征是某物的变化率，y=exp(x) 是一个常数，则变化率为0
    (cond ((constant? exp var) 0)
            x = exp(x), 变化率是1`
           ((same-var？ exp var) 1)
            ((sum? exp)
               (make-sum (deriv(a1 exp) var)
                         (deriv(a2 exp) var)))
            ((product? exp)
            (makes-sum
                (make-product (m1 exp)
                    (deriv (m2 exp) var))
                (make-prodcut (deriv (m1 exp) var)
                    (m2 exp)))))
(define (constant? exp var)
        (and (atom? exp)
            (not (eq? exp var))))

(define (same-var? exp var)
    (and (atom? exp)
        (eq? exp var)))
(define (sum? exp)
        (and (not (atom? exp))
            (eq? (car exp) '+) 
(define (make-sum a1 a2)
    (list '+ a1 a2))
(define a1 car)
(define a2 cdr) 
(define (product exp var)
    (and (not (atom? exp))
        (eq? (car exp) '+)))
(define (make-product m1 m2)
    (list *' m1 m2))
(define m1 cadr)
(define m2 caddr) 

如果不加' +会被lisp引用成一个系统定义的+过程

内嵌元语言，生成新的语言，还是元语言的语法

匹配器 一条规则 一个表达式 返回一个匹配后的表达式或者error

a = 总匹配器 所有规则 一个表达式
如果a无效
    输出a
    程序退出
如果a有效
    总匹配器 所有规则 a
        
~~~
同时检查两棵树，一棵树是pat，另一棵树是exp

一条规则= (pattern skeleton)

(define (instantiate skeleton dict)
    (define (loop s)
        (cond ((atom? s) s)
            #骨架求值 以冒号开头的(define (: x))
            ((skeleton-evaluation? s)
            (evaluate (eval exp s) dict))
            (else (cons (loop (car s))
                        (loop (cdr s))))))
    (loop skeleton)) 
(define (evaluate form dict)
    (if (atom? form)
        (lookup form dict)
        (apply
            (eval (lookup (car form) dict)
            user_initial_environment)
        (mapcar (lambda (v)
                (lookup v dict))
                (cdr form)))))
(define (atom? a)
    (and (not (null? a)) (not (pair a))))
(define constant? number?)
(define variable? symbol?)
(define (arbitrary-constant? pat)
    (eq? (car pat) '?c))
(define (arbitrary-variable? pat)
    (eq? (car pat) '?v))
(define (arbitrary-expression? pat)
    (eq? (car pat) '?))
(define (skeleton-evaluation? s)
    (eq? (car s) ':)
(define pattern car)
(define skeleton cadr)

pat = 一条规则的pattern
exp = 用户传入的表达式
dict = 匹配的键值对[pat[0]:exp[0], pat[1]:exp[1]]
(define (match pat exp dict)
    (cond ((eq? dict 'failed) 'failed)
           ((atom? pat)
            (if (atom? exp)
                (if (eq? pat exp)
                    dict
                    'failed)
                'failed))
           ((arbitrary-constant? pat)
                (if (constant? exp)
                    (extend-dict pat exp dict)
                    'failed))
            ((arbitrary-variable? pat)
                (if (variable? exp)
                    (extend-dict pat exp dict)
                    'failed))
            ((arbitrary-expression? pat)
                (extend-dict pat exp dict))
            ((atom? exp) 'failed)
            (else
                (match (cdr pat)
                        (cdr exp)
                        (match (car pat)
                                (car exp)
                                (dict))))))

(define (simplifier the-rules)
    (define (simplify-exp exp)
        (try-rules (if (compound? exp)
                    (simplify-parts exp)
                    exp)))
    (define (simplify-parts exp)
        (if (null? exp)
            '()
            (cons (simplify-exp (car exp))
                (simplify-parts (cdr exp)))))

    (define (try-rules exp)
        (define (scan rules)
            (if (null? rules)
                exp
                (let ((dict
                            求该条规则的pattern部分
                        (match (pattern (car rules))
                            exp
                            (empty-dictionary)))))
                (if (eq? dict 'failed)
                    (scan (cdr rules))
                     把实例作为exp输入
                    (simplify-exp
                        (instaniate
                            求该条规则的skeleton部分
                            (skeleton (car rules))
                            dict)))))
        (scan the-rules))
    simplify-exp)

(define (empty-dictionary) '())

(define (extend-dictionary pat dat dict)
    (let ((name (variable-name pat)))
                查找name键值是否在字典中
        (let ((v (assq name dict))))
            (cond ((null? v)
                (cons (list name dat) dict))
                如果存在也值相等，直接返回字典
                ((eq? (cadr v) dat) dict)
                (else 'failed))))

(define (lookup var dict)
    (let ((v (assq var dict)))
        (if (null? v) var (cadr v))))
~~~
