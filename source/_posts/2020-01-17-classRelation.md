---
layout: post
title: IDEA查看类关系图
description: 如何使用idea查看类关系图
category: blog
---

## idea-mac-查看类关系图
1. 看继承关系
            快捷键 control + h查看hierarchy,只能查看向上向下继承关系，而不能看实现了哪些接口。
            右键选择Diagrams（也可以使用快捷键command+alt+u，更快捷），然后显示
2. 看接口的实现关系
            command + alt + B会显示出跟这个接口有关系的类。 然后加command+all，鼠标弄开，然后回车，全出来

            然后把需要的拖过来，形成树状图。(我暂时还没找到快捷的方法)
            鼠标放在某个类上，command+enter，直接进看这个类
            
            蓝色的实线是继承关系
            
            白色虚线表示这种关系public abstract class AbstractOrder<T extends BaseOrderResultVO> {  ----》抽象类+泛型
3. 选中类，然后command+option+shift+u
       
