---
title: JShell_User_Guide
date: 2020-05-01 10:54:13
tags: Java
categories: 
---
Shell, Python等语言都支持Shell交互命令, 使用和学习起来非常方便. 而Java需要先编译再执行, 导致Java9之前一直没有Shell可以使用. 2017年9月, Java9 发布, 带来了船新版本的JShell. 虽然现在都Java14了, 但一直没有使用过Java8之后的新功能, 所以今天就来下载个JDK9, 跟着Oracle的文档操作一波, 轻松愉快.

# Preface 

How to use Java Shell (JShell), a Read-Eval-Print Loop (REPL) tool for exploring the Java language.

# Introduction to JShell

The Java Shell Tool (JShell) is an interactive tool for learning the Java programming language and prototyping Java code. The tool is run from the command line.

## Why Use JShell?

Using JShell, you can enter program elements one at a time, immediately see the result, and make adjustments as needed.

JShell helps you try out code and easily explore options as you develop your program. You can test individual statements, try out different variations of a method, and experiment with unfamiliar APIs within the JShell session. JShell doesn't replace an IDE. As you develop your program, paste code into JShell to try it out, and then paste woring code from JShell into your program editor or IDE.


## Starting and Stopping JShell

JShell was introduced in JDK 9. To start JShell, enter the jshell command on the command line. To exit JShell, enter /exit

# Snippets(一则小消息)

Jshell accepts Java statement; variable, method, class definitions; imports; and expressions. These pieces of Java code are referred to as snippets.

## Snippet Transformation
JShell makes it easy to import a needed class when it is first referenced and convert an expression to a variable declaration using keyboard shortcuts.

1. Shift+Tab i : immediately after the identifier to see the options that enable you to add the import to your session.
2. Shift+Tab v : convert an expression to a variable declaration. Sometimes the result type of the expression isn't imported yet. In that case, this offers to both import and create the variable.

# Commands

JShell commands control the enviroment and display information within a session.

Commands are distinguished from snippets by a leading forward slash(/). For information about current var, methods, types, use the /vars, /methods, and types commands.For a list of entered snippets, use the /list command.

# Editing

JShell supports editing input at the jshell prompt and editing in an external editor of your choice.

you can define a keyboard macro by entering `Ctrl+x (`, then entering your text, and finally entering `Ctrl+x )`. To use your macro, enter `Ctrl+x e`.

## External Editor

To edit all existing snippets at once in an editor, use /edit without an option. To edit a specific snippet in an editor, use the /edit command with the snippet name or ID. Use the /list command to get the snippet IDs. (这个功能令修改多个snippets非常方便)

# External Code

External classes are accessed from a JShell session through the class path. External modules are accessed through the module path, additional modules setting, and module exports setting.

## Setting the Class Path

` % jshell --class-path myOwnClassPath`

Point your class path to directories or JAR files that have the packages that you want to access. The code must be compiled into class files. Code in the default package, which is also known as the unnamed package, can’t be accessed from JShell. After you set the class path, these packages can be imported into your session:

`jshell> import my.cool.code.*`

You can also use the /env command to set the class path, as shown in the following example:

`jshell> /env --class-path myOwnClassPath`

The /env command resets the execution state, reloading any current snippets with the new class path setting or other environment setting entered with the command.

# Setting Module Options

Modules are supported in JShell. The module path can be set, additional modules to resolve specified, and module exports given.

Module options can be provided in options to the /env command or on the command line as shown in the following example:

`% jshell --module-path myOwnModulePath  --add-modules my.module`

To see current environment settings, use /env without options.

# Feedback Modes

The feedback mode determines the prompts, feedback, and other interactions within JShell. Predefined modes with different levels of feedback are provided. Custom modes can be created as needed.

## Setting the Feedback Mode

A feedback mode defines the prompts and feedback that are used in your interaction with JShell. Predefined modes are provided for your convenience. You can create custom modes as needed.

The predefined modes can’t be modified, but they can be used as the base of a custom mode. The predefined modes, in descending order of verbosity are verbose, normal, concise, and silent.

# Scripts

A JShell script is a sequence of snippets and JShell commands in a file, one snippet or command per line.

## Startup Scripts

Startup scripts contain snippets and commands that are loaded when a JShell session is started. The default startup script contains common import statements. You can create custom scripts as needed.

Startup scripts are loaded each time the jshell tool is reset. Reset occurs during the initial startup and with the /reset, /reload, and /env commands. If you do not set the script, then, the default startup script, DEFAULT, is used. This default script defines commonly needed import declarations.

For experimentation, it is useful to have print methods that don't need the System.out. prefix. Use the predefined PRINTING script to access the print, println, and printf methods. You can specify more than one startup script with /set start. The following example sets the startup to load both the default imports and printing definitions:

```
jshell> /set start -retain DEFAULT PRINTING

jshell> /exit
|  Goodbye

% jshell
|  Welcome to JShell -- Version 9
|  For an introduction type: /help intro

jshell> println("Hello World!")
Hello World!
```


