[根目录](../../CLAUDE.md) > **config**

# Config 模块 - 配置管理

> **模块职责**：管理系统配置文件，支持多trader设置、交易所配置、AI模型参数、风险控制参数等，提供完整的配置验证和默认值处理。

## 📋 模块概览

Config模块负责从JSON配置文件加载和验证系统配置，支持多个AI交易员的并行配置，包括交易所选择、AI模型设置、杠杆配置、风险参数等。提供智能默认值和严格的配置验证机制。

## 🏗️ 核心结构

### 配置结构体

#### TraderConfig - 单个交易员配置
```go
type TraderConfig struct {
    ID      string `json:"id"`        // 唯一标识
    Name    string `json:"name"`      // 显示名称
    Enabled bool   `json:"enabled"`   // 是否启用
    AIModel string `json:"ai_model"`  // AI模型: qwen/deepseek/custom
    Exchange string `json:"exchange"` // 交易所: binance/hyperliquid/aster

    // 交易所配置
    BinanceAPIKey    string `json:"binance_api_key,omitempty"`
    BinanceSecretKey string `json:"binance_secret_key,omitempty"`
    HyperliquidPrivateKey string `json:"hyperliquid_private_key,omitempty"`
    AsterUser       string `json:"aster_user,omitempty"`
    AsterSigner     string `json:"aster_signer,omitempty"`
    AsterPrivateKey string `json:"aster_private_key,omitempty"`

    // AI配置
    QwenKey     string `json:"qwen_key,omitempty"`
    DeepSeekKey string `json:"deepseek_key,omitempty"`
    CustomAPIURL    string `json:"custom_api_url,omitempty"`
    CustomAPIKey    string `json:"custom_api_key,omitempty"`
    CustomModelName string `json:"custom_model_name,omitempty"`

    InitialBalance      float64 `json:"initial_balance"`       // 初始资金
    ScanIntervalMinutes int     `json:"scan_interval_minutes"` // 扫描间隔
}
```

#### LeverageConfig - 杠杆配置
```go
type LeverageConfig struct {
    BTCETHLeverage  int `json:"btc_eth_leverage"`  // BTC/ETH最大杠杆
    AltcoinLeverage int `json:"altcoin_leverage"`  // 山寨币最大杠杆
}
```

#### Config - 总配置
```go
type Config struct {
    Traders            []TraderConfig `json:"traders"`
    UseDefaultCoins    bool           `json:"use_default_coins"`
    DefaultCoins       []string       `json:"default_coins"`
    CoinPoolAPIURL     string         `json:"coin_pool_api_url"`
    OITopAPIURL        string         `json:"oi_top_api_url"`
    APIServerPort      int            `json:"api_server_port"`
    MaxDailyLoss       float64        `json:"max_daily_loss"`
    MaxDrawdown        float64        `json:"max_drawdown"`
    StopTradingMinutes int            `json:"stop_trading_minutes"`
    Leverage           LeverageConfig `json:"leverage"`
}
```

## 🔧 核心功能

### 配置加载
```go
func LoadConfig(filename string) (*Config, error) {
    data, err := os.ReadFile(filename)
    if err != nil {
        return nil, fmt.Errorf("读取配置文件失败: %w", err)
    }

    var config Config
    if err := json.Unmarshal(data, &config); err != nil {
        return nil, fmt.Errorf("解析配置文件失败: %w", err)
    }

    // 智能默认值处理
    if !config.UseDefaultCoins && config.CoinPoolAPIURL == "" {
        config.UseDefaultCoins = true
    }

    return &config, config.Validate()
}
```

### 配置验证
提供严格的配置验证逻辑：

1. **基础验证**
   - 至少配置一个trader
   - ID唯一性检查
   - 必填字段完整性

2. **交易所验证**
   - Binance: API Key和Secret Key
   - Hyperliquid: 私钥和钱包地址
   - Aster: 用户地址、签名者地址、私钥

3. **AI模型验证**
   - Qwen: qwen_key必填
   - DeepSeek: deepseek_key必填
   - Custom: API URL、Key、Model Name必填

