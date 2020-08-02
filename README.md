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
