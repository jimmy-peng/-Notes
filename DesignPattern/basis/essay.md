# 设计模式


## 工厂模式
* Why
    * 使用工厂模式或者工厂方法，我们只需要这个对象，而不需要了解这个对象的具体创建步骤
    * 我们可以交给一个工厂对象来帮我们生产产品对象，工厂最好也是抽象的类
    * 模块直接的耦合，最好都是面向接口和抽象类


## 迭代器模式
* Why
    * 我们需要一个对象内部的某一集合属性，迭代器防止了暴露对象内存属性细节给外部

## bridge模式
    * 分为类的功能层次和类的实现层次
    * 类的功能层最好是抽象或者接口
    * 类的实现层次对应实际的实现类

* Why
    * 解耦抽象类和它的实现

    ```
    When:
            A             
         /     \
        Aa      Ab
       / \     /  \
     Aa1 Aa2  Ab1 Ab2

     Refactor to
         A         N
      /     \     / \
    Aa(N) Ab(N)  1   2
    ```

## 模板方法

    * 将实现一个目标切割成一组操作集
    * 将操作过程集使用抽象方法实现. 把公共的操作扔在抽象类中，其他全部换成抽象方法

## 适配器模式
    * 客户端和服务端都写好了，但是不能直接通讯
    * 需要在他们之间使用一个适配器，连接他们