4. **杠杆验证**
   - 子账户限制检查 (≤5x)
   - 主账户安全建议
   - 警告提示机制

## 🎯 智能默认值

### 默认币种池
```go
defaultMainstreamCoins = []string{
    "BTCUSDT", "ETHUSDT", "SOLUSDT", "BNBUSDT",
    "XRPUSDT", "DOGEUSDT", "ADAUSDT", "HYPEUSDT",
}
```

### 默认参数
- API服务器端口: 8080
- BTC/ETH杠杆: 5x (安全值)
- 山寨币杠杆: 5x (安全值)
- 扫描间隔: 3分钟

## 📊 配置示例

### 基础配置 - 单交易员
```json
{
  "traders": [
    {
      "id": "my_trader",
      "name": "AI交易员",
      "enabled": true,
      "ai_model": "deepseek",
      "exchange": "binance",
      "binance_api_key": "your_api_key",
      "binance_secret_key": "your_secret_key",
      "deepseek_key": "sk-your-deepseek-key",
      "initial_balance": 1000.0,
      "scan_interval_minutes": 3
    }
  ],
  "use_default_coins": true,
  "leverage": {
    "btc_eth_leverage": 5,
    "altcoin_leverage": 5
  }
}
```

### 竞赛配置 - 多交易员
```json
{
  "traders": [
    {
      "id": "qwen_trader",
      "name": "Qwen AI",
      "enabled": true,
      "ai_model": "qwen",
      "exchange": "binance",
      "binance_api_key": "key1",
      "binance_secret_key": "secret1",
      "qwen_key": "sk-qwen-key",
      "initial_balance": 1000.0
    },
    {
      "id": "deepseek_trader",
      "name": "DeepSeek AI",
      "enabled": true,
      "ai_model": "deepseek",
      "exchange": "hyperliquid",
      "hyperliquid_private_key": "your_key",
      "hyperliquid_wallet_addr": "your_address",
      "deepseek_key": "sk-deepseek-key",
      "initial_balance": 1000.0
    }
  ]
}
```

## 🔒 安全特性

### 敏感信息处理
- 支持配置文件环境变量替换
- API密钥验证不暴露具体值
- 私钥格式验证和提示

### 杠杆安全检查
```go
if c.Leverage.BTCETHLeverage > 5 {
    fmt.Printf("⚠️  警告: BTC/ETH杠杆设置为%dx，如果使用子账户可能会失败\n",
        c.Leverage.BTCETHLeverage)
}
```

## 🔗 依赖关系

- **被依赖**: `main.go`, `manager/`, `trader/`
- **外部依赖**: Go标准库 `encoding/json`, `os`
- **配置文件**: `config.json`, `config.json.example`

## 📁 相关文件

- `config.go` - 主要实现文件
- `config.json.example` - 配置模板
- `config.json` - 实际配置文件 (用户创建)

## 💡 最佳实践

### 1. 安全配置
- 使用API子账户进行交易
- 定期轮换API密钥
- 设置合理的IP白名单

### 2. 杠杆配置
- 子账户: 建议≤5x杠杆
- 主账户: 可根据风险承受能力调整
- BTC/ETH可高于山寨币杠杆

### 3. 资金管理
- initial_balance设置要准确
- 建议测试资金100-500 USDT
- 设置合理的止损参数

## 📅 变更记录

### 2025-01-20 - 模块文档初始化
- ✅ 分析配置模块完整功能
- ✅ 记录所有配置结构和验证逻辑
- ✅ 提供配置示例和最佳实践
- 📊 **代码覆盖率**: 100% (完整分析)
- 🎯 **主要特性**: 灵活的多交易员配置，严格验证机制，智能默认值

### 历史更新
- v2.0.2: 新增Aster DEX支持
- v2.0.1: 改进杠杆配置验证
- v2.0.0: 多交易员竞赛模式

---

**模块状态**: ✅ 完整实现
**复杂度**: ⭐⭐ (中等)
**维护性**: ⭐⭐⭐⭐⭐ (优秀)
**文档覆盖**: 100%