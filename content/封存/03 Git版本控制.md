#Git [视频同步笔记](https://mp.weixin.qq.com/s/Bf7uVhGiu47uOELjmC5uXQ) [Gitee](https://gitee.com) [GitHub](https://github.com)

---

- **Git工作流程**：

![[../Blog/assets/Git.jpg|400]]

> Git的工作区域分别为工作区(work space)、暂存区、本地仓库和远程仓库，本地仓库存储位置位于`.git`的隐藏文件夹中，使用`init`指令创建本地创库。工作区是本地代码存放的地方，使用`add`指令将工作区的文件加入到暂存区，使用`commit`指令将文件放至本地仓库，使用`push`指令将文件上传至远程仓库。
- **初始配置**：
```shell
# 查看配置
git config -l
# 查看系统config
git config --sys --list
# 查看当前用户配置
git config --global --list
# 配置提交代码时的用户名
git config --global user.name "fuck"
# 配置提交代码时的邮箱
git config --global user.email "xxx@gmail.com"
```
- **常用指令**：
```shell
# 在当前目录新建一个Git代码库
git init
# 下载一个项目和它的整个代码历史 例如git clone https://gitee.com/xxx/xxx.git
git clone https://gitee.com/xxx/xxx.git
# 添加指定目录，包括子目录到暂存区
git add [dir]
# 提交所有文件到暂存区
git add .
# 提交暂存区到本地仓库，message是更改信息
git commit -m [message]
# 提交本地仓库到远程仓库
git push
# 显示有变更的文件
git status
```
- **push.bat**：
```shell
@echo off
git add .
git commit -m 'fuck'
git push
pause
```