# 分佈式 Git #

為了便於項目中的所有開發者分享代碼，我們準備好了一台服務器存放遠程 Git 倉庫。經過前面幾章的學習，我們已經學會了一些基本的本地工作流程中所需用到的命令。接下來，我們要學習下如何利用 Git 來組織和完成分佈式工作流程。

特別是，當作為項目貢獻者時，我們該怎麼做才能方便維護者採納更新；或者作為項目維護者時，又該怎樣有效管理大量貢獻者的提交。

## 分佈式工作流程 ##

同傳統的集中式版本控制系統（CVCS）不同，開發者之間的協作方式因著 Git 的分佈式特性而變得更為靈活多樣。在集中式系統上，每個開發者就像是連接在集線器上的節點，彼此的工作方式大體相像。而在 Git 網絡中，每個開發者同時扮演著節點和集線器的角色，這就是說，每一個開發者都可以將自己的代碼貢獻到另外一個開發者的倉庫中，或者建立自己的公共倉庫，讓其他開發者基於自己的工作開始，為自己的倉庫貢獻代碼。於是，Git 的分佈式協作便可以衍生出種種不同的工作流程，我會在接下來的章節介紹幾種常見的應用方式，並分別討論各自的優缺點。你可以選擇其中的一種，或者結合起來，應用到你自己的項目中。

### 集中式工作流 ###

通常，集中式工作流程使用的都是單點協作模型。一個存放代碼倉庫的中心服務器，可以接受所有開發者提交的代碼。所有的開發者都是普通的節點，作為中心集線器的消費者，平時的工作就是和中心倉庫同步數據（見圖 5-1）。

Insert 18333fig0501.png 
圖 5-1. 集中式工作流

如果兩個開發者從中心倉庫克隆代碼下來，同時作了一些修訂，那麼只有第一個開發者可以順利地把數據推送到共享服務器。第二個開發者在提交他的修訂之前，必須先下載合併服務器上的數據，解決衝突之後才能推送數據到共享服務器上。在 Git 中這麼用也決無問題，這就好比是在用 Subversion（或其他 CVCS）一樣，可以很好地工作。

如果你的團隊不是很大，或者大家都已經習慣了使用集中式工作流程，完全可以採用這種簡單的模式。只需要配置好一台中心服務器，並給每個人推送數據的權限，就可以開展工作了。但如果提交代碼時有衝突， Git 根本就不會讓用戶覆蓋他人代碼，它直接駁回第二個人的提交操作。這就等於告訴提交者，你所作的修訂無法通過快近（fast-forward）來合併，你必須先拉取最新數據下來，手工解決衝突合併後，才能繼續推送新的提交。絕大多數人都熟悉和瞭解這種模式的工作方式，所以使用也非常廣泛。

### 集成管理員工作流 ###

由於 Git 允許使用多個遠程倉庫，開發者便可以建立自己的公共倉庫，往裡面寫數據並共享給他人，而同時又可以從別人的倉庫中提取他們的更新過來。這種情形通常都會有個代表著官方發佈的項目倉庫（blessed repository），開發者們由此倉庫克隆出一個自己的公共倉庫（developer public），然後將自己的提交推送上去，請求官方倉庫的維護者拉取更新合併到主項目。維護者在自己的本地也有個克隆倉庫（integration manager），他可以將你的公共倉庫作為遠程倉庫添加進來，經過測試無誤後合併到主幹分支，然後再推送到官方倉庫。工作流程看起來就像圖 5-2 所示：

1. 項目維護者可以推送數據到公共倉庫 blessed repository。
2. 貢獻者克隆此倉庫，修訂或編寫新代碼。
3. 貢獻者推送數據到自己的公共倉庫 developer public。
4. 貢獻者給維護者發送郵件，請求拉取自己的最新修訂。
5. 維護者在自己本地的 integration manger 倉庫中，將貢獻者的倉庫加為遠程倉庫，合併更新並做測試。
6. 維護者將合併後的更新推送到主倉庫 blessed repository。

Insert 18333fig0502.png 
圖 5-2. 集成管理員工作流

在 GitHub 網站上使用得最多的就是這種工作流。人們可以複製（fork 亦即克隆）某個項目到自己的列表中，成為自己的公共倉庫。隨後將自己的更新提交到這個倉庫，所有人都可以看到你的每次更新。這麼做最主要的優點在於，你可以按照自己的節奏繼續工作，而不必等待維護者處理你提交的更新；而維護者也可以按照自己的節奏，任何時候都可以過來處理接納你的貢獻。

### 司令官與副官工作流 ###

這其實是上一種工作流的變體。一般超大型的項目才會用到這樣的工作方式，像是擁有數百協作開發者的 Linux 內核項目就是如此。各個集成管理員分別負責集成項目中的特定部分，所以稱為副官（lieutenant）。而所有這些集成管理員頭上還有一位負責統籌的總集成管理員，稱為司令官（dictator）。司令官維護的倉庫用於提供所有協作者拉取最新集成的項目代碼。整個流程看起來如圖 5-3 所示：

1. 一般的開發者在自己的特性分支上工作，並不定期地根據主幹分支（dectator 上的 master）衍合。
2. 副官（lieutenant）將普通開發者的特性分支合併到自己的 master 分支中。
3. 司令官（dictator）將所有副官的 master 分支併入自己的 master 分支。
4. 司令官（dictator）將集成後的 master 分支推送到共享倉庫 blessed repository 中，以便所有其他開發者以此為基礎進行衍合。

Insert 18333fig0503.png  
圖 5-3. 司令官與副官工作流

這種工作流程並不常用，只有當項目極為龐雜，或者需要多級別管理時，才會體現出優勢。利用這種方式，項目總負責人（即司令官）可以把大量分散的集成工作委託給不同的小組負責人分別處理，最後再統籌起來，如此各人的職責清晰明確，也不易出錯（譯註：此乃分而治之）。

以上介紹的是常見的分佈式系統可以應用的工作流程，當然不止於 Git。在實際的開發工作中，你可能會遇到各種為了滿足特定需求而有所變化的工作方式。我想現在你應該已經清楚，接下來自己需要用哪種方式開展工作了。下節我還會再舉些例子，看看各式工作流中的每個角色具體應該如何操作。

## 為項目作貢獻 ##

接下來，我們來學習一下作為項目貢獻者，會有哪些常見的工作模式。

不過要說清楚整個協作過程真的很難，Git 如此靈活，人們的協作方式便可以各式各樣，沒有固定不變的範式可循，而每個項目的具體情況又多少會有些不同，比如說參與者的規模，所選擇的工作流程，每個人的提交權限，以及 Git 以外貢獻等等，都會影響到具體操作的細節。

首當其衝的是參與者規模。項目中有多少開發者是經常提交代碼的？經常又是多久呢？大多數兩至三人的小團隊，一天大約只有幾次提交，如果不是什麼熱門項目的話就更少了。可要是在大公司裡，或者大項目中，參與者可以多到上千，每天都會有十幾個上百個補丁提交上來。這種差異帶來的影響是顯著的，越是多的人參與進來，就越難保證每次合併正確無誤。你正在工作的代碼，可能會因為合併進來其他人的更新而變得過時，甚至受創無法運行。而已經提交上去的更新，也可能在等著審核合併的過程中變得過時。那麼，我們該怎樣做才能確保代碼是最新的，提交的補丁也是可用的呢？

接下來便是項目改採用的工作流。是集中式的，每個開發者都具有等同的寫權限？項目是否有專人負責檢查所有補丁？是不是所有補丁都做過同行複閱（peer-review）再通過審核的？你是否參與審核過程？如果使用副官系統，那你是不是限定於只能向此副官提交？

