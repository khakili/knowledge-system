# JSON

*推荐一个JSON格式化的网站[BeJson](https://www.bejson.com/)*

## 什么是JSON（引用自-[JSON官网](http://json.org/json-zh.html)）
> JSON(JavaScript Object Notation) 是一种轻量级的数据交换格式。 
>易于人阅读和编写。同时也易于机器解析和生成。 
>它基于JavaScript Programming Language, Standard ECMA-262 3rd Edition - December 1999的一个子集。 
>JSON采用完全独立于语言的文本格式，但是也使用了类似于C语言家族的习惯（包括C, C++, C#, Java, JavaScript, Perl, Python等）。
> 这些特性使JSON成为理想的数据交换语言。

## JSON结构

- 对象

    对象是一个无序的键:值集合。
    一个对象以 { 开始，} 结束。
    键值对以逗号分隔。
    

- 数组

    数组是以[开始，]结束，值的有序集合。

**如果值是字符串，需要用双引号（""）括起来**

    举个栗子：
    错误：
    {'area':{'area':'1','pagetype':'home'},'pagetype':'home'}
    这个不是JSON，因为它的值没有以双引号括起来。
    正确：
    {"area":{"area":"1","pagetype":"home"},"pagetype":"home"}
    

##JSON的序列化与反序列化

**以下序列化与反序列化代码基于com.alibaba.fastjson**

```java
    public class JSONDemo {
    
        public static void main(String[] args) {
            //序列化
            JSONObject object = new JSONObject();
            object.fluentPut("name","khaki")
                    .fluentPut("age","18");
            System.out.println(object.toJSONString());
            //反序列化
            String jsonStr = "{\"area\":{\"area\":\"1\",\"pagetype\":\"home\"},\"pagetype\":\"home\"}";
            JSONObject jsonObject = JSON.parseObject(jsonStr);
            String jsonStr2 =" {'area':{'area':'1','pagetype':'home'},'pagetype':'home'}";
            //③
            JSONObject jsonObject2 = JSON.parseObject(jsonStr2);
        }
    }
```
- 为什么③处的反序列化也是可以成功的呢？
    在使用parseObject(String text)方法时，会使用JSON对象中DEFAULT_PARSER_FEATURE属性，
    也就是默认可以反序列化成功的。如果要强校验JSON，请使用以下代码
```java
        //只针对这个字符串
        String str =" {'area':{'area':'1','pagetype':'home'},'pagetype':'home'}";
        int feature = Feature.config(JSON.DEFAULT_PARSER_FEATURE,Feature.AllowSingleQuotes,false);
        JSON.parse(str,feature);  
```
    
