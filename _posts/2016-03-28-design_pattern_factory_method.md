---
layout: post
title: "设计模式演练——工厂方法模式"
date: 2016-03-28
---
# 设计模式演练——工厂方法模式

### 1.小C的故事

&#160; &#160; &#160; &#160;大家好，我叫张小C，我是一个面包师，专职烤面包，刚入行只会烤一些白切包啊啥的。下面是我的工作。
<pre><code>
/**
 * 程序世界
 *
 * @author bob
 *
 */
public class World {

 public static void main(String[] args) {
   LittleC littleC = new LittleC();
   littleC.work();
 }
}
/**
 * 张小C
 *
 * @author bob
 *
 */
public class LittleC {

 public void work() {
   Oven oven = new Oven();
   Bread bread = oven.cookBread();
   System.out.print(bread.getName()+"做好了！");
   System.out.println(bread.getTaste());
 }

}
</code></pre>

### 2.存在的问题
 未完待续......