還有你的提交權限。有或沒有向主項目提交更新的權限，結果完全不同，直接決定最終採用怎樣的工作流。如果不能直接提交更新，那該如何貢獻自己的代碼呢？是不是該有個什麼策略？你每次貢獻代碼會有多少量？提交頻率呢？

所有以上這些問題都會或多或少影響到最終採用的工作流。接下來，我會在一系列由簡入繁的具體用例中，逐一闡述。此後在實踐時，應該可以借鑑這裡的例子，略作調整，以滿足實際需要構建自己的工作流。

### 提交指南 ###

開始分析特定用例之前，先來瞭解下如何撰寫提交說明。一份好的提交指南可以幫助協作者更輕鬆更有效地配合。Git 項目本身就提供了一份文檔（Git 項目源代碼目錄中 `Documentation/SubmittingPatches`），列數了大量提示，從如何編撰提交說明到提交補丁，不一而足。

首先，請不要在更新中提交多餘的白字符（whitespace）。Git 有種檢查此類問題的方法，在提交之前，先運行 `git diff --check`，會把可能的多餘白字符修正列出來。下面的示例，我已經把終端中顯示為紅色的白字符用 `X` 替換掉：

	$ git diff --check
	lib/simplegit.rb:5: trailing whitespace.
	+    @git_dir = File.expand_path(git_dir)XX
	lib/simplegit.rb:7: trailing whitespace.
	+ XXXXXXXXXXX
	lib/simplegit.rb:26: trailing whitespace.
	+    def command(git_cmd)XXXX

這樣在提交之前你就可以看到這類問題，及時解決以免困擾其他開發者。

接下來，請將每次提交限定於完成一次邏輯功能。並且可能的話，適當地分解為多次小更新，以便每次小型提交都更易於理解。請不要在週末窮追猛打一次性解決五個問題，而最後拖到週一再提交。就算是這樣也請儘可能利用暫存區域，將之前的改動分解為每次修復一個問題，再分別提交和加注說明。如果針對兩個問題改動的是同一個文件，可以試試看 `git add --patch` 的方式將部分內容置入暫存區域（我們會在第六章再詳細介紹）。無論是五次小提交還是混雜在一起的大提交，最終分支末端的項目快照應該還是一樣的，但分解開來之後，更便於其他開發者複閱。這麼做也方便自己將來取消某個特定問題的修復。我們將在第六章介紹一些重寫提交歷史，同暫存區域交互的技巧和工具，以便最終得到一個乾淨有意義，且易於理解的提交歷史。

最後需要謹記的是提交說明的撰寫。寫得好可以讓大家協作起來更輕鬆。一般來說，提交說明最好限制在一行以內，50 個字符以下，簡明扼要地描述更新內容，空開一行後，再展開詳細註解。Git 項目本身需要開發者撰寫詳盡註解，包括本次修訂的因由，以及前後不同實現之間的比較，我們也該借鑑這種做法。另外，提交說明應該用祈使現在式語態，比如，不要說成 「I added tests for」 或 「Adding tests for」 而應該用 「Add tests for」。下面是來自 tpope.net 的 Tim Pope 原創的提交說明格式模版，供參考：

	本次更新的簡要描述（50 個字符以內）

	如果必要，此處展開詳盡闡述。段落寬度限定在 72 個字符以內。
	某些情況下，第一行的簡要描述將用作郵件標題，其餘部分作為郵件正文。
	其間的空行是必要的，以區分兩者（當然沒有正文另當別論）。
	如果並在一起，rebase 這樣的工具就可能會迷惑。
	
	另起空行後，再進一步補充其他說明。

	 - 可以使用這樣的條目列舉式。

	 - 一般以單個空格緊跟短劃線或者星號作為每項條目的起始符。每個條目間用一空行隔開。
	   不過這裡按自己項目的約定，可以略作變化。

如果你的提交說明都用這樣的格式來書寫，好多事情就可以變得十分簡單。Git 項目本身就是這樣要求的，我強烈建議你到 Git 項目倉庫下運行 `git log --no-merges` 看看，所有提交歷史的說明是怎樣撰寫的。（譯註：如果現在還沒有克隆 git 項目源代碼，是時候 `git clone git://git.kernel.org/pub/scm/git/git.git` 了。）

為簡單起見，在接下來的例子（及本書隨後的所有演示）中，我都不會用這種格式，而使用 `-m` 選項提交 `git commit`。不過請還是按照我之前講的做，別學我這裡偷懶的方式。

### 私有的小型團隊 ###

我們從最簡單的情況開始，一個私有項目，與你一起協作的還有另外一到兩位開發者。這裡說私有，是指源代碼不公開，其他人無法訪問項目倉庫。而你和其他開發者則都具有推送數據到倉庫的權限。

這種情況下，你們可以用 Subversion 或其他集中式版本控制系統類似的工作流來協作。你仍然可以得到 Git 帶來的其他好處：離線提交，快速分支與合併等等，但工作流程還是差不多的。主要區別在於，合併操作發生在客戶端而非服務器上。讓我們來看看，兩個開發者一起使用同一個共享倉庫，會發生些什麼。第一個人，John，克隆了倉庫，作了些更新，在本地提交。（下面的例子中省略了常規提示，用 `...` 代替以節約版面。）

	# John's Machine
	$ git clone john@githost:simplegit.git
	Initialized empty Git repository in /home/john/simplegit/.git/
	...
	$ cd simplegit/
	$ vim lib/simplegit.rb 
	$ git commit -am 'removed invalid default value'
	[master 738ee87] removed invalid default value
	 1 files changed, 1 insertions(+), 1 deletions(-)

第二個開發者，Jessica，一樣這麼做：克隆倉庫，提交更新：

	# Jessica's Machine
	$ git clone jessica@githost:simplegit.git
	Initialized empty Git repository in /home/jessica/simplegit/.git/
	...
	$ cd simplegit/
	$ vim TODO 
	$ git commit -am 'add reset task'
	[master fbff5bc] add reset task
	 1 files changed, 1 insertions(+), 0 deletions(-)

現在，Jessica 將她的工作推送到服務器上：

	# Jessica's Machine
	$ git push origin master
	...
	To jessica@githost:simplegit.git
	   1edee6b..fbff5bc  master -> master

John 也嘗試推送自己的工作上去：

	# John's Machine
	$ git push origin master
	To john@githost:simplegit.git
	 ! [rejected]        master -> master (non-fast forward)
	error: failed to push some refs to 'john@githost:simplegit.git'

John 的推送操作被駁回，因為 Jessica 已經推送了新的數據上去。請注意，特別是你用慣了 Subversion 的話，這裡其實修改的是兩個文件，而不是同一個文件的同一個地方。Subversion 會在服務器端自動合併提交上來的更新，而 Git 則必須先在本地合併後才能推送。於是，John 不得不先把 Jessica 的更新拉下來：

	$ git fetch origin
	...
	From john@githost:simplegit
	 + 049d078...fbff5bc master     -> origin/master

此刻，John 的本地倉庫如圖 5-4 所示：

Insert 18333fig0504.png 
圖 5-4. John 的倉庫歷史

雖然 John 下載了 Jessica 推送到服務器的最近更新（fbff5），但目前只是 `origin/master` 指針指向它，而當前的本地分支 `master` 仍然指向自己的更新（738ee），所以需要先把她的提交合併過來，才能繼續推送數據：

	$ git merge origin/master
	Merge made by recursive.
	 TODO |    1 +
	 1 files changed, 1 insertions(+), 0 deletions(-)

