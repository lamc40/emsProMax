# 企业员工管理系统（EMS）

基于 **Django 5 + MySQL + Bootstrap 3** 开发的企业级员工管理系统教学项目，覆盖 Django Web 开发全栈知识点：ORM 模型设计、ModelForm 表单校验、模板继承、AJAX 异步提交、Excel 批量导入、ECharts 数据可视化、文件上传、AI 智能助手（梗识别 + 公告识别 + cnmeme.wiki 热点缓存）等。

## 技术栈

| 类别 | 技术 |
|------|------|
| 后端框架 | Django 5.x |
| 数据库 | MySQL（驱动 mysqlclient） |
| 前端 UI | Bootstrap 3.4.1 + jQuery 3.6.0（本地静态资源） |
| 图表可视化 | ECharts |
| Excel 处理 | openpyxl（模板下载 / 批量导入） |
| AI 大模型 | SiliconFlow API（梗识别 / 公告识别 / JD 生成 / 制度问答） |
| Python | 3.10 / 3.12 |

## 功能模块

### 一、基础 CRUD（核心 Django 知识点）

| 模块 | 说明 | 关键特性 |
|------|------|----------|
| **部门管理** | 列表 / 新增 / 编辑 / 删除 / 批量导入 | 删除后重排剩余 ID 并同步更新员工外键；Excel 批量导入（教程 17 章） |
| **员工管理** | 列表 / 新建（手动表单 + ModelForm） | ModelForm 字段级校验（用户名 ≥ 3 字符、手机号正则） |
| **靓号管理** | 列表 / 搜索 / 增删改 | 号码模糊搜索（`?search=`）+ 全表唯一性校验 |
| **管理员管理** | 列表 / 增删改 / 密码重置 / 切换 | 自定义登录页（验证码 / 图片码） |

### 二、人力资源模块

| 模块 | 说明 |
|------|------|
| **考勤管理** | 打卡记录 / 批量录入 / 编辑 / 删除 |
| **薪资管理** | 薪资记录 / 编辑 / 删除 |
| **公告管理** | 企业公告发布 / 置顶 / AI 自动识别（关键词 + Notice 表检索） |

### 三、任务管理（AJAX + 批量导入）

- **列表**：带翻页（Pagination 组件，默认每页 3 条用于演示分页条）
- **AJAX 异步创建**：`$.ajax` 提交 → 后端校验 → JsonResponse 返回 → 页面不刷新、校验错误实时回显
- **Excel 批量导入**：
  - `/task/template/` — 自动下载带示例数据的 xlsx 模板
  - `/task/import/` — 逐行校验 + 入库，错行返回"第 N 行：..."错误详情
  - 支持 P0/P1/P2/P3 或中文"紧急/高/中/低"两种级别写法
  - 支持 pending/doing/done 或中文"待办/进行中/已完成"两种状态写法
  - 负责人按姓名匹配 UserInfo 表

### 四、AI 智能助手（多轮对话）

| Tab | 功能 |
|-----|------|
| **JD 生成** | AI 根据岗位描述自动生成招聘 JD |
| **制度问答** | Knowledge 表 + 关键词检索 → LLM 回答制度问题 |
| **数据点评** | 读取考勤/薪资数据 → AI 点评 |
| **AI 助手** | 多轮对话 + 梗识别 + 公告识别 |

**梗识别三级缓存架构**：

```
用户输入 → 关键词匹配本地 Meme 表（800+ 条）
         ├── 命中 → 直接解释
         └── 未命中 → 内存 dict 缓存 → cnmeme.wiki 抓取 → 自动回写 Meme 表
```

- 初始梗库 800+ 条：覆盖网络流行语 / 职场梗 / 游戏梗 / 动漫梗 / 综艺梗 / 其他
- 支持多义梗语境判断（"神了"看上下文辨正负、"抽象"看搭配词辨褒贬）
- 热点缓存验证：首次 wiki 抓取 ~13s，二次命中本地 ~0.003s

### 五、数据可视化

| 页面 | 说明 |
|------|------|
| **仪表盘 `/dashboard/`** | 部门人数统计 / 薪资趋势 / 员工分布等综合图表 |
| **ECharts `/chart/list/`** | 柱状图（部门人数）/ 饼图（性别比例）/ 折线图（薪资趋势）/ Highcharts 演示 |

### 六、进阶特性

