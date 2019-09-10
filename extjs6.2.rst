Ext JS 6.2.0 学习
=================
官网文档：https://docs.sencha.com/extjs/6.2.0/index.html

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

  
