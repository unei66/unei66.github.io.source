---
layout: post
title:  "java8"
date:   2018-04-17
categories: java
---

## 行为参数化
让方法接受多种行为作为参数，并在内部使用，来完成不同的行为。

```
public static List<Apple> filterApples(List<Apple> inventory,ApplePredicate p){
    List<Apple> result=new ArrayList<>();
    for(Apple apple:inventory){
        if(p.test(apple)){
            result.add(apple);
        }
    }

    return result;
}
```

