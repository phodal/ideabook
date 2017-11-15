基于Virtual DOM的测试代码生成
===

概况
---

### 背景

当我们在写一些UI测试的时候，我们总需要到浏览器去看一下一些DOM的变化。比如，我们点击了某个下拉菜单，会有另外一个联动的下拉菜单发生了变化。而如果这个事件更复杂的时候，有时我们可能就很难观察出来他们之间的变化。

### ShowCase

![Luffa Screenshot](./images/luffa.jpg)

源码见：[https://github.com/phodal/luffa](https://github.com/phodal/luffa)

### 基本原理

尽管这里的例子是以Jasmine作为例子，但是我想对于React也会有同样的方法。

**一个Jasmine jQuery测试**

如下是一个简单的Jamine jQuery的测试示例：

```javascript
  describe("toHaveCss", function (){
    beforeEach(function (){
      setFixtures(sandbox())
    })

    it("should pass if the element has matching css", function (){
      $("#sandbox").css("display", "none")
      $("#sandbox").css("margin-left", "10px")
      expect($("#sandbox")).toHaveCss({display: "none", "margin-left": "10px"})
    })
});
```

在beforeEach的时候，我们设定了固定的DOM进去，按照用户的行为做一些相应的操作。接着依据这个DOM中的元素变化 ，来作一些断言。

那么，即使我们已经有一个固定的DOM，想要监听这个DOM的变化就是一件容易的事。在我们断言之前，我们就会有一个新的DOM。我们只需要Diff一下这两个DOM的变化，就可以生成这部分测试代码。

步骤
---

###Step 1: Virtual-dom与HyperScript

在寻觅中发现了[virtual-dom](https://github.com/Matt-Esch/virtual-dom)这个库，一个可以支持创建元素、diff计算以及patch操作的库，并且它效率好像还不错。

virtual-dom可以说由下面几部分组成的：

1. createElement，用于创建virtual Node。
2. diff，顾名思义，diff算法。
3. h，用于创建虚拟树的DSL——HyperScript。HyperScript是一个JavaScript的HyperText。
4. patch，用于patch修改的内容。

举例来说，我们有下面一个生成Virtual DOM的函数:

```javascript
function render(count)  {
    return h('div', {
        style: {
            textAlign: 'center',
            lineHeight: (100 + count) + 'px',
            border: '1px solid red',
            width: (100 + count) + 'px',
            height: (100 + count) + 'px'
        }
    }, [String(count)]);
}
```
 
render函数用于生成一个Virtual Node。在这里，我们可以将我们的变量传进去，如1。就会生成如下图所示的节点：

```javascript
{
    "children": [
        {
            "text": "1"
        }
    ],
    "count": 1,
    "descendantHooks": false,
    "hasThunks": false,
    "hasWidgets": false,
    "namespace": null,
    "properties": {
        "style": {
            "border": "1px solid red",
            "height": "101px",
            "lineHeight": "101px",
            "textAlign": "center",
            "width": "101px"
        }
    },
    "tagName": "DIV"
}
```

其中包含中相对应的属性等等。而我们只要调用createElement就可以创建出这个DOM。

如果我们修改了这个节点的一些元素，或者我们render了一个count=2的值时，我们就可以diff两个DOM。如:

```javascript
virtualDom.diff(render(2), render(1))
```

根据两个值的变化就会生成如下的一个对象：

```javascript
{
    "0": {
        "patch": {
            "style": {
                "height": "101px",
                "lineHeight": "101px",
                "width": "101px"
            }
        },
        "type": 4,
        "vNode": {
            ...
        }
    },
    "1": {
        "patch": {
            "text": "1"
        },
        "type": 1,
        "vNode": {
            "text": "2"
        }
    },
    ...
}
```

第一个对象，即0中包含了一些属性的变化。而第二个则是文本的变化——从2变成了1。我们所要做的测试生成便是标记这些变化，并记录之。

### Step 2: 标记DOM变化

由于virtual-dom依赖于虚拟节点vNode，我们需要将fixtures转换为hyperscript。这里我们就需要一个名为html2hyperscript的插件，来解析html。接着，我们就可以diff转换完后的DOM：

```javascript
var leftNode = "", rightNode = "";
var fixtures = '<div id="example"><h1 class="hello">Hello World</h1></div>';
var change = '<div id="example"><h1 class="hello">Hello World</h1><h2>fs</h2></div>';
parser(fixtures, function (err, hscript) {
  leftNode = eval(hscript);
});

parser(change, function (err, hscript) {
  rightNode = eval(hscript);
});

var patches = diff(leftNode, rightNode);
```    

接着，我们需要调用patch函数来做一些相应的改变。

```javascript
luffa.patch(virtualDom.create(leftNode), patches)
```

并且，我们可以尝试在patch阶段做一些处理——输出修改：

```javascript
function printChange(originRootNodeHTML, applyNode) {
  var patchType;

  for (var patchIndex = 0; patchIndex < applyNode.newNodes.length; patchIndex++) {
    patchType = applyNode.newNodes[patchIndex].method;
    switch (patchType) {
      case 'insert':
        printInsert(applyNode);
        break;
      case 'node':
        printNode(applyNode, originRootNodeHTML, patchIndex);
        break;
      case 'remove':
        printRemove(applyNode, originRootNodeHTML, patchIndex);
        break;
      case 'string':
        printString(applyNode, originRootNodeHTML, patchIndex);
        break;
      case 'prop':
        printProp(applyNode, originRootNodeHTML, patchIndex);
        break;
      default:
        printDefault(applyNode, originRootNodeHTML, patchIndex);
    }
  }
}
```

根据不同的类型，作一些对应的输出处理，如pringNode：

```javascript
function printNode(applyNode, originRootNodeHTML, patchIndex) {
  var originNode = $(applyNode.newNodes[patchIndex].vNode).prop('outerHTML') || $(applyNode.newNodes[patchIndex].vNode).text();
  var newNode = $(applyNode.newNodes[patchIndex].newNode).prop('outerHTML');

  console.log('%c' + originRootNodeHTML.replace(originNode, '%c' + originNode + '%c') + ', %c' + newNode, luffa.ORIGIN_STYLE, luffa.CHANGE_STYLE, luffa.ORIGIN_STYLE, luffa.NEW_STYLE);
}
```

用Chrome的console来标记修改的部分，及添加的部分。

最后，我们似乎就可以生成相应的测试代码了。。。
