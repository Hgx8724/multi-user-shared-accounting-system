# 桌面端 UI 原型说明

- 原型目录：`桌面原型/`
- 截图目录：`桌面版/`
- 画布尺寸：1440 × 900
- 设计目标：Windows 桌面应用，后续可使用 Blazor Hybrid、Blazor Web App 或 .NET MAUI 实现
- 导航图标：`桌面原型/icons/` 中的 5 个 SVG

## 1. 一级导航

底部导航固定为五项，从左到右：

| 顺序 | 导航 | 路由建议 | 主要职责 |
| ---: | --- | --- | --- |
| 1 | 首页 | `/home` | 跨账本概览、待办和最近动态 |
| 2 | 账本 | `/ledgers/{ledgerId}` | 账目、成员、设置、审批和审计 |
| 3 | 结算 | `/settlements/{ledgerId}` | 净额匹配、付款指令和结算历史 |
| 4 | 通知 | `/notifications` | 应用内通知与已读状态 |
| 5 | 个人中心 | `/profile` | 账号、角色、设备和安全设置 |

审批、新增账目、成员管理和审计日志都属于账本的子页面，不单独占用一级导航。

## 2. 页面清单

| 文件 | 页面 | 说明 |
| --- | --- | --- |
| `01-登录认证.png` | 登录认证 | 用户名密码登录、注册和账号帮助 |
| `02-首页.png` | 首页 | 跨账本汇总、待处理事项和最近账本 |
| `03-账本.png` | 账本 | 账目列表、筛选、导出和账本内标签页 |
| `04-新增账目.png` | 新增账目 | 多付款人、分摊成员、分摊策略和份额预览 |
| `05-审批中心.png` | 审批中心 | 待审核列表和申请详情 |
| `06-结算中心.png` | 结算中心 | 成员净额、付款指令和批次状态 |
| `07-通知中心.png` | 通知中心 | 通知列表和通知详情 |
| `08-个人中心.png` | 个人中心 | 用户资料、账本角色、偏好和安全 |
| `09-审计日志.png` | 审计日志 | 账本内操作留痕和筛选 |

## 3. 组件与行为

| 组件 | 输入 | 输出/事件 | 主要状态 |
| --- | --- | --- | --- |
| `AppShell` | 当前用户、当前路由 | 导航、退出登录 | 已登录、会话过期 |
| `BottomNav` | 当前 Tab、未读数 | `TabSelected` | 五个一级项、通知红点 |
| `HomeDashboard` | 用户账本汇总 | 打开账本、打开待办 | Loading、Empty、Error |
| `LedgerTabs` | `ledgerId` | `TabSelected` | 账目、成员、设置、审计 |
| `EntryTable` | 筛选条件、分页 | 打开详情、编辑、导出 | Loading、Empty、Forbidden |
| `ExpenseEditor` | 账本、成员、当前用户 | `Submitted`、`SavedDraft` | 校验、提交中、错误 |
| `ApprovalWorkspace` | 待审核申请 | `Approved`、`Rejected` | 待审核、处理中、冲突 |
| `SettlementPanel` | 未结算汇总 | `SettlementConfirmed` | 可结算、空、并发冲突 |
| `NotificationCenter` | 当前用户通知 | `MarkRead` | 未读、已读、空 |
| `ProfilePage` | 当前账号 | 更新资料、安全管理 | 正常、会话失效 |

### 3.1 导航行为

- 登录页不显示底部导航。
- 一级导航仅切换根页面，不重复堆叠相同页面。
- 进入账本子页面时，“账本”保持选中。
- 进入结算详情时，“结算”保持选中。
- 通知未读数来自后端查询，不存为页面局部常量。
- 用户无目标账本权限时，不显示该账本，也不允许通过 ID 直接访问。

### 3.2 账目行为

- 账目只在账本页面中展示。
- 新增账目和审批中心通过账本上下文打开。
- 筛选条件改变后重新请求当前账本数据。
- 导出只导出当前筛选结果。
- 已结清账目不显示修改和撤销操作。
- 手工分摊模式需要展示每个已选成员的可编辑金额。

