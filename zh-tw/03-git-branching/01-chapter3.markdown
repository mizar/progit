# Git 分支 #

幾乎每一種版本控制系統都以某種形式支持分支。使用分支意味著你可以從開發主線上分離開來，然後在不影響主線的同時繼續工作。在很多版本控制系統中，這是個昂貴的過程，常常需要創建一個源代碼目錄的完整副本，對大型項目來說會花費很長時間。

有人把 Git 的分支模型稱為「必殺技特性」，而正是因為它，將 Git 從版本控制系統家族裡區分出來。Git 有何特別之處呢？Git 的分支可謂是難以置信的輕量級，它的新建操作幾乎可以在瞬間完成，並且在不同分支間切換起來也差不多一樣快。和許多其他版本控制系統不同，Git 鼓勵在工作流程中頻繁使用分支與合併，哪怕一天之內進行許多次都沒有關係。理解分支的概念並熟練運用後，你才會意識到為什麼 Git 是一個如此強大而獨特的工具，並從此真正改變你的開發方式。

## 何謂分支 ##

為了理解 Git 分支的實現方式，我們需要回顧一下 Git 是如何儲存數據的。或許你還記得第一章的內容，Git 保存的不是文件差異或者變化量，而只是一系列文件快照。

在 Git 中提交時，會保存一個提交（commit）對象，它包含一個指向暫存內容快照的指針，作者和相關附屬信息，以及一定數量（也可能沒有）指向該提交對象直接祖先的指針：第一次提交是沒有直接祖先的，普通提交有一個祖先，由兩個或多個分支合併產生的提交則有多個祖先。

為直觀起見，我們假設在工作目錄中有三個文件，準備將它們暫存後提交。暫存操作會對每一個文件計算校驗和（即第一章中提到的 SHA-1 哈希字串），然後把當前版本的文件快照保存到 Git 倉庫中（Git 使用 blob 類型的對象存儲這些快照），並將校驗和加入暫存區域：

	$ git add README test.rb LICENSE2
	$ git commit -m 'initial commit of my project'

當使用 `git commit` 新建一個提交對象前，Git 會先計算每一個子目錄（本例中就是項目根目錄）的校驗和，然後在 Git 倉庫中將這些目錄保存為樹（tree）對象。之後 Git 創建的提交對象，除了包含相關提交信息以外，還包含著指向這個樹對象（項目根目錄）的指針，如此它就可以在將來需要的時候，重現此次快照的內容了。

現在，Git 倉庫中有五個對象：三個表示文件快照內容的 blob 對象；一個記錄著目錄樹內容及其中各個文件對應 blob 對象索引的 tree 對象；以及一個包含指向 tree 對象（根目錄）的索引和其他提交信息元數據的 commit 對象。概念上來說，倉庫中的各個對象保存的數據和相互關係看起來如圖 3-1 所示：

Insert 18333fig0301.png 
圖 3-1. 一次提交後倉庫裡的數據

作些修改後再次提交，那麼這次的提交對象會包含一個指向上次提交對象的指針（譯註：即下圖中的 parent 對象）。兩次提交後，倉庫歷史會變成圖 3-2 的樣子：

Insert 18333fig0302.png 
圖 3-2. 多次提交後的 Git 對象數據

現在來談分支。Git 中的分支，其實本質上僅僅是個指向 commit 對象的可變指針。Git 會使用 master 作為分支的默認名字。在若干次提交後，你其實已經有了一個指向最後一次提交對象的 master 分支，它在每次提交的時候都會自動向前移動。

Insert 18333fig0303.png 
圖 3-3. 指向提交數據歷史的分支

那麼，Git 又是如何創建一個新的分支的呢？答案很簡單，創建一個新的分支指針。比如新建一個 testing 分支，可以使用 `git branch` 命令：

	$ git branch testing

這會在當前 commit 對象上新建一個分支指針（見圖 3-4）。

Insert 18333fig0304.png 
圖 3-4. 多個分支指向提交數據的歷史

那麼，Git 是如何知道你當前在哪個分支上工作的呢？其實答案也很簡單，它保存著一個名為 HEAD 的特別指針。請注意它和你熟知的許多其他版本控制系統（比如 Subversion 或 CVS）裡的 HEAD 概念大不相同。在 Git 中，它是一個指向你正在工作中的本地分支的指針。運行 `git branch` 命令，僅僅是建立了一個新的分支，但不會自動切換到這個分支中去，所以在這個例子中，我們依然還在 master 分支裡工作（參考圖 3-5）。

Insert 18333fig0305.png 
圖 3-5. HEAD 指向當前所在的分支

要切換到其他分支，可以執行 `git checkout` 命令。我們現在轉換到新建的 testing 分支：

	$ git checkout testing

這樣 HEAD 就指向了 testing 分支（見圖3-6）。

Insert 18333fig0306.png
圖 3-6. HEAD 在你轉換分支時指向新的分支

這樣的實現方式會給我們帶來什麼好處呢？好吧，現在不妨再提交一次：

	$ vim test.rb
	$ git commit -a -m 'made a change'

圖 3-7 展示了提交後的結果。

Insert 18333fig0307.png 
圖 3-7. 每次提交後 HEAD 隨著分支一起向前移動

