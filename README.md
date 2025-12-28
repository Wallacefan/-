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
|:-----|:--------:|:--------:|:--------:|:------:|
| Figure1  | Figure1.do  |  新replica.dta | Compustat, LBES  | 資料夾"Figure1"內 |
| Table 1 panel A | table1 panelA.do  |  新replica.dta | Compustat | 資料夾"table1_panelA"內 |
| Table 1 panel B | panel b and c.do |  新replica.dta | Compustat | 資料夾"Table 1_Panel B and C"內 |
| Table 1 panel C | panel b and c.do |  新replica.dta；segment.dta；replica_with_NSEG.dta | Compustat | 資料夾"Table 1_Panel B and C"內 |
| Table 2 | Table2.do |  新replica.dta | Compustat, LBES  | 資料夾"Table 2"內 |
| Table 3 | Table3_法一.do |  give.dta；ret.dta | Compustat,CRSP Stock (Annual) | 資料夾"Table3之法一"內 |
| Table 3 | Table3_法二.do |  give.dta | Compustat | 資料夾"Table3之法二"內 |
| Table 4 | JAR_1976-2011_table4.do | 完整主檔.dta ; DISP_FE.dta ; EPS真.dta ; LBES_DISP.dta ; table3大表.dta | Compustat, LBES | 資料夾"Table4"內 |
| Table 7 Panel A | table7 A1.do | 完整主檔.dta；EPS真.dta；LBES_DISP.dta；DISP_FE.dta；新replica.dta | Compustat, LBES  | 資料夾"Table7_PanelA"內 |
| Table 7 Panel B   | table7 Panel B2.do | 完整主檔.dta；EPS真.dta；LBES_DISP.dta；DISP_FE.dta；併檔.dta | Compustat, LBES  | 資料夾"Table 7_Panel B"內 |

<h2>Description of programs/code</h2>

<h3>Figure 1 </h3>

* 區塊0：初始化與Log設定

capture log close; set more off; log using "Figure1.log", replace text

操作細節：關閉舊log → 自動連續執行 → 開啟文字log。

邏輯與目的：建立乾淨環境，完整記錄18組帳戶計算過程。


* 區塊1：讀取Compustat資料

use "新replica.dta", clear; di "Data loaded: N=" _N ", K=" c(k)

操作細節：載入Compustat → 顯示觀測值/變數。

邏輯與目的：驗證原始資料完整性（含18組帳戶）。

* 區塊2：資產負債表品質(DQ_BS)

local bsgroups "act ao ceq dltt intan ivao lct lo ppent pstk txditc" 
foreach g of local bsgroups {
    egen nonmiss_`g' = rownonmiss(``g'')  
    gen dq_`g' = nonmiss_`g' / `total_`g''  
}
egen DQ_BS = rowmean(dq_*)  

操作細節：11組BS帳戶 → 每組非缺漏比例 → 等權重平均。

邏輯與目的：計算資產負債表資料完整度（0-1分數）。

* 區塊3：損益表品質(DQ_IS)

local isgroups "citotal nopi spi txt xido xint xopr"
foreach g of local isgroups { ... 同上計算 ... }
egen DQ_IS = rowmean(dq_*)

操作細節：7組IS帳戶 → 每組非缺漏比例 → 等權重平均。

邏輯與目的：計算損益表資料完整度。

* 區塊4：綜合品質(DQ)與年份

gen DQ = (DQ_BS + DQ_IS)/2 
gen year = fyear 

操作細節：DQ = (DQ_BS + DQ_IS)/2 → 提取年度變數。

邏輯與目的：生成Figure 1 Y軸主變數 + 時間維度。

* 區塊5：年度聚合

collapse (mean) DQ DQ_BS DQ_IS, by(year)

操作細節：公司層面 → 年度平均（刪除個體差異）。

邏輯與目的：生成39年時間序列（1973-2011）。

* 區塊6：繪製Figure 1趨勢圖

