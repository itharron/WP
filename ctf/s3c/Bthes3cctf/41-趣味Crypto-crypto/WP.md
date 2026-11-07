# 41 趣味Crypto · Crypto

- 分类：Crypto ｜ 分值：200 ｜ 状态：平台已解
- 两关都是「只比哈希、不给明文」的交互题；第 2 关是 Rabin 密码，分解 `n` 后四个平方根里有一个正好是口令。

## 题目

- 题号 41，题名「趣味Crypto」，附件 `趣味Crypto.zip`（内含 `趣味Crypto.py`）。
- 程序用 `pbkdf2_hmac('sha256', answer, b"s3c-2026-quwei-crypto", 200000)` 与内置 digest 比对，答案不会出现在源码里。
- 第 1 关：`a = [1, 1, 7, 3]`，`k = 13`，要求两个整数。
- 第 2 关：Rabin，`p,q` 各 64 位且 `≡3 (mod 4)`，`e=2`，`n=163912889473040288242375351586022563317`，`c=62041100841034491160763016680290026337`。

## 思路

- 把附件读一遍就能看出两关都是「你给答案，我只回对不对」，源码里只有 digest，没有明文，硬猜不现实。
- 第 2 关的 `n` 是公开的，先把它分解：两个 64 位因子，直接查 factordb 拿到 `p`、`q`。
- `e=2` 且 `p,q ≡ 3 (mod 4)`，是标准 Rabin：四个根由 `pow(c,(p+1)//4,p)`、`pow(c,(q+1)//4,q)` 组合 CRT 得到。
- 拿到四个根之后逐个算 PBKDF2 摘要和服务端比对，命中的那个才是答案，程序再据此解出最终 flag。
- 第 1 关的「两个整数」没逆出来（只能用 PBKDF2 逐个试），本 WP 用与程序相同的校验函数直接复算第 2 关，效果等价于打通 reveal 分支。

## 关键步骤

1. 解压附件，打开 `趣味Crypto.py`，确认两关的校验方式：都只算 PBKDF2 再比 digest，明文不在源码里。
2. 第 2 关的 `n` 只有 128 bit。浏览器打开 factordb，把 `n` 粘进去查询，直接得到两个因子：

   ```text
   12423010310503732871
   13194297145068849827
   ```

3. 在 python 交互环境里代回校验，并按 Rabin 算出四个平方根：

   ```python
   >>> from Crypto.Util.number import long_to_bytes
   >>> from hashlib import sha256
   >>> p, q, n, c = 12423010310503732871, 13194297145068849827, 163912889473040288242375351586022563317, 62041100841034491160763016680290026337
   >>> p*q == n
   True
   >>> rp, rq = pow(c, (p+1)//4, p), pow(c, (q+1)//4, q)
   >>> roots = [(a*q*pow(q,-1,p) + b*p*pow(p,-1,q)) % n for a in (rp, p-rp) for b in (rq, q-rq)]
   >>> [long_to_bytes(r) for r in roots]
   ```

   四个根里只有一个是 UTF-8 可读的：`b's3c{shu@n_q!!}'`，另外三个是随机字节。

4. 用它跑一遍程序里的 PBKDF2 校验，与服务端的 digest 一致，程序进入 reveal 分支，把 `_VAULT` 用 `sha256(answer + b"::reveal")` 异或开：

   ```python
   >>> key = sha256(b's3c{shu@n_q!!}' + b'::reveal').digest()
   >>> VAULT = bytes.fromhex('1c8a6517dcea7635a615604eb49bc9567e5ea2739c6bc5421a')
   >>> bytes(v ^ key[i % len(key)] for i, v in enumerate(VAULT))
   b's3c{the_correct_@nswer$$}'
   ```

   第 2 关这一段的终端记录（分解结果、四个根、命中哈希、reveal 出 flag）：

   ![Rabin 解密与 reveal](img/01-stage2.png)

## Flag

```text
s3c{the_correct_@nswer$$}
```

## 踩坑 / 备注

- 四个根里只有一个是 UTF-8 可读的，另外三个是随机字节；不要拿随机根去做哈希。
- 程序对答案做了 NFKC 归一化与 `b'...'` 前缀剥离，复算时用裸字符串即可。
- 第 1 关的两个整数没有还原，属于「哈希不可逆 + 空间未知」的卡点；不影响第 2 关与最终 flag 的取得。
- factordb 查不到时再考虑本地工具，128 bit 的 `n` 用 yafu 之类跑一会儿也能出结果。
