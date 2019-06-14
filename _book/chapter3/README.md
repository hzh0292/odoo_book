# 第三章 字段类型和基本视图类型

上一章，我们介绍了如何从无到有地创建一个模块，本篇将介绍Odoo中常见得几种字段类型和基本的视图类型。

## 字段类型

由于Odoo使用了自己编写的ORM框架，因此它有着自己的一套与数据库映射的关系逻辑，简单地说就是由简单字段关系和复杂字段关系组成的。简单的字段关系比如Char,Integer,Float等，复杂字段关系如Many2one，Many2many等。下面就常见的几种字段类型进行简要的介绍。


### 通用字段属性
有些属性是字段共同拥有的，如下表：


| 属性名      | 解释                        |
| ---------- | --------------------------- |
| name       | 字段名                       |
| model_name | 模型名称                     |
| store      | 是否存储在数据库中(默认True)            |
| index      | 字段在数据库中是否索引（默认False)|
| manual     | 是否为自定义字段（默认False）|
| default    | 字段的默认值                |
| string     | 字段的标签值                |
| help       | 帮助提示的值                |
| readonly   | 是否只读(默认False)         |
| required   | 是否必填(默认False)         |
| states     | 根据state设置reaonly和required属性|
| groups     | 可以访问该字段的组的xmlid    |
| related_sudo| 字段是否应该作为admin来访问 |


### Char

字符类型，对应数据库中varchar类型，除了通用类型外接收另外两个参数：

* size: 字符长度，超出的长度将被截断
* trim: 默认True，是否字段值应该被去空白。

### Text

文本类型，对应数据库中的text类型，不限长度，没有额外的参数。

### Html

Html类型，用于存储一段HTML代码，对应数据库中的text类型，接收下面几个参数：

* sanitize：是否过滤安全字符(默认True)
* sanitize_tags： 是否过滤的html标签（只接收白名单列表标签，默认True）
* sanitize_attributes： 是否过滤的html属性（只接收白名单列表属性，默认True）
* sanitize_style： 是否过滤style属性(默认False)
* strip_style： 是否去除style属性中的空白(默认False)
* strip_classes:是否去除class属性中的空白（默认False)

### Date

日期类型，对应于数据库中的date类型，该字段包括如下几个特有属性：

* today: 当前日期
* context_today： 返回客户端时区的当前时间
* to_date: 将时间值转换为date类型的时间
* to_string: 将时间值转为文本

### Datetime

时间类型，对应于数据库中的timestamp类型，该字段包括如下几个特有属性：

* now: 当前时间
* today: 当前日期
* context_timestamp： 客户端时区的当前时间戳
* to_datetime：将时间值转换成datetime类型
* to_string： 将时间值转成文本

### Binary

二进制文件类型，接收三个参数：

* prefetch: 默认False
* context_dependent：默认True
* attachment：默认False，是否作为附件存储

二进制文件作为附件存储时存在服务器指定的文件夹路径中，否则存在数据库中，对应的数据库类型为bytea.

### Selection

下拉选择类型，多选一类型，不同于Many2one类型，Selection类型在数据库中对应的类型是int4或varchar类型。

除了通用的属性，另外接收2个参数：

* selection： 可选的范围值，值和名称组成的元组列表。
* validate: 默认True，是否写入时校验。

Selection包含如下的属性或方法：

* get_values: 返回可选的值列表。

### Reference

引用类型(继承自Selection)，对应数据库类型varchar，Reference字段不同于Many2\*类型的地方在于Many2\*类型的字段的comodel是固定的，而Reference可以提供一种动态的选择。

比如，我们给模型book添加了一个ref字段，类型为Reference，关联两种类型，一个出版商，一个作者，那么ref字段就可以供用户在使用的时候自行选择绑定的记录：

![](odoo-tutorial-2/reference.png)


### Many2one

多对一类型，对应于数据库中的类型是int4，相当于数据库主表中的外键。可选的参数如下：

* comodel_name： 被关联的对象
* domain： 过滤条件
* context： 上下文
* ondelete： 删除时的选项，可选set null(本字段设置null)，restrict(严格控制，只有先删除本字段才允许删除)和cascade(级联删除本字段关联的所有记录)。
* auto_join： 当字段被搜索时是否自动聚合，默认为False
* delegate: 当设置为True时，关联对象的所有字段将在本字段变得可用。

### One2many

一对多类型，返回值是一个关联对象的集合。接收的参数列表如下：

* comodel_name： 关联对象名称
* inverse_name： 在Many2one对象中的字段名 默认为None
* domain： 过滤条件
* context： 上下文
* auto_join： 当字段被搜索时是否自动聚合，默认为False
* limit： 读取的条数限制
* copy: 是否允许拷贝（默认False)