| 模块 | 说明 |
|------|------|
| **订单管理** | Ajax 弹窗 CRUD（教程 18 章） |
| **文件上传** | Form 上传 + ModelForm 上传（教程 20 章） |
| **城市列表** | FileField 缩略图展示（教程 21 章） |

### 七、通用分页组件

自研 `app01/utils/pagination.py`，所有列表页统一复用：

- queryset 切片分页（第 1 页 `[0:page_size]`、第 2 页 `[page_size:page_size*2]`...）
- Bootstrap 分页条：首页 / 上一页 / 数字页码（当前页高亮 ±5 页）/ 下一页 / 尾页 / 跳转输入框
- **搜索 + 分页条件保留**：自动继承 URL 全部 GET 参数，翻页不丢失搜索条件
- 页码越界保护：非数字、超出范围自动修正

```python
from app01.utils.pagination import Pagination

def xxx_list(request):
    queryset = Xxx.objects.all()
    page_object = Pagination(request, queryset, page_size=10)
    return render(request, 'xxx_list.html', {
        'queryset': page_object.page_queryset,
        'page_str': page_object.html(),
    })
```

## 数据模型（17 张表）

| 模型 | 表名 | 主要字段 |
|------|------|----------|
| Role | app01_role | name、code（角色权限系统） |
| Permission | app01_permission | name、code、parent（权限树） |
| Department | app01_department | title 部门名称 |
| **UserInfo** | app01_userinfo | name、password、age、account、gender、depart（外键 Department） |
| Attendance | app01_attendance | employee（外键 UserInfo）、date、status、remark |
| Salary | app01_salary | employee（外键 UserInfo）、month、base、bonus、total |
| Notice | app01_notice | title、content、top（置顶）、publish_time |
| **Task** | app01_task | level（P0~P3）、title、detail、owner（外键 UserInfo）、status、created_at |
| PrettyNum | app01_prettynum | mobile、price、level、status（靓号管理） |
| **Knowledge** | app01_knowledge | 制度知识库条目（制度问答用） |
| ChatHistory | app01_chathistory | AI 对话历史 |
| **Meme** | app01_meme | name、keywords、explanation、origin、category、is_active（梗库，800+ 条） |
| Admin | app01_admin | 管理员账号 |
| Order | app01_order | 订单（教程 18 章） |
| Boss | app01_boss | 文件上传示例 |
| City | app01_city | FileField 缩略图示例 |
| XX | app01_xx | 原始上传示例 |

## 项目结构

```
ems/
├── manage.py                     # Django 管理入口
├── README.md                     # 项目文档（本文件）
├── requirements.txt              # 依赖清单
├── .env.example                  # 大模型配置模板（复制为 .env 使用）
├── ems/                          # 项目配置目录
│   ├── settings.py               # 配置（MySQL 连接、INSTALLED_APPS、MEDIA 等）
│   ├── urls.py                   # 总路由（全部 50+ 条路由集中在此）
│   ├── wsgi.py / asgi.py
└── app01/                        # 业务应用
    ├── models.py                 # 17 个数据模型
    ├── admin.py
    ├── apps.py
    ├── context_processors.py     # 模板上下文处理器
    ├── middleware/               # 自定义中间件（auth）
    ├── management/commands/      # Django 管理命令
    │   ├── init_data.py          # 初始化基础数据
    │   ├── init_memes.py         # 初始化梗库（35 条）
    │   ├── seed_memes.py         # 批量扩充梗库到 800+ 条
    │   └── meme_seed/            # 3 批种子数据文件
    ├── migrations/               # 数据库迁移
    ├── static/                   # 本地静态资源
    │   ├── js/                   # jquery-3.6.0.min.js、echarts.min.js
    │   ├── plugins/bootstrap-3.4.1/
    │   └── css/
    ├── templates/                # 模板（32 个 html）
    │   ├── layout.html           # 父模板（导航栏 + 页面骨架 + jQuery/Bootstrap）
    │   ├── login.html            # 登录页
    │   ├── dashboard.html        # 仪表盘
    │   ├── ai_assistant.html     # AI 助手（4 个 Tab）
    │   ├── task_list.html        # 任务管理（AJAX + 批量导入面板 + 分页）
    │   ├── depart_list.html      # 部门列表（含批量导入面板）
    │   ├── user_list.html        # 员工列表
    │   ├── attendance_*.html     # 考勤（列表 / 批量录入 / 编辑）
    │   ├── salary_*.html         # 薪资
    │   ├── notice_list.html      # 公告列表（置顶标记）
    │   ├── order_list.html       # 订单（Ajax 弹窗）
    │   ├── chart_list.html       # ECharts 三图
    │   ├── upload_*.html         # 文件上传示例
    │   ├── city_list.html        # 城市缩略图
    │   └── pretty_*.html         # 靓号
    ├── utils/                    # 公共组件
    │   ├── pagination.py         # 通用分页组件
    │   ├── bootstrap.py          # BootStrapModelForm / BootStrapForm 基类
    │   ├── form.py               # ModelForm 校验规则
    │   ├── prompts.py            # AI Prompt 模板（CHAT_SYSTEM + build_chat_system）
    │   ├── ai.py                 # AI 工具函数（梗识别、wiki 抓取、知识库检索）
    │   └── check_code.py         # 验证码生成
    └── views/                    # 视图（15 个文件，按模块拆分）
        ├── admin.py              # 登录 / 登出 / 管理员 CRUD / 验证码
        ├── depart.py             # 部门 CRUD + Excel 批量导入（depart_multi）
        ├── user.py               # 员工列表 / 新建
        ├── pretty.py             # 靓号管理 + 模糊搜索
        ├── attendance.py         # 考勤
        ├── salary.py             # 薪资
        ├── notice.py             # 公告
        ├── task.py               # 任务管理 + AJAX 提交 + Excel 批量导入
        ├── dashboard.py          # 仪表盘 + API
        ├── ai.py                 # AI 助手（梗识别 / 公告识别 / wiki 缓存 / JD 生成 / 多轮对话）
        ├── order.py              # 订单（Ajax 弹窗 CRUD）
        ├── chart.py              # ECharts 数据接口
        ├── upload.py             # 文件上传
        └── city.py               # 城市列表
```