還好，合併過程非常順利，沒有衝突，現在 John 的提交歷史如圖 5-5 所示：

Insert 18333fig0505.png 
圖 5-5. 合併 origin/master 後 John 的倉庫歷史

現在，John 應該再測試一下代碼是否仍然正常工作，然後將合併結果（72bbc）推送到服務器上：

	$ git push origin master
	...
	To john@githost:simplegit.git
	   fbff5bc..72bbc59  master -> master

最終，John 的提交歷史變為圖 5-6 所示：

Insert 18333fig0506.png 
圖 5-6. 推送後 John 的倉庫歷史

而在這段時間，Jessica 已經開始在另一個特性分支工作了。她創建了 `issue54` 並提交了三次更新。她還沒有下載 John 提交的合併結果，所以提交歷史如圖 5-7 所示：

Insert 18333fig0507.png 
圖 5-7. Jessica 的提交歷史

Jessica 想要先和服務器上的數據同步，所以先下載數據：

	# Jessica's Machine
	$ git fetch origin
	...
	From jessica@githost:simplegit
	   fbff5bc..72bbc59  master     -> origin/master

於是 Jessica 的本地倉庫歷史多出了 John 的兩次提交（738ee 和 72bbc），如圖 5-8 所示：

Insert 18333fig0508.png 
圖 5-8. 獲取 John 的更新之後 Jessica 的提交歷史

此時，Jessica 在特性分支上的工作已經完成，但她想在推送數據之前，先確認下要並進來的數據究竟是什麼，於是運行 `git log` 查看：

	$ git log --no-merges origin/master ^issue54
	commit 738ee872852dfaa9d6634e0dea7a324040193016
	Author: John Smith <jsmith@example.com>
	Date:   Fri May 29 16:01:27 2009 -0700

	    removed invalid default value

現在，Jessica 可以將特性分支上的工作並到 `master` 分支，然後再併入 John 的工作（`origin/master`）到自己的 `master` 分支，最後再推送回服務器。當然，得先切回主分支才能集成所有數據：

	$ git checkout master
	Switched to branch "master"
	Your branch is behind 'origin/master' by 2 commits, and can be fast-forwarded.

要合併 `origin/master` 或 `issue54` 分支，誰先誰後都沒有關係，因為它們都在上游（upstream）（譯註：想像分叉的更新像是匯流成河的源頭，所以上游 upstream 是指最新的提交），所以無所謂先後順序，最終合併後的內容快照都是一樣的，而僅是提交歷史看起來會有些先後差別。Jessica 選擇先合併 `issue54`：

	$ git merge issue54
	Updating fbff5bc..4af4298
	Fast forward
	 README           |    1 +
	 lib/simplegit.rb |    6 +++++-
	 2 files changed, 6 insertions(+), 1 deletions(-)

正如所見，沒有衝突發生，僅是一次簡單快進。現在 Jessica 開始合併 John 的工作（`origin/master`）：

	$ git merge origin/master
	Auto-merging lib/simplegit.rb
	Merge made by recursive.
	 lib/simplegit.rb |    2 +-
	 1 files changed, 1 insertions(+), 1 deletions(-)

所有的合併都非常乾淨。現在 Jessica 的提交歷史如圖 5-9 所示：

Insert 18333fig0509.png 
圖 5-9. 合併 John 的更新後 Jessica 的提交歷史

現在 Jessica 已經可以在自己的 `master` 分支中訪問 `origin/master` 的最新改動了，所以她應該可以成功推送最後的合併結果到服務器上（假設 John 此時沒再推送新數據上來）：

	$ git push origin master
	...
	To jessica@githost:simplegit.git
	   72bbc59..8059c15  master -> master

至此，每個開發者都提交了若干次，且成功合併了對方的工作成果，最新的提交歷史如圖 5-10 所示：

Insert 18333fig0510.png 
圖 5-10. Jessica 推送數據後的提交歷史

以上就是最簡單的協作方式之一：先在自己的特性分支中工作一段時間，完成後合併到自己的 `master` 分支；然後下載合併 `origin/master` 上的更新（如果有的話），再推回遠程服務器。一般的協作流程如圖 5-11 所示：

Insert 18333fig0511.png 
圖 5-11. 多用戶共享倉庫協作方式的一般工作流程時序

### 私有團隊間協作 ###

現在我們來看更大一點規模的私有團隊協作。如果有幾個小組分頭負責若干特性的開發和集成，那他們之間的協作過程是怎樣的。

假設 John 和 Jessica 一起負責開發某項特性 A，而同時 Jessica 和 Josie 一起負責開發另一項功能 B。公司使用典型的集成管理員式工作流，每個組都有一名管理員負責集成本組代碼，及更新項目主倉庫的 `master` 分支。所有開發都在代表小組的分支上進行。

讓我們跟隨 Jessica 的視角看看她的工作流程。她參與開發兩項特性，同時和不同小組的開發者一起協作。克隆生成本地倉庫後，她打算先著手開發特性 A。於是創建了新的 `featureA` 分支，繼而編寫代碼：

	# Jessica's Machine
	$ git checkout -b featureA
	Switched to a new branch "featureA"
	$ vim lib/simplegit.rb
	$ git commit -am 'add limit to log function'
	[featureA 3300904] add limit to log function
	 1 files changed, 1 insertions(+), 1 deletions(-)

此刻，她需要分享目前的進展給 John，於是她將自己的 `featureA` 分支提交到服務器。由於 Jessica 沒有權限推送數據到主倉庫的 `master` 分支（只有集成管理員有此權限），所以只能將此分支推上去同 John 共享協作：

	$ git push origin featureA
	...
	To jessica@githost:simplegit.git
	 * [new branch]      featureA -> featureA

Jessica 發郵件給 John 讓他上來看看 `featureA` 分支上的進展。在等待他的反饋之前，Jessica 決定繼續工作，和 Josie 一起開發 `featureB` 上的特性 B。當然，先創建此分支，分叉點以服務器上的 `master` 為起點：

	# Jessica's Machine
	$ git fetch origin
	$ git checkout -b featureB origin/master
	Switched to a new branch "featureB"

隨後，Jessica 在 `featureB` 上提交了若干更新：

	$ vim lib/simplegit.rb
	$ git commit -am 'made the ls-tree function recursive'
	[featureB e5b0fdc] made the ls-tree function recursive
	 1 files changed, 1 insertions(+), 1 deletions(-)
	$ vim lib/simplegit.rb
	$ git commit -am 'add ls-files'
	[featureB 8512791] add ls-files
	 1 files changed, 5 insertions(+), 0 deletions(-)

現在 Jessica 的更新歷史如圖 5-12 所示：

Insert 18333fig0512.png 
圖 5-12. Jessica 的更新歷史

Jessica 正準備推送自己的進展上去，卻收到 Josie 的來信，說是她已經將自己的工作推到服務器上的 `featureBee` 分支了。這樣，Jessica 就必須先將 Josie 的代碼合併到自己本地分支中，才能再一起推送回服務器。她用 `git fetch` 下載 Josie 的最新代碼：

	$ git fetch origin
	...
	From jessica@githost:simplegit
	 * [new branch]      featureBee -> origin/featureBee

然後 Jessica 使用 `git merge` 將此分支合併到自己分支中：

	$ git merge origin/featureBee
	Auto-merging lib/simplegit.rb
	Merge made by recursive.
	 lib/simplegit.rb |    4 ++++
	 1 files changed, 4 insertions(+), 0 deletions(-)

