<!DOCTYPE html>
  <html>
   
  <hr>
  <h2>
    <em>實證分析期末作業Replicate_
    Readme File</em>
  </h2>
  <h2>小組成員</h2>
  組長：林楷惟 114353018
  組員：陳心慧 114353010、蔡欣彤 114353012
  、范聖宸 114353101、莊雅淳 114353117
  
  <h2>Data Availability and Provenance Statements</h2>
  
  <h3>Details on each Data Source</h3>
  
| 產出圖表 |  相關 Do 檔  | 相關 .dta 檔 | 資料庫來源 | 雲端硬碟位置 |
|:-----|:--------:|:--------:|:--------:|------:|
| Table 4 | JAR_1976-2011_table4.do | 完整主檔.dta ; DISP_FE.dta ; EPS真.dta ; LBES_DISP.dta ; table3大表.dta | Compustat, LBES | 資料夾"Table4"內 |
| L1   |  `code`  |   $12 | $1600 | $1600 |
| L2   | _italic_ |    $1 | $1600 | $1600 |
| L2   | _italic_ |    $1 | $1600 | $1600 |
| L2   | _italic_ |    $1 | $1600 | $1600 |

<h2>Description of programs/code</h2>
<h3>Table 1 panelA </h3>

*  Step 1. 讀入資料
讀入 新replica.dta 作為基礎資料集。
* Step 2. 建構 DQ_BS（Balance Sheet；Value-Weighted）
定義 BS 子科目清單（Subaccounts）：以 local act_sub ... 等方式列出各群組子科目。
定義 BS parent 群組（用於權重）：local groups act ao ceq dltt intan ivao lct lo ppent pstk txditc
計算各群組揭露比例：
以 egen rownonmiss() 計算該群組子科目「非缺漏數」
揭露比例 dq_g = nonmissing / total_items_in_group
計算價值權重並標準化：
先算 weight_raw_g = abs(parent_g) / total_assets，再把各群組 raw weight 加總為 total_weight_raw
再標準化 weight_g = weight_raw_g / total_weight_raw（確保權重總和為 1）
加權後加總得到 DQ_BS：
vw_dq_g = dq_g * weight_g
DQ_BS = rowtotal(vw_dq_list)
對 DQ_BS 做 1%/99% winsorize：
winsor2 DQ_BS, cut(1 99) replace
* Step 3. 建構 DQ_IS（Income Statement；Equal-Weighted with Screening）
定義 IS 群組與子科目（共 8 組）：Sales、COGS、Op Expenses、Special Items、Interest、Taxes、Minority Interest、Income Below 等。
Parent screening（適用性判斷）：
對每一群組建立 has_is_group_k：當 parent 變數存在且「非缺漏且不等於 0」時，該群組才視為 applicable。
計算分子/分母：
若群組適用（has_is_group_k==1），則該群組的每個子科目都計入分母；子科目若非缺漏則分子 +1。
計算 DQ_IS：
DQ_IS = dq_is_numerator / dq_is_denominator（denominator > 0 才計算）
若 denominator==0 且 numerator==0，程式將 DQ_IS 設為 0（代表沒有任何適用群組）
* Step 4. 計算整體 DQ
DQ = (DQ_BS + DQ_IS) / 2
* Step 5. 樣本限制（Replication Criteria）
補齊年度欄位：
若沒有 year，則優先用 fyear，否則用 year(datadate) 生成。
保留樣本期間：
程式保留 1973–2011：keep if inrange(year, 1973, 2011)
排除產業（依 SIC）：
金融業：6000–6999
公用事業：4900–4999
程式先 destring sic 後再 drop if ...
* Step 6. 產出 Panel A 描述統計並輸出 RTF
用 tabstat 計算 DQ、DQ_BS、DQ_IS 的 mean sd p25 p50 p75，並以 save 將結果存到 r(StatTotal)。
將 r(StatTotal) 存成矩陣 T1，必要時轉置（讓列是變數、欄是統計量）。
設定矩陣列名/欄名（DQ、DQ_BS、DQ_IS；Mean、SD、P25、P50、P75）。
使用 esttab matrix() 輸出為 Word 可開啟的 RTF，並固定到小數點後三位。
輸出檔（Output）
RTF 表格：Table1_PanelA.rtf
三欄分別為：DQ、DQ_BS、DQ_IS
內容為DQ、DQ_BS、DQ_IS 的描述性統計（Mean、SD、P25、P50、P75）
套件需求（Dependencies）
套件需求（Dependencies）
winsor2（用於 winsorize DQ_BS）
estout（用於 esttab 輸出矩陣到 RTF）安裝方式（建議僅需一次）：
ssc install winsor2, replace
ssc install estout, replace
執行方式（How to run）
在 Stata 14 中將工作目錄設為專案資料夾後執行：
do "table1 panelA.do"
Table 2－Correlation Between DQ and Other Measures（Upper: Pearson）


<h3>Table 4 </h3>
首先把完整主檔.dta打開，內含之前計算的DQ、Compustat資料、股價資料及各種所需變數，並與 EPS真.dta 併檔。
併檔後針對actual_EPS 進行平減（按照股價）
導入rangestat進行EPS之標準差計算。
並存入EPS.波動.dta待命。

*  建構自變數

打開完整主檔.dta，計算Sale 的成長率，並立刻進行winsorize，存入
GROWTH。

打開完整主檔，計算ROA，winsorize後存入ROA。

打開LBES_DISP，將每月資料平均至年資料後，計算log(AF)存入AF。

打開完整主檔，計算log(AT)，存入AT。

* 清理資料


打開已整理好的DISP_FE、完整主檔、EPS波動、GROWTH、ROA、AF、AT.dta的變數刪到只剩他們各自的代表性變數，以求後續整理更清楚方便，且把所有變數的閱資料都攤縮成年資料，以便後續1:1併檔，並在名稱前冠以「乾淨」，以示區別。

* 併檔

透過Ticker、fyear把之前的dta全部都用1:1併起來存成併檔，並跑縮尾。

* 迴歸

導入reghdfe, ftools
把fund1設為log_at。(目前fundemental非完整故只能先以部分有的代替之)
如同論文要求的把DISP和FE各自*100以方便閱讀。
為所有變數重新標籤，
用eststo生成各項表達欄位，包括DISP及FE有fund無fund四欄。
並加入表格下方詳細資訊，是否、有無、顯著程度等標記。
最後即可輸出表格到word



  <h2>References</h2>
J of Accounting Research - 2015 - CHEN - A New Measure of Disclosure
Quality: The Level of
Disaggregation of Accounting Data
in Annual Reports

# -

