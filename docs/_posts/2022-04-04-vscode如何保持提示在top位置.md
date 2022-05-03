---
layout: post
title:  "vscode如何保持提示在top位置"
date:   2022-04-04 21:17:38 +0800
categories: vscode
---
参考[stackoverflow提供的方法](https://stackoverflow.com/questions/43857693/visual-studio-code-order-in-autocompletion)

问题：  

I recently switched to Visual Studio Code and I love it! It starts so quickly and I just enjoy the open source environment more than Visual Studio. But there's one problem that I have encountered that bothers me more than it should.
Before I could if I wanted to autocomplete the syntax for an if statement I'd just be able to type in "if" and touble tap tab, but now the autocompletion IntelliSense things pop up in the wrong order:
The red box is the wrong one that gets displayed first and the green box is the one I want at the top. My question is if there's a way to configure it so that I get that statement at the top. It's the same with for-loops, foreach-loops and pretty much every other autocompletion that I want to use.


![](/assets/vscode如何保持提示在top位置/1.png)

解答：

1. Create snippet (or edit) ctrl+shift+p Preferences: Configure User Snippets
2. settings (ctrl+,): "editor.snippetSuggestions": "top",