合併很順利，但另外有個小問題：她要推送自己的 `featureB` 分支到服務器上的 `featureBee` 分支上去。當然，她可以使用冒號（:）格式指定目標分支：

	$ git push origin featureB:featureBee
	...
	To jessica@githost:simplegit.git
	   fba9af8..cd685d1  featureB -> featureBee

我們稱此為_refspec_。更多有關於 Git refspec 的討論和使用方式會在第九章作詳細闡述。

接下來，John 發郵件給 Jessica 告訴她，他看了之後作了些修改，已經推回服務器 `featureA` 分支，請她過目下。於是 Jessica 運行 `git fetch` 下載最新數據：

	$ git fetch origin
	...
	From jessica@githost:simplegit
	   3300904..aad881d  featureA   -> origin/featureA

接下來便可以用 `git log` 查看更新了些什麼：

	$ git log origin/featureA ^featureA
	commit aad881d154acdaeb2b6b18ea0e827ed8a6d671e6
	Author: John Smith <jsmith@example.com>
	Date:   Fri May 29 19:57:33 2009 -0700

	    changed log output to 30 from 25

最後，她將 John 的工作合併到自己的 `featureA` 分支中：

	$ git checkout featureA
	Switched to branch "featureA"
	$ git merge origin/featureA
	Updating 3300904..aad881d
	Fast forward
	 lib/simplegit.rb |   10 +++++++++-
	1 files changed, 9 insertions(+), 1 deletions(-)

Jessica 稍做一番修整後同步到服務器：

	$ git commit -am 'small tweak'
	[featureA ed774b3] small tweak
	 1 files changed, 1 insertions(+), 1 deletions(-)
	$ git push origin featureA
	...
	To jessica@githost:simplegit.git
	   3300904..ed774b3  featureA -> featureA

現在的 Jessica 提交歷史如圖 5-13 所示：

Insert 18333fig0513.png 
圖 5-13. 在特性分支中提交更新後的提交歷史

現在，Jessica，Josie 和 John 通知集成管理員服務器上的 `featureA` 及 `featureBee` 分支已經準備好，可以併入主線了。在管理員完成集成工作後，主分支上便多出一個新的合併提交（5399e），用 fetch 命令更新到本地後，提交歷史如圖 5-14 所示：

Insert 18333fig0514.png 
圖 5-14. 合併特性分支後的 Jessica 提交歷史

許多開發小組改用 Git 就是因為它允許多個小組間並行工作，而在稍後恰當時機再行合併。通過共享遠程分支的方式，無需干擾整體項目代碼便可以開展工作，因此使用 Git 的小型團隊間協作可以變得非常靈活自由。以上工作流程的時序如圖 5-15 所示：

Insert 18333fig0515.png 
圖 5-15. 團隊間協作工作流程基本時序

### 公開的小型項目 ###

上面說的是私有項目協作，但要給公開項目作貢獻，情況就有些不同了。因為你沒有直接更新主倉庫分支的權限，得尋求其它方式把工作成果交給項目維護人。下面會介紹兩種方法，第一種使用 git 託管服務商提供的倉庫複製功能，一般稱作 fork，比如 repo.or.cz 和 GitHub 都支持這樣的操作，而且許多項目管理員都希望大家使用這樣的方式。另一種方法是通過電子郵件寄送文件補丁。

但不管哪種方式，起先我們總需要克隆原始倉庫，而後創建特性分支開展工作。基本工作流程如下：

	$ git clone (url)
	$ cd project
	$ git checkout -b featureA
	$ (work)
	$ git commit
	$ (work)
	$ git commit

你可能想到用 `rebase -i` 將所有更新先變作單個提交，又或者想重新安排提交之間的差異補丁，以方便項目維護者審閱 -- 有關交互式衍合操作的細節見第六章。

在完成了特性分支開發，提交給項目維護者之前，先到原始項目的頁面上點擊「Fork」按鈕，創建一個自己可寫的公共倉庫（譯註：即下面的 url 部分，參照後續的例子，應該是 `git://githost/simplegit.git`）。然後將此倉庫添加為本地的第二個遠端倉庫，姑且稱為 `myfork`：

	$ git remote add myfork (url)

你需要將本地更新推送到這個倉庫。要是將遠端 master 合併到本地再推回去，還不如把整個特性分支推上去來得乾脆直接。而且，假若項目維護者未採納你的貢獻的話（不管是直接合併還是 cherry pick），都不用回退（rewind）自己的 master 分支。但若維護者合併或 cherry-pick 了你的工作，最後總還可以從他們的更新中同步這些代碼。好吧，現在先把 featureA 分支整個推上去：

	$ git push myfork featureA

然後通知項目管理員，讓他來抓取你的代碼。通常我們把這件事叫做 pull request。可以直接用 GitHub 等網站提供的 「pull request」 按鈕自動發送請求通知；或手工把 `git request-pull` 命令輸出結果電郵給項目管理員。

`request-pull` 命令接受兩個參數，第一個是本地特性分支開始前的原始分支，第二個是請求對方來抓取的 Git 倉庫 URL（譯註：即下面 `myfork` 所指的，自己可寫的公共倉庫）。比如現在Jessica 準備要給 John 發一個 pull requst，她之前在自己的特性分支上提交了兩次更新，並把分支整個推到了服務器上，所以運行該命令會看到：

	$ git request-pull origin/master myfork
	The following changes since commit 1edee6b1d61823a2de3b09c160d7080b8d1b3a40:
	  John Smith (1):
	        added a new function

	are available in the git repository at:

	  git://githost/simplegit.git featureA

	Jessica Smith (2):
	      add limit to log function
	      change log output to 30 from 25

	 lib/simplegit.rb |   10 +++++++++-
	 1 files changed, 9 insertions(+), 1 deletions(-)

輸出的內容可以直接發郵件給管理者，他們就會明白這是從哪次提交開始旁支出去的，該到哪裡去抓取新的代碼，以及新的代碼增加了哪些功能等等。

像這樣隨時保持自己的 `master` 分支和官方 `origin/master` 同步，並將自己的工作限制在特性分支上的做法，既方便又靈活，採納和丟棄都輕而易舉。就算原始主幹發生變化，我們也能重新衍合提供新的補丁。比如現在要開始第二項特性的開發，不要在原來已推送的特性分支上繼續，還是按原始 `master` 開始：

	$ git checkout -b featureB origin/master
	$ (work)
	$ git commit
	$ git push myfork featureB
	$ (email maintainer)
	$ git fetch origin

現在，A、B 兩個特性分支各不相擾，如同竹筒裡的兩顆豆子，隊列中的兩個補丁，你隨時都可以分別從頭寫過，或者衍合，或者修改，而不用擔心特性代碼的交叉混雜。如圖 5-16 所示：


Insert 18333fig0516.png 
圖 5-16. featureB 以後的提交歷史

假設項目管理員接納了許多別人提交的補丁後，準備要採納你提交的第一個分支，卻發現因為代碼基準不一致，合併工作無法正確乾淨地完成。這就需要你再次衍合到最新的 `origin/master`，解決相關衝突，然後重新提交你的修改：

	$ git checkout featureA
	$ git rebase origin/master
	$ git push -f myfork featureA

自然，這會重寫提交歷史，如圖 5-17 所示：

Insert 18333fig0517.png 
圖 5-17. featureA 重新衍合後的提交歷史

注意，此時推送分支必須使用 `-f` 選項（譯註：表示 force，不作檢查強制重寫）替換遠程已有的 `featureA` 分支，因為新的 commit 並非原來的後續更新。當然你也可以直接推送到另一個新的分支上去，比如稱作 `featureAv2`。

