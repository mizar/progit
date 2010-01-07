# Git 基礎 #

讀完本章你就能上手使用 Git 了。本章將介紹幾個最基本的，也是最常用的 Git 命令，以後絕大多數時間裡用到的也就是這幾個命令。讀完本章，你就能初始化一個新的代碼倉庫，做一些適當的配置；開始或停止跟蹤某些文件；暫存或提交某些更新。我們還會展示如何讓 Git 忽略某些文件，或是名稱符合特定模式的文件；如何既快且容易地撤消犯下的小錯誤；如何瀏覽項目的更新歷史，查看某兩次更新之間的差異；以及如何從遠程倉庫拉數據下來或者推數據上去。

## 取得項目的 Git 倉庫 ##

有兩種取得 Git 項目倉庫的方法。第一種是在現存的目錄下，通過導入所有文件來創建新的 Git 倉庫。第二種是從已有的 Git 倉庫克隆出一個新的鏡像倉庫來。

### 從當前目錄初始化 ###

要對現有的某個項目開始用 Git 管理，只需到此項目所在的目錄，執行：

	$ git init

初始化後，在當前目錄下會出現一個名為 .git 的目錄，所有 Git 需要的數據和資源都存放在這個目錄中。不過目前，僅僅是按照既有的結構框架初始化好了裡邊所有的文件和目錄，但我們還沒有開始跟蹤管理項目中的任何一個文件。（在第九章我們會詳細說明剛才創建的 `.git` 目錄中究竟有哪些文件，以及都起些什麼作用。）

如果當前目錄下有幾個文件想要納入版本控制，需要先用 git add 命令告訴 Git 開始對這些文件進行跟蹤，然後提交：

	$ git add *.c
	$ git add README
	$ git commit -m 'initial project version'

稍後我們再逐一解釋每條命令的意思。不過現在，你已經得到了一個實際維護著若干文件的 Git 倉庫。

### 從現有倉庫克隆 ###

如果想對某個開源項目出一份力，可以先把該項目的 Git 倉庫複製一份出來，這就需要用到 git clone 命令。如果你熟悉其他的 VCS 比如 Subversion，你可能已經注意到這裡使用的是 clone 而不是 checkout。這是個非常重要的差別，Git 收取的是項目歷史的所有數據（每一個文件的每一個版本），服務器上有的數據克隆之後本地也都有了。實際上，即便服務器的磁盤發生故障，用任何一個克隆出來的客戶端都可以重建服務器上的倉庫，回到當初克隆時的狀態（可能會丟失某些服務器端的掛鉤設置，但所有版本的數據仍舊還在，有關細節請參考第四章）。

克隆倉庫的命令格式為 `git clone [url]`。比如，要克隆 Ruby 語言的 Git 代碼倉庫 Grit，可以用下面的命令：

	$ git clone git://github.com/schacon/grit.git

這會在當前目錄下創建一個名為 「grit」 的目錄，其中內含一個 `.git` 的目錄，並從同步後的倉庫中拉出所有的數據，取出最新版本的文件拷貝。如果進入這個新建的 `grit` 目錄，你會看到項目中的所有文件已經在裡邊了，準備好後續的開發和使用。如果希望在克隆的時候，自己定義要新建的項目目錄名稱，可以在上面的命令最後指定：

	$ git clone git://github.com/schacon/grit.git mygrit

唯一的差別就是，現在新建的目錄成了 mygrit，其他的都和上邊的一樣。

Git 支持許多數據傳輸協議。之前的例子使用的是 `git://` 協議，不過你也可以用 `http(s)://` 或者 `user@server:/path.git` 表示的 SSH 傳輸協議。我們會在第四章詳細介紹所有這些協議在服務器端該如何配置使用，以及各種方式之間的利弊。

## 記錄每次更新到倉庫 ##

現在我們手上已經有了一個真實項目的 Git 倉庫，並從這個倉庫中取出了所有文件的工作拷貝。接下來，對這些文件作些修改，在完成了一個階段的目標之後，提交本次更新到倉庫。

請記住，工作目錄下面的所有文件都不外乎這兩種狀態：已跟蹤或未跟蹤。已跟蹤的文件是指本來就被納入版本控制管理的文件，在上次快照中有它們的記錄，工作一段時間後，它們的狀態可能是未更新，已修改或者已放入暫存區。而所有其他文件都屬於未跟蹤文件。它們既沒有上次更新時的快照，也不在當前的暫存區域。初次克隆某個倉庫時，工作目錄中的所有文件都屬於已跟蹤文件，且狀態為未修改。

在編輯過某些文件之後，Git 將這些文件標為已修改。我們逐步把這些修改過的文件放到暫存區域，然後等最後一次性提交暫存區域的所有文件更新，如此重複。所以使用 Git 時的文件狀態變化週期如圖 2-1 所示。

Insert 18333fig0201.png 
圖 2-1. 文件的狀態變化週期

### 檢查當前文件狀態 ###

要確定哪些文件當前處於什麼狀態，可以用 git status 命令。如果在克隆倉庫之後立即執行此命令，會看到類似這樣的輸出：

	$ git status
	# On branch master
	nothing to commit (working directory clean)

這說明你現在的工作目錄相當乾淨。換句話說，當前沒有任何跟蹤著的文件，也沒有任何文件在上次提交後更改過。此外，上面的信息還表明，當前目錄下沒有出現任何處於未跟蹤的新文件，否則 Git 會在這裡列出來。最後，該命令還顯示了當前所在的分支是 master，這是默認的分支名稱，實際是可以修改的，現在不必多慮。下一章我們就會詳細討論分支和引用。

現在讓我們用 vim 編輯一個新文件 README，保存退出後運行 `git status` 會看到該文件出現在未跟蹤文件列表中：

	$ vim README
	$ git status
	# On branch master
	# Untracked files:
	#   (use "git add <file>..." to include in what will be committed)
	#
	#	README
	nothing added to commit but untracked files present (use "git add" to track)

就是在「Untracked files」這行下面。Git 不會自動將之納入跟蹤範圍，除非你明明白白地告訴它這麼做，因而不用擔心把臨時文件什麼的也歸入版本管理。不過現在我們確實想要跟蹤管理 README 這個文件。

### 跟蹤新文件 ###

使用命令 `git add` 開始跟蹤一個新文件。所以，要跟蹤 README 文件，運行：

	$ git add README

此時再運行 `git status` 命令，會看到 README 文件已被跟蹤，並處於暫存狀態：

	$ git status
	# On branch master
	# Changes to be committed:
	#   (use "git reset HEAD <file>..." to unstage)
	#
	#	new file:   README
	#

只要在 「Changes to be committed」 這行下面的，就說明是已暫存狀態。如果此時提交，那麼該文件此時此刻的版本將被留存在歷史記錄中。你可能會想起之前我們使用 `git init` 後就運行了 `git add` 命令，開始跟蹤當前目錄下的文件。git add 後可以接要跟蹤的文件或目錄的路徑。如果是目錄的話，就說明要遞歸跟蹤所有該目錄下的文件。

### 暫存已修改文件 ###

現在我們修改下之前已跟蹤過的文件 `benchmarks.rb`，然後再次運行 `status` 命令，會看到這樣的狀態報告：

	$ git status
	# On branch master
	# Changes to be committed:
	#   (use "git reset HEAD <file>..." to unstage)
	#
	#	new file:   README
	#
	# Changed but not updated:
	#   (use "git add <file>..." to update what will be committed)
	#
	#	modified:   benchmarks.rb
	#

