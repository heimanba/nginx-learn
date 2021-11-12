### 开始

- package.json

```
"activationEvents": ["onCommand:catCoding.start"],
"contributes": {
    "commands": [
      {
        "command": "catCoding.start",
        "title": "Start new cat coding session",
        "category": "Cat Coding"
      }
    ]
}
```

- extension.ts
```
context.subscriptions.push(
    vscode.commands.registerCommand("catCoding.start", () => {
      // 创建并显示新的webview
      const panel = vscode.window.createWebviewPanel(
        "catCoding", // 只供内部使用，这个webview的标识
        "Cat Coding", // 给用户显示的面板标题
        vscode.ViewColumn.One, // 给新的webview面板一个编辑器视图
        {} // Webview选项。我们稍后会用上
      );
      panel.webview.html = getWebviewContent();
    })
);

function getWebviewContent(): string {
    return `
        <!DOCTYPE html>
        <html lang="en">
        <head>
            <meta charset="UTF-8">
            <meta name="viewport" content="width=device-width, initial-scale=1.0">
            <title>Cat Coding</title>
        </head>
        <body>
            <img src="https://media.giphy.com/media/JIX9t2j0ZTN9S/giphy.gif" width="300" />
        </body>
        </html>
    `;
}
```

### 生命周期
webview插件必须保持住从webview返回的createWebviewPanel,使用`onDidDispose`事件可以在webview被销毁时触发，并且在事件结束之后释放webview资源
```
const panel = vscode.window.createWebviewPanel(
    'catCoding',
    'Cat Coding',
    vscode.ViewColumn.One,
    {}
);


panel.onDidDispose(
    () => {
        // 当面板关闭时，取消webview内容之后的更新
        clearInterval(interval);
    },
    null,
    context.subscriptions
);
```
调用`panel.dispose()`方法可以主动关闭webview

### 移动和可见性
当webview被移动到非激活标签上，就被隐藏起来。但是并不是销毁，当重新激活标签后，VS Code会从`webview.html自`动恢复webview的内容。
![](https://media.githubusercontent.com/media/Microsoft/vscode-docs/master/api/extension-guides/images/webview/basics-restore.gif)

通过`.visible`属性告诉当前webview面板是否可见。
