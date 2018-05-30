---
layout: post
title: 瞎玩物联网系列--Arduino连接LCD1602显示屏
description: 用Arduino控制1602显示屏。
categories: iot
tags: 
---

### 简介

> LCD1602是一种工业字符型液晶，能够同时显示16x02即32个字符。LCD1602液晶显示的原理是利用液晶的物理特性，通过电压对其显示区域进行控制，即可以显示出图形。【[百度百科](https://baike.baidu.com/item/LCD1602)】

#### 引脚说明

引脚  |符号  |说明
-----|------|---
1    |GND   |接地
2    |VCC   |5V正极
3    |V0    |对比度调整，接正极时对比度最弱
4    |RS    |寄存器选择，1数据寄存器（DR），0指令寄存器（IR）
5    |R/W   |读写选择，1度，0写
6    |EN    |使能(enable)端，高电平读取信息，负跳变时执行指令
7~14 |D0~D7 |8位双向数据
15   |BLA   |背光正极
16   |BLK   |背光负极

#### 其它知识点

- 一些简称（本文无用，瞎记）

```
DR      数据寄存器
IR      指令寄存器
DDRAM   显示数据存储器（LCD1602有80字节）
CGROM   字符发生器（内建192个5*7点阵字符）
```

- 3脚电位器

一个滑动变阻器，中间接负极(输出)，两边分别接电源正极和接地(或不接)

![arduino-lcd-01](/images/2018/iot/arduino-lcd-01.jpg)

### 材料

- 大面包板          x1
- 3脚电位器         x1
- LCD 1602        x1
- Arduino UNO     x1

### 接线示意图

![arduino-lcd](/images/2018/iot/arduino-lcd.png)

LCD1602 | --->   | Arduino UNO   | 说明
--------|:------:|---------------|-------
GND     | --->   | GND           |接地
VCC     | --->   | 5V            |5V电源
V0      | --->   |               |连接3脚继电器中间，用于调节对比度
RS      | --->   | 3             |随便接一个输出口，方便接线、画图
R/W     | --->   | GND           |接地，写模式
EN      | --->   | 5             |随便接一个输出口，方便接线、画图
D0~D3   | --->   |               |4位工作模式，不使用
D4~D7   | --->   | 10~13         |其它口也行，方便接线、画图
BLA     | --->   |               |背光，电源正极，可选
BLK     | --->   |               |背光，接地，可选

### 开始抄代码

#### 加载库文件

打开Arduino IDE，选项目 -> 加载库 -> 管理库中搜索LiquidCrystal，然后安装即可，笔者的IDE版本为1.6.12，自带该库。

![arduino-lcd-02](/images/2018/iot/arduino-lcd-02.png)

#### 示例代码, hello word

```java
//引入依赖
#include <LiquidCrystal.h>

// 初始化针脚
const int rs = 3, en = 5, d4 = 10, d5 = 11, d6 = 12 d7 = 13;
LiquidCrystal lcd(rs, en, d4, d5, d6, d7);

void setup() {
    //设置LCD要显示的列数、行数，即2行16列
    lcd.begin(16, 2);
    
    //输出Hello World
    lcd.print("hello, world!");
}

void loop() {
    //设置光标定位到第0列，第1行（从0开始）
    lcd.setCursor(0, 1);
    //打印从重置后的秒数
    lcd.print( millis() / 1000);
}
```

#### 示例代码，自动滚屏

```java
#include <LiquidCrystal.h>
const int rs = 3, en = 5, d4 = 10, d5 = 11, d6 = 12 d7 = 13;
LiquidCrystal lcd(rs, en, d4, d5, d6, d7);

void setup() {
  lcd.begin(16, 2);
}

void loop() {

    //在0列0行显示0~9，间隔500ms变化
    lcd.setCursor(0, 0);
    for (int thisChar = 0; thisChar < 10; thisChar++) {
        lcd.print(thisChar);
        delay(500);
    }

    //设置自动滚屏
    lcd.autoscroll();

    //在16列1行显示0~9，间隔500ms变化
    lcd.setCursor(16, 1);
    for (int thisChar = 0; thisChar < 10; thisChar++) {
        lcd.print(thisChar);
        delay(500);
    }
    //关闭自动滚屏
    lcd.noAutoscroll();

    //未下重循环清屏
    lcd.clear();
}
```

### 参考文章

- [LCD1602_百度百科](https://baike.baidu.com/item/LCD1602)
- [三脚电位器接法图解 - 电工基础 电工论坛](http://www.diangon.com/m395481.html)
- [Arduino - LiquidCrystal](https://www.arduino.cc/en/Reference/LiquidCrystal)