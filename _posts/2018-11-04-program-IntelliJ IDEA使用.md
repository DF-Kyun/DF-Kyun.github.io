---
layout: post
category: "program"
title:  "IntelliJ IDEA使用"
tags: [program ,IntelliJ IDEA]
---

### IntelliJ IDEA使用  

**这篇博客是慕课网上视频的学习笔记，手打出来方便日后查阅**  
[参考博客地址](https://blog.csdn.net/ljxljxljx747/article/details/79085379)  

#### 无处不在的跳转  

项目之间跳转 cltrl+alt+]/[  

快捷键查找（Help下）ctrl+shift+A  

文件之间跳转 recent file Ctrl+E  

	recent changes Ctrl+shift+c   
	
	last edit location  ctrl+Q   

浏览修改位置跳转 ctrl+shift+向左箭头/向右箭头      

Navigate--->Back     Forward  

修改位置的跳转 Ctrl+Shift+Backspace   

Navigate---> Last Edit Location    Next Edit Location  

利用书签进行跳转 搜索bookmarks      

        新建书签（ctrl +shift +F11）
    	展示书签（shift +F11）
    	新建带有标记的书签（ctrl + alt +shift +F11）
        跳转到带标记的书签 ctrl+标记

Help--->Find Action  输入 内容toggle bookmark   

收藏位置和文件和方法  显示收藏 夹alt+2   

添加收藏 Help--->Find Action 输入 内容Add to Favorites  Alt+Shift+F  

字符跳转插件emacsIdea跳转   

安装插件 Help--->Find Action  Ctrl+Shift+A ,输入 内容plugins  

编辑区和文件区来回跳转 alt+1/esc  


#### 精准搜索  

类、文件(Ctrl+Shift+T、 Ctrl+Shift+R)

Navigate--->Class    File

选中include可以搜索jar包

符号  

Ctrl+Alt+Shift+N，可以搜索关键字，包括变量名、函数名和类名等。
字符串  

要搜索字符串，可以通过Ctrl+Shift+F来实现，有几个选项：  

    Match case，是否匹配大小写
    Words，是否是一个单词
    Regex，通过正则表达式搜索
    File mask，可以指定在哪个文件下搜索 
    In Project，在项目下搜索
    Module，在模块下搜索
    Directory，在目录下搜索
    Scope，在指定区域内搜索，还可以自定义搜索区域  


#### 代码助手  

##### 列操作  

光标选中相关操作  

	Shift+右箭头，选中下一个位置，按住Shift并多次按方向键选中多个位置
	Ctrl+右箭头，Move Caret to Next Word把光标移动到下一个单词
	Ctrl+Shift+右箭头，Move Caret to Next Word With Slection把光标移动到下一个单词并选中
	大小写转换，Ctrl+Shift+U，Toggle Case
	Home，Move Caret to Line Start将光标移动到本行开始的位置
	End，Move Caret to Line End将光标移动到本行最后的位置
	Ctrl+Alt+Shift+J，Select All Occurrences，选中所有相同的字符串，多光标操作
	F2，自动将光标定位到报错的位置
	Ctrl+Alt+L，Reformat Code，重新格式代码  

#### live template  

可以使用Setting下的Live Templates定义常用的代码模板，分为以下几步：  

	创建分组：点击+按钮下的Template Group，Create New Group，取组名为“方法”
	创建模板：在“方法”分组下，点击+按钮下的Live Templates创建模板。以main函数为例，下图中Abbreviation为模板名称，Description为模板注释，Template text中的内容为模板，其中$END$表示最后光标停留位置，最后要点击define，选择Java表示这个模板定义为Java语言的模板  

#### postfix
```java
for (int i = 0; i < 100; i++) {
}
```

上面的for循环，只需输入100.fori按回车就可以出来  
可以在Settings下的Postfix Completion查看有哪些后缀实现   
常用的Postfix： 

	fori：for循环，例子：100.fori  
	sout：打印输出，例子：new Date().sout  
	field：引入表达式字段，例子：    

#### Alt+Enter  

自动创建函数  

```java
public static void main(String[] args) {
        f1();//Alt+Enter就会自动创建出下面的函数
}

private static void f1() {
}
```

List Replace  

```java
for (int i = 0; i < list.size(); i++) {
    String item = list.get(i);
}
//选中for循环按住alt+enter出现Replace with foreach
for (String item : list) {
}
```

实现接口  

单词拼写  

```java
System.out.println("usernmea is jone");//username拼写错误，单词下会出现波浪线
//按下alt+enter，出现Typo Change to...
//Intellij会给出一些建议的单词
```

导包  

### 编写高质量代码  

#### 重构  

重构变量，Shift+F6，Rename
重构函数
​	Ctrl+F6，Change Signature，添加参数
​	先在调用的地方添加实参，Alt+Enter

#### 抽取  

抽取变量，Ctrl+Alt+V。Refactor->Extract->Variable  

```java
//抽取前
System.out.println("sun");
System.out.println("sun");
System.out.println("sun");
//抽取后
String sun = "sun";
System.out.println(sun);
System.out.println(sun);
System.out.println(sun);
```

抽取成静态常量，Ctrl+Alt+C  

```java
public static final String SUN = "sun";

public static void main(String[] args) {
    System.out.println(SUN);
    System.out.println(SUN);
    System.out.println(SUN);
}
```

抽取成员变量，Ctrl+Alt+F  

```java
private static String sun;

public static void main(String[] args) {
    sun = "sun";
    System.out.println(sun);
    System.out.println(sun);
    System.out.println(sun);
}
```

抽取函数，Ctrl+Alt+M



