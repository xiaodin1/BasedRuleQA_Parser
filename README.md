# BasedRuleQA_Parser
基于规则匹配的问答系统中的解析器

# 带限定词库的规则匹配
在用规则做QA时，有时需要对相同句子结构的问法做不同的意图分类。例如：“李白是谁”需要分类到意图“问诗人简介”、“李鹏是谁”需要分类到意图”问政治名人简介“。

如果通过正则或机器学习（深度学习）文本分类，只能首先定义成一种意图（问人物简介），然后再对提取到的关键词再进一步通过数据区分为“问诗人简介”、”问政治名人简介“。

本DEMO仿照思必弛平台规则，在带限定词库的规则匹配方式下，定义规则”#诗人#是谁“、”#政治名人#是谁“两条规则，分别对应成“问诗人简介”、”问政治名人简介“两种意图即可，这样通过词库（#诗人#、#政治名人#词库）限制可以直接给出意图类别。

同时可以根据规则自动生成问句，用来给关键词提取模型训练制作语料。

# 效率实测
通过实测1W多条规则（扩展20W条规则），在个人电脑上全部未命中大概300ms左右，耗时跟规则数以及复杂度成正比（扩展词库查询通过词典全部加载进内存基本上不耗时）。

（注：单条规则”[请问|请告诉我]#诗人#的简介"可扩展出三条规则”#诗人#的简介"、”请问#诗人#的简介"、”请告诉我#诗人#的简介"，这代表了规则的复杂度。)


# DEMO输出
```
解析规则：#sys.任意文本##诗人#[的]#诗名#的(介绍|说明|#歌曲#)[啊|哦|#呵呵#|额]

=========================匹配句子=================================
句子：李白的将进酒的介绍哦
关键词：['', '李白', '将进酒']
关联库：['sys.任意文本', '诗人', '诗名']
位置：[(0, 0), (0, 2), (3, 3)]
节点路径：(FULL)-->sys.任意文本(LIB)-->诗人(LIB)-->的(FULL)-->诗名(LIB)-->的(FULL)-->介绍(FULL)-->哦(FULL)
=================================================================


=========================匹配句子=================================
句子：李白将进酒的介绍
关键词：['', '李白', '将进酒']
关联库：['sys.任意文本', '诗人', '诗名']
位置：[(0, 0), (0, 2), (2, 3)]
节点路径：(FULL)-->sys.任意文本(LIB)-->诗人(LIB)-->(FULL)-->诗名(LIB)-->的(FULL)-->介绍(FULL)-->(FULL)
=================================================================


=========================匹配句子=================================
句子：我请问李白冰将进酒的介绍
关键词：['我请问', '李白冰', '将进酒']
关联库：['sys.任意文本', '诗人', '诗名']
位置：[(0, 3), (3, 3), (6, 3)]
节点路径：(FULL)-->sys.任意文本(LIB)-->诗人(LIB)-->(FULL)-->诗名(LIB)-->的(FULL)-->介绍(FULL)-->(FULL)
=================================================================


=================================================================
匹配失败:李白是谁
=================================================================

解析规则：#诗人#是谁

=========================匹配句子=================================
句子：李白是谁
关键词：['李白']
关联库：['诗人']
位置：[(0, 2)]
节点路径：(FULL)-->诗人(LIB)-->是谁(FULL)
=================================================================

解析规则：[请问]#sys.数字#个人是什么字

=========================匹配句子=================================
句子：三个人是什么字
关键词：['三']
关联库：['sys.数字']
位置：[(0, 1)]
节点路径：(FULL)-->(FULL)-->sys.数字(LIB)-->个人是什么字(FULL)
=================================================================


=========================匹配句子=================================
句子：请问三个人是什么字
关键词：['三']
关联库：['sys.数字']
位置：[(2, 1)]
节点路径：(FULL)-->请问(FULL)-->sys.数字(LIB)-->个人是什么字(FULL)
=================================================================


=========================匹配句子=================================
句子：请问十二个人是什么字
关键词：['十二']
关联库：['sys.数字']
位置：[(2, 2)]
节点路径：(FULL)-->请问(FULL)-->sys.数字(LIB)-->个人是什么字(FULL)
=================================================================


======================生成句子=====================================
句子：脴梌李白沁园春的海阔天空额
关键词：['脴梌', '李白', '沁园春', '海阔天空']
关联库：['sys.任意文本', '诗人', '诗名', '歌曲']
位置：[(0, 2), (2, 2), (4, 3), (8, 4)]
节点路径：(FULL)-->sys.任意文本(LIB)-->诗人(LIB)-->(FULL)-->诗名(LIB)-->的(FULL)-->歌曲(LIB)-->额(FULL)
=================================================================


=========================匹配句子=================================
句子：脴梌李白沁园春的海阔天空额
关键词：['脴梌', '李白', '沁园春', '海阔天空']
关联库：['sys.任意文本', '诗人', '诗名', '歌曲']
位置：[(0, 2), (2, 2), (4, 3), (8, 4)]
节点路径：(FULL)-->sys.任意文本(LIB)-->诗人(LIB)-->(FULL)-->诗名(LIB)-->的(FULL)-->歌曲(LIB)-->额(FULL)
=================================================================

```

# 后续问题
1. 系统集成词库暂时只加入了“#sys.任意问题#”，“#sys.数字#”，后续再加入其他内置库;
2. 效率始终是一个需要进一步优化的问题;
3. 花了两天时间把想法实现，解析部分代码有点乱，老想的C的指针去构建图，需要重构一下，匹配部分还算明确，BUG慢慢磨。

# 20200313
已上线到实际项目中，大致思路相同：
一是可以将库拆分;
二是可以使用PyPy，或换成更高性能的语言实现，例如C、go;
三是仿照数据库加入检索方式。
