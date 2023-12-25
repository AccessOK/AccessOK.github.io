# READEME
This is my blog source!
## 配置索引
引用Hexo Configuration
Docs: https://hexo.io/docs/configuration.html
Source: https://github.com/hexojs/hexo/

## 目录
source_dir: source
public_dir: public
tag_dir: tags
archive_dir: archives
category_dir: categories
code_dir: downloads/code
i18n_dir: :lang
skip_render:
## 日期格式配置
date_format: YYYY-MM-DD
time_format: HH:mm:ss
updated_option: 'mtime'
## 主题配置
theme: maupassant
注: 若使用hexo s 之后提示index.html查找失败信息，请先确保themes/maupassant目录下是否有数据。
若该目录下没有数据，请先克隆下相关数据。
```bash
git clone git@github.com:AccessOK/maupassant-hexo.git themes/maupassant
```
## 发布到git
deploy:
  type: git
  repo: git@github.com:AccessOK/AccessOK.github.io.git
  branch: main
注： 

feed:
  type: atom
  path: atom.xml
  limit: 0 # 最大文章數量
  content: false # 是否顯示內文

