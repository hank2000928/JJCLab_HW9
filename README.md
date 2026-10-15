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

### 1-1-3	VSZ 與 RSS 的意義差異

項目	|VSZ	|RSS
---|---|---
全名|	Virtual Memory Size|	Resident Set Size
中文	|虛擬記憶體大小	|常駐實體記憶體大小
代表意義	|行程使用的虛擬位址空間總量|	行程目前駐留於實體 RAM 的記憶體
是否全數占用 RAM|	不一定|	是，但可能含共享頁面
常見單位|	KiB|	KiB

### 1-1-4	TTY 顯示 ? 通常代表什麼?

TTY	|意義|
---|---
pts/0	|虛擬終端機，例如 SSH 或 Terminal
tty1	|本地文字終端機
?	|沒有控制終端機

### 1-1-5	STAT 中 R、S、D、Z、T 各代表什麼狀態？	

狀態|	英文|	詳細意義
---|---|---
R	|Running / Runnable	|行程正在 CPU 上執行，或已準備好執行、等待 CPU 排程。
S	|Interruptible Sleep	|行程處於可中斷睡眠，通常正在等待事件，例如使用者輸入、網路封包或其他條件。
D	|Uninterruptible Sleep	|不可中斷的等待狀態，常見於等待磁碟、網路檔案系統等 I/O 操作完成。
Z	|Zombie	|行程已結束，但父行程尚未讀取並回收其結束狀態，因此仍保留行程表項目。
T	|Stopped / Traced	|行程已暫停，可能由工作控制訊號或除錯追蹤造成。

### 1-1-6 ~ 1-1-7
小題	|問題	|作答
---|---|---
1-1-6	|%CPU 高是否一定是問題？應如何判斷？	|CPU 使用率應配合行程用途、執行時間及系統整體狀態判斷。
1-1-7	|發現 STAT=Z（zombie）時，下一步應查什麼？	|此時應優先查詢 PPID（Parent Process ID），確認負責回收該 Zombie 的父行程。

## 1-2. 實作：找出 sshd 相關行程
### 1-2-1
於虛擬機執行:  
ps aux  
ps -ef  
pgrep sshd  
pgrep -l sshd  
pgrep -u root sshd  
整理後，得到:

欄位|	實際結果|	說明
---|---|---
USER	|root	SSH |服務由 root 帳號執行。
PID	|1847	sshd| 行程的唯一識別碼。
STAT	|Ss|	行程處於可中斷睡眠，而且是 Session Leader。
COMMAND|	sshd: /usr/sbin/sshd -D [listener] ...	|SSH 伺服器正在監聽新的 SSH 連線。

### 1-2-2
<img width="851" height="186" alt="image" src="https://github.com/user-attachments/assets/756dce51-580b-4298-872d-c9a1f1172cc5" />
顯示結果一致

### 1-2-3
工具|	觀察方式|	適用情況
---|---|---
ps	|單次快照	|查詢目前有哪些行程、PID、PPID、執行狀態及資源使用資訊。
top	|即時互動監控	|持續觀察系統 CPU、記憶體使用量，以及目前最消耗資源的行程。
pidstat	|定時採樣與統計|	追蹤特定 PID 的 CPU、記憶體或 I/O 使用情況，適合定量分析或儲存紀錄。

ps 適合快速取得某個時間點的行程資訊與查詢特定 PID；top 適合即時、互動式地觀察整體系統與高負載行程；pidstat 則適合對特定行程進行定時採樣，分析 CPU、記憶體或 I/O 使用率隨時間的變化。  
