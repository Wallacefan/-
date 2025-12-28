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
<h3>Table 4 </h3>
首先把完整主檔.dta打開，內含之前計算的DQ、Compustat資料、股價資料及各種所需變數，並與 EPS真.dta 併檔。
併檔後針對actual_EPS 進行平減（按照股價）
導入rangestat進行EPS之標準差計算。
並存入EPS.波動.dta待命。

<h4>建構自變數：</h4>

打開完整主檔.dta，計算Sale 的成長率，並立刻進行winsorize，存入
GROWTH。

打開完整主檔，計算ROA，winsorize後存入ROA。

打開LBES_DISP，將每月資料平均至年資料後，計算log(AF)存入AF。

打開完整主檔，計算log(AT)，存入AT。

<h4>清理資料：</h4>


打開已整理好的DISP_FE、完整主檔、EPS波動、GROWTH、ROA、AF、AT.dta的變數刪到只剩他們各自的代表性變數，以求後續整理更清楚方便，且把所有變數的閱資料都攤縮成年資料，以便後續1:1併檔，並在名稱前冠以「乾淨」，以示區別。

<h4>併檔：</h4>

透過Ticker、fyear把之前的dta全部都用1:1併起來存成併檔，並跑縮尾。

<h4>迴歸</h4>

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

