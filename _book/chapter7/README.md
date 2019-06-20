# 第七章 视图的继承

贯穿本书始终的一条核心思想就是，**能在不动原生源码的情况下，就不要动原生的代码**。对于视图来说，也是一样。市面上的有些教程，会不负责任的教程会让你直接去修改原生的代码，这样不利于模块的维护和移植，因为一旦开发者离职或是交接时没有说清楚，那么对后续的维护人员来说成本就会变得高昂。

对于视图来说，也是可以通过继承的方式对原生视图进行修改的。下面我们就来教大家如何对原生的视图进行改动。

## XPATH

我们的视图是XML文件，既然要继承，就势必会涉及到XML中标签定位的问题，这就是我们今天的主角——XPATH。

> XPath即为XML路径语言（XML Path Language），它是一种用来确定XML文档中某部分位置的语言。—— 引自百度百科

这里我们简单介绍一下XPATH的语法，我们常用的几种表达式如下：

* / ：从根节点选取
* //：从匹配选择的当前节点选择文档中的节点，而不考虑它们的位置
* .: 选取当前节点
* ..: 选取当前节点的父节点
* @： 选取属性为xx的节点

举例来说，我们要选book_store模块中form视图中的author字段，那么xpath就可以写成下面这样：

```xml
//field[@name='author']
```

### 谓语

XPATH还支持通过谓语来定位相对位置，例如：

```xml
/class/student[1]  #选取属于 class 子元素的第一个 student 元素。
```

常见的谓语列表：

* last(): 最后一个元素
* last()-1: 倒数第二个元素
* position()<3: 前两个元素
* @gender：拥有属性gender的元素
* ID<50： ID小于50的元素
* student[ID<50]/name： student 元素的所有 name 元素，且其中的 ID 元素的值须小于 35

XPATH语法有很多，这里不一一介绍了，需要说明的是学号XPATH不仅对写odoo的模块有用，对于今后的其他框架的开发也是大有益处的。读者可以自行搜素XPATH相关的资料进行学习。

## 视图的继承

视图的继承要掌握以下两个重点：

* 要继承的视图的id
* 对目标字段的修改

### 继承

视图的继承，很简单，只需要在xml文件中添加要给inherit_id字段，并且ref到父视图的xml_id就可以了。

例如，我们在book_store模块中新建一个类拓展原有book.store对象，新增一个分类字段，并显示在form视图的ref字段后面。

```python
class Book(models.Model):
    _inherit = "book_store.book"

    category = fields.Char("分类")
```
视图的继承代码：
```xml
<record id="book_store_book_inherit" model="ir.ui.view">
    <field name="name">继承视图</field>
    <field name="model">book_store.book</field>
    <field name="inherit_id" ref="book_store.book_form"/>
    <field name="arch" type="xml">
        <xpath expr="//field[@name='ref']" position="after">
            <field name="category"/>
        </xpath>
    </field>
</record>
```
升级模块后就可以看到我们的新增加的字段了:

![](category.png)

## 继承的操作

xpath中的position操作，有以下几种:

* inside: 内嵌到节点中
* after: 在节点后添加
* before: 在节点前添加
* replace: 替换掉节点
* attributes: 更新属性

前面的示例中，我们展示了after的用法，before与之相反，是在目标节点的前面添加。如果我们想要替换到原有的节点，比如销售模块中的地址排列，就可以使用replace来完全重写这个节点的代码。

下面重点介绍一下attributes的用法，attributes提供了一种让我们可以修改节点属性的方法，例如，我希望将字段ref的标签改为中文的“参考”，那么我们就可以这么写XPATH:

```xml
<xpath expr="//field[@name='ref']" position="attributes">
    <attribute name="string"/>参考</attribute>
</xpath>
```

## 字段继承的另外一种方式

除了XPATH外，对于field节点，我们有一种简化的写法：

```xml
<record id="book_store_book_inherit" model="ir.ui.view">
    <field name="name">继承视图</field>
    <field name="model">book_store.book</field>
    <field name="inherit_id" ref="book_store.book_form"/>
    <field name="arch" type="xml">
        <field name="ref" position="after">
            <field name="category"/>
        </field>
    </field>
</record>
```

这种写法与XPATH的效果完全一样。

## 继承的优先级

当一个视图被多个视图继承时，页面最终渲染的效果是这些视图叠加的效果。在视图的对象ir.ui.view种有一个字段用于标识视图的优先级，这个字段就是priority，当priority的值越小，它的优先级就越高。

想要查看本页面有哪些视图继承，可以在debug模式下，打开右上角的查看视图-表单（树形等）查看：

![](view.png)