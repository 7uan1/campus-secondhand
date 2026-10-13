# AI 交接文档：校园二手物品交易平台（最新版）

> 本文档写给接手此项目的 AI。读完即可理解项目、运行网站、修改 UI。

## 一、项目一句话说明

在韩留学生校园二手物品交易 Web 平台，全中文界面。支持注册登录、商品发布（图片Base64存库）、浏览搜索、意向申请三态流转、商品留言、收藏。无线上支付，纯线下当面交易。

## 二、技术栈（严格使用，不要替换）

| 层 | 技术 | 说明 |
|---|---|---|
| 后端 | Python Flask 3.1 | |
| 认证 | flask-login | Session-based |
| 数据库 | PostgreSQL | 原生 psycopg2 写 SQL，不用 ORM |
| 前端 | 原生 HTML + Jinja2 模板 + CSS + JS | 不要 React/Vue/Bootstrap |
| 图片 | Base64 存数据库 TEXT 字段 | 不要本地文件/S3 |
| 部署 | Gunicorn + Render | 本地用 python app.py |

## 三、项目结构

```
campus-secondhand/
├── app.py                  # Flask 主程序，全部路由（约390行）
├── init_db.py              # 建表脚本（5张表，CREATE TABLE IF NOT EXISTS）
├── seed_data.py            # 模拟数据（demo用户 + 30条商品）
├── requirements.txt        # 依赖列表
├── .env.example            # 环境变量模板
├── Procfile                # Render 部署用
├── render.yaml             # Render 蓝图
├── README.md
├── static/
│   ├── css/style.css       # 全部样式（约700行，含大菜单+动画）
│   └── js/main.js          # 图片压缩 + 表单校验 + 下拉菜单交互
└── templates/
    ├── base.html           # 基础模板：导航栏（大型悬浮菜单）+ Flash + 页脚
    ├── index.html          # 首页：商品网格 + 搜索 + 分类筛选
    ├── login.html          # 登录页
    ├── register.html       # 注册页（仅3项：用户名/密码/昵称）
    ├── publish.html        # 发布商品页
    ├── detail.html         # 商品详情：大图 + 收藏 + 申请弹窗 + 留言板
    ├── edit_item.html      # 编辑商品页
    └── profile.html        # 个人中心：我的发布/我的申请/我的收藏
```

## 四、数据库设计（5张表）

### users
- id (SERIAL PK), username (UNIQUE), password_hash, nickname, school, created_at
- 注册时 school 默认 '其他'

### items
- id (SERIAL PK), seller_id (FK), title, category, price, condition, description, image_base64 (TEXT), status, created_at
- status: 在售 / 已约定 / 已完成

### applications
- id, item_id (FK), buyer_id (FK), seller_id (FK), status, message, created_at
- UNIQUE(item_id, buyer_id)
- status: 申请中 / 已约定 / 已完成

### messages
- id, item_id (FK), user_id (FK), content, created_at

### favorites
- id, user_id (FK), item_id (FK), created_at
- UNIQUE(user_id, item_id)

## 五、核心业务逻辑

### 三态流转（不要改）
```
申请中 → 已约定 → 已完成
  ↑        ↑         ↑
买家发起  卖家同意   买卖双方确认完成
```
- 卖家同意约定时：application.status → 已约定，同时 item.status → 已约定（首页隐藏）
- 确认完成时：application.status → 已完成，同时 item.status → 已完成

### 图片处理
- 前端 Canvas 压缩到最大宽 800px、质量 0.7，转 data:image/jpeg;base64
- 存入 items.image_base64，模板直接 `<img src="{{ item.image_base64 }}">`

### 首页只显示 status IN ('在售', '已约定')

## 六、导航栏 UI 结构（当前版本）

导航栏包含：
1. **Logo**（左侧）
2. **「全部功能 ▾」大型悬浮菜单**（鼠标 hover 淡入展开，三列布局）
   - 左列「商品分类」：6 大分类卡片（图标+标题+副标题），2列网格
   - 中列「我的账户」：未登录显示发布物品+注册账号；已登录显示用户信息+发布物品+我的发布/申请/收藏+退出
   - 右列「快捷入口」：首页浏览 + 交易小贴士
   - CSS class: `.mega-menu`, `.mega-trigger`, `.mega-panel`, `.mega-column`, `.mega-grid`, `.mega-item`
3. **右上角按钮**：未登录显示「🔐 登录」，已登录显示「+ 发布物品」

## 七、常量

```python
CATEGORIES = ['教材书籍', '电子家电', '生活用品', '家具家居', '服饰鞋包', '其他']
CONDITIONS = ['全新', '几乎全新', '轻微使用痕迹', '明显使用痕迹']
```

## 八、本地运行步骤

```bash
cd campus-secondhand
pip install -r requirements.txt
# 确保 PostgreSQL 运行，创建数据库 campus_secondhand
# 复制 .env.example 为 .env，填入 DATABASE_URL
python init_db.py
python seed_data.py    # 可选：导入demo数据
python app.py
# 打开 http://127.0.0.1:5000
```

测试账号：demo / demo123

## 九、UI 修改指引

- **改导航栏**：编辑 `templates/base.html` 的 `<nav>` 部分，样式在 `static/css/style.css` 搜索 `mega-menu`
- **改首页布局**：编辑 `templates/index.html`，商品卡片样式搜 `.item-card`
- **改颜色主题**：主色 `rgb(193, 86, 52)`（灵山大学橙），全局替换即可
- **改动画**：CSS 末尾 `淡入淡出动画体系` 部分，搜 `pageFadeIn`、`cardFadeInUp`
- **改注册字段**：`templates/register.html` + `app.py` 的 register 路由 + `static/js/main.js` 的 setupRegisterForm

## 十、明确不要做的事

- ❌ 不要引入 React/Vue/Angular
- ❌ 不要用 Bootstrap/Tailwind
- ❌ 不要做线上支付
- ❌ 不要做实时私信（留言板是公开的）
- ❌ 不要做学生实名认证
- ❌ 不要把图片存到本地文件或 S3
- ❌ 不要用 SQLAlchemy/Django ORM
- ❌ 不要改成多语言（全中文）
- ❌ 不要改三态流转逻辑

## 十一、路由清单

| 方法 | 路径 | 功能 |
|---|---|---|
| GET | `/` | 首页（?q=搜索 &category=分类） |
| GET/POST | `/register` | 注册（用户名/密码/昵称） |
| GET/POST | `/login` | 登录 |
| GET | `/logout` | 退出 |
| GET | `/item/<id>` | 商品详情 |
| GET/POST | `/publish` | 发布商品 |
| GET/POST | `/item/<id>/edit` | 编辑商品 |
| POST | `/item/<id>/delete` | 删除商品 |
| POST | `/item/<id>/apply` | 提交意向申请 |
| POST | `/application/<id>/accept` | 卖家同意约定 |
| POST | `/application/<id>/complete` | 确认完成 |
| POST | `/item/<id>/favorite` | 收藏/取消 |
| POST | `/item/<id>/message` | 发表留言 |
| GET | `/profile` | 个人中心（?tab=published/applied/favorites） |

---

**交接完成。按第八节步骤即可启动，修改 UI 参考第九节。**
