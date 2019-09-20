---
title: 对动画混合的研究
date: 2018-07-11 11:58:46
tags:
---

# 现有游戏引擎支持概况

| 游戏引擎 | 一维混合 | 二维混合 | 其它 |
| :---- | :---- | :---- | :---- |
| Unity | 根据输入参数混合 | Simple Directional, Freeform Cartesain, Freeform Directional | Direct Blend  |
| Egret | 动画切换时混合 | 不支持 | |
| LayaAir | 不支持 | 不支持 | |
| PlayCanvas | 动画切换时混合 | 不支持 | |

# Unity

## 二维混合

Unity包含3种二维混合方式：

* 简单定向混合（Simple Directional）
* 自由笛卡尔空间混合（Freeform Cartesian）
* 自由定向混合（Freeform Directional）

### 简单定向混合

简单定向混合将最终结果的影响分为**结点影响部分**和**中心影响部分**。

#### 结点影响部分的计算

我们将本身位于原点的样本点称为**中心点**。除过中心点外，从原点向各个样本点发出射线，整个平面空间会被分割成若干扇形区域。只有输入点所在扇形区域所确定的两个样本点贡献于结点影响部分，根据这两个样本点与输入点的方位来确定结点影响部分的权重。

##### 确定输入点所在的扇形区域

显而易见，我们可以计算出每个样本点与输入点之间的角度差，其中最大的差和最小的差所对应的就是我们需要的样本点。

```ts
function getNodeInfluenceSamples(input: vec2, samples: vec2[])
{
    let calcAngle = (p: vec2) => Math.atan2(p.y, p.x);
    let diffAngles = [];
    // 在遍历的同时我们记录下本身就位于原点的样本点的索引，在计算中心影响部分时使用。
    let iCenter = -1;
    for (let i = 0; i < samples.length; ++i)
        if (samples[i].x == 0 && samples[i].y == 0)
            iCenter = i;
            diffAngles.push({
                index: i,
                diff: calcAngle(samples[i]) - calcAngle(input)});
    diffAngles.sort((a, b) => a.diff - b.diff);
    return {
        first: diffAngles[0].index,
        second: diffAngles[diffAngles.length - 1].index,
        center: iCenter};
}
```

##### 确定结点影响部分的权重

记输入点为 \\(P_i\\)，两个样本点放速度为 \\(P_1\\) 和 \\(P_2\\)，视输入点为两个样本点的线性组合，相应的线性组合系数分别为 \\(t_1\\)、\\(t_2\\)，即存在：
$$ P_i = P_1 \cdot t_1 + P_2 \cdot t_2 $$
注意到样本点与输入点都是二维向量，因此该方程存在 \\(t_1\\)、\\(t_2\\) 的解。
可以用矩阵求解：
{% raw %}
    $$
    \begin{align}
    \begin{vmatrix}
        P_i.x\\
        P_i.y
    \end{vmatrix}
    &=
    \begin{vmatrix}
        P_1.x & P_2.x\\
        P_1.y & P_2.y
    \end{vmatrix}
    \cdot
    \begin{vmatrix}
        t_1\\
        t_2
    \end{vmatrix}
    \\
    \begin{vmatrix}
        t_1\\
        t_2
    \end{vmatrix}
    &=
    inverse(
    \begin{vmatrix}
        P_1.x & P_2.x\\
        P_1.y & P_2.y
    \end{vmatrix}
    )
    \cdot
    \begin{vmatrix}
        P_i.x\\
        P_i.y
    \end{vmatrix}
    \end{align}
    $$
{% endraw %}

```ts
function getNodeInfluenceSamplesCoff(input: vec2, p1: vec2, p2: vec2)
{
    let m = new mat2();
    m.setColumns(p1, p2);
    let t1t2 = mat2.inverse(m) * input;
    return { t1: t1t2.x, t2: t1t2.y };
}
```

简单定向混合认为，结点影响部分的整体权重就用这两个系数的和来衡量，同时，对其进行归一化：
$$ NodeInfluence = clamp(t_1 + t_2, 0, 1) $$


