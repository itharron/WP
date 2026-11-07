# 22 UmbrellaLog · Misc

- 分类：Misc ｜ 分值：500 ｜ 状态：平台已解
- 一句话概括考点：日志取证——按 `incident.log` 给出的规则做字段过滤、状态去重、时间排序，并剔除清场之后补写的伪造记录。

## 题目

- 附件 `logs.zip`（1198 字节，`StaticAttachment` 直链下载），内含 `incident.log`（439）、`security.log`（329）、`virology.log`（1342）。
- 题面：浣熊市地下实验室断电前，保护伞的监控系统留下了几份日志，你能找到真正进入电梯逃逸链的 T 病毒样本吗 ——Sh1y1。
- `virology.log` 里每条样本都带一个 8 位十六进制的 `codon=`，本质就是 ASCII 十六进制。
- 先做过"防藏东西"检查：zip 无注释、EOCD 后无附加字节、三个日志纯 ASCII、LF 统一、无零宽字符——没有隐写，纯考逻辑。

## 思路

- 关键规则全写在 `incident.log` 里，题面只是引子：

```
23:16:05 first confirmed escape trace from ward=B2
23:17:00 collect only ACTIVE T-VIRUS samples with viral_load >= 95
23:20:00 PURGE: all remaining samples destroyed
23:21:44 NOTE: G-VIRUS and NE-ALPHA records are not part of this incident chain
```

- 于是过滤条件是 `project=T-VIRUS` + `ward=B2` + `status=ACTIVE` + `viral_load>=95` 四者同时成立。
- 每个 `codon` 单独 `xxd -r -p` 解出来就是 flag 的一个片段（`7333637b` → `s3c{`），拼装顺序必须按时间戳，不能按文件行序。
- 时间线上还有一层陷阱：`PURGE` 是 23:20:00，之后的记录一定是伪造的。

## 关键步骤

1. 解压，先 `ls -l` 看三个文件的大小，再 `cat incident.log`、`cat security.log` 通读一遍，确认 `ELEVATOR-CAM` 指向 B2、规则出自 `incident.log`。

2. `cat virology.log`，13 条样本一行一条，`codon=` 就是待解码的十六进制：

   ![virology.log 全 13 条样本，codon 即十六进制 ASCII](img/01-virology.png)

3. 在终端按条件逐层过滤。先卡项目和 ward，再卡状态：

   ```bash
   grep 'project=T-VIRUS' virology.log | grep 'ward=B2' | grep 'status=ACTIVE'
   ```

   剩 9 条（`RC-033` 是 `DORMANT`，被这一步去掉）。再按 `viral_load>=95` 去掉 `load=94` 的那条：

   ```bash
   grep 'project=T-VIRUS' virology.log | grep 'ward=B2' | grep 'status=ACTIVE' | grep -v 'viral_load=9[0-4]'
   ```

4. 最后剔除 PURGE 之后补写的伪造记录：

   ```bash
   grep 'project=T-VIRUS' virology.log | grep 'ward=B2' | grep 'status=ACTIVE' | grep -v 'viral_load=9[0-4]' | grep -v '23:20:'
   ```

   剩下 7 条，按时间戳排序是 `RC-031 → RC-032 → RC-034 → RC-035 → RC-037 → RC-038 → RC-039`。注意文件行序不完全等于时间序（`RC-404` 排在第二行，时间却是 23:16:51），所以不能直接按行拼接。

5. 逐条把 `codon` 解成字符，比如第一条：

   ```bash
   echo 7333637b | xxd -r -p
   ```

   输出 `s3c{`。其余六条同样处理，依次得到 `umbr`、`ell4`、`_l0g`、`s_n3`、`ver_`、`lie}`。

6. 七段按时间顺序接起来，就是完整 flag：

   ![过滤 + 排序后的 7 条与拼接结果](img/02-filter.png)

## Flag

```text
s3c{umbrell4_l0gs_n3ver_lie}
```

## 踩坑 / 备注

- `RC-033` 与 `RC-034` 的 codon 完全相同（都是 `ell4`），但前者 `status=DORMANT`，不看状态会拼出 `...umbrell4ell4...`。
- 干扰项都做成了梗：`RC-404 G-VIRUS`→`fake`、`RC-117 ward=B1`→`nope`、`RC-036 load=94`→`_fak`、`NE-010`→`wesk`。
- `RC-040`（23:20:09，PURGE 后 9 秒）→`too_`，且 `RC-039` 已给出闭合的 `lie}`，从结构上也不可能接在后面。
