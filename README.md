# 1. 实现页面模板与data数据替换

- html页面准备：
```
    <div>
        <div id="root">
            <p>{{name}}</p>
            <p>{{message}}</p>
        </div>
    </div>
```

- js逻辑

```
let tmpNode = document.querySelector('#root');
let data = {
    name: 'bc',
    msg: '1111111111111'
}

// 需要保留原来的，因为响应式msg变化会重新用模板渲染
let generateNode = tmpNode.cloneNode(true); // deep 可选
// 是否采用 深度克隆,如果为true,则该节点的所有后代节点也都会被克隆,如果为false,则只克隆该节点本身.

compiler(generateNode, data);
root.parentNode.replaceChild(generateNode, root)
function compiler(template, data) {
    let reg = /\{\{(.+?)\}\}/g;
    let childNodes = template.childNodes || [];
    childNodes.forEach(element => {
        let nodeType = element.nodeType; // 1 元素 3 文本
        if(nodeType === 3) { // 文本节点 {{}}
            let text = element.nodeValue; 
            text = text.replace(reg, (regValue, g1) => { // 第零个表示匹配到的内容，第n个参数表示匹配到的内容
                let key = g1.trim(); // 消除空格
                return data[key]; // 返回数据里面的值
            })
            element.nodeValue = text; // 把进行数据替换后的值替换回去
        } else if (nodeType === 1) { // 元素节点需要考虑子元素
            compiler(element, data); // 递归
        }
    });
}
```

# 2. 实现多级数据对象处理去取值

- 常规方式
```
function getExpr (obj, expr) {

    // name.firstName.xx.xx...
    let pathArr = expr.split('.');
    let result = pathArr.reduce
    ((res, cur, index) => {
        res = res[cur];
        return res;
    }, obj);
    return result;
}
```

- 闭包方式

```
    function createValueByPath(path) {
        let pathArr = path.split('.');

        return function getValueByPath(obj) {
            let result = pathArr.reduce
                ((res, cur, index) => {
                    res = res[cur];
                    return res;
                }, obj);

            return result;
        };
    }

    let getValueByPath = createValueByPath('a.b.c');

    let data = {
        a: {
            b: {
                c: '罗小黑'
            }
        }
    }

    console.log(getValueByPath(data));
```

# 3. 虚拟Dom-提升性能

- 怎么将真正的Dom转化成虚拟Dom
- 怎么将虚拟Dom转化成真正的Dom（思路与深度拷贝类似）

- 常见情况
```

// 标签表示
<div /> => { tag: 'div' }

// 标签内容
文本节点 => { tag: 'div',value }

// 标签属性
<div class="c"/> => { tag: 'div', data: {
    class: 'c'
} }

// 嵌套版本
<div>
    <div>

    </div> 
</div> 

=> { tag: 'div', children: [
    {
        tag: 'div'
    }
] }

```

- 如何将真实Dom转化成虚拟Dom
```
        /**
         * @param {dom} 真实dom. 
         */
        function getVnode(dom) {
            // 使用递归遍历Dom生成虚拟dom

            let nodeType = dom.nodeType;
            let _vNode;

            if (nodeType === 1) { // 元素节点

                let nodeName = dom.nodeName;
                let attributes = dom.attributes;

                // 将属性数组转换成对象表述
                let _attrObj = {};
                Array.from(attributes).forEach(item => { // 属性节点nodeType === 2
                    _attrObj[item.nodeName] = item.nodeValue;
                })

                _vNode = new VNode(nodeName, _attrObj, undefined, dom.nodeType);

                let childNodes = dom.childNodes;// 递归处理子节点 换行，注释也会是子节点，看是否需要处理
                childNodes.forEach((child) => {
                    _vNode.appendChild(getVnode(child));
                })
            } else if (nodeType === 3) {// 文本节点
                _vNode = new VNode(undefined, undefined, dom.nodeValue, dom.nodeType);
            }

            return _vNode;
        }
```

- 如何将虚拟Dom转换成真实Dom

```
        /**
         * @param {Object} vNode 虚拟节点. 
         */
        function parseNode(vNode) {
            return createHtmlDom(vNode);
        }

        function createHtmlDom(node) {
            let dom;
            if (node.nodeType === 1) { // 标签节点
                dom = document.createElement(node.nodeName);
                Object.keys(node.attributes).forEach(key => {
                    dom.setAttribute(key, node.attributes[key])
                })
                let children = node.childNodes;
                children.length && children.forEach(child => {
                    dom.appendChild(createHtmlDom(child))
                })
            } else if (node.nodeType === 3) {// 文本节点
                dom= document.createTextNode(node.nodeValue);
            }

            return dom;
        }
```

- 测试用例

html部分
```
    <div id="root">
        <div class="bc" title="111">222</div>
        <ul>
            <li>1</li>
            <li>2</li>
            <li>3</li>
        </ul>
    </div>
    <div>
        原样渲染
        <div id="render"></div>
    </div>
```
js代码部分
```
        let root = document.querySelector('#root');
        let myVNode = getVnode(root);
        console.log(myVNode);

        let myDom = parseNode(myVNode);
        let rendDom = document.querySelector('#render');
        rendDom.appendChild(myDom);
```