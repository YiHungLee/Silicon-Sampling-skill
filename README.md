# Psychology Survey Agent - Silicon Sampling Skill

一個用於填答心理學研究自陳式量表的 Claude Agent Skill，實現 Silicon Sampling 研究方法論。

## 什麼是 Silicon Sampling?

Silicon Sampling 是一種研究方法，使用 AI 模型模擬人類受試者來填答心理學問卷。這種方法可以用於：

- **預測試問卷**：在正式施測前測試問卷的可行性
- **生成模擬數據**：用於教學或方法論研究
- **探索 AI 心理特徵**：研究 AI 模型在心理學量表上的反應模式
- **補充樣本**：在特定人口統計群體難以取得時作為補充

## 功能特點

- **模擬人類填答**：根據指定的人口統計特徵模擬真實人類的填答行為
- **支援 Likert 量表**：支援各種點數的 Likert 量表（5點、7點等）
- **CSV 輸出格式**：結構化的輸出格式，方便後續統計分析
- **可自訂 Persona**：詳細的人口統計和人格特質設定

## 快速開始

### 1. 準備問卷檔案

建立一個 JSON 格式的問卷檔案：

```json
{
  "questionnaire": {
    "name": "Your Scale Name",
    "scale": {
      "type": "likert",
      "min": 1,
      "max": 5
    },
    "items": [
      {
        "item_id": "Q1",
        "text": "問題內容",
        "reverse_scored": false
      }
    ]
  },
  "persona": {
    "age": 25,
    "gender": "female",
    "education": "bachelor"
  },
  "sample_size": 1
}
```

### 2. 執行 Skill

使用 Claude Agent 載入此 skill 並提供問卷檔案。

### 3. 取得輸出

輸出為 CSV 格式：

```csv
respondent_id,item_id,response,response_time_ms
R001,Q1,4,2340
R001,Q2,3,1890
```

## 檔案結構

```
Silicon-Sampling-skill/
├── skill.yaml          # Skill 配置檔
├── prompt.md           # 主要 prompt 模板
├── README.md           # 說明文件
├── LICENSE             # 授權條款
└── examples/           # 範例檔案
    ├── big_five_bfi10.json    # Big Five 人格量表範例
    ├── phq9.json              # PHQ-9 憂鬱量表範例
    ├── persona_template.json  # Persona 模板
    └── sample_output.csv      # 輸出範例
```

## Persona 設定

Persona 定義了模擬受試者的特徵：

| 欄位 | 說明 | 範例值 |
|------|------|--------|
| `age` | 年齡 | 25 |
| `gender` | 性別 | male, female, non-binary |
| `education` | 教育程度 | elementary, high_school, bachelor, master, doctorate |
| `occupation` | 職業 | 軟體工程師 |
| `personality_traits` | Big Five 人格特質 | 見下方 |
| `cultural_background` | 文化背景 | 台灣 |
| `additional_context` | 額外背景資訊 | 自由文字描述 |

### 人格特質設定

```json
"personality_traits": {
  "extraversion": "low|medium|high",
  "agreeableness": "low|medium|high",
  "conscientiousness": "low|medium|high",
  "neuroticism": "low|medium|high",
  "openness": "low|medium|high"
}
```

## 支援的量表格式

### Likert 量表

```json
{
  "scale": {
    "type": "likert",
    "min": 1,
    "max": 5,
    "labels": {
      "1": "非常不同意",
      "2": "不同意",
      "3": "中立",
      "4": "同意",
      "5": "非常同意"
    }
  }
}
```

## 範例量表

本 skill 提供以下範例量表：

1. **BFI-10 (Big Five Inventory-10)**
   - 10 題簡版五大人格量表
   - 測量外向性、親和性、盡責性、神經質、開放性

2. **PHQ-9 (Patient Health Questionnaire-9)**
   - 9 題憂鬱症狀篩檢量表
   - 0-3 點量表

## 輸出說明

### CSV 欄位

| 欄位 | 說明 |
|------|------|
| `respondent_id` | 受試者編號 |
| `item_id` | 題目編號 |
| `response` | 填答數值 |
| `response_time_ms` | 模擬反應時間（毫秒）|

### Metadata

輸出後會附加 YAML 格式的 metadata：

```yaml
---
metadata:
  persona_summary: "25歲女性，大學學歷，軟體工程師"
  completion_time_total_ms: 23450
  items_completed: 10
  response_style: "傾向中間值，避免極端反應"
```

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