非常有趣，現在 testing 分支向前移動了一格，而 master 分支仍然指向原先 `git checkout` 時所在的 commit 對象。現在我們回到 master 分支看看：

	$ git checkout master

圖 3-8 顯示了結果。

Insert 18333fig0308.png 
圖 3-8. HEAD 在一次 checkout 之後移動到了另一個分支

這條命令做了兩件事。它把 HEAD 指針移回到 master 分支，並把工作目錄中的文件換成了 master 分支所指向的快照內容。也就是說，現在開始所做的改動，將始於本項目中一個較老的版本。它的主要作用是將 testing 分支裡作出的修改暫時取消，這樣你就可以向另一個方向進行開發。

我們作些修改後再次提交：

	$ vim test.rb
	$ git commit -a -m 'made other changes'

現在我們的項目提交歷史產生了分叉（如圖 3-9 所示），因為剛才我們創建了一個分支，轉換到其中進行了一些工作，然後又回到原來的主分支進行了另外一些工作。這些改變分別孤立在不同的分支裡：我們可以在不同分支裡反覆切換，並在時機成熟時把它們合併到一起。而所有這些工作，僅僅需要 `branch` 和 `checkout` 這兩條命令就可以完成。

Insert 18333fig0309.png 
圖 3-9. 分叉了的分支歷史

由於 Git 中的分支實際上僅是一個包含所指對象校驗和（40 個字符長度 SHA-1 字串）的文件，所以創建和銷毀一個分支就變得非常廉價。說白了，新建一個分支就是向一個文件寫入 41 個字節（外加一個換行符）那麼簡單，當然也就很快了。

這和大多數版本控制系統形成了鮮明對比，它們管理分支大多採取備份所有項目文件到特定目錄的方式，所以根據項目文件數量和大小不同，可能花費的時間也會有相當大的差別，快則幾秒，慢則數分鐘。而 Git 的實現與項目複雜度無關，它永遠可以在幾毫秒的時間內完成分支的創建和切換。同時，因為每次提交時都記錄了祖先信息（譯註：即 parent 對象），所以以後要合併分支時，尋找恰當的合併基礎（譯註：即共同祖先）的工作其實已經完成了一大半，實現起來非常容易。Git 鼓勵開發者頻繁使用分支，正是因為有著這些特性作保障。

接下來看看，我們為什麼應該頻繁使用分支。

## 基本的分支與合併 ##

現在讓我們來看一個簡單的分支與合併的例子，實際工作中大體也會用到這樣的工作流程：

1. 開發某個網站。
2. 為實現某個新的需求，創建一個分支。
3. 在這個分支上開展工作。

假設此時，你突然接到一個電話說有個很嚴重的問題需要緊急修補，那麼可以按照下面的方式處理：

1. 返回到原先已經發佈到生產服務器上的分支。
2. 為這次緊急修補建立一個新分支。
3. 測試通過後，將此修補分支合併，再推送到生產服務器上。
4. 切換到之前實現新需求的分支，繼續工作。

### 基本分支 ###

首先，我們假設你正在項目中愉快地工作，並且已經提交了幾次更新（見圖 3-10）。

Insert 18333fig0310.png 
圖 3-10. 一部分簡短的提交歷史

現在，你決定要修補問題追蹤系統上的 #53 問題。順帶說明下，Git 並不同任何特定的問題追蹤系統打交道。這裡為了說明要解決的問題，才把新建的分支取名為 iss53。要新建並切換到該分支，運行 `git checkout` 並加上 `-b` 參數：

	$ git checkout -b iss53
	Switched to a new branch "iss53"

相當於下面這兩條命令：

	$ git branch iss53
	$ git checkout iss53

圖 3-11 示意該命令的結果。

Insert 18333fig0311.png 
圖 3-11. 創建了一個新的分支指針

接下來，你在網站項目上繼續工作並作了一次提交。這會使 `iss53` 分支的指針隨著提交向前推進，因為它處於檢出狀態（或者說，你的 HEAD 指針目前正指向它，見圖3-12）：

	$ vim index.html
	$ git commit -a -m 'added a new footer [issue 53]'

Insert 18333fig0312.png 
圖 3-12. iss53 分支隨工作進展向前推進

現在你就接到了那個網站問題的緊急電話，需要馬上修補。有了 Git ，我們就不需要同時發佈這個補丁和 `iss53` 裡作出的修改，也不需要在創建和發佈該補丁到服務器之前花費很多努力來復原這些修改。唯一需要的僅僅是切換回 master 分支。

不過在此之前，留心你的暫存區或者工作目錄裡，那些還沒有提交的修改，它會和你即將檢出的分支產生衝突從而阻止 Git 為你轉換分支。轉換分支的時候最好保持一個清潔的工作區域。稍後會介紹幾個繞過這種問題的辦法（分別叫做 stashing 和 amending）。目前已經提交了所有的修改，所以接下來可以正常轉換到 master 分支：

	$ git checkout master
	Switched to branch "master"

此時工作目錄中的內容和你在解決問題 #53 之前一模一樣，你可以集中精力進行緊急修補。這一點值得牢記：Git 會把工作目錄的內容恢復為檢出某分支時它所指向的那個 commit 的快照。它會自動添加、刪除和修改文件以確保目錄的內容和你上次提交時完全一樣。

