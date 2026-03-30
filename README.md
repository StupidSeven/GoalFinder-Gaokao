# GoalFinder — 高考志愿智能填报系统

> **助力学子探寻理想路径，科学决策未来。**

`GoalFinder` 是一款参考行业级 **EMS (电力管理系统)** 标准打造的高考志愿填报决策支持系统。它结合了大数据抓取、AI 策略建议与严谨的志愿表诊断功能，旨在消除信息差，为全国考生提供最精准、最易用的填报方案。

## 📖 文档导航

为了让您快速了解本项目，请根据需求查阅以下详细文档：

1. **[核心设计方案 (Design Specification)](file:///Users/tradingfront/Projects/Volunteer/GOALFINDER-DESIGN.md)**
   - 包含：项目背景、竞品分析（夸克/百度/掌上高考）、UI 视觉规范（教育类风格）。
2. **[数据采集与反爬架构 (Data Engineering)](file:///Users/tradingfront/Projects/Volunteer/GOALFINDER-SCRAPER-DESIGN.md)**
   - 包含：Python 分布式抓取、考试院反爬策略应对、人工复核工作流。
3. **[核心推荐算法 (Algorithm Spec)](file:///Users/tradingfront/Projects/Volunteer/GOALFINDER-ALGORITHM-SPEC.md)**
   - 包含：位次偏移法、冲稳保逻辑、志愿表全自动诊断算法。
4. **[开发与上线路线图 (Roadmap)](file:///Users/tradingfront/Projects/Volunteer/GOALFINDER-ROADMAP.md)**
   - 包含：分阶段里程碑、高考出分高峰期数据抢抓时间表。

## 🚀 核心亮点

- **教育美学 UI**：精心设计的“阳光蓝”与“常青绿”界面，缓解填报焦虑。
- **AI 赋能驱动**：利用 LLM 自动解析招生简章，实现全自动信息提取与冲突检测。
- **极致数据准确**：三级复核机制，确保作为决策依据的投档线数据万无一失。
- **全国通用建模**：适配新老高考、专业组制等全国主流报考规则。

---

## 🛠 技术栈预览

- **前端**: Next.js 15, Tailwind CSS, Shadcn UI
- **后端**: FastAPI (Python), PostgreSQL, TimescaleDB
- **数据**: Playwright, Scrapy, Redis, Gemini AI (OCR/NLP)
- **部署**: Docker / Kubernetes (具备高并发弹性缩放能力)