再考慮另一種情形：管理員看過第二個分支後覺得思路新穎，但想請你改下具體實現。我們只需以當前 `origin/master` 分支為基準，開始一個新的特性分支 `featureBv2`，然後把原來的 `featureB` 的更新拿過來，解決衝突，按要求重新實現部分代碼，然後將此特性分支推送上去：

	$ git checkout -b featureBv2 origin/master
	$ git merge --no-commit --squash featureB
	$ (change implementation)
	$ git commit
	$ git push myfork featureBv2

這裡的 `--squash` 選項將目標分支上的所有更改全拿來應用到當前分支上，而 `--no-commit` 選項告訴 Git 此時無需自動生成和記錄（合併）提交。這樣，你就可以在原來代碼基礎上，繼續工作，直到最後一起提交。

好了，現在可以請管理員抓取 `featureBv2` 上的最新代碼了，如圖 5-18 所示：

Insert 18333fig0518.png 
圖 5-18. featureBv2 之後的提交歷史

### 公開的大型項目 ###

許多大型項目都會立有一套自己的接受補丁流程，你應該注意下其中細節。但多數項目都允許通過開發者郵件列表接受補丁，現在我們來看具體例子。

整個工作流程類似上面的情形：為每個補丁創建獨立的特性分支，而不同之處在於如何提交這些補丁。不需要創建自己可寫的公共倉庫，也不用將自己的更新推送到自己的服務器，你只需將每次提交的差異內容以電子郵件的方式依次發送到郵件列表中即可。

	$ git checkout -b topicA
	$ (work)
	$ git commit
	$ (work)
	$ git commit

如此一番後，有了兩個提交要發到郵件列表。我們可以用 `git format-patch` 命令來生成 mbox 格式的文件然後作為附件發送。每個提交都會封裝為一個 `.patch` 後綴的 mbox 文件，但其中只包含一封郵件，郵件標題就是提交消息（譯註：額外有前綴，看例子），郵件內容包含補丁正文和 Git 版本號。這種方式的妙處在於接受補丁時仍可保留原來的提交消息，請看接下來的例子：

	$ git format-patch -M origin/master
	0001-add-limit-to-log-function.patch
	0002-changed-log-output-to-30-from-25.patch

`format-patch` 命令依次創建補丁文件，並輸出文件名。上面的 `-M` 選項允許 Git 檢查是否有對文件重命名的提交。我們來看看補丁文件的內容：

	$ cat 0001-add-limit-to-log-function.patch 
	From 330090432754092d704da8e76ca5c05c198e71a8 Mon Sep 17 00:00:00 2001
	From: Jessica Smith <jessica@example.com>
	Date: Sun, 6 Apr 2008 10:17:23 -0700
	Subject: [PATCH 1/2] add limit to log function

	Limit log functionality to the first 20

	---
	 lib/simplegit.rb |    2 +-
	 1 files changed, 1 insertions(+), 1 deletions(-)

	diff --git a/lib/simplegit.rb b/lib/simplegit.rb
	index 76f47bc..f9815f1 100644
	--- a/lib/simplegit.rb
	+++ b/lib/simplegit.rb
	@@ -14,7 +14,7 @@ class SimpleGit
	   end

	   def log(treeish = 'master')
	-    command("git log #{treeish}")
	+    command("git log -n 20 #{treeish}")
	   end

	   def ls_tree(treeish = 'master')
	-- 
	1.6.2.rc1.20.g8c5b.dirty

如果有額外信息需要補充，但又不想放在提交消息中說明，可以編輯這些補丁文件，在第一個 `---` 行之前添加說明，但不要修改下面的補丁正文，比如例子中的 `Limit log functionality to the first 20` 部分。這樣，其它開發者能閱讀，但在採納補丁時不會將此合併進來。

你可以用郵件客戶端軟件發送這些補丁文件，也可以直接在命令行發送。有些所謂智能的郵件客戶端軟件會自作主張幫你調整格式，所以粘貼補丁到郵件正文時，有可能會丟失換行符和若干空格。Git 提供了一個通過 IMAP 發送補丁文件的工具。接下來我會演示如何通過 Gmail 的 IMAP 服務器發送。另外，在 Git 源代碼中有個 `Documentation/SubmittingPatches` 文件，可以仔細讀讀，看看其它郵件程序的相關導引。

首先在 `~/.gitconfig` 文件中配置 imap 項。每個選項都可用 `git config` 命令分別設置，當然直接編輯文件添加以下內容更便捷：

	[imap]
	  folder = "[Gmail]/Drafts"
	  host = imaps://imap.gmail.com
	  user = user@gmail.com
	  pass = p4ssw0rd
	  port = 993
	  sslverify = false

如果你的 IMAP 服務器沒有啟用 SSL，就無需配置最後那兩行，並且 host 應該以 `imap://` 開頭而不再是有 `s` 的 `imaps://`。保存配置文件後，就能用 `git send-email` 命令把補丁作為郵件依次發送到指定的 IMAP 服務器上的文件夾中（譯註：這裡就是 Gmail 的 `[Gmail]/Drafts` 文件夾。但如果你的語言設置不是英文，此處的文件夾 Drafts 字樣會變為對應的語言。）：

	$ git send-email *.patch
	0001-added-limit-to-log-function.patch
	0002-changed-log-output-to-30-from-25.patch
	Who should the emails appear to be from? [Jessica Smith <jessica@example.com>] 
	Emails will be sent from: Jessica Smith <jessica@example.com>
	Who should the emails be sent to? jessica@example.com
	Message-ID to be used as In-Reply-To for the first email? y

接下來，Git 會根據每個補丁依次輸出類似下面的日誌：

	(mbox) Adding cc: Jessica Smith <jessica@example.com> from 
	  \line 'From: Jessica Smith <jessica@example.com>'
	OK. Log says:
	Sendmail: /usr/sbin/sendmail -i jessica@example.com
	From: Jessica Smith <jessica@example.com>
	To: jessica@example.com
	Subject: [PATCH 1/2] added limit to log function
	Date: Sat, 30 May 2009 13:29:15 -0700
	Message-Id: <1243715356-61726-1-git-send-email-jessica@example.com>
	X-Mailer: git-send-email 1.6.2.rc1.20.g8c5b.dirty
	In-Reply-To: <y>
	References: <y>

	Result: OK

最後，到 Gmail 上打開 Drafts 文件夾，編輯這些郵件，修改收件人地址為郵件列表地址，另外給要抄送的人也加到 Cc 列表中，最後發送。

### 小結 ###

本節主要介紹了常見 Git 項目協作的工作流程，還有一些幫助處理這些工作的命令和工具。接下來我們要看看如何維護 Git 項目，並成為一個合格的項目管理員，或是集成經理。

## 項目的管理 ##

除了貢獻代碼，你還需要知道如何維護一個項目。這包括處理別人用`format-patch`命令產生的補丁和集成遠端代碼庫的更新等等。但無論你是維護代碼庫，還是幫助別人審核、改善收到的補丁，都需要瞭解如何以最簡潔的方式蒐集貢獻者的工作並和他們保持長期的合作關係。

### 使用特性分支進行工作 ###

通常在加入新的代碼時，先在特性分支上做個試驗是個好的主意——建立一個專門用來試驗這些代碼的臨時分支。這樣就能很容易的做出一個獨立的補丁，如果它不工作你還可以先放下它，等到有時間的時候再重新撿起來。創建的分支可以用相關的主題關鍵字命名，比如 `ruby_client` 或者其它類似的描述性詞語，以幫助將來有需要的時候回憶起來。Git本身也把分支名稱分置於不同的命名空間之下，比如 `sc/ruby_client`，`sc` 就是貢獻這些代碼的人名的縮寫。