接下來，你得進行緊急修補。我們創建一個緊急修補分支（hotfix）來開展工作，直到搞定（見圖 3-13）：

	$ git checkout -b 'hotfix'
	Switched to a new branch "hotfix"
	$ vim index.html
	$ git commit -a -m 'fixed the broken email address'
	[hotfix]: created 3a0874c: "fixed the broken email address"
	 1 files changed, 0 insertions(+), 1 deletions(-)

Insert 18333fig0313.png 
圖 3-13. hotfix 分支是從 master 分支所在點分化出來的

有必要作些測試，確保修補是成功的，然後把它合併到 master 分支並發布到生產服務器。用 `git merge` 命令來進行合併：

	$ git checkout master
	$ git merge hotfix
	Updating f42c576..3a0874c
	Fast forward
	 README |    1 -
	 1 files changed, 0 insertions(+), 1 deletions(-)

請注意，合併時出現了 "Fast forward"（快進）提示。由於當前 master 分支所在的 commit 是要併入的 hotfix 分支的直接上游，Git 只需把指針直接右移。換句話說，如果順著一個分支走下去可以到達另一個分支，那麼 Git 在合併兩者時，只會簡單地把指針前移，因為沒有什麼分歧需要解決，所以這個過程叫做快進（Fast forward）。

現在的目錄變為當前 master 分支指向的 commit 所對應的快照，可以發佈了（見圖 3-14）。

Insert 18333fig0314.png 
圖 3-14. 合併之後，master 分支和 hotfix 分支指向同一位置。

在那個超級重要的修補發佈以後，你想要回到被打擾之前的工作。因為現在 `hotfix` 分支和 `master` 指向相同的提交，現在沒什麼用了，可以先刪掉它。使用 `git branch` 的 `-d` 選項表示刪除：

	$ git branch -d hotfix
	Deleted branch hotfix (3a0874c).

現在可以回到未完成的問題 #53 分支繼續工作了（圖3-15）：

	$ git checkout iss53
	Switched to branch "iss53"
	$ vim index.html
	$ git commit -a -m 'finished the new footer [issue 53]'
	[iss53]: created ad82d7a: "finished the new footer [issue 53]"
	 1 files changed, 1 insertions(+), 0 deletions(-)

Insert 18333fig0315.png 
圖 3-15. iss53 分支可以不受影響繼續推進。

不用擔心 `hotfix` 分支的內容還沒包含在 `iss53` 中。如果確實需要納入此次修補，可以用 `git merge master` 把 master 分支合併到 `iss53`，或者等完成後，再將 `iss53` 分支中的更新併入 `master`。

### 基本合併 ###

在問題 #53 相關的工作完成之後，可以合併回 `master` 分支，實際操作同前面合併 `hotfix` 分支差不多，只需檢出想要更新的分支（master），並運行 `git merge` 命令指定來源：

	$ git checkout master
	$ git merge iss53
	Merge made by recursive.
	 README |    1 +
	 1 files changed, 1 insertions(+), 0 deletions(-)

請注意，這次合併的實現，並不同於之前 `hotfix` 的併入方式。這一次，你的開發歷史是從更早的地方開始分叉的。由於當前 master 分支所指向的 commit (C4)並非想要併入分支（iss53）的直接祖先，Git 不得不進行一些處理。就此例而言，Git 會用兩個分支的末端（C4 和 C5）和它們的共同祖先（C2）進行一次簡單的三方合併計算。圖 3-16 標出了 Git 在用於合併的三個更新快照：

Insert 18333fig0316.png 
圖 3-16. Git 為分支合併自動識別出最佳的同源合併點。

Git 沒有簡單地把分支指針右移，而是對三方合併的結果作一新的快照，並自動創建一個指向它的 commit（C6）（見圖 3-17）。我們把這個特殊的 commit 稱作合併提交（merge commit），因為它的祖先不止一個。

值得一提的是 Git 可以自己裁決哪個共同祖先才是最佳合併基礎；這和 CVS 或 Subversion（1.5 以後的版本）不同，它們需要開發者手工指定合併基礎。所以此特性讓 Git 的合併操作比其他系統都要簡單不少。

Insert 18333fig0317.png 
圖 3-17. Git 自動創建了一個包含了合併結果的 commit 對象。

既然你的工作成果已經合併了，`iss53` 也就沒用了。你可以就此刪除它，並在問題追蹤系統裡把該問題關閉。

	$ git branch -d iss53

### 衝突的合併 ###

有時候合併操作並不會如此順利。如果你修改了兩個待合併分支裡同一個文件的同一部分，Git 就無法乾淨地把兩者合到一起（譯註：邏輯上說，這種問題只能由人來解決）。如果你在解決問題 #53 的過程中修改了 `hotfix` 中修改的部分，將得到類似下面的結果：

	$ git merge iss53
	Auto-merging index.html
	CONFLICT (content): Merge conflict in index.html
	Automatic merge failed; fix conflicts and then commit the result.

Git 作了合併，但沒有提交，它會停下來等你解決衝突。要看看哪些文件在合併時發生衝突，可以用 `git status` 查閱：

	[master*]$ git status
	index.html: needs merge
	# On branch master
	# Changed but not updated:
	#   (use "git add <file>..." to update what will be committed)
	#   (use "git checkout -- <file>..." to discard changes in working directory)
	#
	#	unmerged:   index.html
	#

