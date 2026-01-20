# Psychology Survey Agent - Silicon Sampling Skill

一個用於填答心理學研究自陳式量表的 Claude Agent Skill，實現 Silicon Sampling 研究方法論。

## 什麼是 Silicon Sampling?

Silicon Sampling 是一種研究方法，使用 AI 模型模擬人類受試者來填答心理學問卷。這種方法可以用於：

- **預測試問卷**：在正式施測前測試問卷的可行性
- **生成模擬數據**：用於教學或方法論研究
- **探索 AI 心理特徵**：研究 AI 模型在心理學量表上的反應模式
- **補充樣本**：在特定人口統計群體難以取得時作為補充

## 功能特點

- **互動式流程**：自動詢問缺少的資訊（母群、樣本數、量表）
- **母群定義**：根據母群特徵自動生成多樣化的 persona
- **模擬人類填答**：根據 persona 的人口統計特徵和人格特質模擬真實填答行為
- **支援 Likert 量表**：支援各種點數的 Likert 量表（5點、7點等）
- **CSV 輸出格式**：結構化的輸出格式，方便後續統計分析

## 使用流程

```
┌─────────────────────────────────────────────────────────────┐
│  Step 1: 指定母群 (Population)                              │
│  ↓ (若未指定，skill 會詢問)                                 │
├─────────────────────────────────────────────────────────────┤
│  Step 2: 指定樣本數量                                       │
│  ↓ (若未指定，skill 會詢問)                                 │
├─────────────────────────────────────────────────────────────┤
│  Step 3: 提供量表題目                                       │
│  ↓                                                          │
├─────────────────────────────────────────────────────────────┤
│  Step 4: 自動生成 Personas                                  │
│  ↓ (根據母群特徵生成指定數量的獨特 persona)                 │
├─────────────────────────────────────────────────────────────┤
│  Step 5: 模擬填答                                           │
│  ↓ (根據每個 persona 的特徵模擬真實填答行為)                │
├─────────────────────────────────────────────────────────────┤
│  Step 6: 輸出 CSV 結果                                      │
└─────────────────────────────────────────────────────────────┘
```

## 快速開始

### 方式一：互動式使用

直接啟動 skill，它會逐步詢問您：

```
User: 請幫我用 Silicon Sampling 填答問卷

Agent: 在開始模擬填答之前，我需要了解您的目標母群。請提供以下資訊：
       1. 母群名稱/描述
       2. 年齡範圍
       3. 性別分布比例
       ...

User: 台灣大學生，年齡 18-22 歲，男女各半

Agent: 請問您需要生成多少個模擬樣本？

User: 30 個

Agent: 請提供您的量表題目...
```

### 方式二：一次性提供所有資訊

```
User: 請幫我用 Silicon Sampling 填答問卷
      母群：台灣 25-35 歲的上班族
      樣本數：50
      量表：BFI-10 五大人格量表
```

## 母群定義

母群 (Population) 定義了您想要模擬的目標群體特徵：

| 屬性 | 說明 | 範例 |
|------|------|------|
| `name` | 母群名稱 | 台灣大學生 |
| `age_range` | 年齡範圍與分布 | 18-24 歲，平均 20.5 歲 |
| `gender_distribution` | 性別分布比例 | 男 48%、女 50%、其他 2% |
| `education_distribution` | 教育程度分布 | 大學在學 95%、碩士 5% |
| `occupation_types` | 常見職業類型 | 學生、學生(兼職) |
| `cultural_background` | 文化背景 | 台灣 |
| `personality_tendencies` | 人格特質傾向 | Big Five 各維度的平均與變異 |

### 母群範例檔案

```json
{
  "population": {
    "name": "台灣大學生",
    "age_range": {
      "min": 18,
      "max": 24,
      "mean": 20.5,
      "distribution": "normal"
    },
    "gender_distribution": {
      "male": 0.48,
      "female": 0.50,
      "non_binary": 0.02
    },
    "cultural_background": "台灣",
    "personality_tendencies": {
      "extraversion": { "mean": 55, "sd": 18 },
      "openness": { "mean": 62, "sd": 16 }
    }
  }
}
```

