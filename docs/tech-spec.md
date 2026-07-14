# 情绪疏导小助手 - 技术规范文档

## 1. 技术栈选择

### 1.1 编程语言
- **Python 3.10+**

### 1.2 GUI 框架
- **PyQt6**（推荐）或 PySide6
  - 理由：功能强大、组件丰富、样式灵活

### 1.3 数据库
- **SQLite3**（Python内置）
  - 数据库文件：data/emoodie_db.sqlite
  - 自动创建，首次运行时初始化表结构

### 1.4 数据存储格式
- 数据库：结构化数据（心情记录、对话历史等）
- 配置文件：config/settings.json

### 1.5 打包工具
- **PyInstaller**（后期打包为exe）

---

## 2. 项目目录结构

`
情绪疏导小助手/
├── main.py                    # 程序入口
├── requirements.txt           # Python依赖
├── README.md                  # 使用说明
│
├── app/                       # 应用程序核心代码
│   ├── __init__.py
│   ├── main_window.py         # 主窗口
│   ├── styles.qss             # 样式表（淡黄色主题）
│   │
│   ├── modules/               # 功能模块
│   │   ├── mood_record.py     # 心情记录模块
│   │   ├── mood_calendar.py   # 心情日历模块
│   │   ├── history_view.py    # 历史记录模块
│   │   ├── chat_helper.py     # 对话助手模块
│   │   └── ai_summary.py      # AI总结模块
│   │
│   ├── models/                # 数据模型
│   │   ├── mood_entry.py      # 心情记录模型
│   │   └── chat_message.py    # 对话消息模型
│   │
│   ├── services/              # 业务逻辑层
│   │   ├── db_service.py      # 数据库服务
│   │   ├── mood_analyzer.py   # 情绪分析服务
│   │   └── chat_engine.py     # 对话引擎
│   │
│   └── utils/                 # 工具函数
│       ├── date_utils.py      # 日期工具
│       ├── emoji_utils.py     # 表情工具
│       └── backup.py          # 数据备份
│
├── data/                      # 数据目录（运行时创建）
│   ├── emoodie_db.sqlite      # 数据库文件
│   └── backups/               # 备份文件
│
├── config/                    # 配置文件
│   └── settings.json          # 应用配置
│
├── docs/                      # 项目文档
│   ├── requirements.md        # 需求文档
│   ├── tech-spec.md           # 技术规范
│   └── design-guide.md        # UI设计规范
│
├── logs/                      # 开发日志
│   └── YYYY-MM-DD.md
│
└── tests/                     # 测试文件
    ├── test_models.py
    ├── test_services.py
    └── test_modules.py
`

---

## 3. 数据库设计

### 3.1 心情记录表 (mood_entries)

| 字段名 | 类型 | 说明 |
|--------|------|------|
| id | INTEGER PRIMARY KEY | 自增主键 |
| date | TEXT NOT NULL | 日期 (YYYY-MM-DD) |
| time_slot | TEXT | 时间段 (HH:MM)，总体记录为NULL |
| score | INTEGER NOT NULL | 评分 (1-10) |
| emoji | TEXT NOT NULL | 表情符号 |
| mood_label | TEXT NOT NULL | 心情标签 |
| note | TEXT | 备注文字 |
| record_type | TEXT NOT NULL | 'daily' 或 'slot' |
| created_at | TEXT NOT NULL | 创建时间 |
| updated_at | TEXT NOT NULL | 更新时间 |

### 3.2 每日总结表 (daily_summaries)

| 字段名 | 类型 | 说明 |
|--------|------|------|
| id | INTEGER PRIMARY KEY | 自增主键 |
| date | TEXT NOT NULL UNIQUE | 日期 |
| summary_text | TEXT | 总结内容 |
| avg_score | REAL | 平均评分 |
| dominant_emoji | TEXT | 主导表情 |
| generated_at | TEXT NOT NULL | 生成时间 |
| is_auto | BOOLEAN | 是否自动生成 |

### 3.3 对话记录表 (chat_history)

