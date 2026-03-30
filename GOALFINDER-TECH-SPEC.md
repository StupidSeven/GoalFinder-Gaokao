# GoalFinder 技术规格说明书 (v1.0) — 核心架构与数据建模

> **目标**：为研发团队提供清晰的技术路径。对标 EMS 系统的高并发处理能力与数据严谨性，确保高考数据零误差。

---

## 1. 系统架构拓扑 (Technical Architecture)

我们采用 **前后端分离 + 跨平台移动端** 的“三位一体”架构：

### 1.1 技术栈清单
- **后端 (Backend)**: Python 3.11 + **FastAPI** (高性能异步框架)。
- **核心数据库 (Primary DB)**: **PostgreSQL 16** (处理复杂业务关联)。
- **高速缓存 (Cache)**: **Redis 7** (存储全省一分一段表，实现毫秒级位次检索)。
- **考生/管理 Web**: **Next.js 14** (响应式预览与 Admin 统一入口)。
- **用户移动 App**: **React Native (Expo)** (一套代码发布 iOS/Android)。

### 1.2 部署架构
- **容器化**: Docker + GitHub Actions (CI/CD)。
- **弹性扩容**: 针对高考出分周 (72h) 的 QPS 峰值，设计自动横向扩展策略。

---

## 2. 数据库设计 (Core DDL - PostgreSQL)

这是系统最核心的底座。我们将定义以下关键表结构：

```sql
-- 1. 院校与专业基础表
CREATE TABLE majors (
    major_id UUID PRIMARY KEY,
    college_name VARCHAR(100) NOT NULL,
    major_name VARCHAR(200) NOT NULL,
    province_name VARCHAR(50), -- 辽宁
    subject_requirement VARCHAR(50), -- 物理/历史/不限
    degree_type VARCHAR(20), -- 本科/专科
    location VARCHAR(100) -- 地理位置
);

-- 2. 历年录取与趋势表 (核心可视化数据源)
CREATE TABLE admission_history (
    id SERIAL PRIMARY KEY,
    major_id UUID REFERENCES majors(major_id),
    year INTEGER NOT NULL,
    enrollment_plan INTEGER, -- 计划招生人数 (扩招/缩招判断位)
    min_rank INTEGER, -- 最低录取位次
    avg_rank INTEGER,
    min_score DECIMAL(5,2)
);

-- 3. 辽宁省 112 志愿表
CREATE TABLE student_volunteers (
    student_id UUID NOT NULL,
    rank_order INTEGER NOT NULL, -- 1 to 112
    major_id UUID REFERENCES majors(major_id),
    strategy_tag VARCHAR(20), -- 冲/稳/保
    PRIMARY KEY (student_id, rank_order)
);
```

---

## 3. 核心可视化算法：位次-扩招相关性模型

### 3.1 趋势图建模逻辑
系统将计算 **`Enrollment_Shift_Ratio` (扩招偏移率)**：
- $\Delta E = (E_{current} - E_{last\_year}) / E_{last\_year}$
- **逻辑应用**：如果 $\Delta E > 20\%$ (大幅扩招)，系统在趋势图中会自动标记“位次下调风险”，提示考生有更大概率“捡漏”。

### 3.2 地理位置权重计算 (Geo-Weight)
- 用户设置偏好地区 $V_{geo}$。
- 推荐得分 $S = S_{rank} \times (1 + W_{geo})$，其中偏好地院校的权重 $W_{geo}$ 会被动态调高，使其在 112 志愿生成时优先浮动。

---

## 4. API 契约与数据流 (API Design)

| 端点 (Endpoint) | 描述 | 输入参数 | 产出物 |
| :--- | :--- | :--- | :--- |
| `GET /v1/trends` | 获取某专业历史趋势 | `major_id` | 5年位次+人数变动曲线 JSON |
| `POST /v1/recommend` | 生成 112 志愿魔方 | `rank`, `geo_pref`, `subjects` | 112 项分组推荐列表 |
| `POST /v1/admin/review` | 管理员三级复核 | `data_batch_id` | 复核状态确认 |

---

## 5. 安全与隐私 (Compliance)

- **数据加密**: 用户手机号与分数采用 AES-256 加密。
- **访问控制**: 管理员 (Admin) 与普通用户权限逻辑严格隔离，通过 JWT 实现状态保持。
