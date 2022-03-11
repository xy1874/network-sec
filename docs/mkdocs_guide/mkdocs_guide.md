# MkDocs入门指南

&emsp;&emsp;MkDocs是一个便捷易用的静态网页生成工具，详见[官网](https://www.mkdocs.org/)。



## 1. 环境安装

&emsp;&emsp;此处仅介绍Windows 10系统下如何搭建MkDocs开发环境。其他系统下的环境搭建过程类似，此处不赘述。

### 1.1 Python环境安装

&emsp;&emsp;首先，从[Python官网](https://www.python.org/downloads/windows/)下载适用于Windows的最新的Python3环境安装包，如图1-1所示。

<center><img src="../assets/1-1.png"></center>
<center>图1-1 从Python官网下载最新的Python环境安装包</center>

&emsp;&emsp;双击所下载的`python-3.XX.X-amd64.exe`安装包，勾选`Add Python 3.XX to PATH`，并点击`Install Now`，如图1-2所示。

<center><img src="../assets/1-2.png" width = 600></center>
<center>图1-2 为Windows 10安装Python环境</center>

### 1.2 依赖包安装

&emsp;&emsp;安装完毕后，在Windows 10搜索栏输入`powershell`，并在其上右键，选择`以管理员权限运行`，如图1-3所示。

<center><img src="../assets/1-3.png" width = 400></center>
<center>图1-3 以管理员权限打开PowerShell</center>

&emsp;&emsp;在PowerShell中输入`pip install mkdocs`命令并回车，以安装mkdocs。类似地，使用同样的命令依次安装`mkdocs-material`、`mkdocs-material-extensions`、`mkdocs-minify-plugin`、`pymdown-extensions`与`python-markdown-math`。

!!! info "PS :bulb:"
    &emsp;&emsp;也可以从本仓库的stupkt目录下载[requirements.txt文件](https://gitee.com/hitsz-cslab/gitee-page-demo/blob/master/stupkt/requirements.txt)，然后在PowerShell中执行`pip install -r <file-path>/requirements.txt`命令安装依赖包。

&emsp;&emsp;至此，环境搭建完毕。



## 2. MkDocs开发

### 2.1 创建MkDocs工程
&emsp;&emsp;如果希望从零开始创建MkDocs工程，则可以在指定的路径下执行`mkdocs new <proj-name>`命令以创建新的MkDocs工程。

&emsp;&emsp;如果对MkDocs不熟悉，也可以在模板工程或现有工程的基础上进行修改和定制。

### 2.2 MkDocs工程配置

&emsp;&emsp;一个MkDocs工程通常包括`docs文件夹`、`site文件夹`、`mkdocs.yml配置文件`和`README.md`四部分。

- `docs文件夹`

&emsp;&emsp;该文件夹存放的是需要开发者编辑的由Markdown语言描述的静态文本。MkDocs工具将对该文件夹下的Markdown文本进行解析和转换，生成对应的HTML文本。

- `site文件夹`

&emsp;&emsp;该文件夹存放的是MkDocs生成的HTML网页。

- `mkdocs.yml`

&emsp;&emsp;该文件是MkDocs工程的配置文件。开发者通过修改该文件中的配置，不仅可以选择静态网页的主题、自定义网页配色，还可以将`docs文件夹`下的Markdown文本组织成指定的层次结构。

&emsp;&emsp;配置文件内各项配置的含义请参考[本模板工程的配置文件](https://gitee.com/hitsz-cslab/gitee-page-demo/blob/master/mkdocs.yml)中的注释。

- `README.md`

&emsp;&emsp;该文档用于说明MkDocs工程的用途、使用声明等信息。该文件的内容将显示在GitHub或Gitee仓库的首页。

!!! info "PS :bulb:"
    &emsp;&emsp;`stupkt文件夹`可用于存放PDF、压缩包等材料。

### 2.3 Markdown编写

&emsp;&emsp;关于Markdown语法，请直接阅读[官方文档](https://markdown.com.cn/)。

### 2.4 静态网页预览及调试

&emsp;&emsp;开发过程中，若想预览静态网页，可以在MkDocs工程的根目录上打开PowerShell，并执行`mkdocs serve`命令。然后打开浏览器，访问`http://localhost:8000/`即可预览静态网页。

!!! info "PS :bulb:"
    &emsp;&emsp;预览效果通常和最终部署的效果一致。另外，在调试过程中，可能出现因为网页缓存导致的预览效果与实际Markdown代码不一致的情况。此时，可打开网页的开发者工具，禁用缓存。

&emsp;&emsp;需要注意的是，预览时MkDocs工具生成的静态网页是临时的。因此，当文档撰写完毕，在正式部署前，必须在MkDocs工程的根目录上执行`mkdocs build`命令以生成HTML文本。此时，`docs文件夹`下的Markdown文件才与`site目录`下的HTML文件一一对应。


## 3. GiteePage部署

### 3.1 GitHub Desktop安装

&emsp;&emsp;如果熟悉Git工具，可通过相应的命令将本地的MkDocs工程提交到GitHub或Gitee平台，并进行后续的部署工作。否则，也可以先从GitHub官网上下载并安装[GitHub Desktop工具](https://desktop.github.com/)。

### 3.2 创建并同步Gitee仓库

&emsp;&emsp;登录Gitee，加入[Gitee组织](https://gitee.com/hitsz-cslab)后，于组织内新建Gitee空白仓库。

!!! info "PS :bulb:"
    &emsp;&emsp;视情况可创建开源仓库或私有仓库。若创建私有仓库，则无权限用户不可访问仓库本身，但仍可访问GiteePage静态网页。

&emsp;&emsp;然后，在GitHub Desktop工具中，点击`File`->`Clone repository`，并在`URL`处填入上述建立的Gitee仓库链接以及本地文件夹路径，如图1-4所示。

<center><img src="../assets/1-4.png" width = 500></center>
<center>图1-4 通过GitHub Desktop工具将Gitee仓库克隆到本地文件夹</center>

!!! info "PS :bulb:"
    &emsp;&emsp;克隆时，需要输入Gitee仓库创建者或仓库成员的账号密码。

&emsp;&emsp;打开所克隆仓库的本地文件夹，使用快捷键`Ctrl+A`选中所有文件并删除，并将MkDocs工程根目录下的所有文件拷贝进来。

&emsp;&emsp;接下来，回到GitHub Desktop工具。此时，该工具将自动显示当前仓库的变更内容。

&emsp;&emsp;在`Summary (required)`编辑框中输入本次更新内容的概要，点击`Commit to master`按钮，如图1-5所示。

<center><img src="../assets/1-5.png"></center>
<center>图1-5 提交仓库更新内容至master分支</center>

&emsp;&emsp;点击`Push origin`按钮，将更新上传到Gitee仓库，如图1-6所示。

<center><img src="../assets/1-6.png"></center>
<center>图1-6 上传仓库更新内容至Gitee</center>

### 3.3 开启GiteePage服务

&emsp;&emsp;上传完成后，在Gitee仓库页面即可看到已上传的MkDocs工程。此时，点击`服务`->`Gitee Pages`，如图1-7所示。

<center><img src="../assets/1-7.png"></center>
<center>图1-7 进入GiteePage服务</center>

&emsp;&emsp;进入GiteePage服务页面后，在部署目录处输入`site`，并勾选`强制使用HTTPS`，再点击启动按钮，如图1-8所示。

<center><img src="../assets/1-8.png" width = 550></center>
<center>图1-8 启动GiteePage服务</center>

&emsp;&emsp;启动成功后，可点击所显示的链接打开静态网页，如图1-9所示。

<center><img src="../assets/1-9.png" width = 600></center>
<center>图1-9 访问GiteePage静态网页</center>
