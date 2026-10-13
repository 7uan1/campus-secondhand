# 部署指南：校园二手物品交易平台

> 本文档写给负责部署此项目的 AI。按步骤操作即可将项目部署到公网，获得一个任何人都能访问的网址。

## 一、项目概览

- **项目名**：중고캠퍼스 校园二手圈
- **类型**：Python Flask Web 应用 + PostgreSQL 数据库
- **功能**：用户注册登录、商品发布（图片Base64存库）、浏览搜索、意向申请三态流转、留言、收藏
- **当前版本**：导航栏大型悬浮菜单 + 简化注册（用户名/密码/昵称三项）
- **测试账号**：demo / demo123（需先运行 seed_data.py 导入）

## 二、技术栈

- Python 3.10+ / Flask 3.1 / flask-login
- PostgreSQL（原生 psycopg2，无 ORM）
- 原生 HTML + Jinja2 + CSS + JS
- Gunicorn（生产服务器）
- 部署目标：Render（免费层）+ Supabase PostgreSQL（免费层）

## 三、文件清单

```
campus-secondhand/
├── app.py              # Flask 主程序（已配置生产模式：debug=False, 读取PORT环境变量）
├── init_db.py          # 建表脚本（5张表，幂等可重复执行）
├── seed_data.py        # 演示数据（demo用户 + 30条商品）
├── requirements.txt    # 依赖：flask, flask-login, psycopg2-binary, python-dotenv, Werkzeug, gunicorn
├── .env.example        # 环境变量模板
├── Procfile            # Render 进程配置：web: gunicorn app:app
├── render.yaml         # Render Blueprint（可选，repo字段需替换）
├── README.md
├── AI_HANDOFF.md       # 项目完整说明
├── static/
│   ├── css/style.css   # 全部样式
│   └── js/main.js      # 交互脚本
└── templates/          # 8个HTML模板
    ├── base.html       # 导航栏（大型悬浮菜单）
    ├── index.html      # 首页
    ├── login.html
    ├── register.html   # 仅3项注册
    ├── publish.html
    ├── detail.html
    ├── edit_item.html
    └── profile.html
```

## 四、部署方案 A：Render + Supabase（推荐，全免费）

### 步骤 1：准备 GitHub 仓库

1. 将整个项目文件夹推送到 GitHub（不含 `.env` 文件）
2. 确保仓库包含上述所有文件
3. 记录仓库地址

### 步骤 2：创建 Supabase 数据库

1. 注册 https://supabase.com（GitHub 登录）
2. **New Project**
   - Name: `campus-secondhand`
   - Password: 自行设置并记录
   - Region: `Northeast Asia (Tokyo)`（离韩国最近）
   - Pricing: Free
3. 等待创建完成（约2分钟）
4. 左侧 **Project Settings → Database**
5. 找到 **Connection string → URI**，复制完整连接串
6. 将连接串中的 `[YOUR-PASSWORD]` 替换为实际密码
7. 这就是 `DATABASE_URL`，格式类似：
   ```
   postgresql://postgres.xxxxx:yourpassword@aws-0-ap-northeast-2.pooler.supabase.com:6543/postgres
   ```

### 步骤 3：在 Render 部署 Web 服务

1. 注册 https://render.com（GitHub 登录）
2. **New + → Web Service**
3. 连接步骤 1 的 GitHub 仓库
4. 配置：
   - **Name**: `campus-secondhand`（决定网址）
   - **Runtime**: Python
   - **Build Command**: `pip install -r requirements.txt && python init_db.py`
   - **Start Command**: `gunicorn app:app`
   - **Instance Type**: Free
5. **Advanced → Environment Variables**，添加：
   | Key | Value |
   |---|---|
   | `DATABASE_URL` | 步骤 2 拿到的连接串 |
   | `SECRET_KEY` | 任意随机字符串，如 `campus-2026-secret-key-abc123` |
6. 点 **Create Web Service**
7. 等待 3-5 分钟部署完成
8. 页面顶部会显示网址，如 `https://campus-secondhand.onrender.com`

### 步骤 4：导入演示数据（可选）

1. Render 服务页面 → **Shell**
2. 执行：`python seed_data.py`
3. 看到 "种子数据导入完成" 即成功
4. 用 `demo / demo123` 登录测试

### 步骤 5：验证

- 打开网址，首页应显示 30 条商品
- 注册新账号 → 登录 → 发布商品 → 浏览 → 申请加入
- 所有功能应正常工作

## 五、部署方案 B：本地运行（用于测试）

```bash
cd campus-secondhand
python -m venv venv
# Windows: venv\Scripts\activate
# Mac/Linux: source venv/bin/activate
pip install -r requirements.txt

# 确保 PostgreSQL 已安装并运行，创建数据库：
# createdb campus_secondhand

# 创建 .env 文件：
cp .env.example .env
# 编辑 .env，填入 DATABASE_URL

python init_db.py
python seed_data.py    # 可选
python app.py
# 打开 http://127.0.0.1:5000
```

## 六、常见问题

### Q: Render 部署后打开很慢
A: 免费层 15 分钟无访问会休眠，第一次打开需 20-30 秒唤醒，之后正常。可设置 UptimeRobot 免费监控每 5 分钟访问一次防止休眠。

### Q: 数据库连接失败
A: 检查 DATABASE_URL 中的密码是否正确替换，Supabase 项目是否已暂停（免费层 7 天无活动会暂停，登录 Supabase 点 Resume）。

### Q: 图片不显示
A: 确认 items.image_base64 字段有数据，且以 `data:image/jpeg;base64,` 开头。

### Q: 中文乱码
A: 确保数据库编码为 UTF8，Supabase 默认就是 UTF8。

### Q: 如何更新代码
A: 推送到 GitHub 后 Render 会自动重新部署，无需手动操作。

### Q: 500 错误
A: Render 服务页面 → **Logs** 查看具体报错。最常见原因是 DATABASE_URL 未配置或 init_db.py 未执行。

## 七、环境变量说明

| 变量 | 必填 | 说明 |
|---|---|---|
| `DATABASE_URL` | ✅ | PostgreSQL 连接串 |
| `SECRET_KEY` | ✅ | Flask session 加密密钥，任意随机字符串 |
| `SUPABASE_URL` | ❌ | 预留，当前版本未使用 |
| `SUPABASE_KEY` | ❌ | 预留，当前版本未使用 |
| `PORT` | ❌ | Render 自动注入，app.py 会读取 |

## 八、数据库表结构（init_db.py 自动创建）

- **users**: id, username(UNIQUE), password_hash, nickname, school, created_at
- **items**: id, seller_id(FK), title, category, price, condition, description, image_base64(TEXT), status, created_at
- **applications**: id, item_id(FK), buyer_id(FK), seller_id(FK), status, message, created_at, UNIQUE(item_id,buyer_id)
- **messages**: id, item_id(FK), user_id(FK), content, created_at
- **favorites**: id, user_id(FK), item_id(FK), created_at, UNIQUE(user_id,item_id)

## 九、注意事项

- 不要上传 `.env` 文件到 GitHub（含数据库密码）
- 免费层适合课程演示，生产环境需升级付费层
- 项目无线上支付功能，所有交易线下当面完成
- 主色调：灵山大学橙 rgb(193, 86, 52)

---

**部署完成后，将 Render 提供的网址发给任何人即可访问。**
