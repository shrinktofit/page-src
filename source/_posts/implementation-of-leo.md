---
title: LEO - 实现
date: 2018-08-24 11:18:00
tags:
---

{% raw %}
<script src="https://cdn.bootcss.com/vis/4.21.0/vis.min.js"></script>
{% endraw %}

LEO将整个编译过程分为以下阶段：

* 源文件读取

    处理源代码，将源代码的所有字符转换为基本源字符，得到基本源字符流。

* 词法分析

    按照C++标准规定的文法，将源字符输入流解析为预处理符号流。

* 预处理

    按照C++规定的文法与预处理器行为，将预处理符号流解析为符号流。

* 语法分析

    按照C++标准规定的文法，将符号流解析为内部语法格式。

* 语义分析

{% raw %}

<div id="frontend-stream-graph" style="width:600px;height:400px;border:1px solid lightgray;">
</div>

<script>
    // create an array with nodes
    let nodes = new vis.DataSet([
        {id: 1, label: '源输入'},
        {id: 2, label: '基本源字符流'},
        {id: 3, label: '预处理符号流'},
        {id: 4, label: '符号流'},
        {id: 5, label: '语法树'},
        ]);

    // create an array with edges
    let edges = new vis.DataSet([
        {from: 1, to: 2, label: "源文件读取"},
        {from: 2, to: 3, label: "词法分析"},
        {from: 3, to: 4, label: "预处理"},
        {from: 4, to: 5, label: "语法分析"},
        ]);

    // create a network
    var container = document.getElementById('frontend-stream-graph');
    var network = new vis.Network(container, { nodes, edges }, {
        width: "100%",
        height: "100%",
        edges: {
          smooth: true,
          arrows: {to : true }
        }
    });
</script>

{% endraw %}

# 源文件读取

**基本源字符集**（Basic source character set）包含以下字符：

* 大写英文字母`A-Z`；小写英文字母`a-z`

* 罗马数字`0-9`

* `_ { } [ ] # ( ) < > % : ; . ? * + - / ^ & | ~ ! = , \ " ’`

源文件中的非基本源字符会被转换为基本源字符流`\ u code`或`\ U code`，其中_code_是该字符的Unicode编码值。

# 词法分析

词法分析将基本源字符流解析为**预处理符号**（Preprocessing token）流。

# 预处理

预处理将预处理符号流解析为**符号**（Token）流。

# 语法分析

语法分析将符号流解析为**语法树**（Syntax tree）。

语法树包括以下几类：

* 名称
    标识符名称、操作符函数名称、类型转换函数名称

* 表达式

* 语句

* 类型表示
    声明符、声明式

* 声明
    变量声明、函数声明、函数参数声明、类型别名、类声明、枚举声明、枚举量声明、模板声明、模板参数声明

* 不在以上类别的符号

为了前后端之间的独立性，LEO不在语法树中引入类型的概念。
若其它语法树中用到了类型的概念，用声明符和声明式来表示。