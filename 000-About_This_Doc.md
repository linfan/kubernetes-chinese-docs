# Kubernetes文档翻译小组约定

# Kubernetes文档翻译小组约定

## 文档地址

1. 翻译任务认领表：<br>
https://shimo.im/doc/YhENOgq2txyaEQF9
<br>按章节认领，在这个表格中看中章节的“翻译”一栏添上自己的名字即可。

2. Gitbook编辑地址：<br>
https://www.gitbook.com/book/linfan1/kubernetes-chinese-docs/edit
<br>请首先登陆Gitbook，并确保对这个文档有编辑权限。

3. Git Repo地址：<br>
想用Git直接拉取下来编辑的话，可以用下面这个命令哦。
```
git clone https://git.gitbook.com/linfan1/kubernetes-chinese-docs.git
```

## 翻译约定

1. 每个小章节单独建一个文件，文件名按照“__三位数标号-章节英文名.md__”命名。可以参考已经有的文件，文件名记得加上.md后缀。

2. 界面上面“Table Of Contents”部分先不管，这部分内容会不定期进行统一布局和整理。

3. 不论原本内容有没有标题，麻烦大家在每个小节的**第一行**都写个标题，方便后期整理目录。可以参考已有的文件内容。

4. 内容格式按照DockOne的[文章规范](Fingerpost.txt)，基本规范列举如下：
    - 中文部分统一使用中文全角字符，如『』、“”、（）等
    - Markdown格式统一使用英文字符，如[]、()、#等
    - 英文单词前后不需要留空格
    - 注意专用词汇的大小写，如Pod、ReplicationControllers

5. 内容中的链接引用，分为以下两种情况处理：
    - **引用到同一篇文档其他位置的链接**：可以改为叙述方式，例如『参见下文XXX部分内容』
    - **引用到文档以外地址，包括引用另一篇文档的链接**：统一保留原始链接地址，后续再统一进行处理
   所有引用统一采用Markdown的[引用格式](https://github.com/adam-p/markdown-here/wiki/Markdown-Cheatsheet#links)，不应该写成上标的方式。

##名词列表

文档中出现的下列名词统一不翻译，而且文档中全部使用单数形式：

- ***Pod***
- ***Service***
- ***Replication Controller***
- ***Label***
- ***Selector***
- ***Volume***
- ***Secret***
- ***Name***
- ***Namespace***
- ***Annotation***
