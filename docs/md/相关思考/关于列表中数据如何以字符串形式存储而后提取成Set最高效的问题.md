# 关于列表中数据如何以字符串形式存储最高效的问题

最近遇到一个需要将列表存储在配置项中的问题。

用JAVA语言来说的话，就是将一个`List<String>`转换成`String`进行存储。

主要有两种方案，一种是转换成`JSONString`,另一种则是转换成`String,String,String`这样以逗号分隔的方式存储。

本人分别就这两种方式进行了探讨。

## 数据准备阶段

```java
        List<String> list = new ArrayList<>();
        for (int i = 0; i < 10; i++) {
            list.add("PD" + i*1000);
        }
        String jsonString = JSON.toJSONString(list);
        StringBuilder commaSeparatedSB = new StringBuilder();
        for (String s : list) {
            commaSeparatedSB.append(s);
            commaSeparatedSB.append(",");
        }
        commaSeparatedSB.deleteCharAt(commaSeparatedSB.length()-1);
        System.out.println(commaSeparatedSB);
        System.out.println(jsonString);
```

## JSONString相关

```java
		long jsonStart = System.currentTimeMillis();
        HashSet<String> jsonSet = JSON.parseObject(jsonString, HashSet.class);
        long jsonEnd = System.currentTimeMillis();
        long jsonTime = jsonEnd- jsonStart;
        System.out.println("jsonTime = " + jsonTime+"ms");
```

## CommaSeparatedString相关（也就是逗号分隔String）

```java
        String csString = commaSeparatedSB.toString();
        long commaSeparatedStart = System.currentTimeMillis();
        String[] split = csString.split(",");
        Set<String> strings = Arrays.stream(split).collect(Collectors.toSet());
        long commaSeparatedEnd = System.currentTimeMillis();
        long commaSeparatedTime = commaSeparatedEnd - commaSeparatedStart;
        System.out.println("commaSeparatedTime = " + commaSeparatedTime+"ms");
```

这里顺便一提，由于最终得到的列表需要用作Contains进行列表匹配，所以最终以转换成Set的方式进行对比，而非单纯的将String提取为List。故CommaSeparatedString转换为String[]之后还需要进一步转换为`Set`才能进行最终对比。

## 实验开始

### 数据量10

![image-20220419101718537](https://my-first-picture-bed.oss-cn-guangzhou.aliyuncs.com/pic-bed/202204191017608.png)

### 数据量100

![image-20220419101750479](https://my-first-picture-bed.oss-cn-guangzhou.aliyuncs.com/pic-bed/202204191017522.png)

### 数据量10000

![image-20220419101838377](https://my-first-picture-bed.oss-cn-guangzhou.aliyuncs.com/pic-bed/202204191018403.png)

### 数据量1000000

![image-20220419102252609](https://my-first-picture-bed.oss-cn-guangzhou.aliyuncs.com/pic-bed/202204191022635.png)



![image-20220419102031492](https://my-first-picture-bed.oss-cn-guangzhou.aliyuncs.com/pic-bed/202204191020523.png)

## 结论

在数据量足够大的情况下`JSONString`转`String`比`CommaSeparatedString`转`String`效率高两倍左右。

但需要注意的是，这仅仅是时间上对比的结果，空间上，`JSONString`存储要比`CommaSeparatedString`多`2n+2`长度字符数。其中`n`为存储数据个数。常数2为左右括号所占字符数。

在实际运用中，需要综合对比时间和空间存储，才能找到最适合的处理方式。