任何包含未解決衝突的文件都會以未合併（unmerged）狀態列出。Git 會在有衝突的文件裡加入標準的衝突解決標記，可以通過它們來手工定位並解決這些衝突。可以看到此文件包含類似下面這樣的部分：

	<<<<<<< HEAD:index.html
	<div id="footer">contact : email.support@github.com</div>
	=======
	<div id="footer">
	  please contact us at support@github.com
	</div>
	>>>>>>> iss53:index.html

可以看到 `=======` 隔開的上半部分，是 HEAD（即 master 分支，在運行 merge 命令時檢出的分支）中的內容，下半部分是在 `iss53` 分支中的內容。解決衝突的辦法無非是二者選其一或者由你親自整合到一起。比如你可以通過把這段內容替換為下面這樣來解決：

	<div id="footer">
	please contact us at email.support@github.com
	</div>

這個解決方案各採納了兩個分支中的一部分內容，而且我還刪除了 `<<<<<<<`，`=======`，和`>>>>>>>` 這些行。在解決了所有文件裡的所有衝突後，運行 `git add` 將把它們標記為已解決（resolved）。因為一旦暫存，就表示衝突已經解決。如果你想用一個有圖形界面的工具來解決這些問題，不妨運行 `git mergetool`，它會調用一個可視化的合併工具並引導你解決所有衝突：

	$ git mergetool
	merge tool candidates: kdiff3 tkdiff xxdiff meld gvimdiff opendiff emerge vimdiff
	Merging the files: index.html

	Normal merge conflict for 'index.html':
	  {local}: modified
	  {remote}: modified
	Hit return to start merge resolution tool (opendiff):

如果不想用默認的合併工具（Git 為我默認選擇了 `opendiff`，因為我在 Mac 上運行了該命令），你可以在上方"merge tool candidates（候選合併工具）"裡找到可用的合併工具列表，輸入你想用的工具名。我們將在第七章討論怎樣改變環境中的默認值。

退出合併工具以後，Git 會詢問你合併是否成功。如果回答是，它會為你把相關文件暫存起來，以表明狀態為已解決。

再運行一次 `git status` 來確認所有衝突都已解決：

	$ git status
	# On branch master
	# Changes to be committed:
	#   (use "git reset HEAD <file>..." to unstage)
	#
	#	modified:   index.html
	#

如果覺得滿意了，並且確認所有衝突都已解決，也就是進入了緩存區，就可以用 `git commit` 來完成這次合併提交。提交的記錄差不多是這樣：

	Merge branch 'iss53'

	Conflicts:
	  index.html
	#
	# It looks like you may be committing a MERGE.
	# If this is not correct, please remove the file
	# .git/MERGE_HEAD
	# and try again.
	#

如果想給將來看這次合併的人一些方便，可以修改該信息，提供更多合併細節。比如你都作了哪些改動，以及這麼做的原因。有時候裁決衝突的理由並不直接或明顯，有必要略加註解。

## 分支管理 ##

到目前為止，你已經學會了如何創建、合併和刪除分支。除此之外，我們還需要學習如何管理分支，在日後的常規工作中會經常用到下面介紹的管理命令。

`git branch` 命令不僅僅能創建和刪除分支，如果不加任何參數，它會給出當前所有分支的清單：

	$ git branch
	  iss53
	* master
	  testing

注意看 `master` 分支前的 `*` 字符：它表示當前所在的分支。也就是說，如果現在提交更新，`master` 分支將隨著開發進度前移。若要查看各個分支最後一次 commit 信息，運行 `git branch -v`：

	$ git branch -v
	  iss53   93b412c fix javascript issue
	* master  7a98805 Merge branch 'iss53'
	  testing 782fd34 add scott to the author list in the readmes

要從該清單中篩選出你已經（或尚未）與當前分支合併的分支，可以用 `--merge` 和 `--no-merged` 選項（Git 1.5.6 以上版本）。比如 `git branch -merge` 查看哪些分支已被併入當前分支：

	$ git branch --merged
	  iss53
	* master

之前我們已經合併了 `iss53`，所以在這裡會看到它。一般來說，列表中沒有 `*` 的分支通常都可以用 `git branch -d` 來刪掉。原因很簡單，既然已經把它們所包含的工作整合到了其他分支，刪掉也不會損失什麼。

另外可以用 `git branch --no-merged` 查看尚未合併的工作：

	$ git branch --no-merged
	  testing

我們會看到其餘還未合併的分支。因為其中還包含未合併的工作，用 `git branch -d` 刪除該分支會導致失敗：

	$ git branch -d testing
	error: The branch 'testing' is not an ancestor of your current HEAD.

不過，如果你堅信你要刪除它，可以用大寫的刪除選項 `-D` 強制執行，例如 `git branch -D testing`。

## 分支式工作流程 ##

如今有了分支與合併的基礎，你可以（或應該）用它來做點什麼呢？在本節，我們會介紹些使用分支進行開發的工作流程。而正是由於分支管理的便捷，才衍生出了這類典型的工作模式，有機會可以實踐一下。

### 長期分支 ###

