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

- Feature与SerializerFeature枚举属性介绍

    - Feature（反序列化配置）
        
        - AutoCloseSource 这个特性，决定了解析器是否将自动关闭那些不属于parser自己的输入源。 如果禁止，则调用应用不得不分别去关闭那些被用来创建parser的基础输入流InputStream和reader；如果允许，parser只要自己需要获取closed方法（当遇到输入流结束，或者parser自己调用 JsonParder#close方法），就会处理流关闭。
        - AllowComment 该特性决定parser将是否允许解析使用Java/C++ 样式的注释（包括'/'+'*' 和'//' 变量）。 由于JSON标准说明书上面没有提到注释是否是合法的组成，所以这是一个非标准的特性；尽管如此，这个特性还是被广泛地使用。
        - AllowUnQuotedFieldNames  这个特性决定parser是否将允许使用非双引号属性名字， （这种形式在Javascript中被允许，但是JSON标准说明书中没有）。
        - AllowSingleQuotes 该特性决定parser是否允许单引号来包住属性名称和字符串值。
        - InternFieldNames 该特性决定JSON对象属性名称是否可以被String#intern 规范化表示。如果允许，则JSON所有的属性名将会 intern() ；如果不设置，则不会规范化，默认下，该属性是开放的。此外，必须设置CANONICALIZE_FIELD_NAMES为true
        - AllowISO8601DateFormat 这个设置为true则遇到字符串符合ISO8601格式的日期时，会直接转换成日期类。
        - AllowArbitraryCommas 允许多重逗号,如果设为true,则遇到多个逗号会直接跳过;
        - UseBigDecimal 这个设置为true则用BigDecimal类来装载数字，否则用的是double
        - IgnoreNotMatch 忽略不匹配
        - SortFeidFastMatch 如果你用fastjson序列化的文本，输出的结果是按照fieldName排序输出的，parser时也能利用这个顺序进行优化读取。这种情况下，parser能够获得非常好的性能
        - DisableASM 禁用ASM
        - DisableCircularReferenceDetect 禁用循环引用检测
        - InitStringFieldAsEmpty 对于没有值得字符串属性设置为空串
        - SupportArrayToBean 支持数组to对象
        - OrderedField 属性保持原来的顺序
        - DisableSpecialKeyDetect 禁用特殊字符检查
        - UseObjectArray 使用对象数组
        - SupportNonPublicField
        - IgnoreAutoType
        - DisableFieldSmartMatch
        - SupportAutoType
        - NonStringKeyAsString
        - CustomMapDeserializer
        - ErrorOnEnumNotMatch
    
    - SerializerFeature（序列化配置）
    
        - QuoteFieldNames 输出key时是否使用双引号,默认为true 
        - UseSingleQuotes 使用单引号而不是双引号,默认为false
        - WriteMapNullValue 是否输出值为null的字段,默认为false
        - WriteEnumUsingToString 用枚举toString()值输出
        - WriteEnumUsingName 用枚举name()输出
        - UseISO8601DateFormat 使用ISO8601日期格式输出
        - WriteNullListAsEmpty List字段如果为null,输出为[],而非null 
        - WriteNullStringAsEmpty 字符类型字段如果为null,输出为"",而非null
        - WriteNullNumberAsZero 数值字段如果为null,输出为0,而非null 
        - WriteNullBooleanAsFalse Boolean字段如果为null,输出为false,而非null
        - SkipTransientField 如果是true，类中的Get方法对应的Field是transient，序列化时将会被忽略。默认为true
        - SortField 按字段名称排序后输出。 默认为false
        - WriteTabAsSpecial（已废弃） 把\t做转义输出，默认为false
        - PrettyFormat 结果是否格式化,默认为false
        - WriteClassName 序列化时写入类型信息，默认为false。反序列化是需用到
        - DisableCircularReferenceDetect 消除对同一对象循环引用的问题，默认为false
        - WriteSlashAsSpecial 对斜杠'/'进行转义
        - BrowserCompatible 将中文都会序列化为\uXXXX格式，字节数会多一些，但是能兼容IE 6，默认为false
        - WriteDateUseDateFormat 全局修改日期格式,默认为false
        - NotWriteRootClassName
        - DisableCheckSpecialChar 一个对象的字符串属性中如果有特殊字符如双引号，将会在转成json时带有反斜杠转移符。如果不需要转义，可以使用这个属性。默认为false
        - BeanToArray
        - WriteNonStringKeyAsString 
        - NotWriteDefaultValue
        - BrowserSecure
        - IgnoreNonFieldGetter
        - WriteNonStringValueAsString
        - IgnoreErrorGetter
        - WriteBigDecimalAsPlain
        - MapSortField
    
