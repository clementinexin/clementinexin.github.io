---
layout: post
title:  "Java 编码实践-不定期更新"
date: 2017-01-10
categories: Java
tags: Java
published: true
---
* 目录
{:toc}


# 不要重复代码，重复意味着维护的麻烦

## 案例：多重排序

> 实现起来又臭又长的代码自然正确性也得不到保证

### 按照多个属性进行排序，即先分组再组内排序

```java
priceDataList = priceDataList
        .stream()
        .sorted((p1,p2)-> p1.getDiscount() - p2.getDiscount())
        .collect(Collectors.groupingBy(PriceData::getDiscount))
            .entrySet()
            .stream()
            .map(entry -> entry.getValue().stream().sorted((p1,p2)->p1.getDate().compareTo(p2.getDate())))
            .flatMap(Function.identity())
        .collect(Collectors.toList());
```

### comparator实现多个比较器

```java
priceDataList = priceDataList
        .stream()
        .sorted(Comparator.comparing(PriceData::getDiscount).thenComparing(PriceData::getDate))
        .collect(Collectors.toList());
```
