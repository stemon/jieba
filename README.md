### jieba in spark

build jieba Tokenizer in Worker from memory.

### Build jieba instance in Driver

```python
import jieba
# initialize segment model
cut_model = jieba.Tokenizer()
cut_model.initialize()
FREQ, total, initialized = cut_model.dump_kernel()
```

and broadcast:

```python
# 使用两个广播变量来广播jieba的两个模型数据
broadcast_FREQ = sc.broadcast(FREQ)
broadcast_total = sc.broadcast(total)
```

### Read broadcast variable in Worker

```python
seg_model = jieba.Tokenizer(FREQ=broadcast_FREQ.value, total=broadcast_total.value)
[w for w in seg_model.cut(text, HMM=True) if len(w) > 1]
```


### Example

```python
# -*- coding: utf-8 -*-

import jieba

#确定不与site-package里的jieba混淆
print jieba.__file__

def main():
    # initialize segment model
    cut_model = jieba.Tokenizer()
    cut_model.initialize()

    cut_list = cut_model.cut("王者荣耀和绝地求生是两款很火的游戏！")
    print "/ ".join(cut_list)

    cut_model.add_word("王者荣耀")
    cut_model.add_word("绝地求生")
    FREQ, total, initialized = cut_model.dump_kernel()
    seg_model = jieba.Tokenizer(FREQ=FREQ, total=total)

    seg_list = seg_model.cut("王者荣耀和绝地求生是两款很火的游戏！")
    print "/ ".join(seg_list)

if __name__ == "__main__":
    main()
```
