Git 子仓库（Submodule）的使用
在 Git 中，子仓库（Submodule）是一种将一个 Git 仓库作为另一个 Git 仓库的子目录的方式。这允许你将一个项目包含到另一个项目中，同时保持各自的版本控制和提交历史。子仓库非常适合管理外部库或共享组件。

添加子仓库

要在现有的 Git 仓库中添加子仓库，可以使用 git submodule add 命令。这个命令后面需要跟上子仓库的 URL 和你希望将子仓库克隆到的路径。例如：

git submodule add https://github.com/chaconinc/DbConnector DbConnector
复制
执行这个命令后，Git 会将子仓库克隆到指定的路径，并在主仓库中创建一个 .gitmodules 文件，其中记录了子仓库的 URL 和本地路径的映射关系。同时，主仓库的状态会显示有新的文件需要提交，这些文件包括 .gitmodules 和子仓库的目录。

更新子仓库

如果子仓库有更新，你可以进入子仓库的目录，使用 git pull 来获取最新的提交。如果你希望在主仓库中也跟踪这些更新，需要在主仓库中添加并提交子仓库的变更。

cd DbConnector
git pull
cd
git add DbConnector
git commit -m "Update DbConnector submodule"
复制
克隆包含子仓库的项目

当你克隆一个包含子仓库的项目时，默认情况下子仓库的目录会存在，但是里面不会有任何文件。你需要运行 git submodule init 来初始化本地配置文件，然后运行 git submodule update 来从子仓库中抓取所有数据并检出父项目中列出的合适的提交。

git clone https://github.com/chaconinc/MainProject
cd MainProject
git submodule init
git submodule update
复制
如果你希望在克隆时自动初始化和更新每一个子模块，可以给 git clone 命令传递 --recurse-submodules 选项。

git clone --recurse-submodules https://github.com/chaconinc/MainProject
复制
删除子仓库

如果需要从项目中删除子仓库，可以使用 git submodule deinit 命令来反初始化子模块，然后使用 git rm 删除子模块的路径。最后，手动删除 .gitmodules 文件和 .git/modules 目录下的子模块相关数据。

git submodule deinit -f path/to/submodule
git rm -f path/to/submodule
rm -rf .git/modules/path/to/submodule
复制
子仓库的常见问题

使用子仓库时可能会遇到一些问题，例如在不同分支间切换时处理子仓库的状态，或者将子目录转换为子模块时的处理。从 Git 2.13 版本开始，可以使用 --recurse-submodules 选项来简化这些操作。