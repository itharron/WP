# 50 Misc入门指北 · Misc

- 分类：Misc ｜ 分值：200 ｜ 状态：平台已解
- 一份 4 页的 PDF，每页藏一段 flag 碎片，拼装顺序写在文档属性的 `Subject` 里。

## 题目

- 题号 50，题名「Misc入门指北」，题面「听说看完这个可以成为Misc大手子 ——Sh1y1」。
- 附件 `Misc入门指北.pdf`，4 页，内容确实是 Misc 入门教学。

## 思路

- 打开附件翻一遍，正文是正常教程，但每页最后一行都有个 `[memo]` 开头的小尾巴——四段碎片。
- 碎片本身是乱序的：`s7ar7ed_w1th_`、`s3c{C0ngrats_`、`M1sc}`、`0n_gett1ng_`，直接拼不成话。
- 第 3 页正文里写了「顺序记在文档属性里」，去读 PDF 元数据，`Subject = memo-order: 2,4,1,3`。
- 按 2→4→1→3 拼起来就是完整 flag。

## 关键步骤

1. 用阅读器打开 PDF，从第 1 页翻到第 4 页，记下每页末尾 `[memo]` 后面的那截字符串。第 2 页的碎片是 `s3c{C0ngrats_`：

   ![PDF 第 2 页](img/02-page2.png)

2. 翻到第 3 页，正文里明写了「顺序记在文档属性里」，而这一页自己的碎片是 `M1sc}`：

   ![PDF 第 3 页](img/03-page3.png)

3. 回到文件本身查属性。终端里用 `pdfinfo`：

   ```bash
   pdfinfo "Misc入门指北.pdf"
   ```

   输出的 `Subject` 一行是 `memo-order: 2,4,1,3`；用图形阅读器的话就是「文件 → 属性 → 描述」，同样能看到。

4. 想要按页取出四段碎片，省得手抄，也可以用：

   ```bash
   pdftotext -layout "Misc入门指北.pdf" - | grep memo
   ```

   四行依次是 `s7ar7ed_w1th_`、`s3c{C0ngrats_`、`M1sc}`、`0n_gett1ng_`。

5. 按 `2,4,1,3` 的顺序手工拼：

   ```text
   第 2 段  s3c{C0ngrats_
   第 4 段  0n_gett1ng_
   第 1 段  s7ar7ed_w1th_
   第 3 段  M1sc}
   ```

   接起来就是 `s3c{C0ngrats_0n_gett1ng_s7ar7ed_w1th_M1sc}`。

## Flag

```text
s3c{C0ngrats_0n_gett1ng_s7ar7ed_w1th_M1sc}
```

## 踩坑 / 备注

- 四段碎片直接按页序拼是 `s7ar7ed_w1th_s3c{C0ngrats_M1sc}0n_gett1ng_`，一眼假；顺序线索只在元数据里。
- 有些阅读器不显示 `Subject`，`pdfinfo` 最稳；`pdftotext` 提取时 PDF 里没有真正的空格分词，`[memo]` 行可能被拆开，用 `grep memo` 而不是整行等值匹配。
