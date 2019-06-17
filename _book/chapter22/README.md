# 第二章 动作

odoo中的动作，指的是一系列点击的操作，对应不同的应用场景。最常见的就是act_window这个动作，像我们打开form视图、tree视图和search视图的操作都是act_window动作。 odoo中的动作可以分为如下几类：

* act_window: 与视图相关的工作，常见的有form\tree\search\kanban等等。
* act_window_close: 与act_widnow配合使用，用于关闭窗口。
* act_url： 页面跳转相关的动作
* server： 触发服务器动作
* todo： 配置向导
* client： 客户端相关动作

## 通用属性

上面列出的几种动作类型，均是继承自ir.actions.actions对象而来，所有他们有着共同的几个比较重要的属性：

* name: 动作名称
* type: 动作类型，通常为类型名称
* xml_id: 引用的外部xml_id
* help： 说明
* binding_model_id： 在Sidebar中显示菜单项的模型（绑定模型）
* binding_type： 绑定类型，可选值有action,action_form_only和report，默认为action，action与action_form_only的区别在于action会是sidebar上的按钮显示在tree和form视图中，action_form_only则只显示在form中。report用于报表打印。


## act_window

视图动作类型，这个是我们最经常用到的一个动作。我们在前面的例子中创建的视图都会通过绑定一个动作关联到一个菜单中，然后才会在页面中显示出来。当我们单击菜单时，就会触发绑定的动作，找到关联的视图，最后渲染成我们所见到的页面。

下面详细介绍act_window所拥有的属性及作用：

* view_id: 指定动作所绑定的页面，值为页面的xml_id。
* domain: 过滤条件，值为python表达式，过滤目标数据。
* context: 上下文
* res_id: 关联的数据库ID，只有当view_mode参数仅为form时起作用
* res_model: 要打开的视图数据所属模型
* src_model: 该动作绑定的源模型（在该模型上打开动作)
* target: 目标窗口，可选值有current\new\inline\fullscreen\main,默认为current,即在当前窗口打开，new是弹出窗口，就是我们常见的模态窗口。
* view_mode: 视图类型，tree,form,kanban等
* view_type: 用于展示列表视图的类型，可选值有tree和form。
* usage： 在user页面过滤菜单和默认动作的选项
* view_ids：关联的视图对象（one2many)
* views: 显示的视图
* limit： 树形视图显示的条数
* groups_id: 拥有访问权限的组id
* search_view_id: 关联的搜索视图id
* multi: 是否实只在树形视图中显示。

关于ir.actions.act_window的使用示例，这里就不过多介绍了，前面的例子中很多地方都用到了，相信大家对此也已经驾轻就熟。

不过有个问题还是需要大家留意一下，就是我们通常在定义xml的时候要先定义action然后再定义菜单menu对象，否则有可能会出现报错，找不到action的问题。

## act_window_close

ir.actions.act_window_close的作用非常简单，就是关闭当前窗口。常用的场景就是完成一段业务逻辑后，需要将此窗口关闭时。这时，只需要返回一个act_window_close的动作即可。

```python
def btn_OK(self):
    return {
        'type':'ir.actions.act_window_close'
    }
```