現在，在當前主幹分支的基礎上新建一個臨時分支：

        $ git branch sc/ruby_client master

另外，如果你希望立即轉到分支上去工作，可以使用`checkout -b`：

        $ git checkout -b sc/ruby_client master

好了，現在已經準備妥當，可以試著將別人貢獻的代碼合併進來了。之後評估一下有沒有問題，最後再決定是不是真的要併入主幹。

### 採納來自郵件的補丁 ###

如果你收到了一個通過E-mail發來的補丁，你應該先把它應用到一個特性分支上進行評估。有兩種應用補丁的方法：`git apply` 或者 `git am`。

#### 使用 apply 命令應用補丁 ####

如果你收到某人使用`git diff`或者Unix的 `diff`命令生成的補丁，可以使用`git apply`命令來應用它們。假定你把補丁保存在`/tmp/patch-ruby-client.patch`，可以這樣應用補丁：

	$ git apply /tmp/patch-ruby-client.patch

它會修改你工作目錄下的文件。效果基本與運行`patch -p1`命令打補丁一樣，但是它更加嚴格並且不會出現混亂。如果它是`git diff`格式描述的補丁，還會處理文件的添加，刪除和重命名等操作,`patch`命令根本做不到這一點。最後，`git apply`是一個"apply all or abort all"模式的命令,要麼所有補丁都打上去，要麼都沒有，但是`patch`命令會部分的應用補丁文件，可能導致你的工作變得不完整且非常混亂。`git apply`比`patch`嚴格很多，但是它不會自動commit——運行它之後，你必須手動stage，然後commit。

你還可以在真正打補丁之前檢驗一下——使用`git apply --check`來查看補丁是否能夠乾淨的應用。

	$ git apply --check 0001-seeing-if-this-helps-the-gem.patch 
	error: patch failed: ticgit.gemspec:1
	error: ticgit.gemspec: patch does not apply

如果沒有任何輸出，表示補丁被幹淨的應用了。如果檢驗失敗，這個命令將返回一個非零的狀態，需要的話你可以把它用在腳本裡。

#### 使用 am 命令應用補丁 ####

如果貢獻者也使用Git，並且擅長於使用`format-patch`來製作他的補丁，那麼你的工作將會更加輕鬆，因為這樣的補丁裡包含了作者信息和commit信息。請鼓勵你的貢獻者們使用`format-patch`代替`diff`。另外，`git apply`應該只用在老式補丁上。

應用一個由`format-patch`製作的新式補丁,應該使用`git am`。技術上說，`git am`讀取mbox文件——它是一個簡單的，純文本格式的文件。它保存著一或多個e-mail信息，就像這樣：

	From 330090432754092d704da8e76ca5c05c198e71a8 Mon Sep 17 00:00:00 2001
	From: Jessica Smith <jessica@example.com>
	Date: Sun, 6 Apr 2008 10:17:23 -0700
	Subject: [PATCH 1/2] add limit to log function

	Limit log functionality to the first 20

這是`format-patch`命令輸出的開頭幾行，也是一個有效的mbox電郵格式。如果有人用`git send-email`給你發了一個補丁，你可以將它下載到本地，然後運行`git am`命令來應用這個補丁。如果你的E-mail客戶端能夠將多個e-mail文件保存到一起，就可以這樣做——把多個相關補丁保存到一個文件裡，然後使用`git am`一次應用所有的補丁。

同樣的，如果某人上傳了一個由`format-patch`製作的補丁文件讓大家投票決定是否採納時，你可以將該文件保存到本地並使用`git am`應用這個補丁，之後開始評估它：

	$ git am 0001-limit-log-function.patch 
	Applying: add limit to log function

你會看到它被幹淨的應用並自動建立了新的commit。作者信息取自e-mail頭部的`From`和`Date`字段，commit的信息取自`Subject`字段和正文（補丁之前的內容）。舉個例子，如果這個補丁來自我剛剛展示的那個mbox郵件，commit信息看起來就像這樣：

	$ git log --pretty=fuller -1
	commit 6c5e70b984a60b3cecd395edd5b48a7575bf58e0
	Author:     Jessica Smith <jessica@example.com>
	AuthorDate: Sun Apr 6 10:17:23 2008 -0700
	Commit:     Scott Chacon <schacon@gmail.com>
	CommitDate: Thu Apr 9 09:19:06 2009 -0700

	   add limit to log function

	   Limit log functionality to the first 20
`Commit`部分顯示了提交者的信息和提交日期。`Author`部分顯示了原作者信息和補丁的創建時間。

有時,補丁不能被幹淨的應用。這可能是你的主幹分支和補丁的基礎分支相差太遠了，也可能是這個補丁依賴於一些你還沒加入的補丁。這種情況下，`git am`會報錯並詢問你該怎麼做：

	$ git am 0001-seeing-if-this-helps-the-gem.patch 
	Applying: seeing if this helps the gem
	error: patch failed: ticgit.gemspec:1
	error: ticgit.gemspec: patch does not apply
	Patch failed at 0001.
	When you have resolved this problem run "git am --resolved".
	If you would prefer to skip this patch, instead run "git am --skip".
	To restore the original branch and stop patching run "git am --abort".

Git 會在有衝突的文件裡加入衝突解決標記，很像有衝突的合併（merge）或者衍合（rebase）操作產生的結果。解決問題的方法也一樣——首先編輯文件消除衝突，然後暫存（stage）新文件，最後運行`git am --resolved`繼續應用下一個補丁：

	$ (fix the file)
	$ git add ticgit.gemspec 
	$ git am --resolved
	Applying: seeing if this helps the gem

如果你想要Git嘗試用更智能的方法來解決衝突，可以使用`-3`選項進行一次三方合併。這個選項默認是不開啟的，因為如果這個補丁的基礎代碼和你的代碼庫不同將會導致錯誤。如果你的代碼庫和補丁的基準代碼有共同的祖先，那麼`-3`選項將會更加智能的應用這個有爭議的補丁。

	$ git am -3 0001-seeing-if-this-helps-the-gem.patch 
	Applying: seeing if this helps the gem
	error: patch failed: ticgit.gemspec:1
	error: ticgit.gemspec: patch does not apply
	Using index info to reconstruct a base tree...
	Falling back to patching base and 3-way merge...
	No changes -- Patch already applied.

重複應用同一個補丁會產生衝突，加上`-3`選項就可以解決這樣的問題。

如果你一次應用多個mbox格式的補丁，可以使用`am`命令的交互模式，這樣就會在每個補丁前停住，然後詢問你如何操作。

	$ git am -3 -i mbox
	Commit Body is:
	--------------------------
	seeing if this helps the gem
	--------------------------
	Apply? [y]es/[n]o/[e]dit/[v]iew patch/[a]ccept all 

在一次應用多個補丁的情況下，這是一個非常好的方法，如果你忘了補丁的內容，可以先瀏覽一下。或者，那些你已經應用過的補丁就不需要再次應用了。

當所有這些針對新特性的補丁都被評估後，你就可以決定是否在長期分支中將他們集成進來。

### 檢出遠程分支 ###

如果你的貢獻者建立了自己的Git代碼庫，並且他將自己的修改都推送到這個代碼庫裡。需要合併代碼時，他會將他代碼庫的URL連同分支的名字一起發過來，然後你應該把這些分支加為遠程分支並且開始本地合併。