回到采样点上，若求解出的系数中不存在负数，这两个采样点对整个结点影响部分的贡献分别等于它们的系数与系数和之比。自然，它们的权重就等于它们各自对结点影响部分的贡献乘以结点影响部分的权重：
$$ Weight_1 = NodeInfluence \cdot { t_1 \over t_1 + t_2 }  $$
$$ Weight_2 = NodeInfluence \cdot { t_2 \over t_1 + t_2 }  $$

若求解出的系数中存在负数，则表示输入点虽然位于这两个采样点对应的扇形中，但不在这两个采样点以及原点构成的三角形中。在这种情况下，简单定向混合认为这两个采样点的贡献都为 0.5。
$$ Weight_1 = Weight_2 = NodeInfluence \cdot 0.5  $$

```ts
function calcNodeInfluence(result: vec2[], input: vec2, samples: vec2[])
{
    let nodeInfluenceSamples = getNodeInfluenceSamples(input, samples);
    let coff = getNodeInfluenceSamplesCoff(
        input, samples[nodeInfluenceSamples.first], samples[nodeInfluenceSamples.second]);
    let nodeInfluence = clamp(coff.t1 + coff.t2, 0, 1);
    let contrib1 = 0, contrib2 = 0;
    if (coff.t1 < 0 || coff.t2 < 0)
        contrib1 = contrib2 = 0.5;
    else
    {
        contrib1 = coff.t1 / (coff.t1 + coff.t2);
        contrib2 = coff.t2 / (coff.t1 + coff.t2);
    }
    result[nodeInfluenceSamples.first] = nodeInfluence * contrib1;
    result[nodeInfluenceSamples.second] = nodeInfluence * contrib2;
    return {nodeInfluence: nodeInfluence, centerIndex: nodeInfluenceSamples.center};
}
```

#### 中心影响部分的计算

中心影响部分定义为除去结点影响部分的其它部分。因此，其权重为：
$$ CenterInfluence = 1.0 - NodeInfluence $$

若存在中心采样点，则中心影响部分的权重皆视为来自该中心采样点：
$$ Weight_{center} = CenterInfluence  $$
否则，中心影响部分的权重视为平均来自每个样本点（记样本点数目为{% raw %}$N${% endraw %}）：
$$ Weight_i = {CenterInfluence \over N}  $$

```ts
function calcCenterInfluence(result: vec2[], centerIndex: number, nodeInfluence: number)
{
    let centerInfluence = 1 - nodeInfluence;
    if (centerIndex >= 0)
        result[centerIndex] = centerInfluence;
    else
    {
        let average = centerInfluence / result.length;
        for (let i = 0; i < result.length; ++i)
            result[i] += average;
    }
}
```

最后将这两部分组合起来：

```ts
function simpleDirectionalBlend(result: vec2[], input: vec2, samples: vec2[])
{
    let nodeInfluenceCalcResult = calcNodeInfluence(result, input, samples);
    calcCenterInfluence(
        result,
        nodeInfluenceCalcResult.centerIndex,
        nodeInfluenceCalcResult.nodeInfluence);
}
```

### 梯度带插值



# 参考文献

1. [Animation Blending | Learn PlayCanvas](https://developer.playcanvas.com/en/tutorials/animation-blending/)

1. Rune, Skovbo, Johansen. Automated Semi‐Procedural Animation for Character Locomotion[D]. Aarhus University:Rune Skovbo Johansen, 2009.

1. [Unity package: Locomotion System](https://assetstore.unity.com/packages/tools/animation/locomotion-system-7135)

1. [重新实现unity3d的Mecanim动画混合 (1) 2D Simple Directional](https://segmentfault.com/a/1190000006792108)

1. [重新实现unity3d的Mecanim动画混合 (2) 2D Freeform Cartesian](https://segmentfault.com/a/1190000006859384)

{% raw %}
<!-- <canvas id="pdfcanvas" width="480" height="640">
	</canvas>

	<button id="prev">Previous</button>
	<button id="next">Next</button>
	&nbsp; &nbsp;
	<span>Page:
		<span id="page_num"></span> /
		<span id="page_count"></span>
	</span>

	<script src="http://mozilla.github.io/pdf.js/build/pdf.js"></script>
	<script src="/scripts/pdfviewer.js"> </script>
	<script>
		new PDFViewer("/resources/foundation.pdf");
	</script> -->
{% endraw %}