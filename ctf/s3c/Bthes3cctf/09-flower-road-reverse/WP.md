# 9 FlowerRoad · Reverse

- 分类：Reverse ｜ 分值：401 ｜ 状态：平台已解
- 一句话概括考点：披着 Misc 皮的纯静态逆向，识破三块假路牌后逆一个逐字节变换

## 题目

- 题面：**一路生花 ——Sh1y1**，附件 `FlowerRoad.exe`。
- 运行后打印 `The road is blooming.` 与 `flag> `，读入 `%127s`，只回 `correct` 或 `wrong`。
- 二进制里明晃晃躺着三个 `s3c{...}`，但都是诱饵。

## 思路

- 见到 `.exe` 先静态分析：正常 MSVC C++ 运行时，六段，没有加壳。
- 三块“flag”的文案本身就在自曝：`fake_flower_flag`、`r0s3s_4r3_n0t_th3_r04d`、`try_dynamic_debug_next_time`，正经 flag 不会这么写。
- 主函数里有个 `n(n+1) & 1 == 1` 的判断，`n(n+1)` 恒为偶数，该分支永远不成立，正是打印假 flag 的死代码。
- 真正干活的 `check_flag` 先卡 `strlen == 0x27`，再逐字节与 `.rdata` 的 39 字节表比对，因此核心是还原 `transform`。

## 关键步骤

1. 解析 PE 节表，建立 RVA → 文件偏移的映射（`.text`、`.rdata`、`.data` 的 RVA 不能混用）。
2. 在 `.text` 里按 RIP 相对寻址的 `lea` 目标定位字符串引用，得到 main `0x140001350`、`check_flag` `0x140001250`、`transform` `0x140001110`。
3. `transform` 里的恒假条件全是代数陷阱，化简后等价于：
   `c ^= (19i+0x37)&0xff; c += (7i+0x5b)&0xff; c = rol8(c,(i%5)+1); return c ^ 0xC3`。
4. 从 `.rdata:0x1D340` 取 39 字节期望表，按 `ror8 → 减 → 异或` 逆推每个字符。
5. 用脚本直接读表并还原，输出 39 字符，长度与 `0x27` 吻合。

![逆变换脚本](img/01-solve.png)

6. 把结果回代给原程序，程序自己输出 `correct`，闭环。

![运行输出](img/02-run.png)

## Flag

```text
s3c{Y0u_f0und_th3_road_thr0ugh_fl0w3rs}
```

## 踩坑 / 备注

- capstone 打印的 `[rip + 0x...]` 是位移不是绝对地址，必须自己加 `ins.address + ins.size`，否则一条 xref 都匹配不上。
- 循环移位那段看着像 `sar`（算术右移），但输入经 `movzx` 一定是 0..255 正数，等价于 `rol8`，别被符号扩展吓住。
- 一开始把 `.data` 的字符串按 `.rdata` 算了 RVA，找引用全部落空；重算节表后才定位到主函数。
