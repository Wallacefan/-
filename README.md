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
<h3>Table 1 - PanelA </h3>

* Step 1. 讀入資料
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

<h3>Table 2 </h3>

* 讀入資料use "新replica.dta", clear
*  建構 DQ 指標（與 Table 1 同源計算）
依資產負債表科目分組，建構 DQ_BS（value-weighted）：
對每一群組計算揭露比例（非缺漏子科目數／該群組子科目總數）。
以 parent 科目金額在總資產的占比作為 raw weight，並標準化權重使加總為 1。
將各群組揭露比例乘上權重後加總得到 DQ_BS；並對 DQ_BS 進行 1%/99% winsorize（winsor2）。
依損益表科目分組，建構 DQ_IS（equal-weight + applicable screening）：
先用 parent→child screening 判斷各 group 是否適用（parent=0 或缺漏時，該 group 子科目不納入分母）。
以「適用子科目非缺漏數／適用子科目總數」計算 DQ_IS。
定義整體 DQ：DQ = (DQ_BS + DQ_IS) / 2。
*  樣本限制（Sample restrictions）
年度對齊：若缺 year，依序用 fyear 或 datadate 生成 year。
期間限制：僅保留 1993–2011 年度觀測值（keep if inrange(year, 1993, 2011)）。
產業排除：將 sic 轉為數值（destring）後排除
金融業：SIC 6000–6999
公用事業：SIC 4900–4999
*  計算相關係數矩陣（Pearson）
設定變數清單：local vars DQ DQ_BS DQ_IS。
計算 Pearson 相關：
corr `vars'
取出 r(C) 存為矩陣 R_Pearson。
設定矩陣列名與欄名為 DQ、DQ_BS、DQ_IS，並可用 matrix list 預覽。
*　輸出 Table 2（RTF）
使用 esttab 將 R_Combined 輸出為 RTF：
檔名：Table2.rtf
顯示格式：小數點後三位（fmt(3)），不顯示觀測數與多餘標頭（nonumbers、noobs、nomtitles）。
*  輸出資料（Intermediate output）
Table2.rtf
內容：DQ、DQ_BS、DQ_IS 的相關係數矩陣（Pearson）。



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

<h3>Table 7－Panel B </h3>

* A. 建立中介資料集（modules）

* A1. EPS 波動（eps_sd, eps_vol_rank）
Inputs
完整主檔.dta
EPS真.dta
* Steps
合併：merge m:m TICKER fyear using EPS真.dta
平減 EPS：eps_adj = actual_eps / prcc_f（若 prcc_f<=0 則設為 missing）
以 rangestat 計算過去 5 年窗（t-4 到 t）的波動：
rangestat (sd) eps_sd = eps_adj, interval(fyear -4 0) by(gvkey)
以十分位分組：xtile eps_vol_rank = eps_sd, n(10)
* Output
EPS波動.dta
A2. 成長（GROWTH）
* Input
完整主檔.dta
* Steps
sales_growth = (sale - sale[_n-1]) / sale[_n-1]
rangestat (mean) GROWTH = sales_growth, interval(fyear -4 0) by(gvkey)
winsorize：winsor2 GROWTH, cut(1 99) replace
* Output
GROWTH.dta
* A3. ROA
* Input
完整主檔.dta
* Steps
ROA = ib / at
winsorize：winsor2 ROA, cut(1 99) replace
* Output
ROA.dta
* A4. 分析師追蹤（log(AF)）
* Input
LBES_DISP.dta
* Steps
由 STATPERS 產生 fyear
先取月平均 NUMEST，再取年平均得到 AF
log_AF = log(AF)
* Output
AF.dta
* A5. 公司規模（log(AT)）
* Input
完整主檔.dta
* Steps
log_at = log(at)
* Output
AT.dta
* A6. 依變數清理（DISP、|FE|；先不平減）
* Input
DISP_FE.dta
* Steps
保留 TICKER fyear DISP FE_abs
collapse (mean) DISP FE_abs, by(TICKER fyear)
* Output
乾淨DISP_FE.dta
* A7. DQ 清理（含 DQ_BS、DQ_IS、ff12、prcc_f）
* Input
完整主檔.dta
* Steps
保留：TICKER fyear DQ DQ_BS DQ_IS ff12 prcc_f
轉數值（如為字串）：destring ... , replace force
移除關鍵欄位全缺漏觀測值
collapse (mean) DQ DQ_BS DQ_IS ff12 prcc_f, by(TICKER fyear)
* Output
乾淨DQ.dta
* A8–A12. 其餘模組清理（EPS/GROWTH/ROA/AF/AT）
* Inputs
EPS波動.dta, GROWTH.dta, ROA.dta, AF.dta, AT.dta
* Steps
各自保留必要欄位並 collapse (mean) 至 firm-year
* Outputs
乾淨EPS.dta, 乾淨GROWTH.dta, 乾淨ROA.dta, 乾淨AF.dta, 乾淨AT.dta
* B. 1:1 合併為最終分析檔，並於最後進行平減（deflation）
Base dataset
乾淨DISP_FE.dta
* Merge order (all by TICKER fyear)
依序合併：
乾淨EPS.dta
乾淨GROWTH.dta
乾淨ROA.dta
乾淨AF.dta
乾淨AT.dta
乾淨DQ.dta
* Deflation step（關鍵）
合併完成後，以 prcc_f 平減：
DISP = DISP / prcc_f（若 prcc_f>0）
FE_abs = FE_abs / prcc_f（若 prcc_f>0）
若 prcc_f<=0，則將 DISP 與 FE_abs 設為 missing
* Output
併檔.dta（Table 7 Panel B 的最終 firm-year 分析檔）
* C. Panel B1（A route）固定效果迴歸與輸出
* Input
併檔.dta
* Sample restriction
僅保留回歸所需且不缺漏的觀測值：
DISP, FE_abs, DQ_BS, DQ_IS, eps_vol_rank, log_AF, log_at, GROWTH, ROA, ff12, fyear
* Winsorization
winsor2 DISP FE_abs ROA GROWTH, cut(1 99) replace
* Dependent variable scaling
DISP_scaled = DISP * 100
FE_scaled = FE_abs * 100
* Regression specification
固定效果：absorb(ff12 fyear)
控制變數：
Base controls：eps_vol_rank log_AF log_at
Fundamentals：GROWTH ROA
四個模型（以 eststo 儲存）：
DISP（不含 fundamentals）
DISP（含 fundamentals）
|FE|（不含 fundamentals）
|FE|（含 fundamentals）
* Output table
使用 esttab 輸出：
檔名： Table7_PanelB1_Aroute.rtf
顯示：係數三位小數、括號 t 值兩位小數、顯著性星號
僅保留：DQ_BS、DQ_IS





  <h2>References</h2>
J of Accounting Research - 2015 - CHEN - A New Measure of Disclosure
Quality: The Level of
Disaggregation of Accounting Data
in Annual Reports

# -

