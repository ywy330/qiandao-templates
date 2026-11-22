# QD 模板集 · chiphell 每日登录保活

适用于 **QD 签到框架**（[binux/qiandao](https://github.com/binux/qiandao) 及其衍生版，如 `a76yyyy/qiandao`）的 chiphell 模板。

> 模板内不含任何账号信息：Cookie 使用 `{{cookie}}` 变量占位，断言与提取规则为通用正则，可放心使用。

---

## 一、重要说明：chiphell 已经没有"签到"了

2026 年 11 月实测确认：

| 检查项 | 结果 |
|---|---|
| `plugin.php?id=dsu_paulsign:sign`（经典 Discuz 签到插件） | 返回「插件不存在或已关闭」 |
| `k_misign` / `yinxingfei_zzza_sign` / `sg_sign` / `dd_sign` | 同上，全部不存在 |
| 论坛首页、任务中心、个人中心、积分页 | 全站搜不到「签到 / 打卡 / 每日领取」字样，无任何签到入口 |
| 站方积分规则 | **每天登录一次即获得积分** |

所以网上那些"chiphell 签到脚本"现在都会失败——不是脚本坏了，是站点把签到插件下线了。

本模板的做法：**每天带 Cookie 访问一次论坛首页，等效完成"每日登录"**，并把当前积分、用户组写进执行日志，便于确认账号状态。

---

## 二、文件说明

| 文件 | 说明 |
|---|---|
| `chiphell.har` | 标准 HAR 1.2 文件，可在 QD 模板编辑器里直接**上传导入** |
| `chiphell.tpl.json` | 完整模板配置（含断言、变量提取、日志生成），供参考或二次开发 |
| `tpls_history.json` | QD「模板仓库订阅」用的清单文件 |

---

## 三、安装方法

### 方法 A：上传 HAR 导入（推荐，通用）

1. 登录你的 QD 后台 → 顶部导航「模板」/「新建模板」（地址一般是 `/har/edit`）
2. 点「选择文件」，上传 `chiphell.har`
3. 界面上会出现 1 个请求（`GET https://www.chiphell.com/forum.php`），按下面第四节把**断言**和**提取变量**填进去
4. 「网站名称」填 `chiphell`，「网站地址」填 `https://www.chiphell.com`，「间隔」填 `86400`
5. 保存模板 → 新建任务 → 绑定该模板 → 在 `cookie` 变量里填入你的 chiphell Cookie（见第五节）
6. 手动「执行」一次，日志出现 `每日登录保活完成！积分: xxx, 用户组: xxx` 即为成功

### 方法 B：模板仓库订阅

若你的 QD 支持「模板仓库 / 订阅」功能（`a76yyyy/qiandao` 等版本提供）：

1. 进入「模板仓库」→ 添加仓库
2. 仓库地址填本仓库的 GitHub 地址，分支填 `main`
3. QD 会读取根目录的 `tpls_history.json` 并拉取 `chiphell.har`
4. 订阅后仍需在任务里填入自己的 `cookie`

### 方法 C：直接用 JSON 配置

`chiphell.tpl.json` 是模板的完整数据结构（`tpl` 数组 = QD 数据库 `tpl.tpl` 字段内容）。熟悉 QD 结构的开发者可以直接参考其中的请求、断言、变量提取规则自行实现，或用脚本导入。

---

## 四、配置项详解

| 配置项 | 值 | 说明 |
|---|---|---|
| 请求 | `GET https://www.chiphell.com/forum.php` | 论坛首页，携带 Cookie |
| 成功断言 | `action=logout\|退出登录` | 已登录页面才有的退出链接，游客页不含此字符串（实测） |
| 失败断言 | `567\|拦截\|Restricted\|EdgeOne\|需要先登录\|访问已被限制\|请先登录` | 命中即判失败（567 是腾讯 EdgeOne WAF 的拦截码） |
| 提取变量 `jifen` | `积分[:：]\s*([\d,]+)` | 从页面提取当前积分 |
| 提取变量 `usergroup` | `用户组[:：]\s*([^<\s]+)` | 从页面提取用户组 |
| 变量 `cookie` | 你的 chiphell Cookie | 必填，见下一节 |
| 执行间隔 | `86400` 秒 | 每天一次 |

> 注意：不要用「积分」二字做成功断言——游客页面的「积分规则」字样也会命中，会误判为成功。

---

## 五、Cookie 怎么获取

1. 用浏览器登录 [chiphell](https://www.chiphell.com/)
2. 按 `F12` 打开开发者工具 → **Application / 存储** → **Cookies** → `https://www.chiphell.com`
3. 复制以下键值（名字前缀一般固定为 `v2x4_48dd_`，站点换前缀时以实际为准）：
   - `v2x4_48dd_saltkey`
   - `v2x4_48dd_auth`
   - `v2x4_48dd_lastvisit`
   - `v2x4_48dd_lastcheckfeed`（有则带上）
4. 拼接成一行，分号+空格分隔，填入 QD 的 `cookie` 变量：

```
v2x4_48dd_saltkey=xxxx; v2x4_48dd_lastvisit=1700000000; v2x4_48dd_auth=xxxx; v2x4_48dd_lastcheckfeed=123456%7C1700000000
```

> `auth` 才是身份凭证，其余是辅助字段。Cookie 等同于账号密码，**不要提交到公开仓库、不要发给他人**。

---

## 六、常见问题

**Q1：执行报 `567` 或 `Access Restricted` / 页面出现「访问已被限制」？**

chiphell 使用腾讯 EdgeOne（原腾讯云 WAF），会拦截 Python `requests`/`urllib3` 的 TLS 指纹。

- 解决：确保 QD 使用 **pycurl/CurlAsyncHTTPClient**。QD 的 `config.use_pycurl = True` 时（容器内已装 `pycurl`）走 libcurl，TLS 指纹可正常通过；纯 Python 请求会被拦。
- 官方镜像 `a76yyyy/qiandao:*` 已内置 pycurl，一般无需额外处理。

**Q2：日志显示「断言失败」/ 没有 `action=logout`？**

Cookie 过期了，重新获取并更新 `cookie` 变量即可。

**Q3：执行成功，但积分没涨？**

chiphell 是「每天首次登录加积分」，重复访问不叠加。每天成功执行一次即可。

**Q4：需要真实"点签到"的效果怎么办？**

站点已无签到接口。如果 chiphell 后续恢复或改到 App 内签到，需要抓新的接口地址再改模板；本仓库如更新会同步修改版本号。

**Q5：模板里的 `api://util/string/replace` 请求是什么？**

QD 内置的字符串处理 API，仅用于把提取到的积分、用户组拼成一行执行日志，不产生任何网络请求。

---

## 七、已验证环境

| 项 | 版本 |
|---|---|
| QD 框架 | `a76yyyy/qiandao:ja3-latest`（QD 20250803） |
| 容器基础 | Alpine + Python 3.11 + pycurl 7.45.2 / libcurl 8.1.1 (BoringSSL) |
| 验证结果 | 手动执行成功，日志输出 `每日登录保活完成！积分: *****, 用户组: *****` |

---

## 八、免责声明

- 本项目仅供个人学习与自动化技术研究使用，与 chiphell 官方无关。
- Cookie 属于敏感凭据，泄露等于账号失守，请自行妥善保管。
- 请遵守目标站点的用户协议与 robots 规则，合理控制访问频率，不要用于批量操作。
- 因使用本模板导致的账号异常、积分变动或其它后果，由使用者自行承担。

## License

MIT