twoway (line DQ year, lpattern(solid)) ///
       (line DQ_BS year, lpattern(dash)) ///
       (line DQ_IS year, lpattern(dot)), ///
       xlabel(1973(2)2011) ylabel(0(0.1)1) ///
       legend(order(1 "Overall DQ" 2 "DQ_BS" 3 "DQ_IS")) ///
       title("Temporal Trend of Data Quality")

操作細節：折線圖（實線：綜合DQ | 虛線：DQ_BS | 點線：DQ_IS） 
•	X軸：1973-2011年份（每2年標記）
•	Y軸：0-1品質分數範圍。

邏輯與目的：呈現資料品質指標的長期時間趨勢

* 區塊7：執行完成

di "Figure1 Analysis COMPLETED SUCCESSFULLY!"
di "Graph: fig1_DQ (記憶體中，可graph export)"

操作細節：顯示成功訊息 → 圖形儲存於記憶體。

邏輯與目的：確認重現完成，指引圖形匯出。

* 重現步驟總結

1.	將新replica.dta置於Stata工作目錄

2.	執行Figure1.do

3.	檢查Figure1.log與記憶體圖形fig1_DQ → graph export "Figure1.png", replace


<h3>Table 1 - PanelA </h3>

* Step 1. 讀入資料

讀入 新replica.dta 作為基礎資料集。

* Step 2. 建構 DQ_BS（Balance Sheet；Value-Weighted）

1. 定義 BS 子科目清單（Subaccounts）：以 local act_sub ... 等方式列出各群組子科目。
2. 定義 BS parent 群組（用於權重）：local groups act ao ceq dltt intan ivao lct lo ppent pstk txditc
3. 計算各群組揭露比例：
以 egen rownonmiss() 計算該群組子科目「非缺漏數」
揭露比例 dq_g = nonmissing / total_items_in_group
4. 計算價值權重並標準化：
先算 weight_raw_g = abs(parent_g) / total_assets，再把各群組 raw weight 加總為 total_weight_raw
再標準化 weight_g = weight_raw_g / total_weight_raw（確保權重總和為 1）
5. 加權後加總得到 DQ_BS：
vw_dq_g = dq_g * weight_g
DQ_BS = rowtotal(vw_dq_list)
6. 對 DQ_BS 做 1%/99% winsorize：
winsor2 DQ_BS, cut(1 99) replace

* Step 3. 建構 DQ_IS（Income Statement；Equal-Weighted with Screening）

1. 定義 IS 群組與子科目（共 8 組）：Sales、COGS、Op Expenses、Special Items、Interest、Taxes、Minority Interest、Income Below 等。
2. Parent screening（適用性判斷）：
對每一群組建立 has_is_group_k：當 parent 變數存在且「非缺漏且不等於 0」時，該群組才視為 applicable。
3. 計算分子/分母：
若群組適用（has_is_group_k==1），則該群組的每個子科目都計入分母；子科目若非缺漏則分子 +1。
4. 計算 DQ_IS：
DQ_IS = dq_is_numerator / dq_is_denominator（denominator > 0 才計算）
若 denominator==0 且 numerator==0，程式將 DQ_IS 設為 0（代表沒有任何適用群組）

* Step 4. 計算整體 DQ

DQ = (DQ_BS + DQ_IS) / 2

* Step 5. 樣本限制（Replication Criteria）

1. 補齊年度欄位：
若沒有 year，則優先用 fyear，否則用 year(datadate) 生成。
2. 保留樣本期間：
程式保留 1973–2011：keep if inrange(year, 1973, 2011)
3. 排除產業（依 SIC）：
金融業：6000–6999
公用事業：4900–4999
程式先 destring sic 後再 drop if ...

* Step 6. 產出 Panel A 描述統計並輸出 RTF

