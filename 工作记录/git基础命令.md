//配置git仓库全局用户名以及邮箱
git config --global user.name "你的名字"
git config --global user.email "你的邮箱@example.com"
//查看当前的配置
git config --list
//初始化仓库
git init
//查看状态
git status
//添加单个文件
git add 文件名
//提交到本地仓库
git commit -m "提交说明"
//查看详细历史（按 q 键退出）
git log
//查看简洁版历史
git log --oneline
//查看当前所有分支（带 * 号的是当前所在分支）
git branch
//切换到其他的分支
git switch -c feature-login 
//切换回主分支
git switch main
//将开发好的分支合并到主分支
git merge feature-login
//删除分支
git branch -d feature-login


//回退到上一个版本
git reset --soft HEAD^

//回退到上一个版本
git reset --hard HEAD^

// 回退到指定的历史版本（通过 git log --oneline 获取版本号的哈希值）
git reset --hard a1b2c3d
