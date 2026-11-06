# 45 多重迷影 · Misc

- 分类：Misc ｜ 分值：200 ｜ 状态：平台已解
- 附件是一串 emoji，四层套娃：码位偏移 → base64 → base32 → hex → 凯撒。

## 题目

- 题号 45，题名「多重迷影」，题面「不识庐山真面目，只缘身在此山中」，出题人 peipei。
- 附件 `多重迷影.zip`，解压得到 `多重迷影.txt`，内容 88 个 emoji。

## 思路

- 先统计：88 个字符全部落在 `U+1F427`–`U+1F470` 区间，说明是按码位偏移编码的可打印字符。
- 试偏移：`cp - 0x1F3F7` 得到的字节恰好全是 ASCII，是一段以 `==` 结尾的 base64。
- 依次解 base64 → base32 → hex，得到 `l3v{atox_t_gbvx_wtr}`，看着像 `s3c{...}` 但整体错位 7 位，凯撒右移 7 即还原。

## 关键步骤

```python
import base64

s = open('多重迷影.txt', encoding='utf-8').read()
cps = [ord(c) for c in s if ord(c) > 0x2000]

raw = bytes(c - 0x1F3F7 for c in cps)      # -> base64 文本
l2 = base64.b64decode(raw).decode()        # -> base32 文本
l3 = base64.b32decode(l2).decode()         # -> hex 文本
l4 = bytes.fromhex(l3).decode()            # -> l3v{atox_t_gbvx_wtr}
flag = ''.join(chr((ord(c)-97+7) % 26 + 97) if c.isalpha() else c for c in l4)
```

实测四层输出：

![四层解码](img/01-decode.png)

## Flag

```text
s3c{have_a_nice_day}
```

## 踩坑 / 备注

- 偏移值不是常见的 `0x1F000` 一类整数，`0x1F3F7` 是通过「让首字节落在可打印 ASCII」反推出来的，可以写个小循环暴力试 0x1000 个偏移。
- 每一层都要求「解出来仍是可打印文本」再继续，否则很容易在错的分支上越走越远。
