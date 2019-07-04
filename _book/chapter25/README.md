# 第五章 QWeb的继承

## 原生Qweb的继承

基础的QWeb继承与之前的xml的写法都不一样，首先不需要odoo标签的包裹，另外不同于普通的模板，基础模板的写法需要使用templates标签包裹。

例如，我们给Many2one的部件添加一个前缀12345，可以这么写:

```xml
<templates>
    <t t-extend="FieldMany2One">
        <t t-jquery=".o_field_many2one" t-operation="before">
            <a>123456</a>
        </t>
    </t>
</templates>
```

t-extend用于扩展Qweb的内容，配合t-operation可以实现xml中position类似的效果，具体来说，t-operation有以下几种值可以选择:

* append: 节点的主体被附加在上下文节点的末尾（在上下文节点的最后一个子节点之后）
* prepend: 节点的主体被添加到上下文节点（在上下文节点的第一个子节点之前插入）
* before: 节点的主体被插入在上下文节点之前
* after: 节点的主体被插入在上下文节点之后
* inner: 节点的主体替换上下文节点的子节点
* replace: 该节点的主体用于替换上下文节点本身

t-jquery指令采用一个CSS selector。此选择器用于扩展模板以选择应用指定的t-operation的上下文节点。

> 如果使用t-extend，odoo会要求你必须写入一个t-jquery选择器，如果想要使用replace的本身节点话，就会冲突。此时，你应该选择放弃使用t-extend，而使用同名的t-name将原有的代码覆盖掉，就像python的后定义的方法会覆盖掉前面定义的方法一样。

另外，如果想要完全重写某个QWeb部件，那么可以不用上面提到的几种操作，直接简单粗暴的定义一个同名的QWeb部件即可，后加载的部件会覆盖掉之前定义的部件。