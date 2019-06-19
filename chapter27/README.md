
# 第七章 菜单

菜单是odoo种最常见的组件之一，其基本的作用就是作为前端和动作(action)的纽带。菜单的定义，我们在第一部分已经接触到，并且写了很多的示例，相信读者已经熟练的学会了定义一个菜单并绑定相应的动作。

本章，我们将着重认识菜单的本质是什么，以及它所拥有的哪些高级属性，以及它的高级应用。

## 菜单的本质

菜单实际上就是odoo中的一个对象，被命名为ir.ui.menu。

菜单对象所拥有的常见属性有如下几个：

* name : 菜单的名字
* complete_name: 完整的路径，这个是由系统自动得出的
* parent_id: 上级菜单的id
* action: 绑定的动作
* web_icon: 菜单的图标
* sequence: 序号（优先级）
* web_icon_data: 上传的图标文件
* child_id: 子菜单集
* group_ids: 权限组

我们在xml文件中常用的标签:

```xml
<menuitem id=".." parent=".." .../>
```

实际就是正常record的简写。完整的菜单可以写成如下的模式：

```xml
<record id="..." model="ir.ui.menu">
    <field name="name"></field>
    <field name="parent_id"></field>
    <field name="action" ref=""/>
    <field name="web_icon"></field>
    ...
</record>
```

## 菜单的重写与继承

我们知道，在odoo中的model和qweb中的widget都是可以继承和重写的，实际上，对于xml，也是可以重写和继承的。比如说，我们希望对odoo原有菜单进行修改或者移动原有菜单所在的位置，那么该如何去做呢？

>对菜单的修改，不可以使用简写的模式

实际上，菜单的继承和重写非常简单，只需要把xml定义中的id跟要重写的菜单保持一致就可以了，也就是通过覆盖，把对应的属性改成新的值。

例如，我们希望把采购-采购菜单下面的产品菜单单独拿出来作为一个新的菜单“基础数据”的子菜单，那么我们就可以这么写:

```xml
<record id="purchase.product_product_menu" model="ir.ui.menu">
    <field name="parent_id" ref="menu_purchase_main_data"/>
    <field name="name">Products</field>
</record>
```

menu_purchase_main_data是我自己新定义的一个菜单，这样就完成了对原有菜单的父菜单的重写，界面上的变化，就是我们把“产品”菜单移走了。