比如，Jessica給你發了個郵件，告訴你她代碼庫裡的`ruby-client`分支上有一個非常棒的特性，你就可以引入這個遠程分支，然後將代碼檢出到本地來進行測試。

	$ git remote add jessica git://github.com/jessica/myproject.git
	$ git fetch jessica
	$ git checkout -b rubyclient jessica/ruby-client

如果，過了一會她又發郵件給你，說她還有一個很棒的特性在另外一個分支上，你只需要獲取（fetch）然後檢出（checkout），因為前面已經引入了那個遠程分支。

這樣做非常便於和別人保持長期的合作關係。但是，這要求貢獻者都有自己的服務器，也需要你為每個人創建一個遠程分支。有些貢獻者提交代碼補丁不是很頻繁，這時通過郵件接收補丁效率會更高。因為，你自己也不希望創建幾百個分支，並且每個分支只用來接收一兩個補丁。不過，腳本和託管服務會簡化這個過程——根據不同的開發模式會有不同的實現方式。

利用遠程分支的另外一個好處是：能夠得到提交歷史。儘管如何合理的合併代碼還是個問題，但至少不用擔心提交歷史的差異；默認使用三方合併，不需要提供`-3`選項，可以很好的解決補丁和代碼不匹配的問題。

如果你沒有和某人長期合作，但還是希望以遠程分支的方式從他那拉取代碼的話，你可以使用`git pull`命令加上遠程庫的URL。這將執行一次拉取動作又不會把這個URL存為遠程分支。

	$ git pull git://github.com/onetimeguy/project.git
	From git://github.com/onetimeguy/project
	 * branch            HEAD       -> FETCH_HEAD
	Merge made by recursive.

### 決定該引入的代碼 ###

現在，特性分支上已經合併好了貢獻代碼，是時候決定如何處置它們了。本節將回顧一些以前學過的命令，以便你能夠準確的找出執行合併（merge）操作時將被合併進主幹的代碼。

查看特性分支上新增的commit信息是非常有用的操作。在特性分支名字後面加上`--not`選項就會顯示這些信息。比如，你建立了一個叫做`contrib`的特性分支並打上了兩個補丁，你可以運行：

	$ git log contrib --not master
	commit 5b6235bd297351589efc4d73316f0a68d484f118
	Author: Scott Chacon <schacon@gmail.com>
	Date:   Fri Oct 24 09:53:59 2008 -0700

	    seeing if this helps the gem

	commit 7482e0d16d04bea79d0dba8988cc78df655f16a0
	Author: Scott Chacon <schacon@gmail.com>
	Date:   Mon Oct 22 19:38:36 2008 -0700

	    updated the gemspec to hopefully work better

還可以查看每次commit的具體修改。請牢記在`git log`後加上`-p`選項，將展示每次commit的diff。

如果想看看特性分支和其他分支merge時的完整的diff信息，需要一個更高級的技巧。你可能會想到這樣做：

	$ git diff master

這個命令的確會產生一個diff輸出，但是它可能是錯誤的。如果在你建立特性分支之後`master`分支有過快進（move forward）操作，你得到的輸出將是不正確的。這是因為這個Git命令只會簡單的比較特性分支和`master`分支最後一次快照之間的區別。舉例來說，如果你在`master`分支中的某個文件中加了一行，簡單的比較最新快照所得到的結論只能是：特性分支中刪除了這一行。

如果`master`分支是特性分支的直接祖先，不會產生任何問題；但是如果他們兩個在不同的分叉上，產生的diff輸出看起來就像是增加了特性分支上的所有新增代碼，而刪除了`master`分支上的所有新增代碼。

其實，你真正想看到的是新加入到特性分支的代碼——也就是合併時會併入主幹的代碼。你應該比較的是特性分支與它和master分支共同祖先的差異。

技術上說，你可以找出他們的共同祖先，然後與之比較：

	$ git merge-base contrib master
	36c7dba2c95e6bbb78dfa822519ecfec6e1ca649
	$ git diff 36c7db 

但是這樣很不方便，所以Git提供了另外一種捷徑：三點符號。這樣的情景下，可以把三點符號加在當前分支之前，被比較的擁有共同祖先的分支之後（有點拗口，看例程吧）：

	$ git diff master...contrib

這個命令展示了當前特性分支與它和master分支共同祖先的差異。這是一個非常有用的命令，應該牢記。

### 集成貢獻的代碼 ###

特性分支內的工作都完成了，只等待被加入到更主線一些的分支中。隨之而來的問題是：該怎樣做？還有，維護項目的總體步驟是什麼？其實選擇有很多，我們這裡只介紹其中的一部分。

#### 合併代碼的流程 ####

首先介紹一個簡單的合併代碼流程：假設你的`master`分支包含穩定的代碼，然後你在一個分支上開發新的特性直到完成（或者審核完別人貢獻的代碼），接著你把它併入主幹分支，最後刪除這個多餘的分支，再次開頭重複這個開發流程。如果代碼庫裡有兩個分支，分別叫`ruby_client` and `php_client`，見圖5-19。然後你先把`ruby_client`合併進主幹，再合併`php_client`，最後的提交歷史類似於圖5-20。

Insert 18333fig0519.png 
Figure 5-19. 多個特性分支

Insert 18333fig0520.png
Figure 5-20. 合併特性分支之後

這是最簡單的流程，所以在處理大一些的項目時可能會有問題。

如果維護一個有很多開發者的大項目，至少需要將合併過程分為兩步。假設你有兩個長期分支`master`和`develop`，`develop`分支用來合併新的代碼，`master`分支則只升級到穩定的發行版本。通常這兩個分支都會被推送到公開的代碼庫。每次有新的特性需要合併的時候（圖 5-21），首先將它併入`develop`（圖 5-22）；之後，如果有了一個穩定的發行版，`master`會快進到穩定的`develop`分支處（圖 5-23）。

Insert 18333fig0521.png 
Figure 5-21. 合併特性分支之前

Insert 18333fig0522.png 
Figure 5-22. 合併特性分支之後

Insert 18333fig0523.png 
Figure 5-23. 特性分支發佈之後

這樣，人們在clone你的代碼庫時就會有兩個選擇：他們可以檢出最新的穩定版本的代碼，並且還能很方便的跟上更新。也可以檢出開發分支，這樣就能得到最前沿的特性。

你也可以擴展這個概念。將所有的代碼合併到一個單獨的分支，等到這個分支上的代碼穩定並通過測試後，再將它併入develop分支。然後，如果這些代碼能夠穩定的工作一段時間並有理由相信它足夠穩定了，再將主幹分支快進到這裡。

#### 大項目的合併流程 ####

Git項目有四個長期分支：`master`, `next`,和`pu`（建議繼續修改）針對新代碼的，還有`maint`用來維護除錯的。維護者將貢獻者的代碼併入不同的特性分支，方法類似於我上面提到的那樣（見圖5-24）。之後，測試特性看看他們是能夠安全的工作，否則就需要繼續修改。

Insert 18333fig0524.png 
Figure 5-24. 管理複雜的並行貢獻

如果這些代碼可以安全的工作，可以把他們合併進`next`分支，並推送到公開庫來讓更多的人試試這個新加入的特性。

如果這些代碼還需要修改，就將他們合併進`pu`分支。當他們完全穩定時，這個特性就會重新併入`master`，同時也檢查一下`next`分支，將足夠穩定的特性併入`master`。也就是說`master`始終是快進的，`next`偶爾需要rebased，`pu`經常需要rebased（見圖 5-25）

Insert 18333fig0525.png 
Figure 5-25. 將特性併入長期分支

