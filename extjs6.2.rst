Ext JS 6.2.0 学习
=================
官网文档：https://docs.sencha.com/extjs/6.2.0/index.html

xtype 与 alias
----------------------
* xtype 在定义一个类的时候，如果我们指定了它的xtype，那么我们就可以通过这个xtype，而不是类型的全名来创建这些类型。
  ::
    Ext.define('MyApp.PressMeButton', {
        extend: 'Ext.button.Button',
        xtype: 'mybutton',
        text: 'Press Me'
    });
      ......
    items: [
        {
            xtype: 'mybutton',
            fieldLabel: 'Foo'
        },
        ......
    ]
* alias 别名由名称空间和名称通过句号连接组成．像这样：<namespace>.<name>．

  * namespace 描述这是哪种别名，并且必须全部小写．
  * name 允许通过别名进行延迟实例化的别名的名称。名称不应包含任何句号．
  ::

    Ext.define('MyApp.CoolPanel', {
        extend: 'Ext.panel.Panel',
        alias: ['widget.coolpanel'],
        title: 'Yeah!'
    });
    .......
    Ext.widget('panel', {
        items: [
            {xtype: 'coolpanel', html: 'Foo'},
            {xtype: 'coolpanel', html: 'Bar'}
        ]
    });
* xtype和alias实际上可以在一定程度上通用的．xtype是存在于widget命名空间下的alias。如果为一个新的UI组件声明了它的xtype，那么就等于在widget命名空间下为其声明了一个alias。我们也可以通过下面的代码定义刚刚看到的CoolPanel类．alias所能覆盖的类型范围要比xtype广得多。一个alias不仅仅可以用来声明命名空间为widget的各个类型，更可以在plugin，proxy，layout等众多命名空间下声明类型。在ExtJS的内部实现中常常需要将alias中的命名空间剥离才能得到所需要创建的widget类型。在xtype的帮助下，ExtJS可以摆脱掉耗时的字符串分析工作，从而提高性能。因此在定义一个自定义widget的时候，我们应当使用xtype，而在定义其它组件时，我们就不得不使用alias了。由于所有的widget使用同一个命名空间，因此我们建议需要在为自定义widget指定xtype时标识一个前缀，
  ::
    Ext.define('MyApp.CoolPanel', {
        extend: 'Ext.panel.Panel',
        xtype: ‘coolpanel’,
        title: 'Yeah!'
    });

ExtJS 的选择器
-------------
* Ext.query ( selector, [asDom] ) : HTMLElement[]/Ext.dom.Element[]
   Ext.dom.Element.query 的简写。通过CSS选择器选择子节点。
 
   参数：
     * selector CSS选择器
     * asDom 缺省为 true，返回 HTMLElement[]，false 返回 Ext.dom.Element[]
* Ext.fly ( dom, [named] ) : Ext.dom.Element
   Ext.dom.Element.fly 的别名。

* Ext.get  ( el ) : Ext.dom.Element
   Ext.dom.Element.get 的别名。此方法不返回ext.component。此方法返回封装dom元素的ext.dom.element对象。通过组件的ID返回Component，请使用ext.componentmanager.get。
