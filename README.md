# Docs
知识笔记~



### GIT使用说明

- 修改笔记时：不能直接修改master分支的文件，如果要修改文件，必须新创建分支，修改完成后再合并到主分支。

- 创建新笔记时：每次创建空分支，笔记完成后合并到master分支
  1. 先从Empty分支中创建分支，笔记完成后将最后一次commit提交到master分支
     1. checkout到Empty分支，然后选择Add Branch，输入分支名，点击Add Branch & Checkout。
     2. 在该分支中创建、编辑文件，完成后在该分支中commit并push到远程库。
     3. checkout到master分支，然后点击Branch->Cherry-pick，然后在Branches按钮中选择要显示的分支，之后选择要合并到master的一次commit，然后点击Cherry-Pick & commit。