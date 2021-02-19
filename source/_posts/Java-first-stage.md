---
title: Java初步体验及问题小结
date: 2017-02-28 23:23:23
categories: Java
tags: Java
---
![Hexo本地生成后的错误](http://img.jackchen.cn/java-first-stage.jpg)
## 前言
>从去年架设这个网站之后不久便开始了Java的学习。可以说学的挺艰辛的~ 大多数时间感觉上都是在程序的逻辑里迷失方向！当然，也有突然豁然开朗的时候（比较少）~ 但是不管怎么说还是坚持了下来。其实想想也不难，只是你练习和经历的时间还没有足够多。

## 开始
开始觉得时间再也不够用了，因为要一边权衡工作和家庭，一边整合碎片时间来看和练习！在没有任何后端语言基础的状态下，花了大把时间啃掉了大部头《Head First Java》。同时在慕课网上跟着Java工程师的路径学完了第一、二、三季。大多都是语法和模块化的概念和方法论。今天记录下前两天敲完理解完的小程序，分享给和我一样的小白同学们。
<!--more-->
## 第一个案例 —— 随机生成10条不重复长度为10以内的字符串，进行排序

设计要求：
> * 创建完List<String>之后，往里添加10条数据
> * 每条字符串的长度为10以内的随机整数
> * 每条的字符串的字符为随机生成的字符，字符可重复
> * 每条字符串不可重复

完整代码如下，如果问题欢迎大家指正 →

```java
import java.util.ArrayList;  
import java.util.Collections;  
import java.util.List;  
import java.util.Random;  

public class RandomStringSort {

	public void stringSort() {
		List<String> stringList = new ArrayList<String>();
		stringList = randomString();
		System.out.println("-------排序前-------");
		for(String string:stringList) {
			System.out.println("字符串为：" + string);
		}
		Collections.sort(stringList);
		System.out.println("------排序后--------");
		for(String string:stringList) {
			System.out.println("字符串为：" + string);
		}
	}

	public List<String> randomString() {
		List<String> tempList = new ArrayList<String>();
		String base = "abcdefghijklmnopqrstuvwxyz1234567890";
		//添加十条随机字符串
		for(int i=0;i<10;i++) {
			Random random = new Random();
			StringBuffer sb = new StringBuffer();
			//长度为十以内的字符
			int stringLength = random.nextInt(10) + 1;
			do{
				for(int j=0; j<stringLength; j++) {
					int number = random.nextInt(base.length());
					sb.append(base.charAt(number));
				}
			} while(tempList.contains(sb.toString()));
			tempList.add(sb.toString());
		}
		return tempList;
	}


	public static void main(String[] args) {
		// TODO Auto-generated method stub
		RandomStringSort st = new RandomStringSort();
		st.stringSort();
	}

}

```

## 第二个案例 —— 简易的扑克牌比点数游戏

设计要求：
> * 创建一副扑克牌（没有大小王）
> * 创建玩家（玩家有名字，手牌）
> * 洗牌（打乱扑克牌的顺序）
> * 发牌（每人发两张，一人一张来）
> * 比牌（取手中最大点数的牌与另外一个玩家比较，点数大赢，如果点数相等，花色大赢）

完整代码如下，如果问题欢迎大家指正 →

` People类 `
```java
import java.util.ArrayList;
import java.util.List;

/**
 * 创建玩家对象
 * @author Jackchen
 *
 */

public class People {

	public String name;
	public List<String> cards;

	public People(String name) {
		this.name = name;
		this.cards = new ArrayList<String>();
	}
}
```
` CardsGame类 `
```java
import java.util.ArrayList;
import java.util.Collections;
import java.util.List;
import java.util.Scanner;

/**
 * 游戏主程序
 * @author Jackchen
 *
 */

public class CardsGame {
	// 罗列出所有的牌面（除大小王）
	String[] datasList = {
			"方块1","方块2","方块3","方块4","方块5","方块6","方块7","方块8","方块9","方块10","方块J","方块Q","方块K",
			"梅花1","梅花2","梅花3","梅花4","梅花5","梅花6","梅花7","梅花8","梅花9","梅花10","梅花J","梅花Q","梅花K",
			"红桃1","红桃2","红桃3","红桃4","红桃5","红桃6","红桃7","红桃8","红桃9","红桃10","红桃J","红桃Q","红桃K",
			"黑桃1","黑桃2","黑桃3","黑桃4","黑桃5","黑桃6","黑桃7","黑桃8","黑桃9","黑桃10","黑桃J","黑桃Q","黑桃K"
	};

	//一副扑克牌
	List<String> cardsList = new ArrayList<String>();

	//花色
	String[] suitList = {"方块","梅花","红桃","黑桃"};

	//玩家列表
	static List<People> playerList = new ArrayList<People>();

	//创建一副扑克牌，并输出每张牌
	public void createCards() {
		for(int i = 0; i<datasList.length; i++) {
			cardsList.add(datasList[i]);
		}
		System.out.println("创建扑克牌完成");
		System.out.println("扑克牌中有：");
		for(String string:cardsList) {
			System.out.print(string + " ");
		}
	}

	//开始洗牌，打乱顺序
	public void shuffle() {
		Collections.shuffle(cardsList);
		System.out.println();
		System.out.println("洗牌完成");
		System.out.println("扑克牌里有：");
		for(String string:cardsList) {
			System.out.print(string + " ");
		}
	}
	//开始发牌，每人两张牌
	public void dealCards() {
		System.out.println();
		System.out.println("请输入两位玩家的名字：");
		Scanner scanner = new Scanner(System.in);
		for(int i = 0; i < 2; i++) {
			System.out.println("请输入第" + (i+1) + "个玩家的名字");
			String name = scanner.next();
			People player = new People(name);
			playerList.add(player);
		}
		System.out.println("玩家分别是：");
		for(People people : playerList) {
			System.out.println(people.name);
		}
		System.out.println("-----------------------------");
		//发牌
		for(int i=0; i<4; i++) {
			String tempCard = cardsList.get(i);
			if(i%2 == 0) {
				playerList.get(0).cards.add(tempCard);
			} else{
				playerList.get(1).cards.add(tempCard);
			}
		}
		//亮牌
		System.out.println("亮牌如下：");
		for(People people:playerList) {
			System.out.println("玩家：" + people.name + "玩家的手牌是：" + people.cards);
		}
	}
	//比牌面大小定胜负
	public void whoWin(List<People> playerList) {
		//玩家
		People player1 = playerList.get(0);
		People player2 = playerList.get(1);
		//玩家1的两张牌
		String card1ForPlayer1 = player1.cards.get(0);
		String card2ForPlayer1 = player1.cards.get(1);
		//玩家1最大的牌
		String bigCardForPlayer1 = getBigCard(card1ForPlayer1,card2ForPlayer1);
		System.out.println("--------------------");
		System.out.println(player1.name + "最大的牌是：" + bigCardForPlayer1);
		//玩家2的两张牌
		String card1ForPlayer2 = player2.cards.get(0);
		String card2ForPlayer2 = player2.cards.get(1);
		//玩家2最大的牌
		String bigCardForPlayer2 = getBigCard(card1ForPlayer2,card2ForPlayer2);
		System.out.println(player2.name + "最大的牌是：" + bigCardForPlayer2);
		System.out.println("--------------------");
		//玩家中最大的牌为
		String bigCard = getBigCard(bigCardForPlayer1,bigCardForPlayer2);
		if(bigCardForPlayer1.equals(bigCard)) {
			System.out.println("获胜的是：" + player1.name);
		} else if(bigCardForPlayer2.equals(bigCard)) {
			System.out.println("获胜的是：" + player2.name);
		}
	}
	//比较两张牌面的大小
	public String getBigCard(String card1, String card2) {
		//最大的牌面
		String bigCard = null;
		//取得每张牌的数字，因为牌面有10，所以从第二位往后截取字符串
		String num1 = card1.substring(2);
		String num2 = card2.substring(2);
		//取得每张牌的花色
		String suit1 = card1.substring(0,2);
		String suit2 = card2.substring(0,2);
		//两张牌的花色大小
		int level1 = 0;
		int level2 = 0;
		//输出每张牌的数字和花色
		//System.out.println("数字分别为：" + num1 + " " + num2);
		//System.out.println("花色分别为" + suit1 + " " + suit2);
		if (num1.compareTo(num2) > 0) {
		//System.out.println("第一张牌大");
			bigCard = card1;
		} else if(num1.compareTo(num2) == 0) {
		//System.out.println("两张牌一样大");
		//数字相同，比较花色
			for(int i=0; i<suitList.length; i++) {
				if(suit1.equals(suitList[i])) {
    		  //输出花色在数组下的下标，来决定哪个花色大
    		  //System.out.println(i);
					level1 = i;
				}
				if(suit2.equals(suitList[i])) {
          //输出花色在数组下的下标，来决定哪个花色大
					System.out.println(i);
					level2 = i;
				}
			}
			if(level1 > level2) {
        //System.out.println(card1);
				bigCard = card1;
			} else {
        //System.out.println(card2);
				bigCard = card2;
			}
		} else if (num1.compareTo(num2) < 0) {
      //System.out.println("第二张牌大");
			bigCard = card2;
		}
		return bigCard;
	}

	public static void main(String[] args) {
		// TODO Auto-generated method stub
		CardsGame st = new CardsGame();
		st.createCards();
		st.shuffle();
		st.dealCards();
		st.whoWin(playerList);
	}
}
```
## JAVA中字符串函数subString的用法小结
>在第二个案例中我们会用到`subString字符串函数`，在这里我们总结了一下

str＝str.substring(int beginIndex);截取掉str从首字母起长度为beginIndex的字符串，将剩余字符串赋值给str；

str＝str.substring(int beginIndex，int endIndex);截取str中从beginIndex开始至endIndex结束时的字符串，并将其赋值给str;

Demo如下:

```java
class Test
{
 public static void main(String[] args)
 {
  String s1 ="1234567890abcdefgh";
  s1 = s1.substring(10);
  System.out.println(s1);
 }
}
```
运行结果：abcdefgh


```java
class Test
{
 public static void main(String[] args)
 {
  String s1 ="1234567890abcdefgh";
  s1 = s1.substring(0,9);
  System.out.println(s1);
 }
}
```
运行结果：123456789


## 总结
程序的学习还是要多练习，必须要保持每天都能敲到一定量级的代码才可以。还要自己找一些实用的小程序练习，一遍不行就再来一遍，要学会刻意练习。每次练习结束了要思考代码后面的逻辑。其实编程语言知识个工具，重要的是程序背后的逻辑。
