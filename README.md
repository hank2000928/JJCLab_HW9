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
### 1-2-1 從 ps aux 找出 sshd 相關列，記錄 USER、PID、STAT、COMMAND
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

### 1-2-2 用 pgrep -l sshd 對照 PID 是否一致
<img width="851" height="186" alt="image" src="https://github.com/user-attachments/assets/756dce51-580b-4298-872d-c9a1f1172cc5" />
顯示結果一致

### 1-2-3 說明何時用 ps（快照／過濾），何時改用 top／pidstat（持續觀察）
工具|	觀察方式|	適用情況
---|---|---
ps	|單次快照	|查詢目前有哪些行程、PID、PPID、執行狀態及資源使用資訊。
top	|即時互動監控	|持續觀察系統 CPU、記憶體使用量，以及目前最消耗資源的行程。
pidstat	|定時採樣與統計|	追蹤特定 PID 的 CPU、記憶體或 I/O 使用情況，適合定量分析或儲存紀錄。

ps 適合快速取得某個時間點的行程資訊與查詢特定 PID；top 適合即時、互動式地觀察整體系統與高負載行程；pidstat 則適合對特定行程進行定時採樣，分析 CPU、記憶體或 I/O 使用率隨時間的變化。  

## 1-3. pstree 看父子關係
### 1-3-1 -p、-u、-a、-h 各做什麼？

選項	|功能|	詳細說明
-p|	Show PIDs	|在行程名稱後方顯示 PID
-u|	Show UID transitions|	當父子行程的使用者身分不同時，標示使用者變化
-a|	Show command-line arguments	|顯示行程啟動時的命令列參數
-h|	Highlight|	標示目前行程及其祖先行程（視終端機是否支援反白）

### 1-3-2 從輸出指出 sshd（或任一 daemon）的父行程與子行程

<img width="849" height="206" alt="image" src="https://github.com/user-attachments/assets/2b6a355b-5e55-4fdb-9610-f3e36a176510" />
用 ps 與 pstree 兩種方式交叉驗證父子關係，並且目前沒有 SSH 用戶端透過網路登入。

### 1-3-3 為什麼排查時要先搞清楚「該殺父還是殺子」？
終止父行程不一定自動終止所有子行程。實際情況取決於應用程式的訊號處理、行程群組及服務管理機制。
因此排查時應先：
1. 使用 ps、pgrep 找出異常行程的 PID。
2. 使用 pstree 確認父子行程關係。
3. 判斷各行程的功能與依賴關係，再決定是否需要終止或重新啟動。
因此，排查行程異常時，應先確認父子關係，避免誤將負責管理多個子行程的父行程終止，導致整個服務或其他正常工作受到影響。應針對真正異常的行程採取適當措施，並考慮父行程是否會自動重新啟動子行程，以及終止後的資源清理和服務依賴問題。

#練習 2. 訊號、優先級與工作控制
## 2-1. kill / killall / pkill
小題	|問題|	作答
---|---|---
2-1-1	|kill PID 預設送的訊號是什麼？（名稱與編號）	|SIGTERM（Signal Terminate），編號 15
2-1-2	|kill -9 對應哪個訊號？為什麼行程無法捕捉？	|SIGKILL（Signal Kill），編號 9
2-1-3	|為什麼應先試 TERM，把 SIGKILL 當最後手段？	|避免破壞資料與應用程式狀態
2-1-4	|killall 與 pkill -f 的匹配對象有何不同？	|兩者都可以一次對多個行程送出訊號，但匹配方法不同
2-1-5	|寫出查看訊號清單的指令	|執行: kill -l

### 2-1-2
與 SIGTERM 不同，SIGKILL 由 Linux Kernel 處理，行程無法捕捉、忽略或阻擋這個訊號。
因此程式無法透過一般的訊號處理程式執行正常的關閉流程。
不過 SIGKILL 也不保證立即讓所有狀態的行程消失，例如行程正在不可中斷的核心等待狀態時，終止可能延後。

### 2-1-3
例如，一個 Python 程式可能正在寫入檔案。
如果收到 SIGTERM，程式可能有機會完成寫入並關閉檔案。
但收到 SIGKILL 時，程式無法執行自行定義的結束流程，可能造成尚未完成的資料寫入或其他應用程式層級問題。

### 2-1-4
指令	|匹配方式|	範例
---|---|---
kill PID	|指定 PID|	kill 12345
killall sleep	|依行程名稱匹配	|終止名稱為 sleep 的行程
pkill -f PATTERN	|依完整命令列進行正規表示式匹配|	匹配包含指定參數的行程

## 2-2. 實作：對測試行程送訊號（勿對系統服務亂殺）
### 2-2-1 ~ 2-2-2
<img width="849" height="382" alt="image" src="https://github.com/user-attachments/assets/ebd5e150-6526-40aa-95c2-f61187213130" />
結果:  

檢查項目	|實際結果|	判定
---|---|---
kill -l TERM|	15	|正確
kill -l KILL	|9|	正確
sleep 600 &	|背景工作 [1] 5922|	成功
TEST_PID=$!	|5922	|正確取得 PID
ps	|PID 5922，PPID 2829，STAT S|	正常
pgrep|	5922 sleep 600	|PID 一致
kill -TERM "$TEST_PID"	|顯示 Terminated|	成功
ps -p "$TEST_PID"|	只有表頭，沒有行程	|已結束
wait、jobs	|沒有剩餘背景工作	|完成