當一個特性分支最終合併進`master`之後，它將被從代碼庫裡刪除。Git項目還有一個`maint`分支，它是以最近的發行版為基礎派生（fork）出來的，用來提供除錯補丁。所以，當你克隆（clone）Git的代碼庫時，實際上得到四個分支，你可以檢出不同的分支來瞭解不同的開發階段。這取決於你想得到多麼前沿的特性或者做什麼樣的貢獻；維護者都有一個結構化的流程來幫助他們審查新的貢獻。

#### 衍合與挑揀（cherry-pick）的流程 ####

一些維護者更喜歡衍合或者挑揀貢獻者的代碼，而不是簡單的合併，因為這樣能夠保持線性的提交歷史。如果你完成了一個特性的開發，並決定將它引入到主幹代碼中，你可以轉到那個特性分支然後執行衍合命令，好在你的主幹分支上（也可能是`develop`分支之類的）重新提交這些修改。如果這些代碼工作得很好，你就可以快進`master`分支，得到一個線性的提交歷史。

另一個引入代碼的方法是挑揀。挑揀類似於針對某次特定提交的衍合。它首先提取某次提交的補丁，然後試著應用在當前分支上。如果某個特性分支上有多個commits，但你只想引入其中之一就可以使用這種方法。也可能僅僅是因為你喜歡用挑揀，討厭衍合。假設你有一個類似圖 5-26的工程。

Insert 18333fig0526.png 
Figure 5-26. 挑揀（cherry-pick）之前的歷史 

如果你希望拉取`e43a6`到你的主幹分支，可以這樣：

	$ git cherry-pick e43a6fd3e94888d76779ad79fb568ed180e5fcdf
	Finished one cherry-pick.
	[master]: created a0a41a9: "More friendly message when locking the index fails."
	 3 files changed, 17 insertions(+), 3 deletions(-)

這將會引入`e43a6`的代碼，但是會得到不同的SHA-1值，因為應用日期不同。現在你的歷史看起來像圖 5-27.

Insert 18333fig0527.png 
Figure 5-27. 挑揀（cherry-pick）之後的歷史

現在，你可以刪除這個特性分支並丟棄你不想引入的那些commit。

### 給發行版簽名  ###

你可以刪除上次發佈的版本並重新打標籤，也可以像第二章所說的那樣建立一個新的標籤。如果你決定以維護者的身份給發行版簽名，應該這樣做：

	$ git tag -s v1.5 -m 'my signed 1.5 tag'
	You need a passphrase to unlock the secret key for
	user: "Scott Chacon <schacon@gmail.com>"
	1024-bit DSA key, ID F721C45A, created 2009-02-09

完成簽名之後，如何分發PGP公鑰（public key）是個問題。（譯者註：分發公鑰是為了驗證標籤）。還好，Git的設計者想到瞭解決辦法：可以把key（既公鑰）作為blob變量寫入Git庫，然後把它的內容直接寫在標籤裡。`gpg --list-keys`命令可以顯示出你所擁有的key：

	$ gpg --list-keys
	/Users/schacon/.gnupg/pubring.gpg
	---------------------------------
	pub   1024D/F721C45A 2009-02-09 [expires: 2010-02-09]
	uid                  Scott Chacon <schacon@gmail.com>
	sub   2048g/45D02282 2009-02-09 [expires: 2010-02-09]

然後，導出key的內容並經由管道符傳遞給`git hash-object`，之後鑰匙會以blob類型寫入Git中，最後返回這個blob量的SHA-1值：

	$ gpg -a --export F721C45A | git hash-object -w --stdin
	659ef797d181633c87ec71ac3f9ba29fe5775b92

現在你的Git已經包含了這個key的內容了，可以通過不同的SHA-1值指定不同的key來創建標籤。

	$ git tag -a maintainer-pgp-pub 659ef797d181633c87ec71ac3f9ba29fe5775b92

在運行`git push --tags`命令之後，`maintainer-pgp-pub`標籤就會公佈給所有人。如果有人想要校驗標籤，他可以使用如下命令導入你的key：

	$ git show maintainer-pgp-pub | gpg --import

人們可以用這個key校驗你簽名的所有標籤。另外，你也可以在標籤信息裡寫入一個操作嚮導，用戶只需要運行`git show <tag>`查看標籤信息，然後按照你的嚮導就能完成校驗。

### 生成內部版本號 ###

因為Git不會為每次提交自動附加類似'v123'的遞增序列，所以如果你想要得到一個便於理解的提交號可以運行`git describe`命令。Git將會返回一個字符串，由三部分組成：最近一次標定的版本號，加上自那次標定之後的提交次數，再加上一段SHA-1值of the commit you're describing：

	$ git describe master
	v1.6.2-rc1-20-g8c5b85c

這個字符串可以作為快照的名字，方便人們理解。如果你的Git是你自己下載源碼然後編譯安裝的，你會發現`git --version`命令的輸出和這個字符串差不多。如果在一個剛剛打完標籤的提交上運行`describe`命令，只會得到這次標定的版本號，而沒有後面兩項信息。

`git describe`命令只適用於有標註的標籤（通過`-a`或者`-s`選項創建的標籤），所以發行版的標籤都應該是帶有標註的，以保證`git describe`能夠正確的執行。你也可以把這個字符串作為`checkout`或者`show`命令的目標，因為他們最終都依賴於一個簡短的SHA-1值，當然如果這個SHA-1值失效他們也跟著失效。最近Linux內核為了保證SHA-1值的唯一性，將位數由8位擴展到10位，這就導致擴展之前的`git describe`輸出完全失效了。

### 準備發佈 ###

現在可以發佈一個新的版本了。首先要將代碼的壓縮包歸檔，方便那些可憐的還沒有使用Git的人們。可以使用`git archive`：

	$ git archive master --prefix='project/' | gzip > `git describe master`.tar.gz
	$ ls *.tar.gz
	v1.6.2-rc1-20-g8c5b85c.tar.gz

這個壓縮包解壓出來的是一個文件夾，裡面是你項目的最新代碼快照。你也可以用類似的方法建立一個zip壓縮包，在`git archive`加上`--format=zip`選項：

	$ git archive master --prefix='project/' --format=zip > `git describe master`.zip

現在你有了一個tar.gz壓縮包和一個zip壓縮包，可以把他們上傳到你網站上或者用e-mail發給別人。

### 製作簡報 ###

是時候通知郵件列表裡的朋友們來檢驗你的成果了。使用`git shortlog`命令可以方便快捷的製作一份修改日誌（changelog），告訴大家上次發佈之後又增加了哪些特性和修復了哪些bug。實際上這個命令能夠統計給定範圍內的所有提交;假如你上一次發佈的版本是v1.0.1，下面的命令將給出自從上次發佈之後的所有提交的簡介：

	$ git shortlog --no-merges master --not v1.0.1
	Chris Wanstrath (8):
	      Add support for annotated tags to Grit::Tag
	      Add packed-refs annotated tag support.
	      Add Grit::Commit#to_patch
	      Update version and History.txt
	      Remove stray `puts`
	      Make ls_tree ignore nils

	Tom Preston-Werner (4):
	      fix dates in history
	      dynamic version method
	      Version bump to 1.0.2
	      Regenerated gemspec for version 1.0.2

這就是自從v1.0.1版本以來的所有提交的簡介，內容按照作者分組，以便你能快速的發e-mail給他們。


## 小結 ##

你學會了如何使用Git為項目做貢獻，也學會了如何使用Git維護你的項目。恭喜！你已經成為一名高效的開發者。在下一章你將學到更強大的工具來處理更加複雜的問題，之後你會變成一位Git大師。