定义一个One2many对象时，要在关联的对象中定义一个Many2one的字段类反过来关联本对象。比如，一个出版商签约了多个作者，那么出版商和作者就是一对多的关系（当然也可以定义为多对多的关系），用代码表示出来就是下面这个样子：

```python
class Author(models.Model):

    _name = 'book_store.author'

    name = fields.Char('名称', help='作者名称')
    age = fields.Integer('年龄')
    publisher_id = fields.Many2one(
        'book_store.publisher', string='签约出版商', ondelete='no action', required=True, help='')


class Publisher(models.Model):

    _name = "book_store.publisher"

    name = fields.Char('名称', help='出版商名称')
    signed_authors = fields.One2many(
        'book_store.author', 'publisher_id', string="签约作者")
```

对于One2many和Many2many对象，有几种特殊的操作，就是在关联对象中创建或是修改本对象时的操作：

* (0,0,{values}) 根据values里面的信息新建一个记录。
* (1,ID,{values})更新id=ID的记录（写入values里面的数据）
* (2,ID) 删除id=ID的数据（调用unlink方法，删除数据以及整个主从数据链接关系）
* (3,ID) 切断主从数据的链接关系但是不删除这个数据
* (4,ID) 为id=ID的数据添加主从链接关系。
* (5) 删除所有的从数据的链接关系就是向所有的从数据调用(3,ID)
* (6,0,[IDs]) 用IDs里面的记录替换原来的记录（就是先执行(5)再执行循环IDs执行(4,ID）

举例来说，就是我希望在按钮的触发事件中同时创建一个出版商和多个作者，而不是一步一步地创建，就可以这么写：

```python
@api.one
def btn_test(self):
    """测试方法"""
    _logger.info(f"测试创建出版商和作者")
    self.env['book_store.publisher'].sudo().create({
        "name": "超新星出版社",
        "signed_authors": [(0, 0, {'name': '本杰明 巴顿', 'age': 90}),(0,0, {'name': '刘天然', 'age': 28})]
    })
```

### Many2many

多对多类型，参数列表如下：

* comodel_name：关联对象名
* relation：关系表名
* column1：本对象的关联字段名
* column2：关联对象的关联字段名
* domain： 过滤条件
* context： 上下文
* limit: 读取条数限制
  
与One2many类似，但不同的是参数中多了一个relation，用来存储多对多关系的表名，需要注意的是，relation，column1和column2可以不提供，odoo会根据model名称自动生成，但是需要保证column1和column2是不同的值。

### Id

integer类型，对应数据库中的int4，一般不需要定义。

## 视图类型

上面讲述了字段的列表，接下来我们看一下常见的视图类型。

最常见的视图有如下几种：

* form视图
* tree视图
* kanban视图
* graph视图
* gatt视图
...

因为视图是前端展示用的，所以视图的种类多种多样，odoo也可以支持自定义视图，这就给了开发人员很大的灵活拓展性。每种视图的结构不一样，这里只介绍最常见的form和tree视图。

## form 视图

一般的视图都是在record节点中，由arch结点包裹：

```xml
<record id="book_store.book_form" model="ir.ui.view">
      <field name="name">图书</field>
      <field name="model">book_store.book</field>
      <field name="arch" type="xml">
        <form string="图书详情" class="">
          <header>
            <button type="object" name="btn_test" string="测试按钮"/>
          </header>
          <sheet>
            <h1>
              <field name="name" invisible="0"/>
            </h1>
            <group>
              <field name="author" invisible="0"/>
              <field name="date" invisible="0"/>
              <field name="price" invisible="0"/>
              <field name="ref" invisible="0"/>
            </group>
          </sheet>
        </form>
      </field>
    </record>
```

form视图就包括，header、sheet等部分，然后group节点可以对field进行分组，字段field的属性用来控制field的显示。

### tree视图

把form视图代码中的form替换成tree就成了我们的列表视图，这里称之为列表视图是因为tree视图并不是真正的"树视图",真正的树视图是可以展开的。

```xml
<record model="ir.ui.view" id="book_store.list">
      <field name="name">图书列表</field>
      <field name="model">book_store.book</field>
      <field name="arch" type="xml">
        <tree>
          <field name="name"/>
          <field name="author"/>
          <field name="date"/>
          <field name="price"/>
        </tree>
      </field>
    </record>
```

tree视图相对比较简单，一般就是对本对象要显示的属性的罗列，外加一些权限显示的控制。

其他类型的视图，读者可以在官方自带的模块中找到例子，这里就不列举了。



