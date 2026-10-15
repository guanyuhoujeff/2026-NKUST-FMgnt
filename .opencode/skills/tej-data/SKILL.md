---
name: tej-data
description: TEJ 匯出檔的資料說明與 Chen, Lin & Huang (2022) 的變數定義。做資料整理的任何一步之前先讀這份
---

# TEJ 資料說明與變數定義

**文獻**：Chen, Y.-F., Lin, F.-L., & Huang, W.-K. (2022). Environment, social and governance (ESG) and bank loan interest rates. *Corporate Management Review, 42*(2), 51–79.

## 做事的規則

1. **一次只做使用者指定的那一步**，做完就停，不要自己往下做。
2. 每一步的程式寫成一個檔（使用者會指定檔名），用 `python3` 執行。跑不過就自己修、自己重跑。
3. 做完印出使用者要求的數字（例如筆數），直接從程式輸出複製，不可以自己算或手寫。
4. 變數定義照下面的表，不可自己增減、改名或換定義。

## 資料檔（都在 `data/`）

所有檔案：**Big5（cp950）編碼、TAB 分隔、第一列是欄位名稱、每一格前後都有空白要去掉**。

| 檔案 | 內容 | 年份對應 |
|---|---|---|
| `借款明細_2008_2019.txt` | 每一期還沒還完的借款清單 | `年月` 是 TEJ 清單那一期，**不是簽約日**；同一筆借款會在好幾期重複出現。契約年 t ＝ `借款期間(起)` 前四碼（簽約年），只留 t 在 2008–2019 |
| `財報年_一般產業_2007_2018.txt` | 公司財報，**每年有 3、6、9、12 月四期** | 只留 `年/月` 第 5–6 碼是 `12` 的列（年報）；`財報年` ＝ `年/月` 前 4 碼。YYYY 年的財報 → 對到簽約年 **t = YYYY+1** 的借款 |
| `TCRI排行榜_2007_2018.txt` | TCRI 信用評等，**每年有 3、6、9、12 月四期** | 只留 `年/月` 第 5–6 碼是 `12` 的列；`評等年` ＝ `年/月` 前 4 碼。YYYY 年的評等 → 對到簽約年 **t = YYYY+1** 的借款 |
| `公司基本資料_含下市.txt` | 產業別 | 不分年 |

⚠️ 財報和 TCRI 檔的 `年/月` 是 6 位數 `YYYYMM`（例如 `200712`）。**同一家公司每年有四列，沒有先只留 12 月，後面的合併會重複。**

⚠️ 公司代碼欄位名稱不一樣：借款明細叫 `公司代碼`；財報和 TCRI 叫 `公司`；**公司基本資料的第一欄叫 `公司簡稱`，但裡面放的其實是公司代碼**。

## 同一筆借款只算一次

1. **先刪掉五個欄位有任何一個是空白的列**（沒有簽約日或利率，無法認定是哪一筆，後面也算不出變數）。
2. **同一筆借款的認定**：`公司代碼`、`借款銀行碼`、`借款期間(起)`、`借款期間(迄)`、`最低利率％` 五個欄位都一樣，就是同一筆借款。
3. **同一筆只留 `年月` 最早的那一期**；那一期如果有好幾列，都留。

這是為了對齊原文「the loan rate of contract c firm i signed in year t」（t 是簽約年）而加的規則；原文沒有寫怎麼去掉重複。

## 變數定義（原文 Appendix ＋ TEJ 欄位）

| 變數 | 原文定義 | 怎麼算 |
|---|---|---|
| `Rate` | Bank loan rate, the interest rates of each long-term loan contract | `最低利率％` |
| `TCRI` | TCRI, rating from 1 to 9 with 1 the least credit risk | TCRI 檔的 `TCRI` 欄，**只留 1–9**（`-`、`D`、空白都當缺值），當連續數值用 |
| `Size` | Natural log of total assets measured in millions of NT dollars | ln(`資產總額` ÷ 1000)；`資產總額` 單位是千元；≤0 當缺值 |
| `MB` | Market-to-book ratio | `季底普通股市值` ÷ `股東權益總額`（兩者都是千元）；分母為 0 當缺值 |
| `ROA` | Return on total assets | `合併總損益` ÷ `資產總額`（**不要用** `ROA(A)稅後息前`，也不要用 `歸屬母公司淨利（損）`） |
| `Debt` | Total liabilities divided by total assets | `負債比率` ÷ 100 |
| `LLoan` | Natural log of the loan amount measured in millions of NT dollars | ln(`借款金額(台幣)` ÷ 1000)；`借款金額(台幣)` 單位是千元；≤0 當缺值 |
| `LMaturity` | Natural log of the loan maturity measured in days | ln(`借款期間(迄)` − `借款期間(起)` 的天數)；日期格式 YYYYMMDD；天數 ≤0 當缺值 |
| `CollateralD` | 1 if collaterals are required, 0 otherwise | `擔保 (Y/N)` 為 `Y` → 1，其他 → 0 |

## Rate 的極端值：剔除最低 1%、最高 1%（原文沒寫，我們加的）

1. 用整理好、要進迴歸的那一份資料的**全部列**算 `Rate` 的第 1 和第 99 百分位數（pandas 的 `quantile`，預設方法）。
2. **刪掉**（不是改值）`Rate` 比第 1 百分位數小、或比第 99 百分位數大的列；剛好等於百分位數的留下。
3. 只看 `Rate`，其他變數不做這個處理。
