---
layout: post
title: 使用 flowchart.js 支持 markdown 流程图
tags: jekyll
published: false
---

# markdown 流程图

```html
<script src="//cdn.bootcss.com/raphael/2.2.0/raphael-min.js"></script>
<script src="//cdn.bootcss.com/flowchart/1.6.3/flowchart.js"></script>
```


## 例子 用户检测结果

<script src="//cdn.bootcss.com/raphael/2.2.0/raphael-min.js"></script>
<script src="//cdn.bootcss.com/flowchart/1.6.3/flowchart.js"></script>
<script src="/assets/flow.js"></script>

```flow
st=>start: 检测流程
io=>inputoutput: 用户样品(sample bind)
op=>operation: 总部收样 (sys Receive)
sub1=>subroutine: 查找用户基因型检测结果
sub2=>subroutine: 查找套餐检测项目
cond=>condition: 已有用户基因型是否能生成报告?  | 满足
sub3=>subroutine: 生成套餐检测结果
sub4=>subroutine: 生成待检测项目列表
op1=>operation: 发送检测方 (sys Send)
op2=>operation: 上传检测报告 (report upload)
sub5=>subroutine: 录入用户基因型
op3=>operation: 报告审核&预览
op4=>operation: 发布用户
e=>end: 完成

st->io->op->sub1(right)->sub2(right)->cond
cond(yes, right)->sub3->op3->op4->e
cond(no)->sub4->op1->op2->sub5(left)->sub1

```

<script src="{{site.url}}/assets/webfont.js"></script>
<script src="{{site.url}}/assets/snap.svg-min.js" ></script>
<script src="{{site.url}}/assets/underscore-min.js" ></script>
<script src="{{site.url}}/assets/sequence-diagram-min.js" ></script>
<script src="{{site.url}}/assets/jquery-3.1.1.min.js"></script>

<div id="diagram"></div>
<script>
  var diagram = Diagram.parse("A->B: Message");
  diagram.drawSVG("diagram", {theme: 'hand'});
</script>

<div class="diagram">A->B: Message</div>
<script>
$(".diagram").sequenceDiagram({theme: 'hand'});
</script>
