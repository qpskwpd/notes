### 上传/更新项目或文件夹的流程

1. 首先下载并安装git

2. 首次使用设置用户名和邮箱

   ```shell
   git config --global user.name "xxx"
   git config --global user.email "xxx"
   ```

3. 在本地建立一个repository，即一个文件夹，将项目/需要上传的文件夹复制进来

4. 使用`cd`命令进入该文件夹，执行`git init`将目录变成git管理目录

5. 输入`git add .`，把新增文件添加到暂存区，以备提交。

6. 为方便上传不用每次输入密码，可以使用`ssh-keygen`生成一个sshkey，在网页上添加

7. 输入`git commit -m "描述"`生成一个新的提交

8. 连接远程的要上传的仓库，`git remote add origin github仓库地址`

   `git remote rm origin`删除已存在的远程跟踪。

9. 输入`git push --set-upstream origin master/其他分支`

   `git pull origin master`如果远程分支存在内容，需要先拉取到本地。

