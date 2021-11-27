---
layout: post
title: Java_Virtual_Machine_Document_Reading
date: 2020-04-23 08:56:21
tags: JVM
categories: 
---
Oracle官网有很多原汁原味的文档, 非常值得研读和学习. 今天来看一下The Java Virtual Machine Specification 都讲了些什么.

# 1. Introduction

Java是一门多用途, 并发, 面向对象的语言. 它的语法与C,C++相似, 但摒弃了很多令C, C++复杂难懂和不安全的特性. Java最先是用来解决联网的消费产品的软件构建问题. 被设计成支持各种主机架构, 并支持安全构建软件组件. 为了满足这些要求, 被编译过的code在网络传输, 支持任何机器, 保证主机安全的要求中幸存下来.
# 2. The Structure of the Java Virtual Machine 

Chapter 2 gives an overview of the Java Virtual Machine architecture.

# 3. Compiling for the Java Virtual Machine

Chapter 3 introduces compilation of code written in the Java programming language into the instruction set of the Java Virtual Machine.

# 4. The `class` File Format

Chapter 4 occupies a big portion in this document, it specifies the class file format, the hardware- and operating systemindependent binary format used to represent compiled classes and interfaces.

# 5. Loading, Linking, and Initializting

Chapter 5 specifies the start-up of the Java Virtual Machine and the loading,
linking, and initialization of classes and interfaces

# 6. The Java Virtual Machine Instruction Set

Chapter 6 specifies the instruction set of the Java Virtual Machine, presenting the
instructions in alphabetical order of opcode mnemonics.

# 7. Opcode Mnemonics by Opcode

Chapter 7 gives a table of Java Virtual Machine opcode mnemonics indexed by
opcode value

