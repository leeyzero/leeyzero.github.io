---
title: Go json.Unmarshal 精度丢失问题分析（1/2）
date: 2024-04-16 23:37:26
categories:
- 编程语言
tags:
- Go
- JSON
---

## 缘起

最近出现一例json.Unmarshal导致的精度丢失引发的线上问题，虽然这个问题在被及时发现，未对业务造成损失，但细挖这个问题的原因仍然比较有意思。这篇文章会从技术层面深入分析json.Unmarshal精度丢失的原因以及处理建议，以避免后续开发过程中再次踩坑。

在分析这个问题的过程中，发现涉及Go对浮点数数值的处理，又涉及IEEE-745标准中的一些细节，放在一篇文章中会增大文章的阅读难度，故拆分成了两个部分：

- {% post_link go-json-unmarshal-loss-precision-1 Part1 %}： 引出json.Unmarshal处理大整数可能出现精度丢失的问题，并浅层次分析原因以及解决办法。
- {% post_link go-json-unmarshal-loss-precision-2 Part2 %}： 先补充IEEE-745的背景知识，然后解释为什么json.Unmarshal处理大整数可能会出现精度丢失。

## 示例

这个问题的现象是，原始json string是一个字典，其中包含了一个大整数，在业务场景中，需要向该字典中追加一些字段，然后再序列化后进行存储。为了使用上的方便，代码中使用map[string]any去接收json.Unmarshal的结果，然后再使用json.Marshal序列化，结果发现序列化后的大整数跟原始大整数不致。

下面代码片段做了一些简化，同时忽略错误处理细节：

```Go
str := `{"id":16505201442738640729}`

var m map[string]any
json.Unmarshal([]byte(str), &m)

data, _ := json.Marshal(&m)
fmt.Println(string(data))
```

上面代码片段输出：

```shell
{"id":16505201442738640000}
```

原始json string中，id的值是16505201442738640729，经过json.Unmarshal和json.Marshal后，id的值变成了16505201442738640000，看起来出现了精度有丢失。

<!--more-->

## 分析

在上面代码片断中，如果**使用any类型接受整型时，json.Unmarshal会默认使用float64存储整型**：

```Go
str := `{"id":16505201442738640729}`

var m map[string]any
json.Unmarshal([]byte(str), &m)
fmt.Println(m["id"], reflect.TypeOf(m["id"]))
```

输出结果为：

```shell
1.650520144273864e+19 float64
```

在encoding/json包，decode.go中，可以看到其实现：

```Go
// convertNumber converts the number literal s to a float64 or a Number
// depending on the setting of d.useNumber.
func (d *decodeState) convertNumber(s string) (any, error) {
    if d.useNumber {
        return Number(s), nil
    }
    f, err := strconv.ParseFloat(s, 64)
    if err != nil {
        return nil, &UnmarshalTypeError{Value: "number " + s, Type: reflect.TypeOf(0.0), Offset: int64(d.off)}
    }
    return f, nil
}
```

Go在处理json.Unmarshal时，如果未开启[UseNumber](https://pkg.go.dev/encoding/json#Decoder.UseNumber)，默认会将数值类型的字面量转换成float64。如果启用UseNumber，则会使用json.Number存储，从源码中，我们可以看到，json.Number其实底层就是string：

```Go
// A Number represents a JSON number literal.
type Number string

// String returns the literal text of the number.
func (n Number) String() string { return string(n) }

// Float64 returns the number as a float64.
func (n Number) Float64() (float64, error) {
    return strconv.ParseFloat(string(n), 64)
}

// Int64 returns the number as an int64.
func (n Number) Int64() (int64, error) {
    return strconv.ParseInt(string(n), 10, 64)
}
```

看起来问题比较好解决了，可以直接使用json.Decoder，开启UseNumber，json.Unmarshal将数值字面量转化成json.Number，就不会发生精度丢失了。

```Go
str := `{"id":16505201442738640729}`

var m map[string]any
decoder := json.NewDecoder(strings.NewReader(str))
decoder.UseNumber()
decoder.Decode(&m)

fmt.Println(m["id"], reflect.TypeOf(m["id"]))
data, _ := json.Marshal(&m)
fmt.Println(string(data))
```

输出：

```shell
16505201442738640729 json.Number
{"id":16505201442738640729}
```

至此，看起来问题已经解决了，但是我们仍然没有搞明白字面量转成浮点数时，为什么会发生精度丢失？要回答这个问题涉及Go语言中对浮点数的处理方式。这部分内容留在 {% post_link go-json-unmarshal-loss-precision-2 Part2 %} 分析。

需要说明的是：在Go语言中，使用json.Unmarshal函数将JSON数据反序列化为Go结构时，整数类型默认被转换为float64类型的原因是JSON中的数字默认为浮点数。 根据JSON规范，数字可以表示为整数或浮点数，而Go语言中的float64类型可以容纳JSON中的所有数字范围，但float64并不能精确表示所有数值。

## 总结

- json.Unmarshal时，使用any接收时，默认会使用float64存储整数，这可能会导致大整数精度丢失。
- 使用json.Decoder，并开启UseNumber，使用json.Number存储大整数。

## 参考资料

- [json.Unmarshal converts json literal integer to float64 instead of int64 when taget type is of type interface{}](https://github.com/square/go-jose/issues/351)