| 字段名 | 类型 | 说明 |
|--------|------|------|
| id | INTEGER PRIMARY KEY | 自增主键 |
| date | TEXT NOT NULL | 日期 |
| user_message | TEXT NOT NULL | 用户消息 |
| ai_response | TEXT NOT NULL | AI回复 |
| mood_before | INTEGER | 对话前心情评分 |
| mood_after | INTEGER | 对话后心情评分 |
| created_at | TEXT NOT NULL | 创建时间 |

### 3.4 应用设置表 (app_settings)

| 字段名 | 类型 | 说明 |
|--------|------|------|
| key | TEXT PRIMARY KEY | 配置键 |
| value | TEXT | 配置值 |
| updated_at | TEXT | 更新时间 |

---

## 4. 心情与表情映射表

| 评分范围 | 表情 | 标签 | 颜色 |
|----------|------|------|------|
| 1-2 | 😫 | 非常糟糕 | #FF6B6B |
| 3-4 | 😔 | 比较糟糕 | #FFA07A |
| 5-6 | 😐 | 一般 | #FFD93D |
| 7-8 | 🙂 | 比较愉快 | #6BCB77 |
| 9-10 | 😊 | 非常愉快 | #4D96FF |

---

## 5. AI 情绪分析规则

### 5.1 每日总结生成规则

1. **数据收集**：获取当日所有心情记录
2. **统计分析**：
   - 计算平均评分
   - 统计表情分布
   - 分析情绪波动幅度
3. **关键词提取**：从备注中提取高频词
4. **生成总结**：
   - 心情概况（一句话总结）
   - 详细分析（2-3段）
   - 温馨提示（1-2句）

### 5.2 对话回应规则

#### 情绪判断关键词
`python
NEGATIVE_KEYWORDS = ['难过', '伤心', '焦虑', '压力大', '累', '烦']
POSITIVE_KEYWORDS = ['开心', '高兴', '顺利', '感谢', '不错']
NEUTRAL_KEYWORDS = ['一般', '还好', '正常']
`

#### 回应策略
1. 检测到负面情绪 → 共情 + 引导 + 放松建议
2. 检测到正面情绪 → 肯定 + 鼓励保持
3. 中性情绪 → 温和关注 + 开放提问

---

## 6. UI 设计规范

### 6.1 颜色规范

`css
/* 主色调 */
--primary-bg: #FFF9E6;      /* 淡黄背景 */
--primary-accent: #FFD93D;   /* 黄色强调 */
--primary-dark: #F0C419;     /* 深黄 */

/* 辅助色 */
--text-primary: #333333;     /* 主文字 */
--text-secondary: #666666;   /* 次要文字 */
--border-color: #E8E0CC;     /* 边框 */

/* 状态色 */
--success: #6BCB77;          /* 绿色-正面 */
--warning: #FFA07A;          /* 橙色-中性 */
--danger: #FF6B6B;           /* 红色-负面 */
`

### 6.2 字体规范
- 中文字体：微软雅黑 / 思源黑体
- 英文字体：Segoe UI
- 正文字号：14px
- 标题字号：18-24px
- 按钮字号：14px

### 6.3 间距规范
- 元素内边距：8px / 12px / 16px
- 元素外边距：8px / 16px / 24px
- 卡片圆角：8px

---

## 7. 开发规范

### 7.1 代码风格
- 遵循 PEP 8 编码规范
- 函数/变量使用蛇形命名法 (snake_case)
- 类名使用大驼峰命名法 (PascalCase)
- 每个模块添加 docstring

### 7.2 提交规范
- 每次功能完成提交一次代码
- 提交信息格式：[模块名] 简要描述
- 示例：[mood_record] 实现心情记录基本功能

### 7.3 错误处理
- 所有数据库操作使用 try-except
- UI操作需要捕获异常并友好提示
- 关键操作记录日志

---

## 8. 依赖包清单

`	xt
PyQt6>=6.5.0
PyInstaller>=5.0
`

---

*文档版本：v1.0*
*最后更新：2026-07-07*
