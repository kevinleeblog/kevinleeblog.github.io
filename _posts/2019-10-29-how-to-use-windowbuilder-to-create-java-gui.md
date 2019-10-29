---
layout: post
title: "How to use WindowBuilder to create Java GUI"
auther: Kevin Lee
category: 
tags: [Java]
subtitle:
visualworkflow: true
---

### Introduce

WindowBuilder is a plugin for eclipse. It can be easy to generate java code for gui.

### Install

Download https://www.eclipse.org/windowbuilder/download.php Here, I download the zip file

![T7OUpyI]({{site.baseurl}}/img/T7OUpyI.png)

Open Eclipse and click Help -> Install new Software All ticked and keep to install.

![qDztK71]({{site.baseurl}}/img/qDztK71.png)

### How to use

###### Create a Java Project

First, created a new java project named “demo” and click finish button.

![HzTvtrp]({{site.baseurl}}/img/HzTvtrp.png)

###### Create a JFrame

File -> New -> Select a wizard WindowBuilder -> Swing Designer -> JFrame

![Icfrrcp]({{site.baseurl}}/img/Icfrrcp.png)

Named JFrame class “GuiDemo”

![8aQl5aJ]({{site.baseurl}}/img/8aQl5aJ.png)

Switch to Design, it shows GUI design.

![nZDAVNC]({{site.baseurl}}/img/nZDAVNC.png)

###### Set layout for JFrame

Right mouse click Set layout -> Absolute layout

![2e2ZORl]({{site.baseurl}}/img/2e2ZORl.png)

First click componets: JLabel and JButton to JFrame
![jfENPFB]({{site.baseurl}}/img/jfENPFB.png)

Renew to name these componets.
![sQjRWUF]({{site.baseurl}}/img/sQjRWUF.png)

Add Event Handler for button.
![zlqpBY6]({{site.baseurl}}/img/zlqpBY6.png)

![5rRbD6O]({{site.baseurl}}/img/5rRbD6O.png)

Finally, Run it!
![Dgr987U]({{site.baseurl}}/img/Dgr987U.png)