## 路由一览（节选，共 50+ 条）

### 认证
| URL | 说明 |
|-----|------|
| `/login/` | 登录页 |
| `/logout/` | 登出 |
| `/admin/code/` | 登录验证码图片 |

### 部门 / 员工 / 靓号
| URL | 说明 |
|-----|------|
| `/depart/list/` | 部门列表（含批量导入） |
| `/depart/multi/` | Excel 批量导入部门 |
| `/user/list/` | 员工列表 |
| `/pretty/list/` | 靓号列表（搜索 + 分页） |

### 考勤 / 薪资 / 公告
| URL | 说明 |
|-----|------|
| `/attendance/list/` | 考勤列表 |
| `/salary/list/` | 薪资列表 |
| `/notice/list/` | 公告列表 |
| `/notice/top/<nid>/` | 公告置顶 / 取消置顶 |

### 任务管理（重点）
| URL | 说明 |
|-----|------|
| `/task/list/` | 任务列表（分页 + AJAX 新建 + 批量导入面板） |
| `/task/template/` | 下载 Excel 导入模板 |
| `/task/import/` | 上传 Excel 批量导入 |
| `/task/ajax/submit/` | AJAX 异步创建 |
| `/task/ajax/delete/` | AJAX 异步删除 |

### AI 助手（重点）
| URL | 说明 |
|-----|------|
| `/ai/` | AI 助手页面（4 个 Tab） |
| `/ai/chat/` | 多轮对话（梗识别 + 公告识别） |
| `/ai/jd/` | JD 生成 |
| `/ai/knowledge/` | 制度问答 |
| `/ai/meme/` | 梗识别 API（单条输入测试） |
| `/ai/meme/list/` | Meme 梗库管理 |
| `/ai/meme/add/` / `/toggle/` / `/delete/` | 梗库 CRUD |

### 数据可视化
| URL | 说明 |
|-----|------|
| `/dashboard/` | 仪表盘 |
| `/chart/list/` | ECharts 三图（柱 / 饼 / 折线） |

### 进阶演示（教程 18-21 章）
| URL | 说明 |
|-----|------|
| `/order/list/` | 订单（Ajax 弹窗 CRUD） |
| `/upload/form/` | Form 文件上传 |
| `/upload/modal/form/` | ModelForm 文件上传 |
| `/city/list/` | 城市缩略图 |

## 快速开始

### 1. 安装依赖

```bash
pip install -r requirements.txt
```

> 依赖清单：Django 5.x / mysqlclient / Pillow / openpyxl

### 2. 创建数据库

在 MySQL 中执行：

```sql
CREATE DATABASE ems DEFAULT CHARSET utf8mb4;
```

