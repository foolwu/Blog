## 1.初始化项目

### 1.1 安装 Node.js

下载地址：https://nodejs.org/dist/v8.9.4/node-v8.9.4-x64.msi
下载完成后点击安装。
查看 node 版本，命令：node -v
版本：v8.9.4

### 1.2 安装 docsify-cli 工具

命令行执行：

```bash
npm i docsify-cli -g
```

会在这个路径下

 ```
C:\Users\Administrator\AppData\Roaming\npm\node_modules
 ```

生成 docsify-cli 文件夹。

输入命令

```bash
docsify -v
```

检查是否成功。

### 1.3 初始化文档结构

```bash
docsify init ./docs
```

会在当前目录下新建一个 docs 文件夹，并初始化项目，结构如下。

```bash
 -| docs/
    -| .nojekyll 用于阻止 GitHub Pages 会忽略掉下划线开头的文件
    -| index.html 入口文件
    -| README.md 会做为主页内容渲染，编辑即可更新网站内容
```

输入命令：dir 可以查看当前目录。

如果想在特定目录新建项目，例如：cd d:note，会进入 D 盘的 note 文件夹，再执行初始化命令。

### 1.4 本地实时预览

输入如下命令，启动一个本地服务器，可以实时预览效果。默认访问地址：[http://localhost:3000](http://localhost:3000 ) 。

```bash
// 如果当前已在 docs 目录里，则使用 docsify serve 即可
docsify serve docs
```

## 2.部署到 Github

Github、GiTee 都可以用来部署，但是Github的流程更简单，这里以Github为例。

### 2.1 上传到 Github 仓库

新建仓库 Blog，不做任何设置，将 docs 文件夹上传到仓库即可。

### 2.2 开启 GithubPage

进入仓库 Blog，点击 Settings，下滑找到 GitHub Pages，Source 选择 master branch/docs folder，然后 save。按照提示访问  https://xxx.github.io/Blog 就能看到首页了。

