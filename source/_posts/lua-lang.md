---
title: lua 语言
date: 2022-09-13 22:16:59
tags:
  - lua
---

## 语法摘要

Lua 是一门十分简单的脚本语言，关键字列表如下：

|       |       |          |       |        |        |
|-------|-------|----------|-------|--------|--------|
| and   | break | do       | else  | elseif | end    |
| false | for   | function | goto  | if     | in     |
| local | nil   | not      | or    | repeat | return |
| then  | true  | until    | while |        |        |

其他特殊字符构成的 token 有：

|    |    |    |    |    |     |    |
|----|----|----|----|----|-----|----|
| +  | -  | *  | /  | %  | ^   | #  |
| &  | ~  | \| | << | >> | //  |    |
| == | ~= | <= | >= | <  | >   | =  |
| (  | )  | {  | }  | [  | ]   | :: |
| ;  | :  | ,  | .  | .. | ... |    |

其中：

- logic operators: `and`, `or`, `not`;
- logic constants: `true`, `false`;
- arithmetic operators: `+`, `-`, `*`, `/`, `%`, `^`(power), `//`;
- relational operators: `==`, `~=`(inequality), `>`, `<`, `>=`, `<=`;
- bitwise operators: `&`, `|`, `~`(binary as xor, unary as not), `>>`, `<<`
- misc:
  - `..`(`a .. b` -> `String.join(a, b)`)
  - `#`(`#a` -> `a.size()`)
- language constant: `nil`
- control stream:
  - loop:
    - `for`-`do`-`end`
    - `while`-`do`-`end`
    - `repeat`-`until`
    - `break`, `goto`
  - `if`-`then`-`elseif`-`else`-`end`
- function: `function`-`end`
- iterator: `in`

## cheat sheet

### ranges-for

```txt
for id=start,end[,step] do block end
```

这里 `step` 默认为 1。

### generic-for:

```text
for id_1,...,id_n in container do block end
```

### goto

```text
:: label_name ::
goto label_name_p
```
