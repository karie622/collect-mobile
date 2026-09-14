# 把采集页放到 GitHub Pages（无需备案，手机可直开）

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

## 三、三步搞定

### 1. 传到 GitHub

打开目标仓库（没有就新建一个，比如 `shoucang`）→ **Add file → Upload files** →
把 `index.html` 拖进去 → Commit。

### 2. 开 Pages

仓库 **Settings → Pages → Source** 选 `main` 分支、根目录 `/` → Save。
等 1~2 分钟，地址形如：

```
https://karie622.github.io/shoucang/
```

### 3. 手机打开 + 加到桌面

用手机浏览器打开上面这个地址 → 菜单 → **添加到主屏幕**。

顶部会显示 **● 已连接，将真实写入内容数据库**，就说明通了。

## 四、后端地址是怎么决定的

页面会自动判断，不用手动配：

| 页面所在域名 | 后端地址 |
|---|---|
| `*.github.io`（或任何非腾讯云域） | `https://1488221122-fm448vw4ra.ap-beijing.tencentscf.com` |
| `*.tencentscf.com` | 留空，走同源相对路径 |

需要临时改后端时，加参数覆盖一次即可（会存到浏览器本地）：

```
https://karie622.github.io/shoucang/?api=https://别的后端地址
```

## 五、已验证

| 项 | 结果 |
|---|---|
| JS 语法 | `node --check` 通过 |
| 地址解析（github.io / file:// / localhost / tencentscf.com / `?api=` 覆盖） | 5 种场景全部符合预期 |
| CORS 预检 `OPTIONS /api/auto-ingest` | 204，`Access-Control-Allow-Origin: *` |
| 跨域 `POST /api/auto-ingest` | 200，真实落库 |
| 跨域 `POST /api/fetch-url` | 200，抓到「Python 教程」966 字 |

测试数据已清理，内容库 `classified` 保持 **41**。

## 六、如果以后想换回腾讯云域名

需要**已备案**的域名：SCF 控制台 → 高级能力 → 自定义域名 → CNAME 到
`<appid>.ap-beijing.tencentscf.com` → 路径映射 `/` 到函数。备案后体验最好（国内速度、无跨域）。
