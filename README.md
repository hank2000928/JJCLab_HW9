# JJCLab_HW9
參考筆記: https://github.com/wsunccake/sle15_notes/blob/master/practice/ch3.md
-----------------
# 練習 1. Process：ps、pstree、pgrep
## 1-1. ps 選項與欄位
### 1-1-1	ps aux 中 a、u、x 各代表什麼？	

選項|名稱／功能|詳細說明
---|---|---
a	|All users with terminal	|取消只顯示目前使用者行程的限制，包含其他使用者具有控制終端機的行程。
u	|User-oriented format	|使用以使用者為主的詳細格式，顯示 USER、%CPU、%MEM、VSZ、RSS 等欄位。
x	|Include processes without TTY	|取消只顯示具有控制終端機行程的限制，使背景服務、系統 daemon 等也能顯示。

### 1-1-2	ps -ef 與 ps aux 風格差異（UNIX／BSD）簡述	

比較項目|	ps -ef|	ps aux
---|---|---
風格	|UNIX / System V	|BSD
選項格式	|使用 -	|不使用 -
顯示範圍	|所有行程|	所有行程
父行程 PPID	|預設顯示|	預設不顯示
CPU 使用率	|C 欄位，並非 %CPU|	%CPU
記憶體使用率|	預設不顯示|	%MEM
虛擬記憶體 |VSZ	預設不顯示|	顯示
常駐記憶體 |RSS	預設不顯示	|顯示
行程狀態 |STAT	預設不顯示|	顯示

小題	|問題	|作答
---|---|---
1-1-3	|VSZ 與 RSS 的意義差異	|
1-1-4	|TTY 顯示 ? 通常代表什麼？	|
1-1-5	|STAT 中 R、S、D、Z、T 各代表什麼狀態？	|
1-1-6	|%CPU 高是否一定是問題？應如何判斷？	|
1-1-7	|發現 STAT=Z（zombie）時，下一步應查什麼？	|