上述的代码是笔者的一个三方模块中的部分代码，这个模块的作用就是在一段代码执行完成后弹窗告诉用户，执行的结果，当用户点击OK按钮时自动关闭弹窗。详细代码可以参考[这里](https://github.com/block-cat/window_pops/blob/12.0/cwin.py)

## act_url

odoo中还有一个重要但不是特别常见的应用场景，就是页面的跳转或是文件的下载。通常由于已经封装好的many2one和many2many空间都自带了跳转连接，一般不需要我们太多的关注页面的跳转问题。但是当我们需要指定一个跳转页面的时候，就需要使用ir.actions.act_url来帮我们完成这个了。

url可选的参数比较少：

* url: 需要跳转的url
* target: 是在新窗口打开还是本页面跳转(new,self),默认是new

举例来说，我希望增加一个菜单，单击后打开必应搜索，那么我的action就可以这么写：

```xml
<record id="act_bing" model="ir.actions.act_url">
    <field name="name">打开bing网页</field>
    <field name="target">new</field>
    <field name="url">http://www.bing.com</field>
</record>
```

> url的写法需要注意，如果你连接的是站内连接不需要添加前面的host及http协议头，如果你添加的是外链，则需要保证URL完整。

另一个比较实用的场景就是下载，原理是一样，我们通常把附件放到文档管理中，那么我在某些地方想要下载一些文件（比如说模板），那么就可以添加一个按钮用于下载，而这个按钮的后台逻辑就是简单的返回一个act_url即可：

```python
return {
        'type': 'ir.actions.act_url',
        'url': f"/web/content/{doc.id}?download=true",
        'target': 'new',
    }
```

doc.id是通过搜索出来的附件的记录ID。

## server

server类型的action主要的使用场景是执行一段预定义的python代码。server类型的action主要包含如下几个属性：

* sequence: 当要同时执行多个server action时，根据本字段的值排序执行。
* model_id: server脚本要在哪个model对象执行。
* model_name: model对象的名称
* code: 要执行的python代码。（包含一些预置的变量类型，后续章节会讲到)
* child_ids: 子server action列表，最后一个子动作返回的结果作为整个动作的返回结果
* crud_model_id: 要变更的模型id
* crud_model_name: 要创建/变更的模型名称
* link_field_id： 指定当前记录与新记录进行many2one关联的字段
* fields_lines： 创建或复制记录时需要的字段。

[TODO]

## todo

ir.actions.todo虽然被定义在了ir.actions，但它确实是这些对象中的“异类”，它没有继承自ir.actions.actions，这也就是说，ir.actions.todo不是一个动作。todo的属性列表如下：

* action_id: 要执行的动作id
* state: 状态，open或是done，默认为open，当被执行完成后设置为done.
* sequence: 序列，默认为10
* name: 名称

示例：

```xml
<record id="act_todo" model="ir.actions.todo">
    <field name="action_id" ref="act_bing"/>
    <field name="state">open</field>
    <field name="sequence">1</field>
    <field name="type">automatic</field>
</record>
```

type的可选值有如下三个：

* manual： 人工设置
* automatic: 自动设置（每次系统设置，或者安装或是升级系统的时候自动执行）
* once: 仅执行一次

todo的使用场景是当在安装或是升级模块时，需要执行某些特殊的动作。

## client

ir.actions.act_client动作是执行完全定义在客户端的动作，而不经过后台。这样就给我们提供了一种绕过后台定义的widget而实现自己的页面的一种方式。client包含如下几个属性：

* tag: 指定客户端部件的id
* target: 打开方式，可选值：current\new\fullscreen\main
* res_model: 目标模型
* params：根据视图tag一同发给cleint的参数
* params_store：储存的参数

举例：

> 这里涉及到QWeb相关内容，没了解Qweb的同学可以参考第四章和第五章

我们先定义一个菜单，绑定我们的客户端动作：

```xml
<record id="act_bing" model="ir.actions.client">
    <field name="name">打开Bing</field>
    <field name="tag">web.bing</field>
</record>

<menuitem name="打开Bing" id="book_store.menu_open_bing" action="act_bing" parent="book_store.menu_root"/>

```

然后我们创建我们自己的页面结构，简单起见，这里只内嵌了一个Bing的页面：

```xml
<templates xml:space="preserve">
    <t t-name="bing">
        <iframe marginheight="0" marginwidth="0" width="100%" height="910" src="https://www.bing.com" frameborder="0" allowfullscreen="True"></iframe>
    </t>
</templates>
```

然后我们创建我们自己的web widget:

```js
odoo.define('require', function (require) {
    'use strict';

    var core = require("web.core");
    var Widget = require("web.AbstractAction");

    var Bing = Widget.extend({
        template: "bing",

        init: function (parent, data) {
            return this._super.apply(this, arguments);
        },

        start: function () {
            return true;
        },
        on_attach_callback: function () {
            
        }
    });

    core.action_registry.add("web.bing", Bing);

    return {
        Bing: Bing
    };

});
```

> 在v11版本中，Widget需要为Widget的子部件，v12中则需要为AbstractAction

最后，我们将定义的Bing部件，加载到xml页面中：

```xml
<template id="assets_backend" inherit_id="web.assets_backend">
    <xpath expr="script[last()]" position="after">
        <script type="text/javascript" src="/book_store/static/src/js/widget.js"/>
    </xpath>
</template>
```

这样就完成了我们自定义的页面，升级模块我们就能看到效果了：


![](bing.png)