由於 Git 使用簡單的三方合併，所以就算在較長一段時間內，反覆多次把某個分支合併到另一分支，也不是什麼難事。也就是說，你可以同時擁有多個開放的分支，每個分支用於完成特定的任務，隨著開發的推進，你可以隨時把某個特性分支的成果並到其他分支中。

許多使用 Git 的開發者都喜歡以這種方式來開展工作，比如僅在 `master` 分支中保留完全穩定的代碼，即已經發佈或即將發佈的代碼。與此同時，他們還有一個名為 develop 或 next 的平行分支，專門用於後續的開發，或僅用於穩定性測試 —— 當然並不是說一定要絕對穩定，不過一旦進入某種穩定狀態，便可以把它合併到 `master` 裡。這樣，在確保這些已完成的特性分支（短期分支，如前例的 `iss53`）能夠通過所有測試，並且不會引入更多錯誤之後，就可以並到主幹分支中，等待下一次的發布。

本質上我們剛才談論的，是隨著 commit 不停前移的指針。穩定分支的指針總是在提交歷史中落後一大截，而前沿分支總是比較靠前（見圖 3-18）。

Insert 18333fig0318.png 
圖 3-18. 穩定分支總是比較老舊。

或者把它們想像成工作流水線可能會比較容易理解，經過測試的 commit 集合被遴選到更穩定的流水線（見圖 3-19）。

Insert 18333fig0319.png 
圖 3-19. 想像成流水線可能會容易點。

你可以用這招維護不同層次的穩定性。某些大項目還會有個 `proposed`（建議）或 `pu`（proposed updates，建議更新）分支，它包含著那些可能還沒有成熟到進入 `next` 或 `master` 的內容。這麼做的目的是擁有不同層次的穩定性：當這些分支進入到更穩定的水平時，再把它們合併到更高層分支中去。再次說明下，使用多個長期分支的做法並非必需，不過一般來說，對於特大型項目或特複雜的項目，這麼做確實更容易管理。

### 特性分支 ###

在任何規模的項目中都可以使用特性（Topic）分支。一個特性分支是指一個短期的，用來實現單一特性或與其相關工作的分支。可能你在以前的版本控制系統裡從未做過類似這樣的事請，因為通常創建與合併分支消耗太大。然而在 Git 中，一天之內建立，使用，合併再刪除多個分支是常見的事。

我們在上節的例子裡已經見過這種用法了。我們創建了 `iss53` 和 `hotfix` 這兩個特性分支，在提交了若干更新後，把它們合併到主幹分支，然後刪除。該技術允許你迅速且完全的進行語境切換 —— 因為你的工作分散在不同的流水線裡，每個分支裡的改變都和它的目標特性相關，瀏覽代碼之類的事情因而變得更簡單了。你可以把作出的改變保持在特性分支中幾分鐘，幾天甚至幾個月，等它們成熟以後再合併，而不用在乎它們建立的順序或者進度。

現在我們來看一個實際的例子。請看圖 3-20，起先我們在 `master` 工作到 C1，然後開始一個新分支 `iss91` 嘗試修復 91 號缺陷，提交到 C6 的時候，又冒出一個新的解決問題的想法，於是從之前 C4 的地方又分出一個分支 `iss91v2`，幹到 C8 的時候，又回到主幹中提交了 C9 和 C10，再回到 `iss91v2` 繼續工作，提交 C11，接著，又冒出個不太確定的想法，從 `master` 的最新提交 C10 處開了個新的分支 `dumbidea` 做些試驗。

Insert 18333fig0320.png 
圖 3-20. 擁有多個特性分支的提交歷史。

現在，假定兩件事情：我們最終決定使用第二個解決方案，即 `iss91v2` 中的辦法；另外，我們把 `dumbidea` 分支拿給同事們看了以後，發現它竟然是個天才之作。所以接下來，我們拋棄原來的 `iss91` 分支（即丟棄 C5 和 C6），直接在主幹中併入另外兩個分支。最終的提交歷史將變成圖 3-21 這樣：

Insert 18333fig0321.png 
圖 3-21. 合併了 dumbidea 和 iss91v2 以後的歷史。

請務必牢記這些分支全部都是本地分支，這一點很重要。當你在使用分支及合併的時候，一切都是在你自己的 Git 倉庫中進行的 —— 完全不涉及與服務器的交互。

## 遠程分支 ##

遠程分支（remote branch）是對遠程倉庫狀態的索引。它們是一些無法移動的本地分支；只有在進行 Git 的網絡活動時才會更新。遠程分支就像是書籤，提醒著你上次連接遠程倉庫時上面各分支的位置。 

我們用 `(遠程倉庫名)/(分支名)` 這樣的形式表示遠程分支。比如我們想看看上次同 `origin` 倉庫通訊時 `master` 的樣子，就應該查看 `origin/master` 分支。如果你和同伴一起修復某個問題，但他們先推送了一個 `iss53` 分支到遠程倉庫，雖然你可能也有一個本地的 `iss53` 分支，但指向服務器上最新更新的卻應該是 `origin/iss53` 分支。

