---
layout: post
title:  "golang异常处理"
date:   2015-10-31 13:00
categories: golang
---

# defer
+ A deferred function's arguments are evaluated when the defer statement is evaluated.
+ Deferred function calls are executed in Last In First Out order after the surrounding function returns
+ Deferred functions may read and assign to the returning function's named return values
    ```
    func c() (i int) {
        defer func() { i++ }()
        return 1
    }
    ```

# panic
panic是一个内置的方法，可以停止普通流程然后开始panicking。当方法F调用panic，方法F停止，F上的defer方法会正常执行，然后F返回到调用者。对于调用者，F的行为类似于调用一个panic。程序继续执行栈直到当前routine的所有方法返回。panic可以通过直接调用。同样也可能通过运行时错误例如数据越界抛出。

# recover
recover是一个内置的方法，用来恢复对panicking routine的控制。recover只在defer方法中有效。在正常的执行中，调用recover会返回nil，没有任何作用。如果当前routine在panicking，调用recover会捕捉给panic的值
然后恢复正常执行。

# 引用：
https://blog.golang.org/defer-panic-and-recover