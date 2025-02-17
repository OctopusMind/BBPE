# 手动实现BBPE (Byte-level Byte Pair Encoding)
## BBPE原理
BBPE（Bytewise Byte Pair Encoding）是一种字节级别的字节对编码（Byte Pair Encoding），主要用于将文本数据压缩或编码成更紧凑的表示形式。它的原理如下：
1. 初始化词汇表：开始时，BBPE 将每个字符都视为一个词汇。
2. 字节对频率统计：对输入文本进行扫描，统计所有相邻字节对的出现频率。
3. 合并频率最高的字节对：找到出现频率最高的字节对，然后将它们合并成一个新的字节对，并将这个新的字节对添加到词汇表中。
4. 重复步骤 2 和步骤 3：不断重复步骤 2 和步骤 3，直到达到指定的词汇表大小或者达到预设的迭代次数。
5. 编码文本：最终的词汇表包含了所有频率较高的字节对，然后将输入文本中的字节对替换为词汇表中对应的新词汇，从而完成编码。

## BBPE的优点
1. 高效的压缩效果：BBPE 可以根据文本中的重复模式和常见片段来动态地生成词汇表，从而实现高效的文本压缩，尤其适用于包含大量重复内容的文本数据。
2. 适用于多种类型的数据：BBPE 可以应用于各种类型的数据，包括文本数据、图像数据等，因为它是基于字节级别的编码方法。
3. 无损压缩：与许多基于统计方法的压缩算法相比，BBPE 是一种无损压缩算法，可以确保压缩后的数据与原始数据之间没有信息损失。
4. 可解码性：BBPE 使用动态生成的词汇表来编码文本数据，因此可以轻松地将编码后的数据解码回原始数据。
5. 灵活性：BBPE 的压缩效果和词汇表大小可以根据需求进行调整，以满足不同场景下的需求。

## 代码使用
### train.py 训练模型，主要构造vocab.json和merge.txt
1. vocab.json：  
  * 这个文件包含了由 BBPE 算法生成的词汇表（vocabulary），其中每个词汇都是一个字节序列或者子词（subword）。  
  * 词汇表中的每个词汇都与一个整数 ID 相关联，用于在编码和解码过程中进行索引和映射。  
  * 通常，vocab.json 中的词汇按照其频率（或者其他指标）进行排序，以便在 BBPE 算法中进行合并时能够优先选择频繁出现的子词。
2. merge.txt：  
  * 这个文件包含了 BBPE 算法在训练过程中执行的所有合并操作。  
  * 每行都代表一个合并操作，其格式通常是两个子词以及它们合并后的结果，以空格或其他分隔符分开。  
  * 例如，一行可能是 a b ab，表示在某一次迭代中，BBPE 算法将子词 a 和 b 合并成新的子词 ab。

### test.py 测试模型
```
from bbpe import BBPETokenizer
tokenizer = BBPETokenizer("models/斗破苍穹/vocab.json", "models/斗破苍穹/merges.txt")
encode_text = input("请输入要编码的字符串：")
print(f"{'#' * 20}编码'{encode_text}'字符串编码后的结果如下{'#' * 20}")
text_id = tokenizer.encode(encode_text)
id_text = tokenizer.decode(tokenizer.encode(encode_text))
text_token = tokenizer.tokenize(encode_text)
print(f"'{encode_text}'字符串编码之后对应的id: {text_id}")
print(f"'{encode_text}'字符串编码之后再反编码对应的字符串: {encode_text}")
print(f"'{encode_text}'字符串编码之后对应的token: {text_token}")
print('#' * 60)
# 批量编码
print("格式：list[str,……，str]，案例：['你好，你过得怎么样', '嘿，我今天过得很好!']")
batch_encode_text = input("请输入要编码的字符串（按上面格式）：")
batch_encode_text = eval(batch_encode_text)
batch_encode_res = tokenizer.encode_batch(batch_encode_text, num_threads=2)
print(f"#########################编码批量数据结果如下#######################")
print(f"批量编码的数据'{batch_encode_text}'")
print(f"批量编码的结果如下：{batch_encode_res}")
```
执行结果如下：
```
请输入要编码的字符串：中国
####################编码'中国'字符串的结果如下####################
'中国'字符串编码之后对应的id: [181, 269, 121]
'中国'字符串编码之后在反编码对应的字符串: 中国
'中国'字符串编码之后对应的token: ['ä¸Ń', 'åĽ', '½']
############################################################
格式：list[str,……，str]，案例：['你好，你过得怎么样', '嘿，我今天过得很好!']
请输入要编码的字符串（按上面格式）：['你好，你过得怎么样', '嘿，我今天过得很好!']
#########################编码批量数据结果如下#######################
批量编码的数据'['你好，你过得怎么样', '嘿，我今天过得很好!']'
批量编码的结果如下：[[202, 214, 409, 202, 365, 235, 286, 480, 182, 474, 278, 498, 115],
                    [206, 490, 123, 409, 295, 194, 476, 209, 365, 235, 234, 474, 214, 0]]
```
## llama3-8b 分词和qwen分词测试
把text.py代码中的BBPETokenizer("models/斗破苍穹/vocab.json", "models/斗破苍穹/merges.txt")文件名改成llama3和qwen的模型路径即可。  
测试结果和llama3和qwen原始模型分词结果一样。