1. 用 tabstat 計算 DQ、DQ_BS、DQ_IS 的 mean sd p25 p50 p75，並以 save 將結果存到 r(StatTotal)。
2. 將 r(StatTotal) 存成矩陣 T1，必要時轉置（讓列是變數、欄是統計量）。
3. 設定矩陣列名/欄名（DQ、DQ_BS、DQ_IS；Mean、SD、P25、P50、P75）。
4. 使用 esttab matrix() 輸出為 Word 可開啟的 RTF，並固定到小數點後三位。

<h3>Table 1 - PanelB </h3>



<h3>Table 1 - PanelC </h3>



<h3>Table 2 </h3>

* 讀入資料

use "新replica.dta", clear

* 建構 DQ 指標（與 Table 1 同源計算）

1. 依資產負債表科目分組，建構 DQ_BS（value-weighted）：

對每一群組計算揭露比例（非缺漏子科目數／該群組子科目總數）。
以 parent 科目金額在總資產的占比作為 raw weight，並標準化權重使加總為 1。
將各群組揭露比例乘上權重後加總得到 DQ_BS；並對 DQ_BS 進行 1%/99% winsorize（winsor2）。

2. 依損益表科目分組，建構 DQ_IS（equal-weight + applicable screening）：

先用 parent→child screening 判斷各 group 是否適用（parent=0 或缺漏時，該 group 子科目不納入分母）。
以「適用子科目非缺漏數／適用子科目總數」計算 DQ_IS。

3. 定義整體 DQ：DQ = (DQ_BS + DQ_IS) / 2。

* 樣本限制（Sample restrictions）

1. 年度對齊：若缺 year，依序用 fyear 或 datadate 生成 year。
2. 期間限制：僅保留 1993–2011 年度觀測值（keep if inrange(year, 1993, 2011)）。
3. 產業排除：將 sic 轉為數值（destring）後排除
金融業：SIC 6000–6999
公用事業：SIC 4900–4999

* 計算相關係數矩陣（Pearson）

1. 設定變數清單：local vars DQ DQ_BS DQ_IS。
2. 計算 Pearson 相關：
corr `vars'
取出 r(C) 存為矩陣 R_Pearson。
3. 設定矩陣列名與欄名為 DQ、DQ_BS、DQ_IS，並可用 matrix list 預覽。

* 輸出 Table 2（RTF）

使用 esttab 將 R_Combined 輸出為 RTF：
檔名：Table2.rtf
顯示格式：小數點後三位（fmt(3)），不顯示觀測數與多餘標頭（nonumbers、noobs、nomtitles）。

* 輸出資料（Intermediate output）

Table2.rtf
內容：DQ、DQ_BS、DQ_IS 的相關係數矩陣（Pearson）。


<h3>Table 3 PanelA&B_法一 </h3>

* 步驟0：初始化與log設定

set more off
log using "Table3_法一(no table3).log", replace

操作細節：
•	set more off：關閉Stata預設的「畫面暫停」功能（預設每14行暫停，按Enter繼續）
•	log using ...：開啟執行log記錄檔案，記錄所有命令與結果

邏輯與目的：set more off確保程式自動連續執行不中斷，log檔案完整記錄用於除錯、重現驗證與結果檢查。

*步驟1：載入ret.dta並計算sigma_RET

use "ret.dta", clear
gen year = year(date)
keep TICKER year RET
drop if missing(TICKER) | missing(year) | missing(RET)
bysort TICKER year: gen n_ret = _N
bysort TICKER year: egen sigma_RET = sd(RET)
drop if missing(sigma_RET)
drop n_ret RET
duplicates drop TICKER year, force
save "sigma_by_ticker_year.dta", replace

操作細節：
1.	載入ret.dta，清除記憶體
2.	由date產生year變數
3.	僅保留TICKER、year、RET
4.	移除遺漏值
5.	計算每個TICKER-year的觀測數與RET標準差
6.	移除無法計算sd的觀測值，清理變數，儲存中間檔案

操作步驟：載入ret.dta→提取年份→清理遺漏值→計算標準差→儲存中間檔案。

邏輯與目的：股票報酬波動率(sigma_RET)是核心解釋變數，必須由每月報酬資料依公司年度(TICKER-year)計算標準差。此步驟標準化波動率計算，為後續合併準備唯一匹配鍵值。

* 步驟2：載入give.dta並合併sigma_RET

clear all
use "give.dta", clear
capture confirm var year / fyear / date → gen year
rename tic TICKER
replace TICKER = upper(trim(TICKER))
merge m:1 TICKER year using "sigma_by_ticker_year.dta"
drop if _merge_sigma==2

操作細節：
1.	清空記憶體載入give.dta
2.	自動偵測並標準化年份變數
3.	標準化TICKER（大寫+去空格）
4.	m:1合併並檢查匹配率，移除未匹配觀測值

操作步驟：標準化變數名→m:1合併→檢查匹配率→移除未匹配

邏輯與目的：將公司基本面資料與波動率資料精準連結，TICKER+year為唯一匹配鍵。m:1合併確保每個公司年觀測值對應唯一sigma_RET值，建立完整分析資料集。
步驟3：生成分析變數
foreach v in aqp rcp spi at NSEG DQ sic { confirm var `v' }
gen MA = (aqp != . & aqp != 0)
gen Restructure = (rcp != . & rcp != 0)
gen AT_dollar = abs(at)
gen SI = abs(spi) / AT_dollar
gen AT = AT_dollar/1e9
gen log_AT = ln(AT)
gen log_nseg = ln(NSEG)


