Ext JS 6.2.0 学习
=================
官网文档：https://docs.sencha.com/extjs/6.2.0/index.html

Ext.dom.Element 和 Ext.Component的区别。
---------------------------------------
Ext.dom.Element对象是对dom对象的封装，目的是为了跨平台以及增加一些有用的方法。但是Ext.com.Element是不包含外观的，封装的dom原来是怎么样就是怎么样。A	Q`Ext在Element的基础上进一步封装，产生了Component类，这些类含有外观，也就是多加入了一些html之类的进去，更方便开发者使用。 

因此在Component中可以通过el属性来访问该Component所依赖的Element，同样的，Element也可以通过dom属性来访问Element对象所依赖的dom。

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
--------------
* Ext.query ( selector, [asDom] ) : HTMLElement[]/Ext.dom.Element[]
   Ext.dom.Element.query 的简写。通过CSS选择器选择子节点。
 
   参数：
     * selector CSS选择器
     * asDom 缺省为 true，返回 HTMLElement[]，false 返回 Ext.dom.Element[]
* Ext.fly ( dom, [named] ) : Ext.dom.Element
   Ext.dom.Element.fly 的别名。通过id查询，id值不能加"#"号．下同．

* Ext.get  ( el ) : Ext.dom.Element
   Ext.dom.Element.get 的别名。此方法不返回ext.component。此方法返回封装dom元素的ext.dom.element对象。通过组件的ID返回Component，请使用ext.componentmanager.get。

* Ext.ComponentQuery
   提供在ext.componentmanager（全局）或文档指定的ext.container.container中搜索组件的功能，其语法与css选择器类似。返回匹配组件的数组或空数组。

   * 按xtype匹配，匹配继承的类型，因此在下面的代码中，将找到继承自textfield的任何类型的前一个字段．
     ::
    　 prevField = myField.previousNode('textfield');
   * 只匹配确切的类型，通过向xType添加（true）．
     ::
       prevTextField = myField.previousNode('textfield(true)');
   * 可以按组件的id或itemid属性搜索组件，有前缀＂＃＂．
   * 组件xtype和id或itemid可以一起使用，以避免不同类型的组件之间可能的id冲突．
     ::
       Ext.ComponentQuery.query('panel#myPanel');
   * Ext.ComponentQuery.query ( selector, [root] ) : Ext.Component[]

     使用类似CSS选择器查询DOM的方式过滤返回组件数组．
