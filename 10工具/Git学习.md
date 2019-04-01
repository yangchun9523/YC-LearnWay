## Git学习
### 获取Git仓库

- 初始化git仓库: 

```shell
# 初始化git命令：
# 会在当前目录下生成.git子目录，存储git版本控制的所有数据。(与.svn相似)
git init
```

- 克隆已有git仓库: 

```shell
# SSH方式：
# 需要生成git公钥，并添加到github SSH key中
# 参考URL:https://git-scm.com/book/zh/v2/服务器上的-Git-生成-SSH-公钥
git clone git@github.com:yangchun9523/YC-LearnWay.git
# HTTPS方式：
git clone https://github.com/yangchun9523/YC-LearnWay.git
```
### git忽略文件.gitignore配置

```text
# MacOS
.DS_Store

# Compiled class file
*.class

# Log file
*.log

# BlueJ files
*.ctxt

# Mobile Tools for Java (J2ME)
.mtj.tmp/

# Package Files #
*.jar
*.war
*.nar
*.ear
*.zip
*.tar.gz
*.rar

# virtual machine crash logs, see http://www.java.com/en/download/help/error_hotspot.xml
hs_err_pid*
```

### 目前使用到的Git命令总结(持续补充)

```shell
# 0.将本地仓库关联到GitHub仓库上
git remote add origin git@github.com:yangchun9523/YC-LearnWay.git

# 1.创建文件
touch test.md

# 2.将文件添加到暂存区
git add test.md

# 3.数据添加到仓库
git commit -m'描述信息'

# 4.将GitHub仓库更新到本地
git pull origin master
	# 4.1 git fetch:相当于是从远程获取最新版本到本地，不会自动合并
		git fetch origin master:tmp
	  	git diff tmp //比较两次修改的差异
		git merge tmp
	# 4.2 git pull:相当于是从远程获取最新版本并merge到本地
		git pull origin master
		
# 5.将本地仓库代码提交到远程仓库
git push origin master

# 6.向文件中写内容
vim test.md

# 7.删除文件
rm -rf test.java

# 8.将文件从暂存区中删除
git rm test.java

# 9.将文件从仓库中删除
git commit -m'描述消息'

```





