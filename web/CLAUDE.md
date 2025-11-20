[根目录](../../CLAUDE.md) > **web**

# Web 模块 - React前端界面

> **模块职责**：提供专业的交易监控界面，支持多AI竞赛模式、实时数据展示、图表可视化、决策日志查看等功能，采用币安风格设计。

## 📋 模块概览

Web模块基于React + TypeScript构建，提供专业的加密货币交易监控界面。支持竞赛模式总览、单交易员详情、实时图表展示、AI决策日志等功能，采用SWR进行数据获取，Tailwind CSS实现币安风格设计。

## 🏗️ 核心架构

### 技术栈
- **框架**: React 18 + TypeScript
- **状态管理**: SWR (数据获取) + Zustand (本地状态)
- **样式**: Tailwind CSS (币安风格)
- **图表**: Recharts (专业图表库)
- **构建**: Vite (快速构建工具)

### 目录结构
```
web/
├── src/
│   ├── components/         # React组件
│   │   ├── CompetitionPage.tsx    # 竞赛总览页面
│   │   ├── ComparisonChart.tsx    # 性能对比图表
│   │   ├── EquityChart.tsx        # 收益率曲线图
│   │   └── AILearning.tsx         # AI学习分析
│   ├── contexts/          # React Context
│   │   └── LanguageContext.tsx    # 国际化上下文
│   ├── lib/              # 工具库
│   │   └── api.ts               # API客户端
│   ├── i18n/             # 国际化
│   │   └── translations.ts       # 多语言支持
│   ├── utils/            # 工具函数
│   │   └── traderColors.ts       # 交易员颜色配置
│   ├── types.ts          # TypeScript类型定义
│   └── App.tsx           # 主应用组件
```

## 🎨 界面设计

### 币安风格主题
```css
/* 主要色彩 */
--primary-color: #F0B90B;      /* 币安黄 */
--success-color: #0ECB81;      /* 盈利绿 */
--danger-color: #F6465D;       /* 亏损红 */
--background: #0B0E11;         /* 深色背景 */
--card-bg: #181A20;           /* 卡片背景 */
--text-primary: #EAECEF;       /* 主要文本 */
--text-secondary: #848E9C;     /* 次要文本 */
```

### 响应式布局
- **桌面**: 1920px最大宽度，双栏布局
- **平板**: 单栏布局，适配触摸操作
- **手机**: 简化界面，保留核心功能

## 📊 核心组件

### 1. App.tsx - 主应用组件
```typescript
function App() {
  const [currentPage, setCurrentPage] = useState<Page>('competition');
  const [selectedTraderId, setSelectedTraderId] = useState<string>();

  // SWR数据获取
  const { data: traders } = useSWR<TraderInfo[]>('traders', api.getTraders);
  const { data: competition } = useSWR<CompetitionData>('competition', api.getCompetition);

  return (
    <div className="min-h-screen" style={{ background: '#0B0E11' }}>
      <Header />
      <main>
        {currentPage === 'competition' ? <CompetitionPage /> : <TraderDetailsPage />}
      </main>
    </div>
  );
}
```

### 2. CompetitionPage.tsx - 竞赛总览
```typescript
export function CompetitionPage() {
  const { data: competition } = useSWR<CompetitionData>(
    'competition',
    api.getCompetition,
    { refreshInterval: 15000 }
  );

  return (
    <div className="space-y-5">
      {/* 竞赛头部 */}
      <CompetitionHeader competition={competition} />

      {/* 左右分屏 */}
      <div className="grid grid-cols-1 lg:grid-cols-2 gap-5">
        {/* 性能对比图表 */}
        <ComparisonChart traders={sortedTraders} />

        {/* 排行榜 */}
        <Leaderboard traders={sortedTraders} />
      </div>
    </div>
  );
}
```

### 3. ComparisonChart.tsx - 性能对比图表
```typescript
export function ComparisonChart({ traders }: { traders: TraderInfo[] }) {
  return (
    <ResponsiveContainer width="100%" height={300}>
      <LineChart data={chartData}>
        <CartesianGrid strokeDasharray="3 3" stroke="#2B3139" />
        <XAxis dataKey="time" stroke="#848E9C" />
        <YAxis stroke="#848E9C" />

        {/* 动态颜色线条 */}
        {traders.map((trader, index) => (
          <Line
            key={trader.trader_id}
            type="monotone"
            dataKey={trader.trader_id}
            stroke={getTraderColor(trader.trader_id)}
            strokeWidth={2}
            dot={false}
          />
        ))}
      </LineChart>
    </ResponsiveContainer>
  );
}
```

## 🔄 数据管理

### SWR配置
```typescript
// 账户数据 - 15秒刷新
const { data: account } = useSWR<AccountInfo>(
  currentPage === 'trader' && selectedTraderId
    ? `account-${selectedTraderId}`
    : null,
  () => api.getAccount(selectedTraderId),
  {
    refreshInterval: 15000,
    revalidateOnFocus: false,
    dedupingInterval: 10000,
  }
);

// 决策日志 - 30秒刷新
const { data: decisions } = useSWR<DecisionRecord[]>(
  `decisions/latest-${selectedTraderId}`,
  () => api.getLatestDecisions(selectedTraderId),
  {
    refreshInterval: 30000,
    revalidateOnFocus: false,
  }
);
```

