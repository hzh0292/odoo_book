# 第八章 深入视图

## 按钮

按钮的两种类型，object和action。

## 默认视图

当你定义了一个模型，如果你没有写任何关于这个模型的视图，那么Odoo将会为你自动生成默认的视图。

_get_default_form_view 方法用于生成默认form视图
_get_default_search_view 方法用于生成默认的搜索视图
_get_default_tree_view 方法用于生成默认的列表视图
_get_default_pivot_view 方法用于生成默认的透视图
_get_default_kanban_view 方法用于生成默认的看板视图
_get_default_graph_view 方法用于生成默认的graph视图
_get_default_calendar_view 方法用于生成默认的日历视图

## Datetime类型转Date 

在视图中可以使DateTime类型的部件表现得像Date一样，只需要在xml中指定部件

```python
<field name="date" widget="date"/>
```