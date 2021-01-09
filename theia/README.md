【导语】Theia 是一个基于 TS 开发的开源 IDE **框架**，基于它我们可以开发出自己定制化的开发工具，它可以部署到云端使用，也可以打包成桌面应用。

## Theia 是什么

**Eclipse Theia 不是一个 IDE，而是一个用来开发 IDE 的框架。** 它是一个可扩展的平台，基于现代 Web 技术(TypeScript, CSS 和 HTML)实现，是用于开发成熟的、多语言的云计算和桌面类的理想产品。

## 在 docker 中运行

使用 docker 来启动一个基于 Theia 的 IDE 是最简单的了，你只需要确保你当前的系统中安装了 docker 即可。我们可以直接使用它提供的镜像 `theiaide/theia` 来启动：

```bash
# Linux, macOS, 或者 PowerShell 的终端
docker run -it --init -p 3000:3000 -v "$(pwd):/home/project" theiaide/theia:next

# Windows (cmd.exe)
docker run -it --init -p 3000:3000 -v "%cd%:/home/project" theiaide/theia:next
```

执行上面的命令后，会自动的去拉取 `theiaide/theia:next` 的镜像并且在 [http://localhost:3000](http://localhost:3000) 启动 Theia IDE，它会使用你当前目录作为工作目录。其中，`--init` 参数是用来避免[死进程问题](https://github.com/theia-ide/theia-apps/issues/195)的。

假设此刻的目录为：/Users/jerry/workspace/testbox，在该目录下执行上面的命令，我们来看看结果：

![docker run theia image](https://zcd520.gitee.io/pics/theia/docker-run.png)

通过日志我们可以看出，Theia IDE 已经成功启动并且监听 3000 端口了，我们打开浏览器看一下它的庐山真面目：

![result of docker run theia image](https://zcd520.gitee.io/pics/theia/docker-run-result.png)

有没有很亲切的感觉？哈哈，是的，它跟 VS Code 几乎长得一模一样，不仅如此，它同样支持 VS Code 中的插件，所以你可以在 Theia 中尽情的“享用” VS Code 的插件市场。我们先来跑一个 helloworld 感受一下这个 IDE 的能力：

![usage of docker run theia image](https://zcd520.gitee.io/pics/theia/docker-run-result.gif)


## 构建自己的 IDE

如果你不想使用 docker，你完全可以自己构建一个 Theia IDE。接下来我们就基于 Theia，在本地跑起来属于我们自己的 IDE。

1. 环境要求

- Node.js 版本 >= 12.14.1 且 < 13
- Yarn 版本 >= 1.7.0

2. 创建项目

```
mkdir my-theia
cd my-theia
```

接着创建 `package.json` 文件：

```json
{
  "name": "My Cool IDE",
  "dependencies": {
    "@theia/callhierarchy": "next",
    "@theia/file-search": "next",
    "@theia/git": "next",
    "@theia/markers": "next",
    "@theia/messages": "next",
    "@theia/mini-browser": "next",
    "@theia/navigator": "next",
    "@theia/outline-view": "next",
    "@theia/plugin-ext-vscode": "next",
    "@theia/preferences": "next",
    "@theia/preview": "next",
    "@theia/search-in-workspace": "next",
    "@theia/terminal": "next"
  },
  "devDependencies": {
    "@theia/cli": "next"
  }
}
```

通过 package.json 我们看到，其实 Theia 也是个 Node 的包。`dependencies` 中有很多依赖，大致可以推测出，Theia 的功能是由这些包组合起来的，比如`@theia/search-in-workspace` 负责搜索模块，`@theia/terminal` 负责终端模块等；另外，`@theia/cli` 作为 `devDependencies`，我们会在构建与运行时用到它的一些命令。

3. 安装依赖

```
yarn
```

如果下载依赖缓慢，建议切换镜像源地址。安装成功的结果应该如下：

![install theia deps](https://zcd520.gitee.io/pics/theia/theia-yarn-install.png)

4. 构建项目

```
yarn theia build
```

这个命令主要是用来生成项目代码的，包含源码，webpack 配置文件以及 webpack 打包后的文件。运行成功的结果如下：

![theia build](https://zcd520.gitee.io/pics/theia/theia-build.png)

5. 运行 Theia IDE

直接运行
```
yarn theia start
```

就能看到我们的 IDE 跑在了 3000 端口：

![theia start](https://zcd520.gitee.io/pics/theia/theia-start.png)

我们打开 `http://localhost:3000` 看看：

![usage of local run theia image](https://zcd520.gitee.io/pics/theia/local-theia-start.gif)

也是与 VSCode 近乎一致的体验。

6. 封装 npm scripts

在 `package.json` 中添加：

```json
{
  // ..... others
  "scripts": {
    "start": "theia start",
    "build": "theia build"
  }
}
```

以后我们就可以直接用 `yarn xxx` 的方式来执行了。至此，我们本地已经成功构建了一个 IDE ～

7. （进阶）安装插件

其实上一步我们已经有了一个 IDE 了，但是作为开发工具来说，那可能还差点意思。究竟差点什么呢？我们来写一些代码就知道了：

![theia without plugins](https://zcd520.gitee.io/pics/theia/theia-without-plugins.png)


是的，一目了然的结果，没有高亮，并且编码的过程中什么提示也没有，也就是相当于一个长得好看的记事本了。这完全不足以称之为一个 IDE，下面我们就来安装这些插件，使我们的 IDE 强大起来。此时，我们需要更新一下 `package.json` ：

```json
{
  // ... others
  "scripts": {
    "prepare": "yarn run clean && yarn build && yarn run download:plugins",
    "clean": "theia clean",
    "build": "theia build --mode development",
    "start": "theia start --plugins=local-dir:plugins",
    "download:plugins": "theia download:plugins"
  },
  "theiaPluginsDir": "plugins",
  "theiaPlugins": {
    "vscode-builtin-css": "https://github.com/theia-ide/vscode-builtin-extensions/releases/download/v1.39.1-prel/css-1.39.1-prel.vsix",
    "vscode-builtin-html": "https://github.com/theia-ide/vscode-builtin-extensions/releases/download/v1.39.1-prel/html-1.39.1-prel.vsix",
    "vscode-builtin-javascript": "https://github.com/theia-ide/vscode-builtin-extensions/releases/download/v1.39.1-prel/javascript-1.39.1-prel.vsix",
    "vscode-builtin-json": "https://github.com/theia-ide/vscode-builtin-extensions/releases/download/v1.39.1-prel/json-1.39.1-prel.vsix",
    "vscode-builtin-markdown": "https://github.com/theia-ide/vscode-builtin-extensions/releases/download/v1.39.1-prel/markdown-1.39.1-prel.vsix"
  }
}
```

我们更新了 `scripts`，同时又添加了 `theiaPluginsDir` 和 `theiaPlugins` 这两个属性。`theiaPluginsDir` 是用来设置我们的插件存放地址的，`theiaPlugins` 就是我们要安装的插件了。运行项目之前，我们要先运行 `yarn prepare` 来准备环境，我们会在日志中看到插件的下载情况：

![download plugins](https://zcd520.gitee.io/pics/theia/download-plugins.png)

这些插件都会放在当前目录下的 `plugins` 文件夹下。我们再来启动 IDE 看看效果，注意此时 start 带上了参数，指定了插件的目录：

![theia with plugins](https://zcd520.gitee.io/pics/theia/theia-with-plugins.png)


可以看到，借助于插件，我们可以真正的使用这个 IDE 作为生产工具了。

## 打包桌面应用

这个相对来说就比较容易了，只有简单的几步，我们可以直接参考它的 [repo](https://github.com/theia-ide/yangster-electron)。

## 总结

通过上面的例子，我们已经可以构建出一个属于自己的 IDE 了。如果你有自己的服务器，那么按照上面的步骤搭建一个 Cloud IDE，以后出门就不用背着电脑啦，一个平板，甚至一台手机就可以在线编程。