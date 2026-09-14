# 手机采集页（GitHub Pages 托管）

## 线上地址

```
https://karie622.github.io/collect-mobile/
```

仓库：`karie622/collect-mobile`（公开）。后端 API：`https://1488221122-fm448vw4ra.ap-beijing.tencentscf.com`

已通过 GitHub API 完成部署（建库 → 传 `index.html` / `.nojekyll` → 开启 Pages），**无需再手动操作**。
下面记录的是原理和以后要改时的步骤。

---

## 一、为什么不能直接打开 `*.tencentscf.com`

腾讯云函数 URL 的**默认域名对大陆来源 IP 会自动加 `Content-Disposition: attachment`**，
浏览器只能下载、不能渲染页面，且平台会强制覆盖你自己设的值。这是合规要求——默认域名没备案，
不能当网站用。

**API 本身完全不受影响**（`Content-Disposition` 不影响 `fetch`），所以把页面放到别处、
让 SCF 只当后端，是最省事的解法。

## 二、要传的文件

```
C:\Users\循道\WorkBuddy\2026-09-04-18-55-59\pages-github\index.html
```

单文件、零依赖，9KB，已内置后端地址。

## 三、以后要改页面时怎么办

本地源文件改完后，同步到仓库即可（任选一种）：

```bash
# A. 命令行：覆盖后推送
cp "C:/Users/循道/WorkBuddy/2026-09-04-18-55-59/采集-手机版.html" pages-github/index.html
git push          # 若已 clone 本仓库

# B. 网页：仓库 → index.html → 铅笔图标 → 粘贴新内容 → Commit
```

GitHub Pages 会在十几秒到一分钟内自动更新，不用重新开启。

> 注意：`pages-github/index.html` 是从根目录 `采集-手机版.html` **复制**出来的，
> 改完源文件记得重新拷贝一次，两边不会自动同步。

## 四、后端地址是怎么决定的

页面会自动判断，不用手动配：

| 页面所在域名 | 后端地址 |
|---|---|
| `*.github.io`（或任何非腾讯云域） | `https://1488221122-fm448vw4ra.ap-beijing.tencentscf.com` |
| `*.tencentscf.com` | 留空，走同源相对路径 |

需要临时改后端时，加参数覆盖一次即可（会存到浏览器本地）：

```
https://karie622.github.io/collect-mobile/?api=https://别的后端地址
```

## 五、已验证

| 项 | 结果 |
|---|---|
| JS 语法 | `node --check` 通过 |
| 地址解析（github.io / file:// / localhost / tencentscf.com / `?api=` 覆盖） | 5 种场景全部符合预期 |
| 步骤 | 结果 |
|---|---|
| 建库 `karie622/collect-mobile` | 201 |
| 传 `index.html` / `.nojekyll` | 201 / 201 |
| 开启 Pages（`main` + `/`） | 201，`html_url` = `https://karie622.github.io/collect-mobile/` |
| 生效耗时 | **约 20 秒** |
| 线上返回 | 200，`Content-Type: text/html; charset=utf-8`，**无 `Content-Disposition`** |

测试数据已清理，内容库 `classified` 保持 **41**。

## 六、如果以后想换回腾讯云域名

需要**已备案**的域名：SCF 控制台 → 高级能力 → 自定义域名 → CNAME 到
`<appid>.ap-beijing.tencentscf.com` → 路径映射 `/` 到函数。备案后体验最好（国内速度、无跨域）。
