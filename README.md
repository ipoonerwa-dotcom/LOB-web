# LOB Admin Panel

LOB 代币的**白名单管理后台**（TRC20 · TRON 主网）。单文件 HTML，无需构建、无依赖。

支持 **PC / 手机** 两端：
- **PC**：Chrome / Edge / Firefox 装 TronLink 插件
- **手机**：TronLink App 内置 DApp 浏览器

---

## 合约信息

| 项 | 值 |
|------|------|
| 合约地址 | `TGUdHoNkwWJjLYVqVpttTFfy37UYsibeUP` |
| 网络 | TRON 主网 |
| 代币 | LOB（TRC20，18 位精度，总量 110 亿） |
| TronScan | https://tronscan.org/#/contract/TGUdHoNkwWJjLYVqVpttTFfy37UYsibeUP |

---

## 权限模型

后台按连接钱包的角色自动识别，右上角会显示对应标签：

| 角色 | 能做什么 |
|------|---------|
| **Owner**（部署者） | 全部功能：launch / 改窗口 / 改池子 / 白名单 / 设置 admin / 转/弃 owner |
| **Admin**（受限委托） | **只能改白名单**（单个/批量增删+查询） |
| 只读 | 什么写操作都做不了，只能看合约状态 |

**Admin 由 owner 通过 `setAdmin(address)` 设置。合约层强制隔离**：即使 admin 在前端强点 owner 按钮，交易也会 revert `LOB: not owner`。

---

## 手机端使用

### 1. 装 TronLink App
- iOS：App Store 搜「TronLink」
- Android：Google Play 或 https://www.tronlink.org 官网下载

### 2. 导入钱包
- 打开 TronLink App → 用你的助记词/私钥导入或新建
- 切到 **TRON 主网**
- 钱包里存点 TRX 做 gas（**几十 TRX 够用**）

### 3. 打开这个后台
- TronLink App → 底部 **「发现」/「Discover」**
- 顶部地址栏输入本后台的完整 URL（GitHub Pages / Vercel / 你自己部署的地址）
- 页面加载后按下面 PC 端的流程走即可

**注意**：普通手机浏览器（Safari / Chrome）打开会显示"没检测到 TronLink"，必须在 TronLink App 内的 DApp 浏览器打开。

---

## PC 端使用

### 1. 装 TronLink 浏览器插件
- https://www.tronlink.org
- 装完导入钱包，切到 TRON 主网

### 2. 打开 index.html
两种方式：
```bash
# A. 直接双击 index.html
# B. 起本地 HTTP 服务器（推荐，避免 file:// 协议问题）
python -m http.server 8890
# 浏览器打开 http://localhost:8890
```

### 3. 连接钱包 + 加载合约
- 右上角点「连接 TronLink」→ 授权
- 合约地址栏填 `TGUdHoNkwWJjLYVqVpttTFfy37UYsibeUP` → 点「加载」
- 右上角会显示当前角色（owner / admin / 只读）

---

## 常见操作（Admin 版）

### 单个地址增删白名单

在「卖出白名单 → 单个地址」里：
1. 填目标地址
2. 点「加入」或「移出」
3. 系统会先免费**模拟**，估算能量和检查是否会 revert
4. 弹出确认框，点「确定」才广播交易
5. 交易上链后日志会打印 tx hash

### 批量增删

在「批量」里：
- 每行一个 TRON 地址（支持逗号/分号分隔，会自动去重）
- 点「批量加入」或「批量移出」
- **一笔交易搞定一批** → 省 gas、省事

### 查询

- 「查询该地址是否在白名单」：单个查
- 「批量查询」：一次列出全部状态

---

## 关键机制说明

### 三个阶段

| 阶段 | 条件 | 买入 | 卖出 |
|------|------|------|------|
| ① 未启动 | `launched == false` | ❌ 谁也买不到 | ❌ |
| ② 防夹窗口 | `launched && now < openAt` | ✅ 所有人 | 仅白名单 |
| ③ 已开盘 | `now >= openAt` | ✅ | ✅ **所有人，永久** |

**白名单只在阶段 ② 有意义**：让白名单地址在窗口内提前卖。开盘后所有人都能卖，白名单不再有任何特权。

### 硬承诺

- `MAX_PROTECT_WINDOW = 100000 hours` （合约常量，永久固定）
- `openAt` 一旦设定，**只能提前不能推后**（由 `bringOpenForward` 里的 `require(newOpenAt < openAt)` 唯一守卫）
- 到 `openAt` 之后 `_guard()` 第一行直接 return，owner / admin 做什么都影响不了转账

**结论**：任何人买到 LOB 后，最多等到 `openAt` 就能卖，代码保证。

---

## 常见问题

### 手机端连不上钱包
- 只能在 TronLink App 的**发现/浏览器**里打开
- 普通浏览器（Safari / Chrome）会显示"没检测到 TronLink"

### 提示"429" 或"接口调用超时"
- TronGrid 免费额度是 3 次请求/秒，短时间点太多会限流
- 等几秒再试

### 提示"OUT_OF_ENERGY"
- feeLimit 不够：合约层的错误。检查 TRX 余额，充值后重试
- 后台默认 feeLimit = 100 TRX，正常操作绝对够用

### 广播成功但一直没上链
- 去 TronScan 查 tx 状态：https://tronscan.org
- 大部分是 TronLink 那边延迟，等一分钟看看

### 模拟失败 "LOB: not owner or admin"
- 你连的钱包既不是 owner 也不是 admin
- 让 owner 通过 `setAdmin(你的地址)` 把你设为 admin

### 模拟失败 "LOB: zero address"
- 地址栏填了空地址或格式错误的地址（TRON 地址是 `T` 开头 34 字符）

---

## 技术细节

- 单文件 HTML，纯前端，无依赖、无构建
- TronWeb 通过 `window.tronWeb` 注入（TronLink 提供）
- 判链用**创世块哈希**而非节点 URL 字符串（防伪节点）
- 每笔写操作**先免费模拟**（`triggerConstantContract`），拿到真实能量消耗和 revert 原因后才让你确认发送

## License

MIT
