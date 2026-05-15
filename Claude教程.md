1.Nodejs安装

官网下载
https://nodejs.org/zh-cn

在下面找到Windows系统的安装程序下载下来。然后运行安装程序，一路点击下一步完成安装。

然后再本地终端输入命令

```bash
#这个命令是允许在控制台运行Powershell脚本
Set-ExecutionPolicy Unrestricted
#这个命令是查看Nodejs的版本号
node -v 
#这个命令是查看npm的版本号
npm -v

##这里都显示了版本号，我们的Nodejs就安装成功了
```



2.Git安装
https://git-scm.com/

下载Windows系统对应的安装包，一路点击下一步完成安装。

回到桌面，右键在终端打开，输入命令

```bash
git –version
#这里能够成功的打印版本号，Git就安装成功了
```


3.Claude Code安装
来到桌面，右键选择使用终端打开，然后输入命令来安装Claude Code，回车，安装完成
```bash
npm install -g @anthropic-ai/claude-code
```

4.安装  cc switch
官网地址：https://github.com/farion1231/cc-switch/releases
找对应的windows版本下载下来。然后运行安装程序，一路点击下一步完成安装

5.购买国内模型订阅
* deepseek API平台官网：https://platform.deepseek.com
* 点充值充值
* 点击`API Key`，点击创建，名称随便取，复制`API Key`
* 点击接口文档，看到`base_url (Anthropic)`,复制右侧地址：`https://api.deepseek.com/anthropic`

6.配置 cc switch
* 打开cc switch，点击右上角加号“创建”
* 预设供应商选为`deepseek`
* 在`API Key`输入框里输入你之前复制的`API Key`
* 在`请求地址`输入框里输入你之前复制的`https://api.deepseek.com/anthropic`
* 高级选项中，主模型、Haiku 默认模型、Sonnet 默认模型、Opus 默认模型，都可以直接填`deepSeek-v4-flash`或者`deepSeek-v4-pro`,注意不要填错
* 其他可以乱填

7.至此，配置好啦！
* 可以新建一个项目文件夹，右键`在终端打开`
* 输入`claude`即可启动Claude Code让它干活啦！

8.其他配置方法：
* 来到`C:/Users/你的用户名`这个文件夹
* 在顶部菜单点击查看，勾选上显示隐藏文件夹、文件扩展名
* 右键新建文本文档，然后把我们新创建的文件的名字改成 .claude.json
* 右键选记事本，把这一大串文本复制进去
* 将`你的API Key`和`模型厂商的URL`替换为你之前复制的`API Key`和`地址`
{
   "hasCompletedOnboarding": true,
     "env": {    
        "ANTHROPIC_AUTH_TOKEN": "你的API Key",
        "ANTHROPIC_BASE_URL": "模型厂商的URL",
        "API_TIMEOUT_MS": "3000000",
        "CLAUDE_CODE_DISABLE_NONESSENTIAL_TRAFFIC": 1,
        "ENABLE_TOOL_SEARCH": false
    }
}


opencode
1.命令行安装 npm i -g opencode-ai
2.客户端官网安装
3.vscode中安装插件
* Ctrl+Shift+P 打开命令：open opencode 即可打开
* /connect
* /new 开启新会话
* /sessions 切换对话
* /share 分享对话到网址
* /unshared
* /export 导出对话
* /timeline  revert回退时间

