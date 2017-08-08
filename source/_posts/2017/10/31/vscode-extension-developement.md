---
title: Visual Studio Code 扩展/插件开发
date: 2017-10-31
tags: [Javascript]
categories: [Javascript]
---

## 创建项目
### 安装生成器
```bash
npm install -g yo generator-code
```

### 运行生成器
```bash
yo code
```
[yo code](https://code.visualstudio.com/images/yocode_yocode.png)
选择 New Extension (TypeScript),然后按照提示创建工程

### 运行扩展
1. 使用vscode打开刚创建的项目
2. 按F5，稍等下载依赖，然后会自动新打开一个用来调试的新窗口
3. ctrl+shit+p 运行 Hello World命令

## 扩展接口使用
### 命令
1. 编辑`src/extension.ts`添加一条新的命令
```javascript
context.subscriptions.push(vscode.commands.registerCommand('extension.connect',()=>{
    vscode.window.showInformationMessage('Connecting')
}))
```
2. 编辑`package.json`注册这条新命令
```json
"commands": [{
    "command": "extension.sayHello",
    "title": "Hello World"
    },
    {
    "command": "extension.connect",
    "title": "Connect"
    }
]
```
3. 运行插件
4. ctrl+shit+p 输入命令 Hello World 激活该插件，只有激活插件后才能响应其他插件的命令，否则执行命令会报错
```json
"activationEvents": [
    "onCommand:extension.sayHello"  // 在输入这个命令后会激活插件
],
```
激活插件会触发`src/extension.ts`的 activate 方法
```javascript
// this method is called when your extension is activated
// your extension is activated the very first time the command is executed
export function activate(context: vscode.ExtensionContext) {}
```
因此插件的连接资源和其他全局使用的属性可以在这个方法后进行初始化
5. 输入命令 Connect ，然后会弹出消息提示 Connecting，vscode 执行了这个命令内的代码块
6. 激活插件的事件参考
https://code.visualstudio.com/docs/extensionAPI/activation-events
如工作区包含特定文件`.PPYP.json`就自动激活插件
```json
"activationEvents": [
    "workspaceContains:.PPYP.json"
]
```

#### 内置命令
参考 https://code.visualstudio.com/docs/getstarted/settings#_settings-and-security
```javascript
"editor.action.toggleTabFocusMode",
"workbench.action.debug.continue",
"workbench.action.debug.pause",
"workbench.action.debug.restart",
"workbench.action.debug.run",
"workbench.action.debug.start",
"workbench.action.debug.stop",
"workbench.action.focusActiveEditorGroup",
"workbench.action.focusFirstEditorGroup",
"workbench.action.focusSecondEditorGroup",
"workbench.action.focusThirdEditorGroup",
"workbench.action.navigateDown",
"workbench.action.navigateLeft",
"workbench.action.navigateRight",
"workbench.action.navigateUp",
"workbench.action.openNextRecentlyUsedEditorInGroup",
"workbench.action.openPreviousRecentlyUsedEditorInGroup",
"workbench.action.quickOpen",
"workbench.action.quickOpenView",
"workbench.action.showCommands",
"workbench.action.terminal.clear",
"workbench.action.terminal.copySelection",
"workbench.action.terminal.deleteWordLeft",
"workbench.action.terminal.deleteWordRight",
"workbench.action.terminal.findWidget.history.showNext",
"workbench.action.terminal.findWidget.history.showPrevious",
"workbench.action.terminal.focus",
"workbench.action.terminal.focusAtIndex1",
"workbench.action.terminal.focusAtIndex2",
"workbench.action.terminal.focusAtIndex3",
"workbench.action.terminal.focusAtIndex4",
"workbench.action.terminal.focusAtIndex5",
"workbench.action.terminal.focusAtIndex6",
"workbench.action.terminal.focusAtIndex7",
"workbench.action.terminal.focusAtIndex8",
"workbench.action.terminal.focusAtIndex9",
"workbench.action.terminal.focusFindWidget",
"workbench.action.terminal.focusNext",
"workbench.action.terminal.focusPrevious",
"workbench.action.terminal.hideFindWidget",
"workbench.action.terminal.kill",
"workbench.action.terminal.new",
"workbench.action.terminal.paste",
"workbench.action.terminal.runActiveFile",
"workbench.action.terminal.runSelectedText",
"workbench.action.terminal.scrollDown",
"workbench.action.terminal.scrollDownPage",
"workbench.action.terminal.scrollToBottom",
"workbench.action.terminal.scrollToTop",
"workbench.action.terminal.scrollUp",
"workbench.action.terminal.scrollUpPage",
"workbench.action.terminal.selectAll",
"workbench.action.terminal.toggleTerminal",
"workbench.action.togglePanel"
```

### 设置
1. 编辑`package.json`在 contribute 属性内添加 configuration 属性
```json
"configuration": {
    "type": "object",
    "title": "ppyy", // 显示名称
    "properties": {
    "ppyy.port": {
        "type": "number", // string,array,number 类型
        "default": 5566,
        "description": "Communication with ppyy-term`s socket server bind port",
        "scope": "resource"
    }
}
```
2. 在`extension.ts`中使用 sayHello 命令获取配置属性
```javascript
context.subscriptions.push(vscode.commands.registerCommand('extension.sayHello',()=>{
    let ppyyVscodeConfig = vscode.workspace.getConfiguration('ppyy') // 这个参数就是配置的前缀
    console.log(JSON.stringify(ppyyVscodeConfig))
    vscode.window.showInformationMessage('Hello World')
}))
```
然后在编辑源码的 vscode 窗口的调试控制台将会输出`{"port":5566}`

3. 使用 Open Workspace Settings 命令打开设置编辑页面，对应的命令参数是`workbench.action.openWorkspaceSettings`

### 菜单
1. 在`package.json` 的 contribute 属性中添加 menus 
```json
"menus": {
    "editor/context": [{ // 编辑器编辑内容内右键菜单呈现
    "when": "resourceLangId == python", // 当符合条件的时候菜单才生效
    "command": "extension.sayHello",
    "alt": "resourceLangId == lua", // 折这条件
    "group": "hello" // 菜单上下文的分组 默认有navigation 
    }]
}
```
2. 菜单上下文参考
https://code.visualstudio.com/docs/extensionAPI/extension-points#_contributesmenus
```
The global Command Palette - commandPalette
The Explorer context menu - explorer/context 左侧文件树
The editor context menu - editor/context 编辑器内容
The editor title menu bar - editor/title 编辑器左侧...菜单
The editor title context menu - editor/title/context  文件标题菜单
The debug callstack view context menu - debug/callstack/context
The SCM title menu - scm/title
SCM resource groups menus - scm/resourceGroup/context
SCM resources menus - scm/resource/context
The View title menu - view/title 左侧视图标题
The View item menu - view/item/context
```
3. 菜单生效条件参考
https://code.visualstudio.com/docs/getstarted/keybindings#_when-clause-contexts
有常见的一下情况
```
resourceLangId 当前编辑的文件语言，如 resourceLangId == lua
resourceFilename	文件名
```

### 终端
// TODO

### 状态栏栏目
1. 创建一个状态栏目
```javascript
const item = vscode.window.createStatusBarIetm(StatusBarAlignment.LEFT, 100) // 数值越大越靠边
```
2. 方法
```javascript
item.show() // 显示
item.hide() // 隐藏
item.despose() // 释放
```
3. 变更他的属性
```javascript
item.text = 'lala' // 修改显示文字
item.color = 'red' // 修改颜色
item.tooltip = 'help me' // 指针停留显示的提示信息
```

4. 使用图标
支持使用`$()`语法来显示 [GitHub Octicon] (https://octicons.github.com/) 图标
```javascript
item.text = '$(pulse) haha' // 链接连接
item.text = '$(repo-force-push) 100%'  // 强推
```