## Persona 自動生成

根據母群定義，skill 會自動生成指定數量的獨特 persona：

```
Persona R001:
- 年齡: 21
- 性別: 女
- 教育: 大學在學（心理學系三年級）
- 職業: 學生（兼職咖啡店店員）
- 人格特質:
  - 外向性 (E): 62
  - 親和性 (A): 71
  - 盡責性 (C): 55
  - 神經質 (N): 48
  - 開放性 (O): 73
- 背景: 來自台北，對心理學有興趣，社交活躍
```

每個 persona 都會：
- 遵循母群的人口統計分布
- 具有獨特但合理的人格特質組合
- 反映該母群的文化背景特徵

## 輸出格式

### 1. responses.csv - 填答結果

```csv
respondent_id,item_id,response,response_time_ms
R001,Q1,4,2340
R001,Q2,2,1890
R001,Q3,5,3120
R002,Q1,3,2100
R002,Q2,3,2450
```

### 2. personas.csv - Persona 詳細資料

```csv
respondent_id,age,gender,education,occupation,extraversion,agreeableness,conscientiousness,neuroticism,openness
R001,21,female,bachelor_current,student,62,71,55,48,73
R002,23,male,bachelor_current,student,45,58,67,52,61
```

### 3. Summary - 摘要統計

```yaml
---
summary:
  population: "台灣大學生"
  sample_size: 30
  questionnaire: "BFI-10"

  demographics:
    age_mean: 21.3
    age_sd: 1.8
    gender_ratio: "male: 47%, female: 53%"

  response_statistics:
    mean_response_time_ms: 2450
    completion_rate: 100%
```

## 檔案結構

```
Silicon-Sampling-skill/
├── skill.yaml                          # Skill 配置檔
├── prompt.md                           # 主要 prompt 模板
├── README.md                           # 說明文件
├── LICENSE                             # 授權條款
└── examples/
    ├── population_tw_college.json      # 台灣大學生母群定義
    ├── population_general_adult.json   # 台灣一般成人母群定義
    ├── big_five_bfi10.json             # BFI-10 量表範例
    ├── phq9.json                       # PHQ-9 量表範例
    └── sample_output.csv               # 輸出範例
```

## 支援的量表格式

### Likert 量表

**5 點量表：**
1. 非常不同意
2. 不同意
3. 中立
4. 同意
5. 非常同意

**7 點量表：**
1. 非常不同意
2. 不同意
3. 有點不同意
4. 中立
5. 有點同意
6. 同意
7. 非常同意

**頻率量表 (如 PHQ-9)：**
0. 完全沒有
1. 幾天
2. 超過一半的天數
3. 幾乎每天

## 範例量表

本 skill 提供以下範例量表：

1. **BFI-10 (Big Five Inventory-10)**
   - 10 題簡版五大人格量表
   - 測量外向性、親和性、盡責性、神經質、開放性

2. **PHQ-9 (Patient Health Questionnaire-9)**
   - 9 題憂鬱症狀篩檢量表
   - 0-3 點頻率量表

## 研究倫理說明

使用 Silicon Sampling 方法時，請注意：

1. **透明度**：在研究報告中明確說明使用了 AI 模擬數據
2. **效度限制**：AI 模擬的反應可能無法完全代表真實人類
3. **用途限制**：不應將模擬數據用於需要真實人類反應的臨床決策
4. **補充而非替代**：建議作為真實數據的補充而非完全替代

## 授權

MIT License

## 參考文獻

- Argyle, L. P., et al. (2023). Out of One, Many: Using Language Models to Simulate Human Samples. *Political Analysis*.
- Rammstedt, B., & John, O. P. (2007). Measuring personality in one minute or less. *Journal of Research in Personality*.
- Kroenke, K., Spitzer, R. L., & Williams, J. B. (2001). The PHQ-9. *Journal of General Internal Medicine*.
