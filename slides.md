---
theme: seriph
class: text-center
highlighter: shikiji
lineNumbers: false
drawings:
  persist: false
transition: slide-left
title: Web 信息处理与应用实验一
colorSchema: 'light'
mdc: true
---

# Web 信息处理与应用实验一

翁屹禾 傅申 侯博文

<!--
The last comment block of each slide will be treated as slide notes. It will be visible and editable in Presenter Mode along with the slide. [Read more in the docs](https://sli.dev/guide/syntax.html#notes)
-->

---
transition: fade-out
---

# 阶段一：爬虫

<v-click>

- 使用 `lxml` 库解析每个页面的 HTML，获取相关信息
- 将提供的 tag 信息加入到爬取的信息中，保存至 JSON 文件

</v-click>
<v-click>

- 反爬策略：
    - 部分页面只有登录后才可以查看 $\to$ 使用 Cookie 模拟登录
    - 查询频率过高会导致封禁 $\to$ 爬取过程加入随机停顿

</v-click>

---

# 阶段一：数据预处理

<v-clicks depth="1">

- 将数据转换为**关键字** $\to$ **ID** 的映射，保存到倒排索引表中。
    - 使用 `jieba` 库进行分词，并去除停用词
    - 考虑到实验的规模与需求，使用 `set` 作为每个表项的数据结构
- 将倒排索引表组织成数据库，写入文件系统
    ```txt
    database
    └── book
        ├── all_ids            所有 ID 的集合
        ├── indices            倒排索引表中关键字与其条目的对应关系
        └── index              所有关键字的倒排索引表项
            ├── keyword1.idx   keyword1 的 doc_ids
            ├── keyword2.idx   keyword2 的 doc_ids
            ├── ...
            └── keywordn.idx   keywordn 的 doc_ids
    ```
    - 使用 `get(keyword)`，`get_size(keyword)` 来进行基本的查询操作
    - 支持按块压缩和 Lazy Loading

</v-clicks>

---

# 阶段一：布尔查询

<v-clicks depth="1">

- 使用下面的语法对布尔查询进行解析
    $$
    \begin{aligned}
        \mathrm{Query}  & \to \mathrm{Term} \  (\mathbf{OR} \  \mathrm{Term})* \\
        \mathrm{Term}   & \to \mathrm{Factor} \  (\mathbf{AND} \  \mathrm{Factor})* \\
        \mathrm{Factor} & \to \mathbf{NOT}* \  \mathbf{LPAREN} \  \mathrm{Query} \  \mathbf{RPAREN} \  | \  \mathbf{NOT}* \  \mathbf{KEYWORD}
    \end{aligned}
    $$
    - `A AND (B OR C OR NOT D) AND NOT E` $\to$ `AND(A, OR(B, C, NOT(D)), NOT(E))`
- 使用 `Query.parse(query_str)` 解析布尔查询 <br>
  使用 `query.query(database)` 获取查询结果

</v-clicks>

---

# 阶段一：总结

<v-clicks>

1. 加载数据库
    ```python
    movie_database = InvertedIndex.load(MOVIE_DATA_DIR)
    book_database = InvertedIndex.load(BOOK_DATA_DIR)
    ```
2. 解析查询语句
    ```python
    query_str = input("Query (Press CTRL+C to exit): ")
    try:
        query = Query.parse(query_str)
    except Exception as e:
        print("Invalid query:", e)
        continue
    ```
3. 查询并展示
    ```python
    query_books = query.query(book_database)
    query_movies = query.query(movie_database)
    display_results(query_books, query_movies)
    ```

</v-clicks>

---

# 阶段一：总结
\
运行效果：

```txt {1|1-3|1-11|1-13}
$ python main.py
Example query: A AND B AND NOT C / (A OR B) AND C
Query (Press CTRL+C to exit): 信息 AND 科幻
Found books:
- 《三体》 by 刘慈欣: https://book.douban.com/subject/2567698
Found movies:
- 《变形金刚 Transformers》(动作, 科幻): https://movie.douban.com/subject/1794171
- 《变形金刚2 Transformers: Revenge of the Fallen》(动作, 科幻, 冒险): https://movie.douban.com/subject/2146991
- 《侏罗纪公园 Jurassic Park》(科幻, 惊悚, 冒险): https://movie.douban.com/subject/1292523
- 《我，机器人 I, Robot》(动作, 科幻, 悬疑, 惊悚): https://movie.douban.com/subject/1308843
--------------------------------------------
Query (Press CTRL+C to exit): ^C
exiting...
```

---

# 阶段二：推荐

<div style="display: flex; justify-content: space-between; flex-direction: column; height: 90%;">

<div>
<li>
    使用 GraphRec, Text Embedding 和 LightGCN<sup>[1]</sup> 三种模型进行推荐
    <li>
        提取 LightGCN-Pytorch<sup>[2]</sup> 中的模型代码，并进行修改
    </li>
</li>
<li>比较三种模型的训练速度和预测效果</li>
</div>


<!-- refs --->
<div style="font-size: 8pt;">
<hr color="black"/>

\[1\] [https://arxiv.org/abs/2002.02126](https://arxiv.org/abs/2002.02126) <br>
\[2\] [https://github.com/gusye1234/LightGCN-PyTorch](https://github.com/gusye1234/LightGCN-PyTorch)

</div>
</div>

---
layout: center
---

# Thanks
