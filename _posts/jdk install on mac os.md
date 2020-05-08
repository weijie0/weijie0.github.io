# Java - MacOS下JDK安装、升级教程（附：JDK1.8官方下载地址）

### 1，JDK 的下载与安装

（1）首先访问 **Oracle** 官网（[www.oracle.com](https://www.oracle.com/index.html)），点击页面最下方的“**Java for Developers**”链接。

[![原文:Java - MacOS下JDK安装、升级教程（附：JDK1.8官方下载地址）](https://www.hangge.com/blog_uploads/201905/2019053115582287112.png)](https://www.hangge.com/blog/cache/detail_2453.html#)



（2）找到需要的版本后，点击旁边的“**JDK DOWNLOAD**”按钮：

[![原文:Java - MacOS下JDK安装、升级教程（附：JDK1.8官方下载地址）](https://www.hangge.com/blog_uploads/201905/2019053116023370229.png)](https://www.hangge.com/blog/cache/detail_2453.html#)



（3）选择“**Accept Lisence Agreement**”同意协议，然后点击 **Mac OS X x64** 后面的下载链接即可下载。

[![原文:Java - MacOS下JDK安装、升级教程（附：JDK1.8官方下载地址）](https://www.hangge.com/blog_uploads/201905/2019053116070064455.png)](https://www.hangge.com/blog/cache/detail_2453.html#)



（4）下载完成后双击安装包，按提示即可完成安装。

[![原文:Java - MacOS下JDK安装、升级教程（附：JDK1.8官方下载地址）](https://www.hangge.com/blog_uploads/201905/201905311654337858.png)](https://www.hangge.com/blog/cache/detail_2453.html#)

（5）我们可以通过如下路径中找到安装刚刚好的 **jdk1.8.0_211.jdk**

```
/Library/Java/JavaVirtualMachines
```

[![原文:Java - MacOS下JDK安装、升级教程（附：JDK1.8官方下载地址）](https://www.hangge.com/blog_uploads/201905/2019053116594599919.png)](https://www.hangge.com/blog/cache/detail_2453.html#)


（6）而其中 **Contents** 下的 **Home** 文件夹，则是该 **JDK** 的根目录。

[![原文:Java - MacOS下JDK安装、升级教程（附：JDK1.8官方下载地址）](https://www.hangge.com/blog_uploads/201905/2019053117022068377.png)](https://www.hangge.com/blog/cache/detail_2453.html#)

### 2，配置系统的环境变量

（1）前面的安装我们只是把 **JDK1.8** 的文件复制到操作系统上。要让应用知道 **JDK1.8** 环境的存在，接下来我们还需要配置系统的环境变量。

（2）打开终端，执行如下命令进入 **Home** 目录：

```
cd ~
```


（3）执行如下命令打开启动文件：

```
vi .bash_profile
```


（4）按 **I** 键进入编辑状态，添加如下内容（具体路径根据前面安装结果决定）：

```
export JAVA_HOME=/Library/Java/JavaVirtualMachines/jdk1.8.0_211.jdk/Contents/Home
```

[![原文:Java - MacOS下JDK安装、升级教程（附：JDK1.8官方下载地址）](https://www.hangge.com/blog_uploads/201905/2019053117114210257.png)](https://www.hangge.com/blog/cache/detail_2453.html#)

（5）编辑完毕后，按下 **ESC** 键，输入“**:wq**”保存退出。

### 3，验证是否安装成功

在终端中执行 **java -version** 命令，出现版本信息的话则说明 **JDK** 安装配置成功了。

[![原文:Java - MacOS下JDK安装、升级教程（附：JDK1.8官方下载地址）](https://www.hangge.com/blog_uploads/201905/2019053117150526245.png)](https://www.hangge.com/blog/cache/detail_2453.html#)