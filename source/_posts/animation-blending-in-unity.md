---
title: Unity中的动画混合
date: 2018-07-11 11:58:46
tags:
---

# 二维混合

Unity包含3种二维混合方式：

* 简单方向性混合（Simple Directional）
* 自由形态下的笛卡尔空间混合（Freeform Cartesian）
* 自由形态下的方向性混合（Freeform Directional）

## 简单方向性混合

简单方向性混合思路是将最终结果的影响分为两部分：*结点影响部分*、*中心影响部分*。

### 结点影响部分的计算

将原点和各个样本点连接，整个平面空间会被分割成若干扇形区域，其中输入点所在扇形区域所确定的两个样本点构成了结点影响部分，并根据这两个样本点与输入点的方位来确定结点影响部分的值。

#### 确定输入点所在的扇形区域

显而易见，我们可以计算出每个样本点与输入点之间角度的差，最大的差和最小的差所对应的就是我们需要的样本点。

```ts
function getNodeInfluenceSamples(input: vec2, samples: vec2[])
{
    let calcAngle = (p: vec2) => Math.atan2(p.y, p.x);
    let diffAngles = [];
    for (let sample in samples)
        diffAngles.push(calcAngle(sample) - calcAngle(input));
    diffAngles.sort((a, b) => a - b);
    return { first: diffAngles[0], second: diffAngles[diffAngles.length - 1] };
}
```

#### 确定结点影响部分的权重

简单方向性混合认为，这两个样本点与它们权重相乘之和就构成了输入点，记输入点为{% raw %} $P_i$ {% endraw %}，两个样本点为{% raw %} $P_1$ {% endraw %}和{% raw %} $P_2$ {% endraw %}，其权重分别为{% raw %} $t_1$ {% endraw %}、{% raw %} $t_2$ {% endraw %}，即存在：
{% raw %}
$$ P_i = P_1 \cdot t_1 + P_2 \cdot t_2 $$
{% endraw %}
注意到样本点与输入点都是二维向量，因此该方程存在{% raw %} $t_1$ {% endraw %}、{% raw %} $t_2$ {% endraw %}的解。


# 参考文献

1. Rune, Skovbo, Johansen. Automated Semi‐Procedural Animation for Character Locomotion[D]. Aarhus University:Rune Skovbo Johansen, 2009.

1. [Unity package: Locomotion System](https://assetstore.unity.com/packages/tools/animation/locomotion-system-7135)

1. [重新实现unity3d的Mecanim动画混合 (1) 2D Simple Directional](https://segmentfault.com/a/1190000006792108)

1. [重新实现unity3d的Mecanim动画混合 (2) 2D Freeform Cartesian](https://segmentfault.com/a/1190000006859384)

<!-- {% raw %}
<canvas id="pdfcanvas" width="480" height="640">
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
	</script>
{% endraw %} -->

<!-- {% pdfviewer %} -->