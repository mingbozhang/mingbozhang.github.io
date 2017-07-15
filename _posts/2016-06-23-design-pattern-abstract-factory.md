---
layout: post
title: "设计模式演练——抽象工厂模式"
date: 2016-06-23
---

{% include toc.html %}  


### 1.小C的故事

&#160; &#160; &#160; &#160;下面讲述的是关于x星系喵星和汪星两个宿敌星球之间的故事。首先把镜头聚焦到喵星，它是主角登场的地方。（本故事纯属虚构，如有雷同，纯属巧合）   
&#160; &#160; &#160; &#160;喵星纪元9035年，汪星精锐舰队聚集在近喵星轨道，企图一举拿下喵星。大部分喵星人已经被转移到防空工事中。喵星国王下令出动最强战力迎击来敌。喵小c是战斗机编队到王牌飞行员，参加了上百场战斗，获得了很多荣誉，他是本次先头部队的一员。   
&#160; &#160; &#160; &#160;“装载初级武器，装载初级防御罩，装载初级逃生仓。准备完毕，出发！”，航空港的扩音机发出急促的声音。伴随着引擎的轰鸣，战斗机编队已经离开跑道，朝汪星舰队飞去。下面是当前的源代码：

这个是战斗机的代码，喵小c就坐在飞机里，所以代码就省了。
{% highlight java %}    

/**
 * 战斗机
 * @author bob
 *
 */
public class Battleplane {
	/**
	 * 初级逃生舱
	 */
	private PrimaryEscapeCompartment primaryEscapeCompartment;
	
	/**
	 * 初级防御罩
	 */
	private PrimaryShield primaryShield;
	
	/**
	 * 初级武器
	 */
	private PrimaryWeapon primaryWeapon;

	public PrimaryEscapeCompartment getPrimaryEscapeCompartment() {
		return primaryEscapeCompartment;
	}

	public void setPrimaryEscapeCompartment(PrimaryEscapeCompartment primaryEscapeCompartment) {
		this.primaryEscapeCompartment = primaryEscapeCompartment;
	}

	public PrimaryShield getPrimaryShield() {
		return primaryShield;
	}

	public void setPrimaryShield(PrimaryShield primaryShield) {
		this.primaryShield = primaryShield;
	}

	public PrimaryWeapon getPrimaryWeapon() {
		return primaryWeapon;
	}

	public void setPrimaryWeapon(PrimaryWeapon primaryWeapon) {
		this.primaryWeapon = primaryWeapon;
	}
	
}


{% endhighlight %}   

下面是战斗机需要装备的设备
{% highlight java %}   
/**
 * 初级逃生舱
 * 
 * @author bob
 *
 */
public class PrimaryEscapeCompartment {

	public String description() {
		return "逃生能力一般";
	}
}

/**
 * 初级防护罩
 * 
 * @author bob
 *
 */
public class PrimaryShield {

	public String protect(){
		return "防御了一次攻击，但是防御罩受损";
	}
}

/**
 * 初级武器
 * @author bob
 *
 */
public class PrimaryWeapon {
	
	public String shoot(){
		return "突突突";
	}
	
}

{% endhighlight %}    

下面是战斗机起飞的机场
{% highlight java %}  
 package com.bob.designpatterns.abstractfactory.stepone;

/**
 * 机场
 * @author bob
 *
 */
public class Airport {
	
	public static void main(String[] args) {
		//起飞前的准备
		Battleplane battleplane = new Battleplane();
		prepareToFly(battleplane);
		
		//起飞
	}
	
	/**
	 * 战斗机起飞前的准备工作
	 * @param battleplane
	 */
	public static void prepareToFly(Battleplane battleplane){
		battleplane.setPrimaryEscapeCompartment(new PrimaryEscapeCompartment());
		battleplane.setPrimaryShield(new PrimaryShield());
		battleplane.setPrimaryWeapon(new PrimaryWeapon());
	}
	

}

{% endhighlight %}   

