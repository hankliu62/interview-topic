<!--
 * @owner: hank.liu
 * @team: 卡鲁秋
-->

# Git 面试知识点总结

本部分主要是笔者在复习 Git 相关知识和一些相关面试题时所做的笔记，主要是个人复习使用，如果出现错误，希望并感谢大家指出，如总结答案与标准有出入，请轻喷，谢谢🙏


## git rebase 和 git merge 有啥区别？

- git rebase: 会把你当前分支的 commit 放到公共分支的最后面,所以叫变基。
- git merge: 会把公共分支和你当前的commit 合并在一起，形成一个新的 commit 提交

## git fetch 和 git pull 有啥区别？

- git pull: 相当于是从远程获取最新版本并merge到本地
- git fetch: 相当于是从远程获取最新版本到本地，不会自动merge