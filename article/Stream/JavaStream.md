# Java Stream
Stream中的不同函数就像Spark中的操作一样，分为转化类操作与触发计算类操作，所以如果像下面的代码并不会执行：
```Java
lists.stream()
     .map(pojo -> {
            return mapper.insert(pojo);
        })
```
增加`collect`函数可以触发计算:
```Java
lists.stream()
     .map(pojo -> {
            return mapper.insert(pojo);
        }).collect(Collectors.toList());
```