文件 benchmarks.rb 出現在 「Changed but not updated」 這行下面，說明已跟蹤文件的內容發生了變化，但還沒有放到暫存區。要暫存這次更新，需要運行 `git add` 命令（這是個多功能命令，根據目標文件的狀態不同，此命令的效果也不同：可以用它開始跟蹤新文件，或者把已跟蹤的文件放到暫存區，還能用於合併時把有衝突的文件標記為已解決狀態等）。現在讓我們運行 `git add` 將 benchmarks.rb 放到暫存區，然後再看看 `git status` 的輸出：

	$ git add benchmarks.rb
	$ git status
	# On branch master
	# Changes to be committed:
	#   (use "git reset HEAD <file>..." to unstage)
	#
	#	new file:   README
	#	modified:   benchmarks.rb
	#

現在兩個文件都已暫存，下次提交時就會一併記錄到倉庫。假設此時，你想要在 benchmarks.rb 裡再加條註釋，重新編輯存盤後，準備好提交。不過且慢，再運行 `git status` 看看：

	$ vim benchmarks.rb 
	$ git status
	# On branch master
	# Changes to be committed:
	#   (use "git reset HEAD <file>..." to unstage)
	#
	#	new file:   README
	#	modified:   benchmarks.rb
	#
	# Changed but not updated:
	#   (use "git add <file>..." to update what will be committed)
	#
	#	modified:   benchmarks.rb
	#

見鬼！benchmarks.rb 文件出現了兩次！一次算未暫存，一次算已暫存，這怎麼可能呢？好吧，實際上 Git 只不過暫存了你運行 git add 命令時的版本，如果現在提交，那麼提交的是添加註釋前的版本，而非當前工作目錄中的版本。所以，運行了 `git add` 之後又作了修訂的文件，需要重新運行 `git add` 把最新版本重新暫存起來：

	$ git add benchmarks.rb
	$ git status
	# On branch master
	# Changes to be committed:
	#   (use "git reset HEAD <file>..." to unstage)
	#
	#	new file:   README
	#	modified:   benchmarks.rb
	#

### 忽略某些文件 ###

一般我們總會有些文件無需納入 Git 的管理，也不希望它們總出現在未跟蹤文件列表。通常都是些自動生成的文件，像是日誌或者編譯過程中創建的等等。我們可以創建一個名為 .gitignore 的文件，列出要忽略的文件模式，來看一個簡單的例子：

	$ cat .gitignore
	*.[oa]
	*~

第一行告訴 Git 忽略所有以 .o 或 .a 結尾的文件。一般這類對象文件和存檔文件都是編譯過程中出現的，我們用不著跟蹤它們的版本。第二行告訴 Git 忽略所有以波浪符（`~`）結尾的文件，許多文本編輯軟件（比如 Emacs）都用這樣的文件名保存副本。此外，你可能還需要忽略 log，tmp 或者 pid 目錄，以及自動生成的文檔等等。要養成一開始就設置好 .gitignore 文件的習慣，以免將來誤提交這類無用的文件。

文件 .gitignore 的格式規範如下：

* 所有空行或者以註釋符號 ＃ 開頭的行都會被 Git 忽略。
* 可以使用標準的 glob 模式匹配。
* 匹配模式最後跟反斜槓（`/`）說明要忽略的是目錄。
* 要忽略指定模式以外的文件或目錄，可以在模式前加上驚嘆號（`!`）取反。

所謂的 glob 模式是指 shell 所使用的簡化了的正則表達式。星號（`*`）匹配零個或多個任意字符；`[abc]` 匹配任何一個列在方括號中的字符（這個例子要麼匹配一個 a，要麼匹配一個 b，要麼匹配一個 c）；問號（`?`）只匹配一個任意字符；如果在方括號中使用短劃線分隔兩個字符，表示所有在這兩個字符範圍內的都可以匹配（比如 `[0-9]` 表示匹配所有 0 到 9 的數字）。