可能有點亂，我們不妨舉例說明。假設你們團隊有個地址為 `git.ourcompany.com` 的 Git 服務器。如果你從這裡克隆，Git 會自動為你將此遠程倉庫命名為 `origin`，並下載其中所有的數據，建立一個指向它的 `master` 分支的指針，在本地命名為 `origin/master`，但你無法在本地更改其數據。接著，Git 建立一個屬於你自己的本地 `master` 分支，始於 `origin` 上 `master` 分支相同的位置，你可以就此開始工作（見圖 3-22）：

Insert 18333fig0322.png 
圖 3-22. 一次 Git 克隆會建立你自己的本地分支 master 和遠程分支 origin/master，它們都指向 origin/master 分支的最後一次提交。

要是你在本地 `master` 分支做了會兒事情，與此同時，其他人向 `git.ourcompany.com` 推送了內容，更新了上面的 `master` 分支，那麼你的提交歷史會開始朝不同的方向發展。不過只要你不和服務器通訊，你的 `origin/master` 指針不會移動（見圖 3-23）。

Insert 18333fig0323.png 
圖 3-23. 在本地工作的同時有人向遠程倉庫推送內容會讓提交歷史發生分歧。

可以運行 `git fetch origin` 來進行同步。該命令首先找到 `origin` 是哪個服務器（本例為 `git.ourcompany.com`），從上面獲取你尚未擁有的數據，更新你本地的數據庫，然後把 `origin/master` 的指針移到它最新的位置（見圖 3-24）。

Insert 18333fig0324.png 
圖 3-24. git fetch 命令會更新 remote 索引。

為了演示擁有多個遠程分支（不同的遠程服務器）的項目是個什麼樣，我們假設你還有另一個僅供你的敏捷開發小組使用的內部服務器 `git.team1.ourcompany.com`。可以用第二章中提到的 `git remote add` 命令把它加為當前項目的遠程分支之一。我們把它命名為 `teamone`，表示那一整串 Git 地址（見圖 3-25）。

Insert 18333fig0325.png 
圖 3-25. 把另一個服務器加為遠程倉庫

現在你可以用 `git fetch teamone` 來獲取小組服務器上你還沒有的數據了。由於當前該服務器上的內容是你 `origin` 服務器上的子集，Git 不會下載任何數據，而只是簡單地創建一個名為 `teamone/master` 的分支來指向 `teamone` 服務器上 `master` 所指向的更新 `31b8e`（見圖 3-26）。

Insert 18333fig0326.png 
圖 3-26. 你在本地有了一個指向 teamone 服務器上 master 分支的索引。

### 推送 ###

要想和其他人分享某個分支，你需要把它推送到一個你擁有寫權限的遠程倉庫。你的本地分支不會被自動同步到你引入的遠程分支中，除非你明確執行推送操作。換句話說，對於無意分享的，你盡可以保留為私人分支，而只推送那些協同工作的特性分支。

如果你有個叫 `serverfix` 的分支需要和他人一起開發，可以運行 `git push (遠程倉庫名) (分支名)`：

	$ git push origin serverfix
	Counting objects: 20, done.
	Compressing objects: 100% (14/14), done.
	Writing objects: 100% (15/15), 1.74 KiB, done.
	Total 15 (delta 5), reused 0 (delta 0)
	To git@github.com:schacon/simplegit.git
	 * [new branch]      serverfix -> serverfix

這其實有點像條捷徑。Git 自動把 `serverfix` 分支名擴展為 `refs/heads/serverfix:refs/heads/serverfix`，意為「取出我的 serverfix 本地分支，推送它來更新遠程倉庫的 serverfix 分支」。我們將在第九章進一步介紹 `refs/heads/` 部分的細節，不過一般使用的時候都可以省略它。也可以運行 `git push origin serverfix:serferfix` 來實現相同的效果，它的意思是「提取我的 serverfix 並更新到遠程倉庫的 serverfix」。通過此語法，你可以把本地分支推送到某個命名不同的遠程分支：若想把遠程分支叫作 `awesomebranch`，可以用 `git push origin serverfix:awesomebranch` 來推送數據。

接下來，當你的協作者再次從服務器上獲取數據時，他們將得到一個新的遠程分支 `origin/serverfix`：

	$ git fetch origin
	remote: Counting objects: 20, done.
	remote: Compressing objects: 100% (14/14), done.
	remote: Total 15 (delta 5), reused 0 (delta 0)
	Unpacking objects: 100% (15/15), done.
	From git@github.com:schacon/simplegit
	 * [new branch]      serverfix    -> origin/serverfix

值得注意的是，在 fetch 操作抓來新的遠程分支之後，你仍然無法在本地編輯該遠程倉庫。換句話說，在本例中，你不會有一個新的 `serverfix` 分支，有的只是一個你無法移動的 `origin/serverfix` 指針。

如果要把該內容合併到當前分支，可以運行 `git merge origin/serverfix`。如果想要一份自己的 `serverfix` 來開發，可以在遠程分支的基礎上分化出一個新的分支來：

	$ git checkout -b serverfix origin/serverfix
	Branch serverfix set up to track remote branch refs/remotes/origin/serverfix.
	Switched to a new branch "serverfix"

這會切換到新建的 `serverfix` 本地分支，其內容同遠程分支 `origin/serverfix` 一致，你可以在裡面繼續開發了。

### 跟蹤分支 ###

