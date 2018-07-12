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

简单方向性混合认为，这两个样本点在它们分别的权重下的和就构成了输入点，即存在：

{% raw %}
<script src="http://cdn.mathjax.org/mathjax/latest/MathJax.js?config=TeX-AMS-MML_HTMLorMML"></script>
<p>
    When \(a \ne 0\), there are two solutions to \(ax^2 + bx + c = 0\) and they are
  $$x = {-b \pm \sqrt{b^2-4ac} \over 2a}.$$
</p>
{% endraw %}

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