### 3.3 结算行为

- 结算页不重新选择成员和分摊策略。
- 页面只展示冻结的付款、分摊快照和计算出的净额。
- 付款指令默认 `OPEN`。
- 确认结算后显示报告和新的付款指令状态。
- 服务端返回版本冲突时提示刷新，不覆盖本地旧数据。

### 3.4 个人中心行为

- 显示账号基本信息。
- 显示用户在不同账本中的角色。
- 显示当前登录设备、退出登录和会话管理入口。
- 用户认证信息不进入普通页面状态，只由认证服务维护。

## 4. 导航 SVG 使用

5 个 SVG 的描边已统一为主题色 `#235C52`。

当前原型使用背景图方式：

```css
.nav-asset {
  width: 22px;
  height: 22px;
  background: var(--icon) center / contain no-repeat;
}

.nav-home { --icon: url("icons/Home.svg"); }
.nav-ledger { --icon: url("icons/Ledger.svg"); }
.nav-settlement { --icon: url("icons/Settlement.svg"); }
.nav-notification { --icon: url("icons/Notification.svg"); }
.nav-user { --icon: url("icons/User.svg"); }
```

未选中时通过透明度弱化：

```css
.nav-item:not(.active) .nav-asset {
  opacity: 0.46;
}
```

不直接使用 Mask，是为了避免本地文件协议和部分运行环境对 SVG Mask 的兼容差异。

## 5. 与 C# 前端代码的对应

### 5.1 Blazor

建议将 Figma/HTML 组件一一映射为 Razor 组件：

```razor
<BottomNav ActiveTab="ActiveTab"
           UnreadCount="UnreadCount"
           OnTabSelected="NavigateToTab" />
```

```csharp
public enum AppTab
{
    Home,
    Ledger,
    Settlement,
    Notifications,
    Profile
}
```

页面状态由 ViewModel 或页面服务提供，不把业务逻辑写进纯展示组件。

```csharp
public sealed record HomeDashboardViewModel(
    int LedgerCount,
    decimal MonthlyExpense,
    int PendingTaskCount,
    IReadOnlyList<LedgerSummary> RecentLedgers);
```

### 5.2 组件参数与事件

- 组件输入通过 `[Parameter]` 定义。
- 组件输出通过 `EventCallback<T>` 定义。
- 权限、业务校验和数据请求由应用服务完成。
- 纯展示组件不直接查询数据库或拼接 URL。

### 5.3 工具建议

- 设计：Figma
- 桌面 IDE：Visual Studio 2026 Community
- Web/桌面前端：Blazor
- 移动端：.NET MAUI
- 后端：ASP.NET Core Web API
- 数据访问：Entity Framework Core
- 数据库：MySQL 8.0
- 组件测试：bUnit
- 端到端测试：Playwright
- API 客户端：NSwag/OpenAPI 生成

## 6. Figma CSS 导出说明

Figma 的整页 CSS 导出不适合直接用于生产代码，原因是：

- Auto Layout 容易导出为绝对定位。
- 嵌套组件会产生大量重复 class。
- Mask、Effect、Constraints 等属性容易产生不兼容代码。
- 完整页面的视觉层级不等于最终组件层级。

推荐只从 Figma 提取：

- 颜色变量
- 字号和行高
- 间距
- 尺寸
- SVG/PNG 资源

HTML/CSS、Blazor 组件或 XAML 布局应在代码工程中重新组织。

## 7. 当前版本

当前桌面版原型位于：

```text
UI界面/
├── 桌面原型/
│   ├── index.html
│   ├── styles.css
│   └── icons/
└── 桌面版/
    ├── 00-桌面UI总览.png
    ├── 01-登录认证.png
    ├── ...
    └── 09-审计日志.png
```

`桌面版/` 目录仅保存 PNG；`桌面原型/` 保存可编辑源文件。
