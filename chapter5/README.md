
# 第五章 API简介

细心的朋友一定注意到了，odoo中的方法经常会看到由api装饰的方法，这是Odoo从8.0起引入的新的编程API，那么为什么要引入新API呢，这要从更早的历史说起。


## 7.0之前的API

当年那是Odoo还叫OpenERP的时代，OpenObject对象的方法通常都需要带着几个固定的参数：cr,uid,ids,context等等，写起来很繁琐，比如下面的例子：

```python
def btn_import(self,cr,uid,ids,context=None):
    ...
```

到了8.0，可能Odoo官方的开发人员也觉得这样写起来太繁琐了，于是乎，他们引入了新API，封装在api.py文件中，主要有一下几种类型：

* model
* multi
* one
* constrains
* depends
* onchange
* returns

下面我们就这几种类型进行详细的介绍。

## one

作用： 表示被封装的方法接受单一的对象，用于处理一些单个对象的方法。
返回值： \[None\](部分不负责的教程里说不返回值)
v7版本写法： def funct(self,cr,uid,ids,context)

举例：
视图文件中添加一个按钮，对应的触发方法：

```python
@api.one
def button_test(self):
    self.name = "xxx"
```

## multi

作用：表示被封装的方法接收一个集合对象，正好与one对应。
返回值： 被封装方法的返回值
v7版本写法： model.method(cr, uid, ids, args, context=context)

multi典型的用法即odoo对象的write方法：

```python
@api.multi
def write(self, vals):
    res = super(Book, self).write(vals)
    return res
```

这里的self是一个集合对象，vals是待更新的字段和值的字典，这里的例子我们没有用到self，只是调用了父类的write方法，将vals写入到数据库中。如果我们要修改某个值，需要对self进行迭代，然后更新字段的值。

## model

作用：self代表一个记录集合，但内容与记录集无关，只是与model类型相关。
返回值： 被装饰方法的返回值
v7版本写法： model.method(cr, args)

典型应用是对象的create方法：

```python
@api.model
def create(self,vals):
    return super(Book,self).create(vals)
```

self是个对象集合，但是creat方法内部却与self这个集合没有关系，只是用了self的model类型，将vals的值进行创建操作。

## contrains

作用：给某个字段添加限制条件
v7版本：是通过_constrains属性关联验证方法的方式实现

到了新API，不需要添加_constrains属性了，直接给校验方法添加constrains装饰即可。

```python
age = fields.Integer(string="age")  

＠api.constrains('age')
def _check_age(self):
    if self.age<16:
        raise ValueError(_('Age must be older than 16'))
```

这样就限制了age字段只能输入比16大的数据。

### 另外一种constrains

odoo支持另外一种添加限制的方式，即通过sql约束的方式。方法是在odoo类对象中添加_sql_constrains属性，值是一个包含了元组的列表，元组的三个值分别是约束名，约束条件和警告信息，看一个例子：

```python
_sql_constraints = [
    ('name_description_check',
     'CHECK(name != description)',
     "The title of the course should not be the description"),
    ('name_unique',
     'UNIQUE(name)',
     "The course title must be unique"),
]
```

这样做的好处是，在数据库层面就限制了数据的校验，而不是在代码层面的校验，显然效率会更高。缺点是在添加限制之前，数据库中不能存在违反约束的数据，否则约束会添加失败。

## depends

depends主要用于compute方法，v8当中已经取消了function字段，对于任何fields都可以通过添加compute属性动态赋值。depends就是用来标该方法依赖于哪些字段的装饰。

```python
@api.depends('date')
def _get_book_age(self):
    self.age = (datetime.now().date() - self.date).days
```

对于compute方法来说，加不加depends装饰的区别在于，加了depends的方法会在依赖的字段发生改变时重新计算本字段的值，而不加depends的方法只在触发的第一次调用，也就是说不会持续更新。

## returns

returns的用法主要是用来指定返回值的格式，它接受三个参数，第一个为返回值的model，第二个为向下兼容的method，第三个为向上兼容的method。

第一个参数如果是对象本身，则写'self',如果是其他对象，则写其他对象名如：@api.returns('ir.ui.view')。

例如：

```python
@api.multi    @api.returns('mail.message', lambda value: value.id)
def message_post(self, **kwargs):
    if self.env.context.get('mark_so_as_sent'):
        self.filtered(lambda o: o.state == 'draft').with_context(tracking_disable=True).write({'state': 'sent'})
        self.env.user.company_id.set_onboarding_step_done('sale_onboarding_sample_quotation_state')
    return super(SaleOrder, self.with_context(mail_post_autofollow=True)).message_post(**kwargs)
```

returns主要用于确保新旧API返回值的一致，并不常用。