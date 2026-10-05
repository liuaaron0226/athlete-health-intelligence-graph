# AHIG — Athlete Health Intelligence Graph

AHIG 想回答的是：一個不是該領域專家的人，在 AI 協助下，怎麼取得**可以追查、也知道自己限制**的健康答案。

它不直接產生建議。每個問題交付一張**決策卡**，同時寫出目前知道什麼、適用條件、仍不知道什麼，以及可以回頭查證的來源識別碼。

> **本 repo 是從私人工作區抽出的公開子集，個人健康資料不在此。**
> 個人健康紀錄、論文全文與原文引句都放在外部的私有資料根（由環境變數 `AHIG_PRIVATE_ROOT` 指定），不進 Git。
> 少數工作紀錄原本寫到擁有者的個人情況，公開版已改寫成 `〔公開版移除：…〕`。

## 證據管線與對應程式

| 管線層 | 做什麼 | 主要位置 |
|---|---|---|
| 1. 搜尋契約 | 先把問題和來源範圍寫成契約並凍結，再照契約搜尋 | `ahig/calibration/b11-carbohydrate/`（搜尋／範圍契約與查詢式）、`ahig/ahig/contracts/freeze.py`、`ahig/ahig/search/runner.py`、`ahig/ahig/scope/` |
| 2. 去重與篩選 | 用 DOI／PMID／PMCID 這類精確識別碼合併同一篇；判讀不一致的送去稽核，不自動裁掉 | `ahig/ahig/search/candidates.py`、`screening.py`、`screening_decisions.py`、`screening_driver.py`、`llm_second_review.py`、`statistical_termination.py`、`ahig/ahig/stats/family.py` |
| 3. 全文與引用 | 取得開放取用全文，解析成節段，並保留文件版本、解析器版本與節段位置 | `ahig/ahig/search/fulltext.py`、`ahig/ahig/extraction/` |
| 4. 品質檢查 | 核對數值與引文是否真的支持主張；算得出來的數字由程式重算 | `ahig/ahig/stats/deterministic.py`、`adjudication.py`、`grade.py`、`ahig/ahig/gates/`、`ahig/shapes/`（含 canary）、`ahig/schema/` |
| 5. 決策卡 | 每個問題一張，寫出結論、適用條件與限制 | `decisions/` |

## 三張決策卡

- `decisions/001-stress-eating-interventions.md`：壓力性進食
- `decisions/002-sleep-and-fat-loss.md`：睡眠
- `decisions/003-rate-of-loss-and-muscle.html`：減重速度與肌肉保留

決策卡是**盤點既有系統性回顧**的結果，原文引句留在私有資料根。各問題的盤點工作紀錄放在 `ahig/domains/`。

## 其他目錄

- `docs/adr/`：系統層級的架構決策（0006–0017），包括公私分離、AI 判讀的信任模型、凍結 B.11 改交付決策卡、M1 的處置
- `ahig/docs/`：AHIG 內部的 ADR、契約文件與遷移說明
- `docs/new-question-recipe.md`：新問題的處理方式
- `ahig/README.md`：程式本身的說明

## 執行測試

需要 Python 3.11 以上。

```bash
cd ahig
pip install jsonschema numpy scipy rdflib pyshacl scikit-learn pytest
python tests/run_tests.py          # 單元與整合測試
python -m ahig.cli verify --all    # 十個階段的完整驗證（含私密路徑掃描）
```

公開驗證不需要設定 `AHIG_PRIVATE_ROOT`。`tests/test_extraction_bridge.py` 與 `tests/test_extraction_chain.py` 會 import `pytest`，所以沒裝 pytest 時這兩個檔案會失敗。

## 限制

- **完整原文抽取已凍結。** 它在校準階段曾產生原文中不存在的數字，也曾引用不支持主張的段落。這部分沒有通過內容驗收，不用來直接下結論（ADR-0015）。
- **M1 未達成。** M1 原定為「固定測試集通過內容驗收」，但測試集從未選出（ADR-0016、ADR-0017）。
- **AI 判讀未經領域專家審核。** 證據由 AI 判讀並留下紀錄，由擁有者抽查，不是醫學專家的審核（ADR-0009）。給家人使用之前，必須有專業人員參與。
- **尚未完成：** 手錶資料整合、家人介面。
- **3D 臥推示意工具不在本 repo。** 該工具以 BodyParts3D 模型製作（BodyParts3D, © The Database Center for Life Science, licensed under CC Attribution 4.0 International）；它是模型示意，不是人體量測。