從遠程分支檢出的本地分支，稱為_跟蹤分支(tracking branch)_。跟蹤分支是一種和遠程分支有直接聯繫的本地分支。在跟蹤分支裡輸入 `git push`，Git 會自行推斷應該向哪個服務器的哪個分支推送數據。反過來，在這些分支裡運行 `git pull` 會獲取所有遠程索引，並把它們的數據都合併到本地分支中來。

在克隆倉庫時，Git 通常會自動創建一個 `master` 分支來跟蹤 `origin/master`。這正是 `git push` 和 `git pull` 一開始就能正常工作的原因。當然，你可以隨心所欲地設定為其它跟蹤分支，比如 `origin` 上除了 `master` 之外的其它分支。剛才我們已經看到了這樣的一個例子：`git checkout -b [分支名] [遠程名]/[分支名]`。如果你有 1.6.2 以上版本的 Git，還可以用 `--track` 選項簡化：

	$ git checkout --track origin/serverfix
	Branch serverfix set up to track remote branch refs/remotes/origin/serverfix.
	Switched to a new branch "serverfix"

要為本地分支設定不同於遠程分支的名字，只需在前個版本的命令裡換個名字：

	$ git checkout -b sf origin/serverfix
	Branch sf set up to track remote branch refs/remotes/origin/serverfix.
	Switched to a new branch "sf"

現在你的本地分支 `sf` 會自動向 `origin/serverfix` 推送和抓取數據了。

### 刪除遠程分支 ###

如果不再需要某個遠程分支了，比如搞定了某個特性並把它合併進了遠程的 `master` 分支（或任何其他存放穩定代碼的地方），可以用這個非常無厘頭的語法來刪除它：`git push [遠程名] :[分支名]`。如果想在服務器上刪除 `serverfix` 分支，運行下面的命令：

	$ git push origin :serverfix
	To git@github.com:schacon/simplegit.git
	 - [deleted]         serverfix

咚！服務器上的分支沒了。你最好特別留心這一頁，因為你一定會用到那個命令，而且你很可能會忘掉它的語法。有種方便記憶這條命令的方法：記住我們不久前見過的 `git push [遠程名] [本地分支]:[遠程分支]` 語法，如果省略 `[本地分支]`，那就等於是在說「在這裡提取空白然後把它變成`[遠程分支]`」。

## 衍合 ##

把一個分支整合到另一個分支的辦法有兩種：`merge（合併）` 和 `rebase（衍合）`。在本章我們會學習什麼是衍合，如何使用衍合，為什麼衍合操作如此富有魅力，以及我們應該在什麼情況下使用衍合。

### 衍合基礎 ###

請回顧之前有關合併的一節（見圖 3-27），你會看到開發進程分叉到兩個不同分支，又各自提交了更新。

Insert 18333fig0327.png 
圖 3-27. 最初分叉的提交歷史。

之前介紹過，最容易的整合分支的方法是 `merge` 命令，它會把兩個分支最新的快照（C3 和 C4）以及二者最新的共同祖先（C2）進行三方合併。如圖 3-28 所示：

Insert 18333fig0328.png 
圖 3-28. 通過合併一個分支來整合分叉了的歷史。

其實，還有另外一個選擇：你可以把在 C3 裡產生的變化補丁重新在 C4 的基礎上打一遍。在 Git 裡，這種操作叫做_衍合（rebase）_。有了 `rebase` 命令，就可以把在一個分支裡提交的改變在另一個分支裡重放一遍。

在這個例子裡，可以運行下面的命令：

	$ git checkout experiment
	$ git rebase master
	First, rewinding head to replay your work on top of it...
	Applying: added staged command

它的原理是回到兩個分支（你所在的分支和你想要衍合進去的分支）的共同祖先，提取你所在分支每次提交時產生的差異（diff），把這些差異分別保存到臨時文件裡，然後從當前分支轉換到你需要衍合入的分支，依序施用每一個差異補丁文件。圖 3-29 演示了這一過程：

Insert 18333fig0329.png 
圖 3-29. 把 C3 裡產生的改變衍合到 C4 中。

現在，你可以回到 master 分支然後進行一次快進合併（見圖 3-30）：

Insert 18333fig0330.png 
圖 3-30. master 分支的快進。

現在，合併後的 C3（即現在的 C3'）所指的快照，同三方合併例子中的 C5 所指的快照內容一模一樣了。最後整合得到的結果沒有任何區別，但衍合能產生一個更為整潔的提交歷史。如果視察一個衍合過的分支的歷史記錄，看起來更清楚：彷彿所有修改都是先後進行的，儘管實際上它們原來是同時發生的。

你可以經常使用衍合，確保在遠程分支裡的提交歷史更清晰。比方說，某些項目自己不是維護者，但想幫點忙，就應該儘可能使用衍合：先在一個分支裡進行開發，當準備向主項目提交補丁的時候，再把它衍合到 `origin/master` 裡面。這樣，維護者就不需要做任何整合工作，只需根據你提供的倉庫地址作一次快進，或者採納你提交的補丁。

請注意，合併結果中最後一次提交所指向的快照，無論是通過一次衍合還是一次三方合併，都是同樣的快照內容，只是提交的歷史不同罷了。衍合按照每行改變發生的次序重演發生的改變，而合併是把最終結果合在一起。

### 更多有趣的衍合 ###

