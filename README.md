<h2> vueTree</h2>

the tree for Vue2.0

run
> npm run dev

附赠vue脚手架一套

<h2>树的结构</h2>

多根节点结构（index.vue）

```
.list	// 整个列表
|
|-- .root	// 根节点（可能存在多个），包含他的子节点
|	|
|	|----	.topItem	// 根节点（不包含他的子节点）
|	|
|	|--	--	子节点	// 根节点的子节点
|
|-- .root	// 根节点（可能存在多个），包含他的子节点
|	|
|	|----	.topItem	// 根节点（不包含他的子节点）
|	|
|	|--	--	子节点	// 根节点的子节点
|
...
```

子节点的结构（递归结构）（treeItem.vue）

```
子节点
|
|----	.item	// 该子节点的DOM
|
|---- div
		|
		|----	下级子节点1
		|----	下级子节点2
		....
```


<h2>CSS：</h2>

<h3>1、背景颜色</h3>

由于HTML自身的限制问题，建议根节点和子节点，使用同样的背景颜色，并通过``options.listStyle.backgroundColor``来设置。

原因是子节点的宽度受限于父框体的宽度影响，导致当深层子节点把整个list的根结点的宽度撑开后（触发list滚动条），``.topItem``的根结点的宽度，只能保持和撑开前的宽度一致（即使设置其width为100%只是和撑开前的保持一致）。

那么问题就来了，当``.list``被撑开后，他的实际宽度是大于``.topItem``的，因此``.topItem``最右边就会有一部分是没撑满``.list``的区域。

验证代码如下：

```
<style>
    #box {
        width: 200px;
        overflow: auto;
        border: 1px solid #000;
    }

    .container > div {
        height: 50px;
    }

    .moreWidth {
        width: 300px;
        background-color: green;
    }

    .normal {
        background-color: blue;
        width: 100%;
    }
</style>
<div id="box">
    <div class="container">
        <div class="moreWidth"></div>
        <div class="normal"></div>
    </div>
</div>
```

将滚动条向右拖动，注意蓝色区域右边的部分，即可证明。

因此，解决方案有四种：

1. 使用其他能满足这个需求的树结构，但由于这个问题，解决的可能性不大；
2. 设置``.list``的css属性``overflow: hidden;``，然后让所有子节点的width为100%，并且文本内容超出部分显示三个点即可。缺点是假如树的层级较深，那么深层的节点，也许显示不了几个字；
3. 不设置子节点的背景色，如果有背景颜色需求，通过``.list``的背景颜色设置即可。缺点是，不能通过根节点和子节点的不同颜色来区分根节点和子节点（用其他方式来区分）；
4. 在【2】的基础上，首先限制最大缩进值，其次让文本内容超出部分不显示三个点，而是改用高度自适应（即更高的行高）来达成目的；

总结一下，可以通过背景颜色来区分根节点和子节点的解决办法是【2】【4】，可以适用于更多层子节点的解决办法是【3】。

伯乐在线的论坛版块，发帖回帖解决这个办法的方案是【4】，附链接：[点击查看](http://group.jobbole.com/)

Vue-tree组件通过``settings.isOverflowHidden.enabled``来配置，值为true时采用【方案2】（但文字超出部分没有用三个点，而是走马灯的形式），为false时采用【方案3】。

