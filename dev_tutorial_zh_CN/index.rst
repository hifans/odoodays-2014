Odoo V8的后台模块开发简介
==================================

Raphael Collet (rco@odoo.com)


中文翻译：先安科技（contact@openerp.cn)


大纲
------

* Odoo的基本架构
* 开放学院（Open Academy）模块示例


Odoo的基本架构
===================


Odoo的基本架构
-------------------

* 三层架构 客户端/服务器/数据库
* Javascript语言开发的Web客户端
* Python语言开发的服务器和后台模块
    * MVC框架结构


开放学院模块（示例）    
==================================


模块
------

* 管理课程，学期，选课
* 学习
    * 模块的结构
    * 模型的定义
    * 视图的菜单的定义


模块的结构    
-------------------

Odoo模块的组成
    * python模块（实体模型），
    * Odoo模块定义文件，
    * XML和CSV数据文件（基础数据，视图，菜单，工作流等）,
    * 前台资源文件(Javascript, CSS).


开放学院模块
------------------

Odoo模块定义文件``__odoo__.py``::

    {
        'name': 'Open Academy',
        'version': '1.0',
        'category': 'Tools',
        'summary': 'Courses, Sessions, Subscriptions',
        'description': "...",
        'depends' : ['base'],
        'data' : ['view/menu.xml'],
        'images': [],
        'demo': [],
        'application': True,
    }


课程模型    
----------------

在Python类中定义数据模型和它的字段::

    from odoo import Model, fields

    class Course(Model):
        _name = 'openacademy.course'

        name = fields.Char(string='Title', required=True)
        description = fields.Text()


使用XML来定义菜单    
----------------------------

.. code-block:: xml

    <?xml version="1.0" encoding="UTF-8"?>
    <openerp>
        <data>

            <menuitem name="Open Academy" id="menu_root" sequence="110"/>

            <menuitem name="General" id="menu_general" parent="menu_root"/>

            <record model="ir.actions.act_window" id="action_courses">
                <field name="name">Courses</field>
                <field name="res_model">openacademy.course</field>
                <field name="view_mode">tree,form</field>
            </record>

            <menuitem name="Courses" id="menu_courses" parent="menu_general"
                sequence="1" action="action_courses"/>
        </data>
    </openerp>


创建一个表单视图    
----------------------------

.. code-block:: xml

    <record model="ir.ui.view" id="course_form">
        <field name="name">course form view</field>
        <field name="model">openacademy.course</field>
        <field name="arch" type="xml">

            <form string="Course" version="7.0">
                <sheet>
                    <h1>
                        <field name="name" placeholder="Course Title"/>
                    </h1>
                    <notebook>
                        <page string="Description">
                            <field name="description"/>
                        </page>
                    </notebook>
                </sheet>
            </form>

        </field>
    </record>


学期模型    
----------------

.. code::

    class Session(Model):
        _name = 'openacademy.session'

        name = fields.Char(required=True)
        start_date = fields.Date()
        duration = fields.Integer(help="Duration in days")
        seats = fields.Integer(string="Number of Seats")


关系型字段
---------------

来将学期与课程和教师关联::

    class Session(Model):
        _name = 'openacademy.session'

        ...

        course = fields.Many2one('openacademy.course', required=True)
        instructor = fields.Many2one('res.partner')


.. nextslide::

将课程与学期作对应的关联::

    class Course(Model):
        _name = 'openacademy.course'

        ...

        responsible = fields.Many2one('res.users')
        sessions = fields.One2many('openacademy.session', 'course')


.. nextslide::

将学期与业务伙伴对象关联作为该开课的选课学生::

    class Session(Model):
        _name = 'openacademy.session'

        ...

        attendees = fields.Many2many('res.partner')


计算字段
------------

该字段的值是由计算获得::

    class Session(Model):
        _name = 'openacademy.session'

        ...

        taken_seats = fields.Float(compute='_compute_taken_seats')

        @api.one
        @api.depends('attendees', 'seats')
        def _compute_taken_seats(self):
            if self.seats:
                self.taken_seats = 100.0 * len(self.attendees) / self.seats
            else:
                self.taken_seats = 0.0


关于self
----------

模型的实例是**记录集**.

记录集是一个一体两面的概念:
    * 即可以是记录的集合
    * 单个记录

.. code::

    for session in self:
        print session.name
        print session.course.name

    assert self.name == self[0].name


"Onchange"函数的用法
-------------------------

当某些字段输入值时改变表单其他字段的值::

    class Session(Model):
        _name = 'openacademy.session'

        ...

        @api.onchange('course')
        def _onchange_course(self):
            if not self.name:
                self.name = self.course.name


默认值
---------

设置表单中字段的默认值::

    class Session(Model):
        _name = 'openacademy.session'

        ...

        active = fields.Boolean(default=True)
        start_date = fields.Date(default=fields.Date.today)

        ...


模型约束条件
------------------

防止错误输入的存入::

    from odoo.exceptions import Warning

    class Session(Model):
        _name = 'openacademy.session'

        ...

        @api.one
        @api.constrains('instructor', 'attendees')
        def _check_instructor(self):
            if self.instructor in self.attendees:
                raise Warning("Instructor of session '%s' "
                    "cannot attend its own session" % self.name)


更多东东
------------

* 扩展现有模型
* 更多的视图类型
* 工作流
* 报表
* 安全
* 翻译


V8的后台模块
=====================


总结
------

* Odoo模块结构简单
* 模型定义简单高效
    * 使用Python标准功能(decorators, descriptors)
    * 记录集支持“批”处理功能
    * 多种模型钩子接口（默认值，约束条件，计算字段等等）