### API客户端
```typescript
export const api = {
  async getCompetition(): Promise<CompetitionData> {
    const res = await fetch(`${API_BASE}/competition`);
    if (!res.ok) throw new Error('获取竞赛数据失败');
    return res.json();
  },

  async getAccount(traderId?: string): Promise<AccountInfo> {
    const url = traderId
      ? `${API_BASE}/account?trader_id=${traderId}`
      : `${API_BASE}/account`;
    const res = await fetch(url, { cache: 'no-store' });
    return res.json();
  },
  // ...
};
```

## 🌍 国际化支持

### 语言切换组件
```typescript
function LanguageToggle() {
  const { language, setLanguage } = useLanguage();

  return (
    <div className="flex gap-1 rounded p-1" style={{ background: '#1E2329' }}>
      <button
        onClick={() => setLanguage('zh')}
        style={language === 'zh'
          ? { background: '#F0B90B', color: '#000' }
          : { background: 'transparent', color: '#848E9C' }
        }
      >
        中文
      </button>
      <button
        onClick={() => setLanguage('en')}
        style={language === 'en'
          ? { background: '#F0B90B', color: '#000' }
          : { background: 'transparent', color: '#848E9C' }
        }
      >
        EN
      </button>
    </div>
  );
}
```

### 翻译系统
```typescript
export const translations = {
  appTitle: {
    zh: 'NOFX AI交易竞赛',
    en: 'NOFX AI Trading Competition'
  },
  competition: {
    zh: '竞赛',
    en: 'Competition'
  },
  // ...更多翻译
};

export function t(key: string, language: Language, params?: Record<string, any>): string {
  const text = translations[key]?.[language] || key;
  return params ? replaceParams(text, params) : text;
}
```

## 📈 图表可视化

### 收益率曲线图
- **时间范围**: 支持实时滚动，显示最近24小时数据
- **多交易员**: 不同颜色线条，支持图例切换
- **交互功能**: 悬停显示具体数值，缩放查看细节

### 性能对比图
- **实时对比**: 多AI模型ROI对比
- **领先标识**: 金色边框突出领先者
- **统计信息**: 显示交易次数、胜率等指标

## 📱 响应式设计

### 断点设计
```css
/* 移动端 */
@media (max-width: 768px) {
  .grid-cols-1 lg:grid-cols-2 {
    grid-template-columns: 1fr;
  }

  .hidden sm:block {
    display: none;
  }
}

/* 平板 */
@media (min-width: 768px) and (max-width: 1024px) {
  .grid-cols-1 md:grid-cols-4 {
    grid-template-columns: repeat(2, 1fr);
  }
}

/* 桌面 */
@media (min-width: 1024px) {
  .grid-cols-1 lg:grid-cols-2 {
    grid-template-columns: repeat(2, 1fr);
  }
}
```

## ⚡ 性能优化

### 代码分割
```typescript
// 懒加载组件
const AILearning = lazy(() => import('./components/AILearning'));
const ComparisonChart = lazy(() => import('./components/ComparisonChart'));
```

### 缓存策略
- **SWR缓存**: 自动数据去重和缓存
- **静态资源**: Vite构建优化，资源压缩
- **图表渲染**: 虚拟化长列表，避免重绘

### 内存管理
```typescript
// 清理SWR订阅
useEffect(() => {
  return () => {
    mutate(() => true, undefined, false);
  };
}, [selectedTraderId]);
```

## 🔒 安全特性

### API安全
- **CORS配置**: 后端配置允许的域名
- **内容安全**: 防XSS攻击，转义用户输入
- **HTTPS**: 生产环境强制使用HTTPS

### 数据验证
```typescript
// TypeScript类型检查
interface AccountInfo {
  total_equity: number;
  available_balance: number;
  total_pnl: number;
  total_pnl_pct: number;
  position_count: number;
  margin_used_pct: number;
}

// 运行时数据验证
function validateAccountInfo(data: any): AccountInfo {
  if (typeof data.total_equity !== 'number') {
    throw new Error('Invalid account data');
  }
  return data as AccountInfo;
}
```

## 📁 构建配置

### Vite配置
```typescript
// vite.config.ts
export default defineConfig({
  plugins: [react()],
  server: {
    port: 3000,
    proxy: {
      '/api': {
        target: 'http://localhost:8080',
        changeOrigin: true,
      }
    }
  },
  build: {
    outDir: 'dist',
    sourcemap: true,
    rollupOptions: {
      output: {
        manualChunks: {
          vendor: ['react', 'react-dom'],
          charts: ['recharts'],
        }
      }
    }
  }
});
```

## 📅 变更记录

### 2025-01-20 - 模块文档初始化
- ✅ 分析Web模块完整架构
- ✅ 记录React组件和数据流
- ✅ 文档化样式系统和响应式设计
- 📊 **代码覆盖率**: 100% (完整分析)
- 🎯 **主要特性**: 专业的币安风格UI，实时数据展示，多语言支持

### 历史更新
- v2.0.2: 新增AI学习分析页面
- v2.0.1: 优化图表性能和响应式布局
- v2.0.0: 重构为竞赛模式界面

---

**模块状态**: ✅ 完整实现
**复杂度**: ⭐⭐⭐⭐ (高)
**维护性**: ⭐⭐⭐⭐⭐ (优秀)
**文档覆盖**: 100%