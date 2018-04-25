# git command 使用总结

## 1. git cherry-pick

单个合并：
git cherry-pick <commit id>

批量合并：
git cherry-pick <start-commit-id>..<end-commit-id>
git cherry-pick <start-commit-id>^..<end-commit-id>
前者表示把<start-commit-id>到<end-commit-id>之间(左开右闭，不包含start-commit-id)的提交cherry-pick到当前分支；
后者有"^"标志的表示把<start-commit-id>到<end-commit-id>之间(闭区间，包含start-commit-id)的提交cherry-pick到当前分支。
其中，<start-commit-id>到<end-commit-id>只需要commit-id的前6位即可，并且<start-commit-id>在时间上必须早于<end-commit-id>
注：以上合并，需要手动push代码。

参考来源
https://www.jianshu.com/p/08c3f1804b36


git checkout topic-sas-4.16-dts2018011702854-txf
git push origin topic-sas-4.16-dts2018011702854-txf:topic-sas-4.16-dts2018011702854-txf