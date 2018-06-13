---
title: Executing the right project in Visual Studio
layout: post
tags : [visual, studio, tips, tricks]

---

Often, when you're working you want to quickly open a file, edit that and run it so you can test it. What happens when you press `F5` in Visual Studio is that it will start some other project (when you have a solution with more than 1 project). In this post you'll learn a simple trick that allows you to start the project where the file you just edited is.

# The trick
You just need to go to the solution settings and choose **Current selection** as the **Startup Project** like so:

![Starting the Correct Project in Visual Studio](https://i.imgur.com/0SH9GAT.png)

Once you do that, every time you select a file, the project in which that file resides will become active so that when you press `F5` to run it will start that specific project. Notice how when you select a different file, the project name becomes **bolded** in the **Solution Explorer**.

![Demo on how it works](https://i.imgur.com/5WlLExw.gif)

Hopefully, this trick will help you at least in some cases. Of course, it won't every time (in library projects for example).