&#160; &#160; &#160; &#160;像大部分的剧情一样，喵星的首轮自卫反击战以失败落幕，喵小c顶着主角光环成功飞回。  
&#160; &#160; &#160; &#160;"中级战斗装备研制完成，马上装备到战斗机上了"，战斗后勤部得到了上级通知，同时愁上心头....WHY?

### 2.存在的问题
&#160; &#160; &#160; &#160;请看Airport类中prepareToFly中负责产生战斗装备（直接new了），紧密得耦合在一起。另外请看Battleplane类中也与初级的装备耦合严重。除了耦合问题以外战斗装备分为多个部分，替换的时候容易漏替，要知道初级战斗装备和中级战斗装备之间不能混搭，一旦搭错将会出现不兼容的情况，给战斗机带来额外风险。所以要解决两个问题：1.保证灵活得升级现有装备  2.要按照级别统一升级，不能有部分升级的情况。
 
### 3.抽象工厂模式登场   
&#160; &#160; &#160; &#160;抽象工厂模式的意图：提供一个创建一系列相关或者相互依赖的对象的接口，而无需指定他们具体的类。 听起来略微有些抽象，我们继续讲故事。      

&#160; &#160; &#160; &#160;让我们将时间回退到1年前的喵星。改变历史，解决1年后的困境。我门重新设计一下：

经过酝酿，新的设计出现了：
<img src="/img/in-post/design_pattern/abstract_factory/abstract_factory.png">
抽象工厂模式常见的实现方式是使用工厂方法模式实现，也可以使用原型模式实现。本文中的实现方式为前者。关于原型模式的实现方式，下一篇写完原型模式后会补充上来。

下面只给出Airport部分代码，完整代码在https://github.com/mingbozhang/designpattern   

{% highlight java %}
package com.bob.designpatterns.abstractfactory.steptwo;

/**
 * 机场
 * 
 * @author bob
 *
 */
public class Airport {

	/**
	 * 战斗准备系统
	 */
	private static FightPrepareSys fightPrepareSys;

	public static void setFightPrepareSys(FightPrepareSys fightPrepareSys) {
		Airport.fightPrepareSys = fightPrepareSys;
	}

	public static void main(String[] args) {
		// 设置初级战斗准备系统
		// Airport.setFightPrepareSys(new PrimaryFightPrepareSys());
		// 设置中级战斗准备系统
		Airport.setFightPrepareSys(new MidFightPrepareSys());

		// 起飞前的准备
		Battleplane battleplane = new Battleplane();
		prepareToFly(battleplane);

		// 起飞
	}

	/**
	 * 战斗机起飞前的准备工作
	 * 
	 * @param battleplane
	 */
	public static void prepareToFly(Battleplane battleplane) {
		battleplane.setEscapeCompartment(fightPrepareSys.getEscapeCompartment());
		battleplane.setShield(fightPrepareSys.getShield());
		battleplane.setWeapon(fightPrepareSys.getWeapon());
	}

}

{% endhighlight %} 


### 4.解决了什么，失去了什么
&#160; &#160; &#160; &#160;引入抽象工厂后，对于多个系列的产品（对应故事中战斗装备各系列）更容易管理，限制取得产品与同系列产品版本风格保持一致。使创建产品与使用者解藕，根据抽象类进行交互。   
&#160; &#160; &#160; &#160;如果想要新增加一个产品的创建，且同时保证使用者的简洁实用，则比较困难。

### 5.具体应用场景
&#160; &#160; &#160; &#160;如果一个系统需要对应多个系列的产品则可以使用抽象工厂模式。举个《设计模式》书中的例子：当制作一款编辑器界面包含多种风格，界面元素包括各个控件，如：按钮、滚动条等。可以创建一个抽象的WidgetFactory包含创建各种控件的抽象方法。具体风格再实现Style1WidgetFactory、Style2WidgetFactory，生产各自风格系列的具体控件。这样就达到风格可以灵活扩展，且切换风格的时候非常简单，整个系统中只有在初始化具体工厂的时候需要改动。

### 6.参考
《设计模式－可复用面向对象软件的基础》 