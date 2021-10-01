#### 打印JSON等信息
![](https://img.alicdn.com/imgextra/i3/O1CN01GVrkSl1XIhgwvGPnb_!!6000000002901-2-tps-1598-822.png)

```
const prettyoutput = require('prettyoutput');
const white = (str) => str; // 使用cli默认的颜色

const logOutputs = (outputs, indent = 0) => {
    // Clear any existing content
    process.stdout.write(ansiEscapes.eraseDown);
    process.stdout.write(
      white(
        prettyoutput(
          outputs,
          {
            colors: {
              keys: "bold",
              dash: null,
              number: null,
              string: null,
              true: null,
              false: null,
            },
            maxDepth: 10,
          },
          indent
        )
      )
    );
}
```