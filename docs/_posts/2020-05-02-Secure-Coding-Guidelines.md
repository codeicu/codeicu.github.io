---
title: Secure_Coding_Guidelines
date: 2020-05-02 23:42:39
tags: Java
categories:
---

# Introduction
(这一段写得全面又简练)

One of the main design considerations for the Java platform is to provide a restricted environment for executing code with different permission levels. Java comes with its own unique set of security challenges. While the Java security architecture can in many cases help to protect users and systems from hostile or misbehaving code, it cannot defend against implementation bugs that occur in trusted code. Such bugs can inadvertently(无意) open the very holes that the security architecture was designed to contain. In severe cases local programs may be executed or Java security disabled. These bugs can potentially be used to steal confidential data from machine and intranet(内联网), misuse system resources, prevent useful operation of the machine, assist further attacks, and many other malicious activities.

The choice of language system impacts the robustness of any software program. The Java language and virtual machine provide many features to mitigate(减小) common programming mistakes.

(书籍推荐)  
To minimize the likelihood of security vulnerabilities caused by programmer error, Java developers should adhere to recommended coding guidelines. Existing publications, such as Effective Java, provide excellent guidelines related to Java software design. Others, such as Software Security: Building Security In, outline guiding principles for software security. Any implementation bug can have serious security ramifications(众多复杂后果) and could appear in any layer of the software stack.

While sections 0 through 3 are generally applicable across different types of software, most of the guidelines in sections 4 through 9 focus on applications that interact with untrusted code (though some guidelines in these sections are still relevant for other situations)

## 0 Foundamentals

The following general principles apply throughout Java security.

1. Prefer to have obviously no flows rather than no obvious flaws.

Creating secure code is not necessarily(不可避免) easy. Despite the unusually robust nature of Java, flaws can slip past with surprising ease. Design and write code that does not require clever logic to see that it is safe. Specifically, follow the guidelines in this document unless there is a very strong reason not to.

2. Design APIs to avoid security concerns

It is better to design APIs with security in mind. Trying to retrofit(翻新) security into an existing API is more difficult and error prone(易遭受). For example, making a class final prevents a malicious subclass from adding finalizers, cloning, and overriding random methods (Guideline 4-5). Any use of the SecurityManager highlights an area that should be scrutinized.

3. Avoid duplication

Duplication of code and data causes many problems. Both code and data tend not to be treated consistently when duplicated, e.g., changes may not be applied to all copies.

4. Restrict privileges

Despite best efforts, not all coding flaws will be eliminated even in well reviewed code. However, if the code is operating with reduced privileges, then exploitation of any flaws is likely to be thwarted. The most extreme form of this is known as the principle of least privilege. Using the Java security mechanism this can be implemented statically by restricting permissions through policy files and dynamically with the use of the java.security.AccessController.doPrivileged mechanism (see Section 9).

5. Establish trust boundaries

In order to ensure that a system is protected, it is necessary to establish trust boundaries. Data that crosses these boundaries should be sanitized(令人不愉快) and validated before use. Trust boundaries are also necessary to allow security audits to be performed efficiently. Code that ensures integrity of trust boundaries must itself be loaded in such a way that its own integrity is assured.

For instance, a web browser is outside of the system for a web server. Equally, a web server is outside of the system for a web browser. Therefore, web browser and server software should not rely upon the behavior of the other for security.

6. Minimise the number of permission checks

Java is primarily an object-capability language. SecurityManager checks should be considered a last resort(诉求). Perform security checks at a few defined points and return an object (a capability) that client code retains so that no further permission checks are required. Note, however, that care must be taken by both the code performing the check and the caller to prevent the capability from being leaked to code without the proper permissions. See Section 9 for additional information.

7.  Encapsulate(概况)

Allocate behaviors and provide succinct(言简意赅) interfaces. Fields of objects should be private and accessors avoided. The interface of a method, class, package, and module should form a coherent(合乎逻辑) set of behaviors, and no more.

8. Document security-related information

API documentation should cover security-related information such as required permissions, security-related exceptions, caller sensitivity (see Guidelines 9-8 through 9-11 for additional on this topic), and any preconditions or postconditions that are relevant to security. Documenting this information in comments for a tool such as Javadoc can also help to ensure that it is kept up to date.

写出一个程序很简单, 但写出面面俱到的程序非常难, 每一个领域都有人花费大量的时间精力去研究. 学习所有的知识是不现实的, 所以, 要有自己专长.