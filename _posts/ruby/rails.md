---
layout: post
title:  "rails"
date:   2018-04-17
categories: ruby
---

## quickstart
+ rails generate controller article 
+ rake routes
+ rails generate model Article title:string body:text
+ rails g scaffold user name:string age:int
+ rake db:migrate


## 数据校验
1. 数据验证方法
    在数据存入数据库之前，也有几种验证数据的方法，包括数据库原生的约束、客户端验证和控制器层验证。下面列出这几种验证方法的优缺点：

+ 数据库约束和存储过程无法兼容多种数据库，而且难以测试和维护。然而，如果其他应用也要使用这个数据库，最好在数据库层做些约束。此外，数据库层的某些验证（例如在使用量很高的表中做唯一性验证）通过其他方式实现起来有点困难。
+ 客户端验证很有用，但单独使用时可靠性不高。如果使用 JavaScript 实现，用户在浏览器中禁用 JavaScript 后很容易跳过验证。然而，客户端验证和其他验证方式相结合，可以为用户提供实时反馈。
+ 控制器层验证很诱人，但一般都不灵便，难以测试和维护。只要可能，就要保证控制器的代码简洁，这样才有利于长远发展。

你可以根据实际需求选择使用合适的验证方式。Rails 团队认为，模型层数据验证最具普适性。
