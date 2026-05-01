# CLAUDE.md — Global Rules

Last Updated: 2026-04-12

---

## Critical Rules (Immutable)

- Türkçe iletişim kur (kullanıcı aksini belirtmedikçe)
- Kod yazmadan önce mevcut kodu oku ve anla
- Test yazmadan feature tamamlama
- Security best practices uygula (OWASP Top 10)
- Commit mesajları Conventional Commits formatında
- Hallucination yapma — emin değilsen söyle

## Available Custom Agents (21)

Auto-invoked via `Task` tool based on description matching. All agent configs are English; communication language stays Turkish per the Critical Rules.

### Tier 1 — Domain Custom (6)
| Agent | Usage |
|-------|-------|
| `content-creator` | Blog, social, marketing copy, docs, case studies |
| `creative-designer` | Brand identity, visual concepts, taglines, campaigns |
| `ecommerce-analyst` | Revenue, segmentation, churn, pricing, competitive analysis |
| `meta-ads-specialist` | Meta Ads copy, A/B tests, audience, budget optimization |
| `ui-ux-designer` | UI/UX, accessibility, design systems, responsive layout |
| `ai-visual-prompter` | AI image prompts (Midjourney, DALL-E, SD, Flux), bias-aware visual generation |

### Tier 2 — Engineering Adopt (3)
| Agent | Usage |
|-------|-------|
| `fullstack-developer` | Full-stack development, bug fix, refactoring, architecture |
| `data-analyst` | SQL, visualization, reporting, trend analysis |
| `ai-engineer` | MCP, skill design, agent architecture, prompt engineering |

### Tier 3 — Code Quality (4)
| Agent | Usage |
|-------|-------|
| `code-reviewer` | Code review for quality, security, maintainability |
| `debugger` | Bug diagnosis, root cause analysis, error log analysis |
| `oracle` | Deep audit, second opinion, complex bug investigation |
| `refactoring-expert` | Clean code, technical debt reduction, systematic refactoring |

### Tier 4 — Framework Experts (3)
| Agent | Usage |
|-------|-------|
| `nextjs-expert` | Next.js App Router, Server Components, routing, hydration |
| `react-expert` | React hooks, re-rendering, state management, component patterns |
| `typescript-expert` | TypeScript types, generics, build optimization, type safety |

### Tier 5 — Research (1)
| Agent | Usage |
|-------|-------|
| `ux-researcher` | User research, personas, journey maps, usability testing |

### Tier 6 — Workflow (4)
| Agent | Usage |
|-------|-------|
| `sc-business-panel-experts` | Multi-expert business strategy panel (Christensen, Porter, Drucker, Godin, Kim & Mauborgne) |
| `sc-pm-agent` | Multi-agent orchestration, workflow coordination |
| `ecc-planner` | Structured implementation planning for complex features |
| `ecc-build-error-resolver` | Minimal-diff build error and TypeScript error fixes |

> 91 additional agents archived at `~/.claude/agents/_archive/` (recoverable with `mv`).

## Available Custom Skills (26)

### Web Development
- `nextjs-app-router` — Next.js App Router, RSC, Server Actions
- `typescript-standards` — TypeScript best practices, branded types
- `api-development` — REST API tasarımı, error handling, validation
- `prisma-orm` — Prisma ORM, migrations, query optimization
- `tailwind-design-system` — Tailwind CSS, design tokens, responsive
- `nextauth-authentication` — NextAuth.js v5, OAuth, session management
- `neon-serverless` — Neon serverless PostgreSQL

### Testing & Quality
- `testing` — Genel test stratejisi
- `vitest-unit-testing` — Unit test yazımı (Vitest)
- `playwright-e2e` — E2E test (Playwright)
- `code-review` — PR review workflow (5 adım)
- `security-compliance` — Security audit, OWASP
- `performance-optimization` — Core Web Vitals, query profiling

### Infrastructure & DevOps
- `aws-infrastructure` — AWS cloud altyapısı
- `gcp-infrastructure` — Google Cloud Platform
- `vercel-deployment` — Vercel deployment
- `git-workflow` — Git branch strategy, conventional commits
- `database-migrations` — DB migration yönetimi
- `monitoring-logging` — Sistem izleme, log yönetimi

### UX & Content
- `accessibility-ux` — WCAG uyumluluğu, erişilebilirlik
- `seo-optimization` — SEO, meta tags, semantic HTML
- `feedback-analyzer` — Müşteri geri bildirim analizi

### Prompt Templates (docs/examples'dan dönüşüm)
- `business-analysis` — CFO perspektifli iş analizi (5 senaryo)
- `research-tasks` — Sistematik araştırma (5 senaryo)
- `coding-tasks` — Production-ready kodlama (6 senaryo)
- `document-creation` — Profesyonel doküman (5 senaryo)

## Learned Conventions [Auto-updated]

<!-- Claude oturumlar arası öğrendiklerini buraya yazar -->
<!-- Yeni convention eklerken [VERIFY] etiketi kullan -->

## Session Learnings [Temporary]

<!-- Geçici oturum notları — periyodik olarak gözden geçir -->
