---
title: 简单易懂的 Mathematica 张量
date: 2022-09-22 21:15:10
tags:
  - Mathematica
---

## 张量的定义

我们要处理的是广义相对论中常见的张量对象，度规、联络、爱因斯坦和里奇。我们将使用数组作为张量：

```wl
tensorQ[any_] := AtomQ[any] || (
  ListQ[any] && Equal@@Dimensions[any]
)

tensorDim[tensor_?tensorQ] := If[ListQ[tensor], First@Dimensions[tensor], 0];
```

## 上上下下左左右右ABAB

见内置上下文 ``SymbolicTensors` ``.

## 指标

利用内置函数我们很容易处理指标升降：

```wl
downOnce[tensor_, metric_, pos_] := With[{rank = TensorRank@tensor},
  If[rank === 0, Return@tensor]; (* {1,1} is invalid contraction term *)
  TensorTranspose[
    TensorContract[TensorProduct[tensor, metric], 
      {{pos, rank+1}}], 
      Range@rank /. {pos->rank, rank->pos} (* replace once *)
    ]
];
upOnce[tensor_, metric_, pos_] := downOnce[tensor, Inverse@metric, pos];
```

说明：
- `TensorProduct` 计算张量积；
- `TensorContract` 对指标对收缩；
- `TensorTranspose` 交换张量的不同层，这里就是交换指标；
- 为什么不用 `Cycles`？这是因为他不允许两个循环下标重合，这样不如我们这里的方便。

考虑这些，我们可以写一个简单的任意的升降函数：

```wl
changeIndex[any___] := $Failed;
changeIndex[tensor:_?tensorQ, metric:_?tensorQ, 
  change:Rule[init: {(Up|Down) ...}, fin: {(Up|Down|$) ...}]] := Module[
  {
    dim,
    rank = TensorRank[tensor],
    ups, downs
  },
  dim = tensorDim[tensor];
  Assert[TensorRank@metric == 2 && 
    tensorDim[metric] == tensor];
  Assert[Head@Quiet[Inverse@metric] =!= Inverse]; 
  (*Non-singular metric needed.*)
  Assert[Length@init === Length@fin === rank];
  ups = Select[init[[#]] == Down&]@Flatten@Position[fin, Up];
  downs = Select[init[[#]] == Up&]@Flatten@Position[fin, Down];
  Fold[downOnce[#1, metric, #2]&, Fold[
      upOnce[#1, metric, #2]&, tensor, ups
    ], downs]
]
```

或者随你的喜好改变 api：

```wl
changeIndex2[tensor_?tensorQ, metric_?tensorQ,
  change:{Rule[_Integer | {___Integer}, Up|Down]...}] := 
  Block[
    {
      rank = TensorRank[tensor],
      flatted = change /. rl:Rule[___]:>Splice[Thread[rl]], 
      rev
    },
    rev = flatted /. {Up->Down, Down->Up};
    changeIndex[tensor, metric, 
      Rule@@(ReplacePart[ConstantArray[Up, rank], #]&/@{rev, change})
    ]
  ]
```

## 抽象

我们现在希望将我们的 Tensor 封装一下，打包上指标一类的信息，这样方便我们调用，为此我们使用 Mathematica 的 OOP 技术(我也不是很清楚这是不是能叫做 OOP)来完成。

```wl
abstractTensorObject[data: _?tensorQ]; (*object*)

abstractTensorObject[___][_[___]] := $Failed; (*member function call*)
abstractTensorObject[data_]@"rank"[] := TensorRank[arr];
abstractTensorObject[data_]@"dim"[] := tensorDim[arr];
abstractTensorObject[data_]@"data"[] := data;
abstractTensorObject[data_]@"changeIndex"[
  change:Rule[init: {(Up|Down) ...}, fin: {(Up|Down|$) ...}],
  "Metric"->metric:_?metricQ:$Metric
]
```

