---
layout: post
title: Emacs shortcuts
subtitle: Emacs常用快捷键
image: /ico/icon-emacs.png
tags: [emacs, org, LaTeX]
---

## 1. Universal emacs shortcuts
### General shortcuts

| Shortcuts        | Description         |
| :--------------- | :------------------ |
| C-x C-c          | 关闭Emacs           |
| C-g              | 取消执行            |
| C-SPC            | 设置mark            |
| M-x              | 执行Emacs命令       |
| M-!              | 执行外部shell命令   |
| C-u M-! date     | 插入当前时间        |
| [M-x multi-term] | 在Emacs中打开多终端 |

### Files

| Shortcuts | Description |
| :-------- | :---------- |
| C-f       | 打开文件    |
| C-x C-s   | 保存文件    |
| C-x s     | 另存为文件  |

### Cursor moves

| Shortcuts      | Description    |
| :------------- | :------------- |
| C-p            | 光标上移一行   |
| C-n            | 光标下移一行   |
| C-f            | 光标右移       |
| C-b            | 光标左移       |
| M-f or C-RIGHT | 光标右移一字   |
| M-b or C-LEFT  | 光标左移一字   |
| C-a            | 光标移至行头   |
| C-e            | 光标移至行尾   |
| M-<            | 光标移至文档头 |
| M->            | 光标移至文档尾 |
| C-v            | 向下翻页       |
| M-v            | 向上翻页       |

### Edit

| Shortcuts    | Description      |
| :----------- | :--------------- |
| M-DEL        | 往回删除一字     |
| M-d          | 往后删除一字     |
| C-k          | 从光标处删至行尾 |
| C-x u or C-/ | 撤销             |
| M-w          | 复制             |
| C-w          | 剪切             |
| C-y          | 粘贴             |
| C-s          | 向后搜索         |
| C-r          | 向前搜索         |
| C-x h        | 全选             |
| M-%          | 替换             |

### Windows

| Shortcuts      | Description                        |
| :------------- | :--------------------------------- |
| C-x 0          | 关掉当前的视窗                     |
| C-x 1          | 使当前的视窗占满窗口               |
| C-x o [or C-\] | 切换到另一个视窗                   |
| C-x 2          | 把当前的视窗水平分割成上下两个视窗 |
| C-x 3          | 把当前的视窗垂直分割成左右两个视窗 |
| C-x 5 2        | 新建一个窗口                       |
| C-x C-b        | 查看并切换至已打开的buffer         |
| C-x b          | 切换至已打开的buffer               |
| [C-`]          | 切换至下一个标签页                 |

### Bookmarks

| Shortcuts           | Description              |
| :------------------ | :----------------------- |
| C-x r l             | 查看书签列表并切换至书签 |
| C-x r b             | 切换至书签               |
| C-x r m             | 保存书签                 |
| M-x bookmark-delete | 删除书签                 |

## 2. org-mode shortcuts
### General shortcuts

| Shortcuts    | Description                                              |
| :----------- | :------------------------------------------------------- |
| TAB          | 切换光标所在大纲的显示状态，即折叠，打开下一级，打开全部 |
| S-TAB        | 切换整个文档的显示状态，即折叠，打开下一级，打开全部     |
| M-RET        | 插入一个同级标题或列表                                   |
| M-LEFT/RIGHT | 将当前标题升/降级                                        |
| C-c C-x C-h  | 查看快捷键                                               |
| C-c C-l      | 创建或修改链接                                           |
| C-c C-e      | 导出                                                     |
| S-LEFT/RIGHT | 切换标题状态，即DONE，TODO，无                           |

### Tables

| Shortcuts    | Description                    |
| :----------- | :----------------------------- |
| C-c RET      | 在输入表头之后生成表格结构     |
| C-c RET      | 添加水平分割线并跳到下一行     |
| C-c C-c      | 调整表格缩进                   |
| C-c ^        | 根据当前列排序                 |
| TAB          | 移动到下一区域，必要时新建一行 |
| S-TAB        | 移动到上一区域                 |
| M-S-LEFT     | 删除当前列                     |
| M-S-RIGHT    | 在当前列的右侧新建列           |
| M-S-UP       | 删除当前行                     |
| M-S-DOWN     | 在当前行的上侧新建行           |
| C-c SPC      | 清空当前单元格                 |
| M-LEFT/RIGHT | 将当前列向左/右移动            |
| M-UP/DOWN    | 将当前行向上/下移动            |

### Tags

| Shortcuts | Description                                  |
| :-------- | :------------------------------------------- |
| C-c C-c   | 在#+TAGS处刷新元数据，在标题处创建或编辑标签 |
| C-c / m   | 搜索标签并按树状结构显示                     |

## 3. AUCTex

| Shortcuts        | Description              |
| :--------------- | :----------------------- |
| C-c C-s          | 插入section              |
| C-c C-j or M-RET | 插入item                 |
| C-c C-e          | 插入environment          |
| C-c ;            | 注释或取消注释已选中区域 |
| C-c C-c          | 对主文件执行编译命令     |
| C-c C-k          | 停止任务                 |