你還可以在衍合分支以外的地方衍合。以圖 3-31 的歷史為例。你創建了一個特性分支 `server` 來給服務器端代碼添加一些功能，然後提交 C3 和 C4。然後從 C3 的地方再增加一個 `client` 分支來對客戶端代碼進行一些修改，提交 C8 和 C9。最後，又回到 `server` 分支提交了 C10。

Insert 18333fig0331.png 
圖 3-31. 從一個特性分支裡再分出一個特性分支的歷史。

假設在接下來的一次軟件發佈中，你決定把客戶端的修改先合併到主線中，而暫緩併入服務端軟件的修改（因為還需要進一步測試）。你可以僅提取對客戶端的改變（C8 和 C9），然後通過使用 `git rebase` 的 `--onto` 選項來把它們在 master 分支上重演：

	$ git rebase --onto master server client

這基本上等於在說「檢出 client 分支，找出 `client` 分支和 `server` 分支的共同祖先之後的變化，然後把它們在 `master` 上重演一遍」。是不是有點複雜？不過它的結果，如圖 3-32 所示，非常酷：

Insert 18333fig0332.png 
圖 3-32. 衍合一個特性分支上的另一個特性分支。

現在可以快進 master 分支了（見圖 3-33）：

	$ git checkout master
	$ git merge client

Insert 18333fig0333.png 
圖 3-33. 快進 master 分支，使之包含 client 分支的變化。

現在你決定把 `server` 分支的變化也包含進來。可以直接把 `server` 分支衍合到 `master` 而不用手工轉到 `server` 分支再衍合。`git rebase [主分支] [特性分支]` 命令會先檢出特性分支 `server`，然後在主分支 `master` 上重演：

	$ git rebase master server

於是 `server` 的進度應用到 `master` 的基礎上，如圖 3-34：

Insert 18333fig0334.png 
圖 3-34. 在 master 分支上衍合 server 分支。

然後快進主分支 `master`：

	$ git checkout master
	$ git merge server

現在 `client` 和 `server` 分支的變化都被整合了，不妨刪掉它們，把你的提交歷史變成圖 3-35 的樣子：

	$ git branch -d client
	$ git branch -d server

Insert 18333fig0335.png 
圖 3-35. 最終的提交歷史

### 衍合的風險 ###

呃，奇妙的衍合也不是完美無缺的，一句話可以總結這點：

**永遠不要衍合那些已經推送到公共倉庫的更新。**

如果你遵循這條金科玉律，就不會出差錯。否則，人民群眾會仇恨你，你的朋友和家人也會嘲笑你，唾棄你。

在衍合的時候，實際上拋棄了一些現存的 commit 而創造了一些類似但不同的新 commit。如果你把commit 推送到某處然後其他人下載並在其基礎上工作，然後你用 `git rebase` 重寫了這些commit 再推送一次，你的合作者就不得不重新合併他們的工作，這樣當你再次從他們那裡獲取內容的時候事情就會變得一團糟。

下面我們用一個實際例子來說明為什麼公開的衍合會帶來問題。假設你從一個中央服務器克隆然後在它的基礎上搞了一些開發，提交歷史類似圖 3-36：

Insert 18333fig0336.png 
圖 3-36. 克隆一個倉庫，在其基礎上工作一番。

現在，其他人進行了一些包含一次合併的工作（得到結果 C6），然後把它推送到了中央服務器。你獲取了這些數據並把它們合併到你本地的開發進程裡，讓你的歷史變成類似圖 3-37 這樣：

Insert 18333fig0337.png 
圖 3-37. 獲取更多提交，併入你的開發進程。

接下來，那個推送 C6 上來的人決定用衍合取代那次合併；他們用 `git push --force` 覆蓋了服務器上的歷史，得到 C4'。然後你再從服務器上獲取更新：

Insert 18333fig0338.png 
圖 3-38. 有人推送了衍合過的 C4'，丟棄了你作為開發基礎的 C6。

這時候，你需要再次合併這些內容，儘管之前已經做過一次了。衍合會改變這些 commit 的 SHA-1 校驗值，這樣 Git 會把它們當作新的 commit，然而這時候在你的提交歷史早就有了 C4 的內容（見圖 3-39）:

Insert 18333fig0339.png 
圖 3-39. 你把相同的內容又合併了一遍，生成一個新的提交 C8。

你遲早都是要併入其他協作者提交的內容的，這樣才能保持同步。當你做完這些，你的提交歷史裡會同時包含 C4 和 C4'，兩者有著不同的 SHA-1 校驗值，但卻擁有一樣的作者日期與提交說明，令人費解！更糟糕的是，當你把這樣的歷史推送到服務器，會再次把這些衍合的提交引入到中央服務器，進一步迷惑其他人。

如果把衍合當成一種在推送之前清理提交歷史的手段，而且僅僅衍合那些永遠不會公開的 commit，那就不會有任何問題。如果衍合那些已經公開的 commit，而與此同時其他人已經用這些 commit 進行了後續的開發工作，那你有得麻煩了。

## 小結 ##

讀到這裡，你應該已經學會了如何創建分支並切換到新分支；在不同分支間轉換；合併本地分支；把分支推送到共享服務器上，同世界分享；使用共享分支與他人協作；以及在分享之前進行衍合。

