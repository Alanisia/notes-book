# Go

## 基本类型

|类型|长度（字节）|默认值|说明|
|---|---|---|---|
|`bool`|1|`false`||
|`byte`|1|0|`uint8`|
|`rune`|4|0|Unicode Code Point, `int32`|
|`int`, `uint`|4/8|0|32/64位|
|`int8`, `uint8`|1|0|-128~127/0~255（C: `short`）|
|`int16`, `uint16`|2|0|-32768~32767/0~65535|
|`int32`, `uint32`|4|0|-21亿~21亿/0~42亿|
|`int64`, `uint64`|8|0||
|`float32`|4|0.0||
|`float64`|8|0.0||
|`complex64`|8|||
|`complex128`|8|||
|`uintptr`|4/8||以存储指针的`uint32`或`uint64`整数|
|array|||值类型|
|`string`||`""`|值类型|
|`struct`|||值类型|
|slice||`nil`|引用类型|
|`map`||`nil`|引用类型|
|channel（`chan`）||`nil`|引用类型|
|`interface`||`nil`|接口|
|function（`func`）||`nil`|函数|

***TODO***

## 数组

***TODO***

## Slice（切片）

***TODO***

## 指针

***TODO***

## Map

***TODO***

## 面向对象

***TODO***

## 泛型

***TODO***

## CGo

***TODO***