### 3. 配置数据库和大模型

**数据库**：按需修改 `ems/settings.py` 中的连接信息：

```python
DATABASES = {
    'default': {
        'ENGINE': 'django.db.backends.mysql',
        'NAME': 'ems',
        'USER': 'root',
        'PASSWORD': '123456',
        'HOST': '127.0.0.1',
        'PORT': 3306,
    }
}
```

**大模型**（AI 助手功能需要）：复制 `.env.example` 为 `.env`，填入你的 SiliconFlow API Key：

```bash
copy .env.example .env
```

编辑 `.env`：

```
AI_API_KEY=sk-xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx
AI_BASE_URL=https://api.siliconflow.cn/v1
AI_MODEL=Qwen/Qwen2.5-7B-Instruct
```

### 4. 执行数据库迁移

```bash
python manage.py migrate
```

### 5. 初始化数据（可选）

```bash
python manage.py init_data       # 基础数据（部门、管理员等）
python manage.py seed_memes      # 批量导入 800+ 条梗数据（网络流行语/职场/游戏/动漫/综艺等）
```

### 6. 启动服务

```bash
python manage.py runserver 8000
```

浏览器访问 <http://127.0.0.1:8000/>（未登录自动跳 `/login/`，已登录跳 `/dashboard/`）。

## AI 梗识别系统详解

### 三级缓存架构

```
用户输入文本
  │
  ├── Step 1：本地 Meme 表关键词匹配（800+ 条，毫秒级）
  │   ├── 命中 → 读取 explanation 返回
  │   └── 未命中 ↓
  │
  ├── Step 2：进程内内存 dict（_wiki_cache）
  │   ├── 命中 → 直接返回
  │   └── 未命中 ↓
  │
  └── Step 3：cnmeme.wiki 在线抓取（~3-5s，首次）
      ├── 成功 → 自动写入 Meme 表 + 回写 _wiki_cache
      └── 失败 → 返回"未识别到梗"

热点缓存效果：首次 ~13s → 二次命中 ~0.003s（4000x 提速）
```

### 语境判断（多义梗处理）

部分梗有正负两解，prompt 教 AI 结合语气/标点/主题判断：

| 梗 | 正面语境 | 负面语境 |
|----|---------|---------|
| 神了 | 这操作太神了！| 有个同事真神了... |
| 抽象 | 这画很抽象艺术 | 这需求太抽象了 |
| 666 | 干得漂亮 666 | 周末还要加班 666 |
| 绝了 | 这蛋糕绝了！好吃 | 这 bug 绝了，复现不了 |

### 梗库种子数据（meme_seed/）

| 文件 | 分类 | 数量 |
|------|------|------|
| batch1_internet.py | 网络流行语（yyds、emo、躺平...） | ~150 |
| batch2_workplace_gaming.py | 职场梗（996、打工人、画大饼）+ 游戏梗（带不动、猪队友） | ~250 |
| batch3_anime_variety_other.py | 动漫梗 + 综艺梗 + 其他 | ~400 |

## 学习要点索引

- **BootstrapModelForm 统一样式**：`utils/bootstrap.py` 重写 `__init__` 循环添加 `form-control` + placeholder
- **字段级校验钩子**：`utils/form.py` 中 `clean_mobile` 唯一性校验（编辑时 `exclude(id=self.instance.pk)`）
- **AJAX 异步表单**：`task_list.html` → `task_ajax_submit()` → `JsonResponse` + 校验错误回显 + 不刷新页面
- **Excel 批量导入**：`task_import()` 接收 `request.FILES` → `openpyxl.load_workbook()` → 逐行校验 + `Task.objects.create()`
- **cnmeme.wiki 抓取**：`_fetch_meme_from_wiki()` 正则提取 `<article>` 正文 + 自动回写数据库
- **热点缓存**：`_fetch_and_cache_meme()` 三级缓存（内存 dict → Meme 表 → wiki 抓取）
- **模板继承 + JS 块**：jQuery 在 `{% block content %}` **之后**加载 → 内联 `<script>` 必须放 `{% block js %}`
- **删除后重排 ID**：原生 SQL + 临时关闭外键检查 `SET FOREIGN_KEY_CHECKS=0`
- **通用分页组件**：queryset 切片 + URL 参数继承（搜索条件不丢失）
- **ECharts 集成**：CDN 加载 + 后端返回 JSON + 前端 `setOption()` 渲染
