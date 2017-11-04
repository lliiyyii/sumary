## 问题描述
To https://github.com/lliiyyii/xxx.git
 ! [rejected]        master -> master (fetch first)
error: failed to push some refs to 'https://github.com/lliiyyii/xxx.git'
hint: Updates were rejected because the remote contains work that you do
hint: not have locally. This is usually caused by another repository pushing
hint: to the same ref. You may want to first integrate the remote changes
hint: (e.g., 'git pull ...') before pushing again.
hint: See the 'Note about fast-forwards' in 'git push --help' for details.

## 解决方法
git pull --rebase https://github.com/lliiyyii/xxx.git master

## 问题原因
git上的readme没有在本地目录中

