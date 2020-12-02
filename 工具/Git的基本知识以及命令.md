# Git的基本知识以及命令



## Git仓库的三个区域

- 工作区：平时编写代码的存储位置
- 暂存区：写好的代码在提交之前，暂存的区域
- 历史版本区：代码被提交后，以一个个版本记录的形式，保存在版本库

![Git三大分区.png](https://user-gold-cdn.xitu.io/2018/8/9/1651f14704cb69b3?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)



## Git的基本命令

### 文件修改相关

```
// 创建git仓库
git init
```

```
// 将工作区的内容，添加进暂存区
git add xxx
git add .		# 把所有修改和新增（删除的不包含）的文件提交到暂存区
git add -u		# 把所有修改和删除（新增的不包含）的文件提交到暂存区
git add -A		# 把所有修改、新增、删除的文件提交到暂存区
```

```
// 查看当前工作区中文件的状态
git status
// 红色：在工作区中，没有提交到暂存区
// 绿色：在暂存区中，没有提交到历史区
```

如果在提交的时候，有些内容不想提交，我们可以增加git提交的忽略文件： `.gitignore` (没有文件名只有后缀名)

```
// .gitignore文件内容示例
.idea
target
log
logs
*.iml
```

```
// 把暂存区中的文件提交到历史区
git commit
git commit -m '注释内容'
// 把提交到暂存区和提交到历史区合并到一起完成。
// 只适合已经提交过一次的文件，被修改后可以快速提交。
// 但是对于新增的文件，一次都没有提交过的，是不允许这样操作的。
git commit -a -m '注释内容'
```

```
// 把暂存区的内容撤回工作区（覆盖现有工作区的内容，且无法找回）
git checkout .
// 只能在当前暂存区还未提交commit的时候生效，如果已提交，则暂存区和工作区一致，这个命令就没有效果
// 解决方法：
git reset HEAD .	// 在暂存区中，回滚到上一次暂存区中记录的内容
git checkout .		// 将经过回滚的暂存区，替换到工作区中
```

```
// 查看不同区域的代码的不同，可以基于GUI或者IDEA可视化界面来查看
git diff			// 工作区 -- 暂存区
git diff master		// 工作区 -- 历史区
git diff --cached	// 暂存区 -- 历史区
```

```
// 回滚
git reset --hard 版本号（commit id）
git reset --hard HEAD~	// 回滚至当前的上一个版本
git reset --hard HEAD^n	// 回滚至当前的上n个版本
// 查看版本号
git log
```

![img](https://user-gold-cdn.xitu.io/2019/8/12/16c84ff492a9de1a?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)

### 远程仓库

- 关联本地仓库和远程仓库

  ```
  git remote add {name}(如origin) {远程仓库地址}
  git remote add origin git@github.com:project/test.git
  git push github master
  git push gitee master
  
  git remote rm {name}	// 移除关联
  git remote -v	// 查看当前仓库和哪些远程仓库保持关联
  ```

- 指定本地分支与远程分支进行链接

  ```
  git branch --set-upstream dev origin/dev
  ```

- 同步本地历史区和远程仓库的信息

  ```
  // 将本地信息推送到远程仓库
  git push {name} {branch}
  git push origin master
  
  // 将远程仓库信息拉取到本地
  git pull {name} {branch}
  git pull origin master
  ```

- git fetch

  更新git remote中所有的远程仓库所包含分支的最新commit-id，将其记录到.git/FETCH_HEAD文件中

  ```git pull = git fetch + git merge```

  ```
  // 将某个远程主机的更新全部取回本地
  git fetch {远程主机名}
  // 取回指定分支
  git fetch {远程主机名} {分支名}
  git fetch origin master
  // 取回更新后，会返回一个FETCH_HEAD ，指的是某个branch在服务器上的最新状态，我们可以在本地通过它查看刚取回的更新信息：
  git log -p PETCH_HEAD
  // 我们可以通过这些信息来判断是否产生冲突，以确定是否将更新merge到当前分支。
  ```

  ```
  git fetch origin master
  git merge FETCH_HEAD
  // 以上效果等同git pull
  // 可以将远程分支取回（合并）到本地指定的分支（temp），如不存在则创建
  git fetch origin master:temp
  git pull origin master:temp
  ```

- 克隆远程仓库

  ```
  // 在本地创建仓库，并与远程仓库保持链接(origin)，也克隆了所有信息
  git clone 仓库地址 仓库别名(nullable)
  ```


- 查看远程仓库的信息

  ```
  git remote
  // 显示更详细的分支
  git remote -V
  ```



### 分支相关

- 创建、切换、删除分支

  ```
  // 查看所有分支以及当前所在分支
  git branch
  // 创建分支
  git branch dev
  // 切换到分支
  git branch dev
  // 创建并切换到分支
  git branch -b dev
  // 删除分支
  git branch -d dev
  ```

- 合并分支

  ```
  // 把当前分支合并到branch分支上
  git merge {branch}
  // master为发布版本的分支，禁止在此之上进行commit
  // 在dev、test等分支上进行开发、测试，并commit几个记录之后，将稳定的dev版本merge到master上以此来发布
  git merge master	// dev -> master
  ```

- 保存工作现场

  ```
  // 在切换至其他分支去修改之前，必须保存当前分支的工作内容
  git stash
  // 查看已经保存的工作内容
  git stash list
  // 恢复工作现场，并从stash list中删除
  git stash pop 或者 git stash apply
  // 恢复工作现场，并删除stash的内容（默认不删除）
  git stash drop
  // 恢复至指定的stash
  git stash apply stash@{0}（一种编号）
  ```



### 其他

- 创建标签

  ```
  git tag {name}
  git tag v1.0
  // 查看所有标签
  git tag
  ```