我們再看一個 .gitignore 文件的例子：

	# 此為註釋 – 將被 Git 忽略
	*.a       # 忽略所有 .a 結尾的文件
	!lib.a    # 但 lib.a 除外
	/TODO     # 僅僅忽略項目根目錄下的 TODO 文件，不包括 subdir/TODO
	build/    # 忽略 build/ 目錄下的所有文件
	doc/*.txt # 會忽略 doc/notes.txt 但不包括 doc/server/arch.txt

### 查看已暫存和未暫存的更新 ###

實際上 `git status` 的顯示比較簡單，僅僅是列出了修改過的文件，如果要查看具體修改了什麼地方，可以用 `git diff` 命令。稍後我們會詳細介紹 `git diff`，不過現在，它已經能回答我們的兩個問題了：當前作的哪些更新還沒有暫存？有哪些更新已經暫存起來準備好了下次提交？ `git diff` 會使用文件補丁的格式顯示具體添加和刪除的行。

假如再次修改 README 文件後暫存，然後編輯 benchmarks.rb 文件後先別暫存，運行 `status` 命令，會看到：

	$ git status
	# On branch master
	# Changes to be committed:
	#   (use "git reset HEAD <file>..." to unstage)
	#
	#	new file:   README
	#
	# Changed but not updated:
	#   (use "git add <file>..." to update what will be committed)
	#
	#	modified:   benchmarks.rb
	#

要查看尚未暫存的文件更新了哪些部分，不加參數直接輸入 `git diff`：

	$ git diff
	diff --git a/benchmarks.rb b/benchmarks.rb
	index 3cb747f..da65585 100644
	--- a/benchmarks.rb
	+++ b/benchmarks.rb
	@@ -36,6 +36,10 @@ def main
	           @commit.parents[0].parents[0].parents[0]
	         end

	+        run_code(x, 'commits 1') do
	+          git.commits.size
	+        end
	+
	         run_code(x, 'commits 2') do
	           log = git.commits('master', 15)
	           log.size

此命令比較的是工作目錄中當前文件和暫存區域快照之間的差異，也就是修改之後還沒有暫存起來的變化內容。

若要看已經暫存起來的文件和上次提交時的快照之間的差異，可以用 `git diff --cached` 命令。（Git 1.6.1 及更高版本還允許使用 `git diff --staged`，效果是相同的，但更好記些。）來看看實際的效果：

	$ git diff --cached
	diff --git a/README b/README
	new file mode 100644
	index 0000000..03902a1
	--- /dev/null
	+++ b/README2
	@@ -0,0 +1,5 @@
	+grit
	+ by Tom Preston-Werner, Chris Wanstrath
	+ http://github.com/mojombo/grit
	+
	+Grit is a Ruby library for extracting information from a Git repository

請注意，單單 `git diff` 不過是顯示還沒有暫存起來的改動，而不是這次工作和上次提交之間的差異。所以有時候你一下子暫存了所有更新過的文件後，運行 `git diff` 後卻什麼也沒有，就是這個原因。

像之前說的，暫存 benchmarks.rb 後再編輯，運行 `git status` 會看到暫存前後的兩個版本：

	$ git add benchmarks.rb
	$ echo '# test line' >> benchmarks.rb
	$ git status
	# On branch master
	#
	# Changes to be committed:
	#
	#	modified:   benchmarks.rb
	#
	# Changed but not updated:
	#
	#	modified:   benchmarks.rb
	#

現在運行 `git diff` 看暫存前後的變化：

	$ git diff 
	diff --git a/benchmarks.rb b/benchmarks.rb
	index e445e28..86b2f7c 100644
	--- a/benchmarks.rb
	+++ b/benchmarks.rb
	@@ -127,3 +127,4 @@ end
	 main()

	 ##pp Grit::GitRuby.cache_client.stats 
	+# test line
	and git diff --cached to see what you've staged so far:
	$ git diff --cached
	diff --git a/benchmarks.rb b/benchmarks.rb
	index 3cb747f..e445e28 100644
	--- a/benchmarks.rb
	+++ b/benchmarks.rb
	@@ -36,6 +36,10 @@ def main
	          @commit.parents[0].parents[0].parents[0]
	        end

	+        run_code(x, 'commits 1') do
	+          git.commits.size
	+        end
	+              
	        run_code(x, 'commits 2') do
	          log = git.commits('master', 15)
	          log.size

### 提交更新 ###

現在的暫存區域已經準備妥當可以提交了。在此之前，請一定要確認還有什麼修改過的或新建的文件還沒有 `git add` 過，否則提交的時候不會記錄這些還沒暫存起來的變化。所以，每次準備提交前，先用 `git status` 看下，是不是都已暫存起來了，然後再運行提交命令 `git commit`：

	$ git commit

這種方式會啟動文本編輯器以便輸入本次提交的說明。（默認會啟用 shell 的環境變量 $EDITOR 所指定的軟件，一般都是 vim 或 emacs。當然也可以按照第一章介紹的方式，使用 `git config --global core.editor` 命令設定你喜歡的編輯軟件。）

編輯器會顯示類似下面的文本信息（本例選用 Vim 的屏顯方式展示）：

	# Please enter the commit message for your changes. Lines starting
	# with '#' will be ignored, and an empty message aborts the commit.
	# On branch master
	# Changes to be committed:
	#   (use "git reset HEAD <file>..." to unstage)
	#
	#       new file:   README
	#       modified:   benchmarks.rb 
	~
	~
	~
	".git/COMMIT_EDITMSG" 10L, 283C

可以看到，默認的提交消息包含最後一次運行 `git status` 的輸出，放在註釋行裡，另外開頭還有一空行，供你輸入提交說明。你完全可以去掉這些註釋行，不過留著也沒關係，多少能幫你回想起這次更新的內容有哪些。（如果覺得這還不夠，可以用 `-v` 選項將修改差異的每一行都包含到註釋中來。）退出編輯器時，Git 會丟掉註釋行，將說明內容和本次更新提交到倉庫。

也可以使用 -m 參數後跟提交說明的方式，在一行命令中提交更新：

	$ git commit -m "Story 182: Fix benchmarks for speed"
	[master]: created 463dc4f: "Fix benchmarks for speed"
	 2 files changed, 3 insertions(+), 0 deletions(-)
	 create mode 100644 README

好，現在你已經創建了第一個提交！可以看到，提交後它會告訴你，當前是在哪個分支（master）提交的，本次提交的完整 SHA-1 校驗和是什麼（`463dc4f`），以及在本次提交中，有多少文件修訂過，多少行添改和刪改過。

記住，提交時記錄的是放在暫存區域的快照，任何還未暫存的仍然保持已修改狀態，可以在下次提交時納入版本管理。每一次運行提交操作，都是對你項目作一次快照，以後可以回到這個狀態，或者進行比較。

### 跳過使用暫存區域 ###

儘管使用暫存區域的方式可以精心準備要提交的細節，但有時候這麼做略顯繁瑣。Git 提供了一個跳過使用暫存區域的方式，只要在提交的時候，給 `git commit` 加上 `-a` 選項，Git 就會自動把所有已經跟蹤過的文件暫存起來一併提交，從而跳過 `git add` 步驟：

	$ git status
	# On branch master
	#
	# Changed but not updated:
	#
	#	modified:   benchmarks.rb
	#
	$ git commit -a -m 'added new benchmarks'
	[master 83e38c7] added new benchmarks
	 1 files changed, 5 insertions(+), 0 deletions(-)

看到了嗎？提交之前不再需要 `git add` 文件 benchmarks.rb 了。

### 移除文件 ###

要從 Git 中移除某個文件，就必須要從已跟蹤文件清單中移除（確切地說，是從暫存區域移除），然後提交。可以用 `git rm` 命令完成此項工作，並連帶從工作目錄中刪除指定的文件，這樣以後就不會出現在未跟蹤文件清單中了。

如果只是簡單地從工作目錄中手工刪除文件，運行 `git status` 時就會在 「Changed but not updated」 部分（也就是_未暫存_清單）看到：

	$ rm grit.gemspec
	$ git status
	# On branch master
	#
	# Changed but not updated:
	#   (use "git add/rm <file>..." to update what will be committed)
	#
	#       deleted:    grit.gemspec
	#

然後再運行 `git rm` 記錄此次移除文件的操作：

	$ git rm grit.gemspec
	rm 'grit.gemspec'
	$ git status
	# On branch master
	#
	# Changes to be committed:
	#   (use "git reset HEAD <file>..." to unstage)
	#
	#       deleted:    grit.gemspec
	#

最後提交的時候，該文件就不再納入版本管理了。如果刪除之前修改過並且已經放到暫存區域的話，則必須要用強制刪除選項 `-f`（譯註：即 force 的首字母），以防誤刪除文件後丟失修改的內容。

另外一種情況是，我們想把文件從 Git 倉庫中刪除（亦即從暫存區域移除），但仍然希望保留在當前工作目錄中。換句話說，僅是從跟蹤清單中刪除。比如一些大型日誌文件或者一堆 `.a` 編譯文件，不小心納入倉庫後，要移除跟蹤但不刪除文件，以便稍後在 `.gitignore` 文件中補上，用 `--cached` 選項即可：

	$ git rm --cached readme.txt

後面可以列出文件或者目錄的名字，也可以使用 glob 模式。比方說：

	$ git rm log/\*.log

注意到星號 `*` 之前的反斜槓 `\`，因為 Git 有它自己的文件模式擴展匹配方式，所以我們不用 shell 來幫忙展開（譯註：實際上不加反斜槓也可以運行，只不過按照 shell 擴展的話，僅僅刪除指定目錄下的文件而不會遞歸匹配。上面的例子本來就指定了目錄，所以效果等同，但下面的例子就會用遞歸方式匹配，所以必須加反斜槓。）。此命令刪除所有 `log/` 目錄下擴展名為 `.log` 的文件。類似的比如：

	$ git rm \*~

會遞歸刪除當前目錄及其子目錄中所有 `~` 結尾的文件。

### 移動文件 ###

不像其他的 VCS 系統，Git 並不跟蹤文件移動操作。如果在 Git 中重命名了某個文件，倉庫中存儲的元數據並不會體現出這是一次改名操作。不過 Git 非常聰明，它會推斷出究竟發生了什麼，至於具體是如何做到的，我們稍後再談。

既然如此，當你看到 Git 的 `mv` 命令時一定會困惑不已。要在 Git 中對文件改名，可以這麼做：

	$ git mv file_from file_to

它會恰如預期般正常工作。實際上，即便此時查看狀態信息，也會明白無誤地看到關於重命名操作的說明：

	$ git mv README.txt README
	$ git status
	# On branch master
	# Your branch is ahead of 'origin/master' by 1 commit.
	#
	# Changes to be committed:
	#   (use "git reset HEAD <file>..." to unstage)
	#
	#       renamed:    README.txt -> README
	#

其實，運行 `git mv` 就相當於運行了下面三條命令：

	$ mv README.txt README
	$ git rm README.txt
	$ git add README

如此分開操作，Git 也會意識到這是一次改名，所以不管何種方式都一樣。當然，直接用 `git mv` 輕便得多，不過有時候用其他工具批處理改名的話，要記得在提交前刪除老的文件名，再添加新的文件名。

## 查看提交歷史 ##

在提交了若干更新之後，又或者克隆了某個項目，想回顧下提交歷史，可以使用 `git log` 命令。

接下來的例子會用我專門用於演示的 simplegit 項目，運行下面的命令獲取該項目源代碼：

	git clone git://github.com/schacon/simplegit-progit.git

然後在此項目中運行 `git log`，應該會看到下面的輸出：

	$ git log
	commit ca82a6dff817ec66f44342007202690a93763949
	Author: Scott Chacon <schacon@gee-mail.com>
	Date:   Mon Mar 17 21:52:11 2008 -0700

	    changed the verison number

	commit 085bb3bcb608e1e8451d4b2432f8ecbe6306e7e7
	Author: Scott Chacon <schacon@gee-mail.com>
	Date:   Sat Mar 15 16:40:33 2008 -0700

	    removed unnecessary test code

	commit a11bef06a3f659402fe7563abf99ad00de2209e6
	Author: Scott Chacon <schacon@gee-mail.com>
	Date:   Sat Mar 15 10:31:28 2008 -0700

	    first commit

默認不用任何參數的話，`git log` 會按提交時間列出所有的更新，最近的更新排在最上面。看到了嗎，每次更新都有一個 SHA-1 校驗和、作者的名字和電子郵件地址、提交時間，最後縮進一個段落顯示提交說明。

`git log` 有許多選項可以幫助你搜尋感興趣的提交，接下來我們介紹些最常用的。

我們常用 `-p` 選項展開顯示每次提交的內容差異，用 `-2` 則僅顯示最近的兩次更新：

	$ git log –p -2
	commit ca82a6dff817ec66f44342007202690a93763949
	Author: Scott Chacon <schacon@gee-mail.com>
	Date:   Mon Mar 17 21:52:11 2008 -0700

	    changed the verison number

	diff --git a/Rakefile b/Rakefile
	index a874b73..8f94139 100644
	--- a/Rakefile
	+++ b/Rakefile
	@@ -5,7 +5,7 @@ require 'rake/gempackagetask'
	 spec = Gem::Specification.new do |s|
	-    s.version   =   "0.1.0"
	+    s.version   =   "0.1.1"
	     s.author    =   "Scott Chacon"

	commit 085bb3bcb608e1e8451d4b2432f8ecbe6306e7e7
	Author: Scott Chacon <schacon@gee-mail.com>
	Date:   Sat Mar 15 16:40:33 2008 -0700

	    removed unnecessary test code

	diff --git a/lib/simplegit.rb b/lib/simplegit.rb
	index a0a60ae..47c6340 100644
	--- a/lib/simplegit.rb
	+++ b/lib/simplegit.rb
	@@ -18,8 +18,3 @@ class SimpleGit
	     end

	 end
	-
	-if $0 == __FILE__
	-  git = SimpleGit.new
	-  puts git.show
	-end
	\ No newline at end of file

在做代碼審查，或者要快速瀏覽其他協作者提交的更新都作了哪些改動時，就可以用這個選項。此外，還有許多摘要選項可以用，比如 `--stat`，僅顯示簡要的增改行數統計：

	$ git log --stat 
	commit ca82a6dff817ec66f44342007202690a93763949
	Author: Scott Chacon <schacon@gee-mail.com>
	Date:   Mon Mar 17 21:52:11 2008 -0700

	    changed the verison number

	 Rakefile |    2 +-
	 1 files changed, 1 insertions(+), 1 deletions(-)

	commit 085bb3bcb608e1e8451d4b2432f8ecbe6306e7e7
	Author: Scott Chacon <schacon@gee-mail.com>
	Date:   Sat Mar 15 16:40:33 2008 -0700

	    removed unnecessary test code

	 lib/simplegit.rb |    5 -----
	 1 files changed, 0 insertions(+), 5 deletions(-)

	commit a11bef06a3f659402fe7563abf99ad00de2209e6
	Author: Scott Chacon <schacon@gee-mail.com>
	Date:   Sat Mar 15 10:31:28 2008 -0700

	    first commit

	 README           |    6 ++++++
	 Rakefile         |   23 +++++++++++++++++++++++
	 lib/simplegit.rb |   25 +++++++++++++++++++++++++
	 3 files changed, 54 insertions(+), 0 deletions(-)

每個提交都列出了修改過的文件，以及其中添加和移除的行數，並在最後列出所有增減行數小計。

還有個常用的 `--pretty` 選項，可以指定使用完全不同於默認格式的方式展示提交歷史。比如用 `oneline` 將每個提交放在一行顯示，這在提交數很大時非常有用。另外還有 `short`，`full` 和 `fuller` 可以用，展示的信息或多或少有些不同，請自己動手實踐一下看看效果如何。

	$ git log --pretty=oneline
	ca82a6dff817ec66f44342007202690a93763949 changed the verison number
	085bb3bcb608e1e8451d4b2432f8ecbe6306e7e7 removed unnecessary test code
	a11bef06a3f659402fe7563abf99ad00de2209e6 first commit

但最有意思的是 `format`，可以定製要顯示的記錄格式，這樣的輸出便於後期編程提取分析，像這樣：

	$ git log --pretty=format:"%h - %an, %ar : %s"
	ca82a6d - Scott Chacon, 11 months ago : changed the verison number
	085bb3b - Scott Chacon, 11 months ago : removed unnecessary test code
	a11bef0 - Scott Chacon, 11 months ago : first commit

表 2-1 列出了常用的格式佔位符寫法及其代表的意義。

	選項	 說明
	%H	提交對象（commit）的完整哈希字串
	%h	提交對象的簡短哈希字串
	%T	樹對象（tree）的完整哈希字串
	%t	樹對象的簡短哈希字串
	%P	父對象（parent）的完整哈希字串
	%p	父對象的簡短哈希字串
	%an	作者（author）的名字
	%ae	作者的電子郵件地址
	%ad	作者修訂日期（可以用 -date= 選項定製格式）
	%ar	作者修訂日期，按多久以前的方式顯示
	%cn	提交者(committer)的名字
	%ce	提交者的電子郵件地址
	%cd	提交日期
	%cr	提交日期，按多久以前的方式顯示
	%s	提交說明

你一定奇怪_作者（author）_和_提交者（committer）_之間究竟有何差別，其實作者指的是實際作出修改的人，提交者指的是最後將此工作成果提交到倉庫的人。所以，當你為某個項目發去補丁，然後某個核心成員將你的補丁併入項目時，你就是作者，而那個核心成員就是提交者。我們會在第五章再詳細介紹兩者之間的細緻差別。

用 oneline 或 format 時結合 `--graph` 選項，可以看到開頭多出一些 ASCII 字符串表示的簡單圖形，形象地展示了每個提交所在的分支及其分化衍合情況。在我們之前提到的 Grit 項目倉庫中可以看到：

	$ git log --pretty=format:"%h %s" --graph
	* 2d3acf9 ignore errors from SIGCHLD on trap
	*  5e3ee11 Merge branch 'master' of git://github.com/dustin/grit
	|\  
	| * 420eac9 Added a method for getting the current branch.
	* | 30e367c timeout code and tests
	* | 5a09431 add timeout protection to grit
	* | e1193f8 support for heads with slashes in them
	|/  
	* d6016bc require time for xmlschema
	*  11d191e Merge branch 'defunkt' into local

以上只是簡單介紹了一些 `git log` 命令支持的選項。表 2-2 還列出了一些其他常用的選項及其釋義。

	選項 說明
	-p 按補丁格式顯示每個更新之間的差異。
	--stat 顯示每次更新的文件修改統計信息。
	--shortstat 只顯示 --stat 中最後的行數修改添加移除統計。
	--name-only 僅在提交信息後顯示已修改的文件清單。
	--name-status 顯示新增、修改、刪除的文件清單。
	--abbrev-commit 僅顯示 SHA-1 的前幾個字符，而非所有的 40 個字符。
	--relative-date 使用較短的相對時間顯示（比如，「2 weeks ago」）。
	--graph 顯示 ASCII 圖形表示的分支合併歷史。
	--pretty 使用其他格式顯示歷史提交信息。可用的選項包括 oneline，short，full，fuller 和 format（後跟指定格式）。

### 限制輸出長度 ###

除了定製輸出格式的選項之外，`git log` 還有許多非常實用的限制輸出長度的選項，也就是只輸出部分提交信息。之前我們已經看到過 `-2` 了，它只顯示最近的兩條提交，實際上，這是 `-<n>` 選項的寫法，其中的 `n` 可以是任何自然數，表示僅顯示最近的若干條提交。不過實踐中我們是不太用這個選項的，Git 在輸出所有提交時會自動調用分頁程序（pager），要看更早的更新只需翻到下頁即可。

另外還有按照時間作限制的選項，比如 `--since` 和 `--until`。下面的命令列出所有最近兩週內的提交：

	$ git log --since=2.weeks

你可以給出各種時間格式，比如說具體的某一天（「2008-01-15」），或者是多久以前（「2 years 1 day 3 minutes ago」）。

還可以給出若干搜索條件，列出符合的提交。用 `--author` 選項顯示指定作者的提交，用 `--grep` 選項搜索提交說明中的關鍵字。（請注意，如果要得到同時滿足這兩個選項搜索條件的提交，就必須用 `--all-match` 選項。）

如果只關心某些文件或者目錄的歷史提交，可以在 `git log` 選項的最後指定它們的路徑。因為是放在最後位置上的選項，所以用兩個短劃線（`--`）隔開之前的選項和後面限定的路徑名。

表 2-3 還列出了其他常用的類似選項。

	選項 說明
	-(n)	僅顯示最近的 n 條提交
	--since, --after 僅顯示指定時間之後的提交。
	--until, --before 僅顯示指定時間之前的提交。
	--author 僅顯示指定作者相關的提交。
	--committer 僅顯示指定提交者相關的提交。

來看一個實際的例子，如果要查看 Git 倉庫中，2008 年 10 月期間，Junio Hamano 提交的但未合併的測試腳本（位於項目的 t/ 目錄下的文件），可以用下面的查詢命令：

	$ git log --pretty="%h:%s" --author=gitster --since="2008-10-01" \
	   --before="2008-11-01" --no-merges -- t/
	5610e3b - Fix testcase failure when extended attribute
	acd3b9e - Enhance hold_lock_file_for_{update,append}()
	f563754 - demonstrate breakage of detached checkout wi
	d1a43f2 - reset --hard/read-tree --reset -u: remove un
	51a94af - Fix "checkout --track -b newbranch" on detac
	b0ad11e - pull: allow "git pull origin $something:$cur

Git 項目有 20,000 多條提交，但我們給出搜索選項後，僅列出了其中滿足條件的 6 條。

### 使用圖形化工具查閱提交歷史 ###

有時候圖形化工具更容易展示歷史提交的變化，隨 Git 一同發佈的 gitk 就是這樣一種工具。它是用 Tcl/Tk 寫成的，基本上相當於 `git log` 命令的可視化版本，凡是 `git log` 可以用的選項也都能用在 gitk 上。在項目工作目錄中輸入 gitk 命令後，就會啟動圖 2-2 所示的界面。

Insert 18333fig0202.png 
圖 2-2. gitk 的圖形界面

上半個窗口顯示的是歷次提交的分支祖先圖譜，下半個窗口顯示當前點選的提交對應的具體差異。

## 撤消操作 ##

任何時候，你都有可能需要撤消剛才所做的某些操作。接下來，我們會介紹一些基本的撤消操作相關的命令。請注意，有些操作並不總是可以撤消的，所以請務必謹慎小心，一旦失誤，就有可能丟失部分工作成果。

### 修改最後一次提交 ###

有時候我們提交完了才發現漏掉了幾個文件沒有加，或者提交信息寫錯了。想要撤消剛才的提交操作，可以使用 `--amend` 選項重新提交：

	$ git commit --amend

此命令將使用當前的暫存區域快照提交。如果剛才提交完沒有作任何改動，直接運行此命令的話，相當於有機會重新編輯提交說明，而所提交的文件快照和之前的一樣。

啟動文本編輯器後，會看到上次提交時的說明，編輯它確認沒問題後保存退出，就會使用新的提交說明覆蓋剛才失誤的提交。

如果剛才提交時忘了暫存某些修改，可以先補上暫存操作，然後再運行 `--amend` 提交：

	$ git commit -m 'initial commit'
	$ git add forgotten_file
	$ git commit --amend 

上面的三條命令最終得到一個提交，第二個提交命令修正了第一個的提交內容。

### 取消已經暫存的文件 ###

接下來的兩個小節將演示如何取消暫存區域中的文件，以及如何取消工作目錄中已修改的文件。不用擔心，查看文件狀態的時候就提示了該如何撤消，所以不需要死記硬背。來看下面的例子，有兩個修改過的文件，我們想要分開提交，但不小心用 `git add *` 全加到了暫存區域。該如何撤消暫存其中的一個文件呢？`git status` 命令的輸出會告訴你怎麼做：

	$ git add .
	$ git status
	# On branch master
	# Changes to be committed:
	#   (use "git reset HEAD <file>..." to unstage)
	#
	#       modified:   README.txt
	#       modified:   benchmarks.rb
	#

就在 「Changes to be committed」 下面，括號中有提示，可以使用 `git reset HEAD <file>...` 的方式取消暫存。好吧，我們來試試取消暫存 benchmarks.rb 文件：

	$ git reset HEAD benchmarks.rb 
	benchmarks.rb: locally modified
	$ git status
	# On branch master
	# Changes to be committed:
	#   (use "git reset HEAD <file>..." to unstage)
	#
	#       modified:   README.txt
	#
	# Changed but not updated:
	#   (use "git add <file>..." to update what will be committed)
	#   (use "git checkout -- <file>..." to discard changes in working directory)
	#
	#       modified:   benchmarks.rb
	#

這條命令看起來有些古怪，先別管，能用就行。現在 benchmarks.rb 文件又回到了之前已修改未暫存的狀態。

### 取消對文件的修改 ###

如果覺得剛才對 benchmarks.rb 的修改完全沒有必要，該如何取消修改，回到之前的狀態（也就是修改之前的版本）呢？`git status` 同樣提示了具體的撤消方法，接著上面的例子，現在未暫存區域看起來像這樣：

	# Changed but not updated:
	#   (use "git add <file>..." to update what will be committed)
	#   (use "git checkout -- <file>..." to discard changes in working directory)
	#
	#       modified:   benchmarks.rb
	#

在第二個括號中，我們看到了拋棄文件修改的命令（至少在 Git 1.6.1 以及更高版本中會這樣提示，如果你還在用老版本，我們強烈建議你升級，以獲取最佳的用戶體驗），讓我們試試看：

	$ git checkout -- benchmarks.rb
	$ git status
	# On branch master
	# Changes to be committed:
	#   (use "git reset HEAD <file>..." to unstage)
	#
	#       modified:   README.txt
	#

可以看到，該文件已經恢復到修改前的版本。你可能已經意識到了，這條命令有些危險，所有對文件的修改都沒有了，因為我們剛剛把之前版本的文件複製過來重寫了此文件。所以在用這條命令前，請務必確定真的不再需要保留剛才的修改。如果只是想回退版本，同時保留剛才的修改以便將來繼續工作，可以用下章介紹的 stashing 和分支來處理，應該會更好些。

記住，任何已經提交到 Git 的都可以被恢復。即便在已經刪除的分支中的提交，或者用 `--amend` 重新改寫的提交，都可以被恢復（關於數據恢復的內容見第九章）。所以，你可能失去的數據，僅限於沒有提交過的，對 Git 來說它們就像從未存在過一樣。

## 遠程倉庫的使用 ##

要參與任何一個 Git 項目的協作，必須要瞭解該如何管理遠程倉庫。遠程倉庫是指託管在網絡上的項目倉庫，可能會有好多個，其中有些你只能讀，另外有些可以寫。同他人協作開發某個項目時，需要管理這些遠程倉庫，以便推送或拉取數據，分享各自的工作進展。管理遠程倉庫的工作，包括添加遠程庫，移除廢棄的遠程庫，管理各式遠程庫分支，定義是否跟蹤這些分支，等等。本節我們將詳細討論遠程庫的管理和使用。

### 查看當前的遠程庫 ###

要查看當前配置有哪些遠程倉庫，可以用 `git remote` 命令，它會列出每個遠程庫的簡短名字。在克隆完某個項目後，至少可以看到一個名為 origin 的遠程庫，Git 默認使用這個名字來標識你所克隆的原始倉庫：

	$ git clone git://github.com/schacon/ticgit.git
	Initialized empty Git repository in /private/tmp/ticgit/.git/
	remote: Counting objects: 595, done.
	remote: Compressing objects: 100% (269/269), done.
	remote: Total 595 (delta 255), reused 589 (delta 253)
	Receiving objects: 100% (595/595), 73.31 KiB | 1 KiB/s, done.
	Resolving deltas: 100% (255/255), done.
	$ cd ticgit
	$ git remote 
	origin

也可以加上 `-v` 選項（譯註：此為 --verbose 的簡寫，取首字母），顯示對應的克隆地址：

	$ git remote -v
	origin	git://github.com/schacon/ticgit.git

如果有多個遠程倉庫，此命令將全部列出。比如在我的 Grit 項目中，可以看到：

	$ cd grit
	$ git remote -v
	bakkdoor  git://github.com/bakkdoor/grit.git
	cho45     git://github.com/cho45/grit.git
	defunkt   git://github.com/defunkt/grit.git
	koke      git://github.com/koke/grit.git
	origin    git@github.com:mojombo/grit.git

這樣一來，我就可以非常輕鬆地從這些用戶的倉庫中，拉取他們的提交到本地。請注意，上面列出的地址只有 origin 用的是 SSH URL 鏈接，所以也只有這個倉庫我能推送數據上去（我們會在第四章解釋原因）。

### 添加遠程倉庫 ###

要添加一個新的遠程倉庫，可以指定一個簡單的名字，以便將來引用，運行 `git remote add [shortname] [url]`：

	$ git remote
	origin
	$ git remote add pb git://github.com/paulboone/ticgit.git
	$ git remote -v
	origin	git://github.com/schacon/ticgit.git
	pb	git://github.com/paulboone/ticgit.git

現在可以用字串 pb 指代對應的倉庫地址了。比如說，要抓取所有 Paul 有的，但本地倉庫沒有的信息，可以運行 `git fetch pb`：

	$ git fetch pb
	remote: Counting objects: 58, done.
	remote: Compressing objects: 100% (41/41), done.
	remote: Total 44 (delta 24), reused 1 (delta 0)
	Unpacking objects: 100% (44/44), done.
	From git://github.com/paulboone/ticgit
	 * [new branch]      master     -> pb/master
	 * [new branch]      ticgit     -> pb/ticgit

現在，Paul 的主幹分支（master）已經完全可以在本地訪問了，對應的名字是 `pb/master`，你可以將它合併到自己的某個分支，或者切換到這個分支，看看有些什麼有趣的更新。

### 從遠程倉庫抓取數據 ###

正如之前所看到的，可以用下面的命令從遠程倉庫抓取數據到本地：

	$ git fetch [remote-name]

此命令會到遠程倉庫中拉取所有你本地倉庫中還沒有的數據。運行完成後，你就可以在本地訪問該遠程倉庫中的所有分支，將其中某個分支合併到本地，或者只是取出某個分支，一探究竟。（我們會在第三章詳細討論關於分支的概念和操作。）

如果是克隆了一個倉庫，此命令會自動將遠程倉庫歸於 origin 名下。所以，`git fetch origin` 會抓取從你上次克隆以來別人上傳到此遠程倉庫中的所有更新（或是上次 fetch 以來別人提交的更新）。有一點很重要，需要記住，fetch 命令只是將遠端的數據拉到本地倉庫，並不自動合併到當前工作分支，只有當你確實準備好了，才能手工合併。

如果設置了某個分支用於跟蹤某個遠端倉庫的分支（參見下節及第三章的內容），可以使用 `git pull` 命令自動抓取數據下來，然後將遠端分支自動合併到本地倉庫中當前分支。在日常工作中我們經常這麼用，既快且好。實際上，默認情況下 `git clone` 命令本質上就是自動創建了本地的 master 分支用於跟蹤遠程倉庫中的 master 分支（假設遠程倉庫確實有 master 分支）。所以一般我們運行 `git pull`，目的都是要從原始克隆的遠端倉庫中抓取數據後，合併到工作目錄中當前分支。

### 推送數據到遠程倉庫 ###

項目進行到一個階段，要同別人分享目前的成果，可以將本地倉庫中的數據推送到遠程倉庫。實現這個任務的命令很簡單： `git push [remote-name] [branch-name]`。如果要把本地的 master 分支推送到 `origin` 服務器上（再次說明下，克隆操作會自動使用默認的 master 和 origin 名字），可以運行下面的命令：

	$ git push origin master

只有在所克隆的服務器上有寫權限，或者同一時刻沒有其他人在推數據，這條命令才會如期完成任務。如果在你推數據前，已經有其他人推送了若干更新，那你的推送操作就會被駁回。你必須先把他們的更新抓取到本地，並到自己的項目中，然後才可以再次推送。有關推送數據到遠程倉庫的詳細內容見第三章。

### 查看遠程倉庫信息 ###

我們可以通過命令 `git remote show [remote-name]` 查看某個遠程倉庫的詳細信息，比如要看所克隆的 `origin` 倉庫，可以運行：

	$ git remote show origin
	* remote origin
	  URL: git://github.com/schacon/ticgit.git
	  Remote branch merged with 'git pull' while on branch master
	    master
	  Tracked remote branches
	    master
	    ticgit

除了對應的克隆地址外，它還給出了許多額外的信息。它友善地告訴你如果是在 master 分支，就可以用 `git pull` 命令抓取數據合併到本地。另外還列出了所有處於跟蹤狀態中的遠端分支。

實際使用過程中，`git remote show` 給出的信息可能會像這樣：

	$ git remote show origin
	* remote origin
	  URL: git@github.com:defunkt/github.git
	  Remote branch merged with 'git pull' while on branch issues
	    issues
	  Remote branch merged with 'git pull' while on branch master
	    master
	  New remote branches (next fetch will store in remotes/origin)
	    caching
	  Stale tracking branches (use 'git remote prune')
	    libwalker
	    walker2
	  Tracked remote branches
	    acl
	    apiv2
	    dashboard2
	    issues
	    master
	    postgres
	  Local branch pushed with 'git push'
	    master:master

它告訴我們，運行 `git push` 時缺省推送的分支是什麼（譯註：最後兩行）。它還顯示了有哪些遠端分支還沒有同步到本地（譯註：第六行的 caching 分支），哪些已同步到本地的遠端分支在遠端服務器上已被刪除（譯註：Stale tracking branches 下面的兩個分支），以及運行 `git pull` 時將自動合併哪些分支（譯註：前四行中列出的 issues 和 master 分支）。

### 遠程倉庫的刪除和重命名 ###

在新版 Git 中可以用 `git remote rename` 命令修改某個遠程倉庫的簡短名稱，比如想把 `pb` 改成 `paul`，可以這麼運行：

	$ git remote rename pb paul
	$ git remote
	origin
	paul

注意，對遠程倉庫的重命名，也會使對應的分支名稱發生變化，原來的 `pb/master` 分支現在成了 `paul/master`。

碰到遠端倉庫服務器遷移，或者原來的克隆鏡像不再使用，又或者某個參與者不再貢獻代碼，那麼需要移除對應的遠端倉庫，可以運行 `git remote rm` 命令：

	$ git remote rm paul
	$ git remote
	origin

## 打標籤 ##

同大多數 VCS 一樣，Git 也可以對某一時間點上的版本打上標籤。人們在發佈某個軟件版本（比如 v1.0 等等）的時候，經常這麼做。本節我們一起來學習如何列出所有可用的標籤，如何新建標籤，以及各種不同類型標籤之間的差別。

### 列顯已有的標籤 ###

列出現有標籤的命令非常簡單，直接運行 `git tag` 即可：

	$ git tag
	v0.1
	v1.3

顯示的標籤按字母順序排列，所以標籤的先後並不表示重要程度的輕重。

我們可以用特定的搜索模式列出符合條件的標籤。在 Git 自身項目倉庫中，有著超過 240 個標籤，如果你只對 1.4.2 系列的版本感興趣，可以運行下面的命令：

	$ git tag -l 'v1.4.2.*'
	v1.4.2.1
	v1.4.2.2
	v1.4.2.3
	v1.4.2.4

### 新建標籤 ###

Git 使用的標籤有兩種類型：輕量級的（lightweight）和含附註的（annotated）。輕量級標籤就像是個不會變化的分支，實際上它就是個指向特定提交對象的引用。而含附註標籤，實際上是存儲在倉庫中的一個獨立對象，它有自身的校驗和信息，包含著標籤的名字，電子郵件地址和日期，以及標籤說明，標籤本身也允許使用 GNU Privacy Guard (GPG) 來簽署或驗證。一般我們都建議使用含附註型的標籤，以便保留相關信息；當然，如果只是臨時性加註標簽，或者不需要旁註額外信息，用輕量級標籤也沒問題。

### 含附註的標籤 ###

創建一個含附註類型的標籤非常簡單，用 `-a` （譯註：取 annotated 的首字母）指定標籤名字即可：

	$ git tag -a v1.4 -m 'my version 1.4'
	$ git tag
	v0.1
	v1.3
	v1.4

而 `-m` 選項則指定了對應的標籤說明，Git 會將此說明一同保存在標籤對象中。如果在此選項後沒有給出具體的說明內容，Git 會啟動文本編輯軟件供你輸入。

可以使用 `git show` 命令查看相應標籤的版本信息，並連同顯示打標籤時的提交對象。

	$ git show v1.4
	tag v1.4
	Tagger: Scott Chacon <schacon@gee-mail.com>
	Date:   Mon Feb 9 14:45:11 2009 -0800

	my version 1.4
	commit 15027957951b64cf874c3557a0f3547bd83b3ff6
	Merge: 4a447f7... a6b4c97...
	Author: Scott Chacon <schacon@gee-mail.com>
	Date:   Sun Feb 8 19:02:46 2009 -0800

	    Merge branch 'experiment'

我們可以看到在提交對象信息上面，列出了此標籤的提交者和提交時間，以及相應的標籤說明。

### 簽署標籤 ###

如果你有自己的私鑰，還可以用 GPG 來簽署標籤，只需要把之前的 `-a` 改為 `-s` （譯註： 取 Signed 的首字母）即可：

	$ git tag -s v1.5 -m 'my signed 1.5 tag'
	You need a passphrase to unlock the secret key for
	user: "Scott Chacon <schacon@gee-mail.com>"
	1024-bit DSA key, ID F721C45A, created 2009-02-09

現在再運行 `git show` 會看到對應的 GPG 簽名也附在其內：

	$ git show v1.5
	tag v1.5
	Tagger: Scott Chacon <schacon@gee-mail.com>
	Date:   Mon Feb 9 15:22:20 2009 -0800

	my signed 1.5 tag
	-----BEGIN PGP SIGNATURE-----
	Version: GnuPG v1.4.8 (Darwin)

	iEYEABECAAYFAkmQurIACgkQON3DxfchxFr5cACeIMN+ZxLKggJQf0QYiQBwgySN
	Ki0An2JeAVUCAiJ7Ox6ZEtK+NvZAj82/
	=WryJ
	-----END PGP SIGNATURE-----
	commit 15027957951b64cf874c3557a0f3547bd83b3ff6
	Merge: 4a447f7... a6b4c97...
	Author: Scott Chacon <schacon@gee-mail.com>
	Date:   Sun Feb 8 19:02:46 2009 -0800

	    Merge branch 'experiment'

稍後我們再學習如何驗證已經簽署的標籤。

### 輕量級標籤 ###

輕量級標籤實際上就是一個保存著對應提交對象的校驗和信息的文件。要創建這樣的標籤，一個 `-a`，`-s` 或 `-m` 選項都不用，直接給出標籤名字即可：

	$ git tag v1.4-lw
	$ git tag
	v0.1
	v1.3
	v1.4
	v1.4-lw
	v1.5

現在運行 `git show` 查看此標籤信息，就只有相應的提交對象摘要：

	$ git show v1.4-lw
	commit 15027957951b64cf874c3557a0f3547bd83b3ff6
	Merge: 4a447f7... a6b4c97...
	Author: Scott Chacon <schacon@gee-mail.com>
	Date:   Sun Feb 8 19:02:46 2009 -0800

	    Merge branch 'experiment'

### 驗證標籤 ###

可以使用 `git tag -v [tag-name]` （譯註：取 verify 的首字母）的方式驗證已經簽署的標籤。此命令會調用 GPG 來驗證簽名，所以你需要有簽署者的公鑰，存放在 keyring 中，才能驗證：

	$ git tag -v v1.4.2.1
	object 883653babd8ee7ea23e6a5c392bb739348b1eb61
	type commit
	tag v1.4.2.1
	tagger Junio C Hamano <junkio@cox.net> 1158138501 -0700

	GIT 1.4.2.1

	Minor fixes since 1.4.2, including git-mv and git-http with alternates.
	gpg: Signature made Wed Sep 13 02:08:25 2006 PDT using DSA key ID F3119B9A
	gpg: Good signature from "Junio C Hamano <junkio@cox.net>"
	gpg:                 aka "[jpeg image of size 1513]"
	Primary key fingerprint: 3565 2A26 2040 E066 C9A7  4A7D C0C6 D9A4 F311 9B9A

若是沒有簽署者的公鑰，會報告類似下面這樣的錯誤：

	gpg: Signature made Wed Sep 13 02:08:25 2006 PDT using DSA key ID F3119B9A
	gpg: Can't check signature: public key not found
	error: could not verify the tag 'v1.4.2.1'

### 後期加註標簽 ###

你甚至可以在後期對早先的某次提交加註標簽。比如在下面展示的提交歷史中：

	$ git log --pretty=oneline
	15027957951b64cf874c3557a0f3547bd83b3ff6 Merge branch 'experiment'
	a6b4c97498bd301d84096da251c98a07c7723e65 beginning write support
	0d52aaab4479697da7686c15f77a3d64d9165190 one more thing
	6d52a271eda8725415634dd79daabbc4d9b6008e Merge branch 'experiment'
	0b7434d86859cc7b8c3d5e1dddfed66ff742fcbc added a commit function
	4682c3261057305bdd616e23b64b0857d832627b added a todo file
	166ae0c4d3f420721acbb115cc33848dfcc2121a started write support
	9fceb02d0ae598e95dc970b74767f19372d61af8 updated rakefile
	964f16d36dfccde844893cac5b347e7b3d44abbc commit the todo
	8a5cbc430f1a9c3d00faaeffd07798508422908a updated readme

我們忘了在提交 「updated rakefile」 後為此項目打上版本號 v1.2，沒關係，現在也能做。只要在打標籤的時候跟上對應提交對象的校驗和（或前幾位字符）即可：

	$ git tag -a v1.2 9fceb02

可以看到我們已經補上了標籤：

	$ git tag 
	v0.1
	v1.2
	v1.3
	v1.4
	v1.4-lw
	v1.5

	$ git show v1.2
	tag v1.2
	Tagger: Scott Chacon <schacon@gee-mail.com>
	Date:   Mon Feb 9 15:32:16 2009 -0800

	version 1.2
	commit 9fceb02d0ae598e95dc970b74767f19372d61af8
	Author: Magnus Chacon <mchacon@gee-mail.com>
	Date:   Sun Apr 27 20:43:35 2008 -0700

	    updated rakefile
	...

### 分享標籤 ###

默認情況下，`git push` 並不會把標籤傳送到遠端服務器上，只有通過顯式命令才能分享標籤到遠端倉庫。其命令格式如同推送分支，運行 `git push origin [tagname]` 即可： 

	$ git push origin v1.5
	Counting objects: 50, done.
	Compressing objects: 100% (38/38), done.
	Writing objects: 100% (44/44), 4.56 KiB, done.
	Total 44 (delta 18), reused 8 (delta 1)
	To git@github.com:schacon/simplegit.git
	* [new tag]         v1.5 -> v1.5

如果要一次推送所有（本地新增的）標籤上去，可以使用 `--tags` 選項：

	$ git push origin --tags
	Counting objects: 50, done.
	Compressing objects: 100% (38/38), done.
	Writing objects: 100% (44/44), 4.56 KiB, done.
	Total 44 (delta 18), reused 8 (delta 1)
	To git@github.com:schacon/simplegit.git
	 * [new tag]         v0.1 -> v0.1
	 * [new tag]         v1.2 -> v1.2
	 * [new tag]         v1.4 -> v1.4
	 * [new tag]         v1.4-lw -> v1.4-lw
	 * [new tag]         v1.5 -> v1.5

現在，其他人克隆共享倉庫或拉取數據同步後，也會看到這些標籤。

## 技巧和竅門 ##

在結束本章之前，我還想和大家分享一些 Git 使用的技巧和竅門。很多使用 Git 的開發者可能根本就沒用過這些技巧，我們也不是說在讀過本書後非得用這些技巧不可，但至少應該有所瞭解吧。說實話，有了這些小竅門，我們的工作可以變得更簡單，更輕鬆，更高效。

### 自動完成 ###

如果你用的是 Bash shell，可以試試看 Git 提供的自動完成腳本。下載 Git 的源代碼，進入  `contrib/completion` 目錄，會看到一個 `git-completion.bash` 文件。將此文件複製到你自己的用戶主目錄中（譯註：按照下面的示例，還應改名加上點：cp git-completion.bash ~/.git-completion.bash），並把下面一行內容添加到你的 `.bashrc` 文件中：

	source ~/.git-completion.bash

也可以為系統上所有用戶都設置默認使用此腳本。Mac 上將此腳本複製到 `/opt/local/etc/bash_completion.d` 目錄中，Linux 上則複製到 `/etc/bash_completion.d/` 目錄中即可。這兩處目錄中的腳本，都會在 Bash 啟動時自動加載。

如果在 Windows 上安裝了 msysGit，默認使用的 Git Bash 就已經配好了這個自動完成腳本，可以直接使用。

在輸入 Git 命令的時候可以敲兩次跳格鍵（Tab），就會看到列出所有匹配的可用命令建議：

	$ git co<tab><tab>
	commit config

此例中，鍵入 git co 然後連按兩次 Tab 鍵，會看到兩個相關的建議（命令） commit 和 config。繼而輸入 `m<tab>` 會自動完成 `git commit` 命令的輸入。

命令的選項也可以用這種方式自動完成，其實這種情況更實用些。比如運行 `git log` 的時候忘了相關選項的名字，可以輸入開頭的幾個字母，然後敲 Tab 鍵看看有哪些匹配的：

	$ git log --s<tab>
	--shortstat  --since=  --src-prefix=  --stat   --summary

這個技巧不錯吧，可以節省很多輸入和查閱文檔的時間。

### Git 命令別名 ###

Git 並不會推斷你輸入的幾個字符將會是哪條命令，不過如果想偷懶，少敲幾個命令的字符，可以用 `git config` 為命令設置別名。來看看下面的例子：

	$ git config --global alias.co checkout
	$ git config --global alias.br branch
	$ git config --global alias.ci commit
	$ git config --global alias.st status

現在，如果要輸入 `git commit` 只需鍵入 `git ci` 即可。而隨著 Git 使用的深入，會有很多經常要用到的命令，遇到這種情況，不妨建個別名提高效率。

使用這種技術還可以創造出新的命令，比方說取消暫存文件時的輸入比較繁瑣，可以自己設置一下：

	$ git config --global alias.unstage 'reset HEAD --'

這樣一來，下面的兩條命令完全等同：

	$ git unstage fileA
	$ git reset HEAD fileA

顯然，使用別名的方式看起來更清楚。另外，我們還經常設置 `last` 命令：

	$ git config --global alias.last 'log -1 HEAD'

然後要看最後一次的提交信息，就變得簡單多了：
	
	$ git last
	commit 66938dae3329c7aebe598c2246a8e6af90d04646
	Author: Josh Goebel <dreamer3@example.com>
	Date:   Tue Aug 26 19:48:51 2008 +0800

	    test for current head

	    Signed-off-by: Scott Chacon <schacon@example.com>

可以看出，實際上 Git 只是簡單地在命令中替換了你設置的別名。不過有時候我們希望運行某個外部命令，而非 Git 的附屬工具，這個好辦，只需要在命令前加上 `!` 就行。如果你自己寫了些處理 Git 倉庫信息的腳本的話，就可以用這種技術包裝起來。作為演示，我們可以設置用 `git visual` 啟動 `gitk`：

	$ git config --global alias.visual "!gitk"

## 小結 ##

到目前為止，你已經學會了最基本的 Git 操作：創建和克隆倉庫，作出更新，暫存並提交這些更新，以及查看所有歷史更新記錄。接下來，我們將學習 Git 的必殺技特性：分支模型。

