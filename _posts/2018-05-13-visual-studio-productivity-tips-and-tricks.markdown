---
title: Visual Studio Productivity Tips & Tricks
layout: post
tags : [vs, visual, studio, productivity, tips, tricks]

---

You can boost your Visual Studio productivity if you know these little **tips and tricks**. We'll go through some of the most useful **code snippets**, **keyboard shortcuts** and **extensions**.

## Code snippets

Code snippets are a way to **write frequent code faster**. This is achieved by typing a mnemonic followed by `TAB TAB` in order for it to be replaced with the final code.

### 1. `svm` - `static void main`
When you start a new project you obviously want to write the `main` method first. In order to do so faster you can use the `svm` snippet. After you write `svm` press `TAB TAB` and the snippet will be replaced with the `main` method:

![VS snippet: svm](https://i.imgur.com/lhduOMd.gif)

### 2. `cw` - `Console.WriteLine`
Normally you would be making GUI programs so why should you care about being able to write to the command line faster?!

Well, if you're like me, when you encounter a problem that it's more difficult you want to simplify it as much as possible. One way of doing this is to create a separate command line project to isolate the problem to the most essential parts. In that case, if you want to **quickly display a message** you can use the `cw` snippet (as before - followed by `TAB TAB`):

![VS snippet: cw](https://i.imgur.com/bZPK7rI.gif)

### 3. `mbox` - `MessageBox.Show`
In the same style as the previous snippet, this one helps us when we want to show a quick message to the user. Normally the `MessageBox` is a bit old and not so user friendly, but it works quite well when debugging.

One quick note, though, if you're using Windows Forms then you can leave the namespace untouched, but if you're using WPF you have 2 options: either add the `System.Windows.Forms` reference or just **delete the** `Forms` **part from the inserted line**. I prefer the second options (also demoed below) because you keep **using WPF without needing to depend also on Windows Forms**:

![VS snippet: mbox](https://i.imgur.com/S0QbfGf.gif)

### 4. `prop` & `propfull` - C# Properties
Another thing that's quite often and useful is **adding C# properties quickly**. After the snippet is inserted, you can continue pressing `TAB` to cycle between the type and the name. If you need also the property backing field you should use `propfull`.

![VS snippet: prop](https://i.imgur.com/aW2CqlR.gif)

### 5. `propdp` & `propa` - Dependency Properties & Attached Properties
When creating a new user control you often need to write new **dependency properties**. Unfortunately, the syntax for doing so is quite complex and difficult to get right the first time. In the same style there's also `propa` that inserts an **attached property**.

![VS snippet: propdp](https://i.imgur.com/ucypHKY.gif)


## Keyboard shortcuts

Keyboard shortcuts are by far the **quickest way to improve your productivity**. In the next lines I'll show the shortcuts that I consider critical.

### The most potent shortcuts
- quick fix: `CTRL + .`
- search commands: `CTRL + Q`
- search type: `CTRL + T`

### Navigation shortcuts
- navigate to definition: `F12`
- find references: `SHIFT + F12`
- navigate backward: `CTRL + -`
- navigate foreward: `CTRL + SHIFT + -`

### Code handling
- format code: `CTRL + K,D`
- comment code: `CTRL + K,C`
- uncomment code: `CTRL + K,U`
- move line up: `ALT + ↑`
- move line down: `ALT + ↓`
- rename: `F2`

### Personal favorite: quick fix
The most amazing shortcut has to be the *quick fix*! If you have an error, a warning or a code suggestion you just have to press `CTRL + .` to get a list of quick fixes and choose the one you want. It's the shortcut that can fix almost any problem.

In the example below I used [programming by intention](http://wiki.c2.com/?ProgrammingByIntention) to first call the method I need and then using quick fix to add a placeholder for the method thus saving my time I would have spent writing it. I know, I know, it's so small it doesn't matter... but it adds up after a while.

![VS keyboard shortcut: quick fix](https://i.imgur.com/EanCj5Z.gif)

### Personal favorite: expand/contract selection
- expand selection: `ALT + SHIFT + =`
- contract selection: `ALT + SHIFT + -`

This shortcut is very useful when you want to refactor code. Let's say that you want to extract a method for the conditional expression in an `if` statement. If you move your caret (using the keyboard) you could press `ALT + SHIFT + =` until you select the expression you want and then press `CTRL + R,M` to extract a method. Here's how the selection works:

![VS keyboard shortcut: expand/contract selection](https://i.imgur.com/ddTTjGI.gif)

Just a **quick warning** though: this was added in Visual Studio 2017 15.5.2 so in previous versions you won't be able to do this (although this functionality might be supported through an extension).

## Extensions
Extensions are the last pillar - they offer stuff that's not built-in Visual Studio. **WARNING:** Please be aware that **too many extensions will slow down Visual Studio**.

Here are the extensions that I consider most useful:
- [Productivity Power Tools 2017](https://marketplace.visualstudio.com/items?itemName=VisualStudioProductTeam.ProductivityPowerPack2017)
- [Roslynator (refactoring)](https://marketplace.visualstudio.com/items?itemName=josefpihrt.Roslynator2017)
- [Refactoring Essentials for Visual Studio](https://marketplace.visualstudio.com/items?itemName=SharpDevelopTeam.RefactoringEssentialsforVisualStudio) ([official website](http://vsrefactoringessentials.com/))
- [Developer Assistant](https://marketplace.visualstudio.com/items?itemName=OneCodeTeam.DeveloperAssistant)
- [Hide Main Menu](https://marketplace.visualstudio.com/items?itemName=MatthewJohnsonMSFT.HideMainMenu)
- [Color Picker](https://marketplace.visualstudio.com/items?itemName=ShemeerNS.ColorPicker)
- [Viasfora (colors matching parenthesis)](https://marketplace.visualstudio.com/items?itemName=TomasRestrepo.Viasfora)
- [Spell Checker](https://marketplace.visualstudio.com/items?itemName=EWoodruff.VisualStudioSpellCheckerVS2017andLater)
- [VSColorOutput](https://marketplace.visualstudio.com/items?itemName=MikeWard-AnnArbor.VSColorOutput)
- [SonarLint](https://marketplace.visualstudio.com/items?itemName=SonarSource.SonarLintforVisualStudio2017)
- [Visual Studio IntelliCode](https://marketplace.visualstudio.com/items?itemName=VisualStudioExptTeam.VSIntelliCode)

Hopefully all these **tips and tricks** will help you **improve your productivity**.