操作細節：
1.	檢查8個必要原始變數
2.	產生二元變數MA、Restructure
3.	計算標準化變數SI、AT(億美元)、log變數

操作步驟：檢查原始變數→產生二元變數→標準化計算

邏輯與目的：依論文Table 3定義，將原始財務資料轉換為標準化分析變數。虛擬變數捕捉重組活動存在性，log變數處理偏態分佈，SI標準化結構複雜度，確保變數定義完全一致。

* 步驟4：樣本篩選

keep if inrange(year, 1976, 2011)
sic字串→數字 → sic2 = floor(sic/100)
drop if inlist(sic2, 49,60-69)

操作細節：
1.	時間範圍限制1976-2011
2.	自動處理sic產業代碼，排除金融+公用事業

操作步驟：時間限制→產業代碼處理→排除金融公用事業。

邏輯與目的：符合論文樣本定義，非金融非公用事業為標準篩選，避免特殊產業特性（如高槓桿）干擾重組活動與揭露品質的因果關係。

* 步驟5：Winsorize極值處理

capture ssc install winsor2, replace
foreach v in SI sigma_RET log_AT log_nseg {
    winsor2 `v', replace cuts(1 99)
}

操作細節：自動安裝winsor2，對4個連續變數1%-99%尾部截斷。

操作步驟：自動安裝winsor2→1%-99%尾部截斷。

邏輯與目的：標準計量經濟學做法，減少極端離群值對迴歸係數估計的影響，確保統計推論穩健性。

*　步驟6：統計分析

pwcorr DQ Restructure MA SI sigma_RET log_AT log_nseg, star(0.10)
reghdfe DQ Restructure MA SI sigma_RET log_AT log_nseg, ///
    absorb(i.year i.sic2) vce(cluster year sic2)

操作細節：
•	Panel A：Pearson相關矩陣+顯著性標記
•	Panel B：固定效果迴歸+雙向cluster標準差

操作步驟：Panel A相關矩陣→Panel B固定效果迴歸。

邏輯與目的：Panel A檢驗變數間相關性，Panel B測試重組活動對DQ的主效應，控制公司規模、複雜度、波動率，年產業固定效果+cluster標準誤處理時間序列與跨產業相關性。

* 步驟7：RTF表格輸出

Part 6. RTF輸出：Panel A（相關性表）
tempname fhA
local today = c(current_date)
local corrfile "Table3_PanelA_`today'.rtf"
file open `fhA' using "`corrfile'", write replace
file write `fhA' "{\rtf1\ansi\deff0" _n
file write `fhA' "\b Table 3 Panel A: Correlation matrix (Pearson)\b0\par" _n
... (RTF表格格式化相關矩陣，對角線顯示"-") ...
file close `fhA'

** Part 7. RTF輸出：Panel B（迴歸結果） 
tempname fhB
local fname "Table3_PanelB_`today'.rtf"
file open `fhB' using "`fname'", write replace
file write `fhB' "{\rtf1\ansi\ansicpg1252" _n
... ** 表格必要條件：7行結構**…
1. 變數名稱行
2. 預測符號行  
3. 係數×100行
4. t統計量行 (括號)
5. NOBS行
6. Adjusted R²行
7. 註解行

file close `fhB'
操作細節：手動撰寫RTF原始碼，Panel A相關矩陣(對角線"-")，Panel B七行表格(變數名、預測符號、係數×100、t值、N、Adj R²、註解)。

邏輯與目的：生成符合學術期刊標準的RTF格式迴歸表，直接匯入文書處理軟體。係數乘以100轉換為百分比形式，提升經濟意義的可讀性；完整註解闡述模型規格、樣本篩選標準、穩健性處理程序及資料來源，確保結果完全可重現。

* 重現步驟總結

1.	將ret.dta、give.dta置於Stata工作目錄
2.	執行Table3_法一.do
3.	檢查是否確實輸出panelA&panelB表格與確實出現「Table 3 重現完成！」訊息

<h3>Table 3 PanelA&B_法二 </h3>

* 區塊1：初始化與Log設定

capture log close
set more off
log using "Table3_法二.log", replace text

操作細節：關閉舊log → 關閉畫面暫停 → 開啟文字格式log記錄檔。

邏輯與目的：確保乾淨執行環境，完整記錄重現過程。

* 區塊2：資料載入（僅give.dta）

clear all
use "give.dta", clear
display "載入 give.dta: " _N " 觀測值"
count; if r(N)==0 { display as error "錯誤！"; exit 2000 }
操作細節：清空記憶體 → 載入give.dta → 顯示總觀測值 → 若為0則報錯中止。

邏輯與目的：法二核心特色 - 驗證單一輸入檔案可用性，無需外部ret.dta。

* 區塊3：年度變數生成

capture confirm var fyear
if _rc gen year = year(datadate)

操作細節：檢查fyear是否存在 → 若無則由datadate日期變數提取年份。

邏輯與目的：標準化時間變數格式，確保後續年份固定效果可用。

* 區塊4：分析變數生成（法二核心）

foreach v in aqp rcp spi at NSEG re DQ sic { confirm var `v' }
gen MA = (aqp != . & aqp != 0)
gen Restructure = (rcp != . & rcp != 0)
gen AT_dollar = abs(at); gen SI = abs(spi)/AT_dollar
gen AT = AT_dollar/1e9; gen log_AT = ln(AT)
gen log_nseg = ln(NSEG)
destring gvkey, generate(gvkey_n)
bysort gvkey_n: egen sigma_RET = sd(re)
gen sigma_RET_100 = sigma_RET * 100
操作細節：
1.	檢查9個必要變數（含re關鍵變數）
2.	生成重組dummy變數：MA、Restructure
3.	標準化變數：SI、log_AT、log_nseg
4.	法二核心：gvkey內re標準差→sigma_RET→×100

邏輯與目的：法二最大創新 - 使用give.dta內建re (Retained Earnings)按gvkey公司層級計算波動率。

* 區塊5：樣本篩選

keep if inrange(year, 1976, 2011)
capture confirm var sic
if !_rc {
    destring sic, replace ignore(" ")
    gen sic2 = floor(sic/100)
    drop if inlist(sic2, 49,60,61,62,63,64,65,66,67,68,69)
}
操作細節：保留1976-2011年 → sic字串轉數字 → sic2前2碼 → 排除金融(sic2=60-69)/公用事業(sic2=49)。

邏輯與目的：與法一完全一致樣本定義，確保結果可比性。

* 區塊6：Winsorize極值處理

capture ssc install winsor2, replace
foreach v in SI sigma_RET_100 log_AT log_nseg {
    winsor2 `v', replace cuts(1 99)
}
操作細節：自動安裝winsor2 → 對4連續變數執行1%-99%雙尾截斷。

邏輯與目的：標準計量經濟學穩健性處理，消除離群值影響。

* 區塊7：最終樣本驗證

pwcorr DQ Restructure MA SI sigma_RET_100 log_AT log_nseg, star(0.10)
reghdfe DQ `xvars', absorb(i.year i.sic2) vce(cluster year sic2)

操作細節：Panel A相關矩陣 + Panel B固定效果迴歸。

邏輯與目的：與法一相同統計規格。

* 區塊8：Panel A - Pearson相關矩陣

pwcorr DQ Restructure MA SI sigma_RET_100 log_AT log_nseg, star(0.10)
matrix M = r(C)
操作細節：計算7變數Pearson相關矩陣 → 顯著性星號(p<0.10) → 儲存矩陣M。

邏輯與目的：生成Table 3 Panel A統計基礎。

* 區塊9：儲存中間樣本

local today = c(current_date)
save "give_final_`today'.dta", replace

操作細節：以當日日期命名儲存完整分析樣本。

邏輯與目的：保留清理後資料，方便驗證與重複使用。

* 區塊10：Panel A RTF表格輸出

tempname fhA; local corrfile "Table3_PanelA_`today'.rtf"
file open `fhA' using "`corrfile'", write replace
file write `fhA' "{\rtf1\ansi\deff0" _n
... **使用file write逐行輸出矩陣M → RTF表格** ...
file close `fhA'

操作細節：file open開檔 → file write逐行生成RTF語法 → 矩陣M轉表格（對角線"-"）→ file close。

邏輯與目的：生成論文標準Panel A相關矩陣表格


* 區塊11：Panel B主迴歸分析

reghdfe DQ `xvars', absorb(i.year i.sic2) vce(cluster year sic2)
matrix b_raw = e(b); matrix V = e(V)
matrix b100 = b * 100; matrix t_vals = b/se

操作細節：固定效果迴歸 → 提取係數矩陣 → 計算t統計量 → 係數×100。

邏輯與目的：核心計量模型，年產業雙固定效果 + 雙向cluster標準差。

* 區塊12：Panel B RTF表格輸出

tempname fhB; local fname "Table3_PanelB_`today'.rtf"
file open `fhB' using "`fname'", write replace
... **使用file write生成7行標準結構** ...
file close `fhB'

操作細節：file write → 7行結構（變數名/預測符號/係數×100/t值/N/R²/註解）→ RTF檔案完成。

邏輯與目的：生成論文標準Panel B迴歸表，註解明確法二sigma_RET來源。

重現步驟總結
1.	將give.dta置於Stata工作目錄

2.	執行Table3_法二.do

3.	檢查是否確實輸出Panel A & Panel B表格與確實出現「完美完成！（法二 - 僅 give.dta）」訊息

<h3>Table 3 法一法二比較表格 </h3>

| 項目 |  法一  | 法二 | 
|:-----|:--------:|:--------:|
| 輸入檔案 | ret.dta + give.dta | 僅 give.dta |
| sigma_RET來源  |月報酬RET  | Retained Earnings (re) |
| 匹配層級 | TICKER-year | gvkey-level |
| 程式區塊 | 7區塊 | 12區塊 |
| 中間檔案 | sigma_by_ticker_year.dta | give_final_*.dta | 


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

<h3>Table 7－Panel A </h3>

* 1.產生各項控制變數與中介變數：

關於 EPS 波動，先把 EPS 檔併入主檔，然後對每家公司計算過去四年（fyear −4 到 fyear）的 EPS 調整後值（以股價平減後的 actual_eps/prcc_f）之標準差，並以 rangestat 計算該區間的標準差，最後將其分為十等分（xtile eps_vol_rank, n(10)）作為 EPS 波動等級。關於 GROWTH，以銷售額計算年成長率，並以 rangestat 取過去四年的平均值。ROA 的定義為 ib 除以 at。分析師覆蓋度 AF 的來源是 LBES 的月度資料：先以公司、年度與月份群組計算每月的分析師人數平均（af_month），再以公司年度群組取月均值得到 AF，最後生成 log_AF（目前 DO 未對 AF=0 特別處理）。規模變數以 log_at = log(at) 得到。

* 2.DISP 與 FE 的處理：

讀入 DISP_FE.dta，並以公司與年度（TICKER fyear）群組將 monthly 或 analyst-level 的 DISP 與 FE_abs 取平均（collapse (mean) DISP FE_abs, by(TICKER fyear)），把每個公司在該年度的值匯總成 firm-year 水準。在完成與其他檔案的合併之後，使用對應的期末股價 prcc_f 對 DISP 與 FE_abs 做 price-normalization，再把 DISP 與 FE_abs 乘以 100 生成 DISP_scaled 與 FE_scaled 以便於表格呈現。

* 3.DQ_OP 與 DQ_FIN 的建構：

在新replica.dta中以 local lists 列出Compustat mnemonics，分別歸於 op_assets、op_liabs、op_inc、fin_debt、fin_eq、fin_inc 等群組。對於 lists 中實際存在的變數，我們為每一個變數生成一個非缺失指標 nm_<var>，表示該子科目在該公司是否有揭露；並實作 parent→child 的 screening，當父項為 0 或遺漏時，將該父項下的子科目對應的 nm_ 設為 missing，確保只有 applicable 的子科目被計入分母。相同地，對 ppent、intan、ao、txt、xopr 等父項執行相同邏輯；對於融資面也針對 dltt、dlc、pstk、ceq、xint、citotal 等父項對相應子項做相同處理。最後，以等權的方式對所有 applicable 的 nm_ 子項取平均，分別用 egen DQ_OP = rowmean(...) 與 egen DQ_FIN = rowmean(...) 得到每家公司年度的營運揭露分數與融資揭露分數，接著以公司年度群組 collapse 成 firm-year 平均並存檔。

* 4.合併樣本：

將多個已清理的子檔（如 DISP_FE、EPS 波動、GROWTH、ROA、AF、AT、DQ 等）以公司與年度（TICKER fyear）進行一對一合併（merge 1:1 TICKER fyear），合併後的最終檔案命名為 併檔.dta。合併完成之後，對被解釋變數與控制變數再次進行尺度處理與清理，包括以 prcc_f 做 price-normalization（如前述），以及針對 DISP、FE_abs、ROA、GROWTH 進行 1%/99% 的 winsorize（winsor2 DISP FE_abs ROA GROWTH, cut(1 99)）。

* 5.迴歸估計：

在最終整理完成的 firm-year 資料上，使用 reghdfe 執行 Table 7 Panel A1 的四個主要迴歸規格。第一列（Column 1），以 DISP_scaled 為被解釋變數，並僅放入基本控制變數（Base Controls）；第二列（Column 2），以 DISP_scaled 為被解釋變數，並加入基本控制變數與公司基本面控制（Fundamentals）；第三列（Column 3），以 FE_scaled（|FE|）為被解釋變數且僅放基本控制；第四列（Column 4），以 FE_scaled 為被解釋變數並加入基本控制與公司基本面控制。所有迴歸皆吸收產業固定效果（ff12）與年度固定效果（fyear），標準誤以產業（ff12）與年度（fyear）進行群集校正（vce(cluster ff12 fyear)）。最終使用 esttab 將四欄回歸結果輸出為 Table7_PanelA1.rtf，並在表中只呈現 DQ_OP 與 DQ_FIN 的係數與標準誤，以便直接比較二者對 DISP 與 |FE| 的影響。

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

