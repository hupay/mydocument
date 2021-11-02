# 说明
git用的频率有点低啊，用ssh提交又忘记怎么弄了。只好建个仓库记录这些琐事。

## ssh
主要参考这篇官方文档[使用 SSH 连接到 GitHub](https://docs.github.com/cn/authentication/connecting-to-github-with-ssh)。请注意，这篇文章讲述的步骤，可以让我们使用“[Git for Windows](https://gitforwindows.org/)”安装后自带的Git Bash能够克隆ssh链接的仓库。

如果我们使用[tortoisegit](https://tortoisegit.org/download/)来管理git仓库，还需要使用安装时自带的PuTTYgen来生成ppk文件。在“Conversions”菜单->“Import key”（路径如：C:\Users\\<用户名>\.ssh），选择生成的“id_ed25519”文件。然后在“Actions”栏目里，点击“Save private key”按钮。这样就得到了一个ppk文件。

最后，在使用tortoisegit克隆时，选择ppk文件路径即可。
