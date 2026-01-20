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
      "neuroticism": { "mean": 52, "sd": 17 },
      "extraversion": { "mean": 55, "sd": 18 },
      "openness": { "mean": 62, "sd": 16 },
      "agreeableness": { "mean": 60, "sd": 15 },
      "conscientiousness": { "mean": 50, "sd": 18 }
    }
  }
}
```

## Persona 自動生成

根據母群定義，skill 會自動生成指定數量的獨特 persona，使用 NEO-PI-R 30 構面人格模型：

```
Persona R001:
- 年齡: 21
- 性別: 女
- 教育: 大學在學（心理學系三年級）
- 職業: 學生（兼職咖啡店店員）
- 人格特質 (NEO-PI-R 30 Facets, PR 分數 1-100, 50=母群平均):
  【神經質 N】焦慮:45, 憤怒敵意:38, 憂鬱:42, 自我意識:55, 衝動:52, 脆弱:40
  【外向性 E】熱情:68, 合群:72, 獨斷:58, 活力:65, 尋求刺激:60, 正向情緒:70
  【開放性 O】想像:75, 審美:78, 感受:72, 嘗試:68, 理念:70, 價值:65
  【親和性 A】信任:62, 坦誠:70, 利他:75, 順從:58, 謙虛:55, 溫柔:68
  【盡責性 C】勝任感:60, 條理:52, 責任:58, 追求成就:62, 自律:48, 深思熟慮:55
- 背景: 來自台北，對心理學有興趣，社交活躍
```

每個 persona 都會：
- 遵循母群的人口統計分布
- 具有獨特但合理的人格特質組合
- 反映該母群的文化背景特徵

## 輸出格式

### 1. responses.csv - 填答結果 (Wide Format)

每列為一個 persona，每欄為一個題目：

```csv
respondent_id,BFI10_1,BFI10_2,BFI10_3,BFI10_4,BFI10_5,BFI10_6,BFI10_7,BFI10_8,BFI10_9,BFI10_10
R001,2,4,5,2,4,4,2,1,4,2
R002,4,3,3,4,3,2,3,4,2,3
```

### 2. personas.csv - Persona 詳細資料 (含 30 構面)

```csv
respondent_id,age,gender,education,occupation,background,N1_anxiety,N2_hostility,N3_depression,N4_self_consciousness,N5_impulsiveness,N6_vulnerability,E1_warmth,E2_gregariousness,E3_assertiveness,E4_activity,E5_excitement_seeking,E6_positive_emotions,O1_fantasy,O2_aesthetics,O3_feelings,O4_actions,O5_ideas,O6_values,A1_trust,A2_straightforwardness,A3_altruism,A4_compliance,A5_modesty,A6_tender_mindedness,C1_competence,C2_order,C3_dutifulness,C4_achievement_striving,C5_self_discipline,C6_deliberation
R001,21,female,bachelor_3,student_parttime,"台北，心理系",45,38,42,55,52,40,68,72,58,65,60,70,75,78,72,68,70,65,62,70,75,58,55,68,60,52,58,62,48,55
R002,23,male,bachelor_4,student,"新竹，資工系",52,45,55,60,48,58,42,38,45,50,55,40,60,55,58,52,65,60,55,58,52,62,65,55,55,48,52,58,45,52
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
├── SKILL.md                            # Skill 定義檔（遵循 Anthropic Skills 規範）
├── README.md                           # 說明文件
├── LICENSE                             # 授權條款
├── references/                         # 台灣人口統計參考資料
│   ├── 114-12_Age_composition_by_city.csv   # 民國114年各縣市年齡性別分布
│   └── 113_education.csv                    # 民國113年教育程度分布
└── examples/
    ├── population_tw_college.json      # 台灣大學生母群定義
    ├── population_general_adult.json   # 台灣一般成人母群定義
    ├── big_five_bfi10.json             # BFI-10 量表範例
    ├── phq9.json                       # PHQ-9 量表範例
    └── sample_output.csv               # 輸出範例
```

## 預設母群

若使用者未指定母群，skill 會使用 **台灣成年人口** 作為預設，依據 `references/` 資料夾中的人口統計資料：

| 屬性 | 分布 |
|------|------|
| 性別 | 男 49.2%、女 50.8% |
| 年齡 | 20-49歲 49.4%、50-64歲 30.5%、65歲以上 20.1% |
| 教育 | 國小以下 10.8%、國中 9.6%、高中職 28.8%、專科 11.7%、大學 30.4%、研究所 8.7% |

資料來源：內政部戶政司、行政院主計總處

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
