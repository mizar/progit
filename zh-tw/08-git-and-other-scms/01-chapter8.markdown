# Git 與其他系統 #

世界不是完美的。大多數時候，將所有接觸到的項目全部轉向 Git 是不可能的。有時我們不得不為某個項目使用其他的版本控制系統（VCS, Version Control System ），其中比較常見的是 Subversion 。你將在本章的第一部分學習使用 `git svn` ，Git 為 Subversion 附帶的雙向橋接工具。

或許現在你已經在考慮將先前的項目轉向 Git 。本章的第二部分將介紹如何將項目遷移到 Git：先介紹從 Subversion 的遷移，然後是 Perforce，最後介紹如何使用自定義的腳本進行非標準的導入。

## Git 與 Subversion ##

當前，大多數開發中的開源項目以及大量的商業項目都使用 Subversion 來管理源碼。作為最流行的開源版本控制系統，Subversion 已經存在了接近十年的時間。它在許多方面與 CVS 十分類似，後者是前者出現之前代碼控制世界的霸主。

Git 最為重要的特性之一是名為 `git svn` 的 Subversion 雙向橋接工具。該工具把 Git 變成了 Subversion 服務的客戶端，從而讓你在本地享受到 Git 所有的功能，而後直接向 Subversion 服務器推送內容，彷彿在本地使用了 Subversion 客戶端。也就是說，在其他人忍受古董的同時，你可以在本地享受分支合併，使暫存區域，衍合以及 單項挑揀等等。這是個讓 Git 偷偷潛入合作開發環境的好東西，在幫助你的開發同伴們提高效率的同時，它還能幫你勸說團隊讓整個項目框架轉向對 Git 的支持。這個 Subversion 之橋是通向分佈式版本控制系統（DVCS, Distributed VCS ）世界的神奇隧道。

### git svn ###

Git 中所有 Subversion 橋接命令的基礎是 `git svn` 。所有的命令都從它開始。相關的命令數目不少，你將通過幾個簡單的工作流程瞭解到其中常見的一些。

值得警戒的是，在使用 `git svn` 的時候，你實際是在與 Subversion 交互，Git 比它要高級複雜的多。儘管可以在本地隨意的進行分支和合併，最好還是通過衍合保持線性的提交歷史，儘量避免類似與遠程 Git 倉庫動態交互這樣的操作。

避免修改歷史再重新推送的做法，也不要同時推送到並行的 Git 倉庫來試圖與其他 Git 用戶合作。Subersion 只能保存單一的線性提交歷史，一不小心就會被搞糊塗。合作團隊中同時有人用 SVN 和 Git，一定要確保所有人都使用 SVN 服務來協作——這會讓生活輕鬆很多。

### 初始設定 ###

為了展示功能，先要一個具有寫權限的 SVN 倉庫。如果想嘗試這個範例，你必須複製一份其中的測試倉庫。比較簡單的做法是使用一個名為 `svnsync` 的工具。較新的 Subversion 版本中都帶有該工具，它將數據編碼為用於網絡傳輸的格式。

要嘗試本例，先在本地新建一個 Subversion 倉庫：

	$ mkdir /tmp/test-svn
	$ svnadmin create /tmp/test-svn

然後，允許所有用戶修改 revprop —— 簡單的做法是添加一個總是以 0 作為返回值的 pre-revprop-change 腳本：

	$ cat /tmp/test-svn/hooks/pre-revprop-change 
	#!/bin/sh
	exit 0;
	$ chmod +x /tmp/test-svn/hooks/pre-revprop-change

現在可以調用 `svnsync init` 加目標倉庫，再加源倉庫的格式來把該項目同步到本地了：

	$ svnsync init file:///tmp/test-svn http://progit-example.googlecode.com/svn/ 

這將建立進行同步所需的屬性。可以通過運行以下命令來克隆代碼：

	$ svnsync sync file:///tmp/test-svn
	Committed revision 1.
	Copied properties for revision 1.
	Committed revision 2.
	Copied properties for revision 2.
	Committed revision 3.
	...

別看這個操作只花掉幾分鐘，要是你想把源倉庫複製到另一個遠程倉庫，而不是本地倉庫，那將花掉接近一個小時，儘管項目中只有不到 100 次的提交。 Subversion 每次只複製一次修改，把它推送到另一個倉庫裡，然後週而復始——驚人的低效，但是我們別無選擇。

### 入門 ###

有了可以寫入的 Subversion 倉庫以後，就可以嘗試一下典型的工作流程了。我們從 `git svn clone` 命令開始，它會把整個 Subversion 倉庫導入到一個本地的 Git 倉庫中。提醒一下，這裡導入的是一個貨真價實的 Subversion 倉庫，所以應該把下面的 `file:///tmp/test-svn` 換成你所用的 Subversion 倉庫的 URL：

	$ git svn clone file:///tmp/test-svn -T trunk -b branches -t tags
	Initialized empty Git repository in /Users/schacon/projects/testsvnsync/svn/.git/
	r1 = b4e387bc68740b5af56c2a5faf4003ae42bd135c (trunk)
	      A    m4/acx_pthread.m4
	      A    m4/stl_hash.m4
	...
	r75 = d1957f3b307922124eec6314e15bcda59e3d9610 (trunk)
	Found possible branch point: file:///tmp/test-svn/trunk => \
	    file:///tmp/test-svn /branches/my-calc-branch, 75
	Found branch parent: (my-calc-branch) d1957f3b307922124eec6314e15bcda59e3d9610
	Following parent with do_switch
	Successfully followed parent
	r76 = 8624824ecc0badd73f40ea2f01fce51894189b01 (my-calc-branch)
	Checked out HEAD:
	 file:///tmp/test-svn/branches/my-calc-branch r76

這相當於針對所提供的 URL 運行了兩條命令—— `git svn init` 加上 `gitsvn fetch` 。可能會花上一段時間。我們所用的測試項目僅僅包含 75 次提交並且它的代碼量不算大，所以只有幾分鐘而已。不過，Git 仍然需要提取每一個版本，每次一個，再逐個提交。對於一個包含成百上千次提交的項目，花掉的時間則可能是幾小時甚至數天。

`-T trunk -b branches -t tags` 告訴 Git 該 Subversion 倉庫遵循了基本的分支和標籤命名法則。如果你的主幹(譯註：trunk，相當於非分佈式版本控制裡的master分支，代表開發的主線），分支或者標籤以不同的方式命名，則應做出相應改變。由於該法則的常見性，可以使用 `-s` 來代替整條命令，它意味著標準佈局（s 是 Standard layout 的首字母），也就是前面選項的內容。下面的命令有相同的效果：

	$ git svn clone file:///tmp/test-svn -s

現在，你有了一個有效的 Git 倉庫，包含著導入的分支和標籤：

	$ git branch -a
	* master
	  my-calc-branch
	  tags/2.0.2
	  tags/release-2.0.1
	  tags/release-2.0.2
	  tags/release-2.0.2rc1
	  trunk

值得注意的是，該工具分配命名空間時和遠程引用的方式不盡相同。克隆普通的 Git 倉庫時，可以以 `origin/[branch]` 的形式獲取遠程服務器上所有可用的分支——分配到遠程服務的名稱下。然而 `git svn` 假定不存在多個遠程服務器，所以把所有指向遠程服務的引用不加區分的保存下來。可以用 Git 探測命令 `show-ref` 來查看所有引用的全名。

	$ git show-ref
	1cbd4904d9982f386d87f88fce1c24ad7c0f0471 refs/heads/master
	aee1ecc26318164f355a883f5d99cff0c852d3c4 refs/remotes/my-calc-branch
	03d09b0e2aad427e34a6d50ff147128e76c0e0f5 refs/remotes/tags/2.0.2
	50d02cc0adc9da4319eeba0900430ba219b9c376 refs/remotes/tags/release-2.0.1
	4caaa711a50c77879a91b8b90380060f672745cb refs/remotes/tags/release-2.0.2
	1c4cb508144c513ff1214c3488abe66dcb92916f refs/remotes/tags/release-2.0.2rc1
	1cbd4904d9982f386d87f88fce1c24ad7c0f0471 refs/remotes/trunk

而普通的 Git 倉庫應該是這個模樣：

	$ git show-ref
	83e38c7a0af325a9722f2fdc56b10188806d83a1 refs/heads/master
	3e15e38c198baac84223acfc6224bb8b99ff2281 refs/remotes/gitserver/master
	0a30dd3b0c795b80212ae723640d4e5d48cabdff refs/remotes/origin/master
	25812380387fdd55f916652be4881c6f11600d6f refs/remotes/origin/testing

這裡有兩個遠程服務器：一個名為 `gitserver` ，具有一個 `master`分支；另一個叫 `origin`，具有 `master` 和 `testing` 兩個分支。

注意本例中通過 `git svn` 導入的遠程引用，（Subversion 的)標籤是當作遠程分支添加的，而不是真正的 Git 標籤。導入的 Subversion 倉庫彷彿是有一個帶有不同分支的 tags 遠程服務器。

### 提交到 Subversion ###

有了可以開展工作的（本地）倉庫以後，你可以開始對該項目做出貢獻並向上游倉庫提交內容了，Git 這時相當於一個 SVN 客戶端。假如編輯了一個文件並進行提交，那麼這次提交僅存在於本地的 Git 而非 Subversion 服務器上。

	$ git commit -am 'Adding git-svn instructions to the README'
	[master 97031e5] Adding git-svn instructions to the README
	 1 files changed, 1 insertions(+), 1 deletions(-)

接下來，可以將作出的修改推送到上游。值得注意的是，Subversion 的使用流程也因此改變了——你可以在離線狀態下進行多次提交然後一次性的推送到 Subversion 的服務器上。向 Subversion 服務器推送的命令是 `git svn dcommit`：

	$ git svn dcommit
	Committing to file:///tmp/test-svn/trunk ...
	       M      README.txt
	Committed r79
	       M      README.txt
	r79 = 938b1a547c2cc92033b74d32030e86468294a5c8 (trunk)
	No changes between current HEAD and refs/remotes/trunk
	Resetting to the latest refs/remotes/trunk

所有在原 Subversion 數據基礎上提交的 commit 會一一提交到 Subversion，然後你本地 Git 的 commit 將被重寫，加入一個特別標識。這一步很重要，因為它意味著所有 commit 的 SHA-1 指都會發生變化。這也是同時使用 Git 和 Subversion 兩種服務作為遠程服務不是個好主意的原因之一。檢視以下最後一個 commit，你會找到新添加的 `git-svn-id` （譯註：即本段開頭所說的特別標識）：

	$ git log -1
	commit 938b1a547c2cc92033b74d32030e86468294a5c8
	Author: schacon <schacon@4c93b258-373f-11de-be05-5f7a86268029>
	Date:   Sat May 2 22:06:44 2009 +0000

	    Adding git-svn instructions to the README

	    git-svn-id: file:///tmp/test-svn/trunk@79 4c93b258-373f-11de-be05-5f7a86268029

注意看，原本以 `97031e5` 開頭的 SHA-1 校驗值在提交完成以後變成了 `938b1a5` 。如果既要向 Git 遠程服務器推送內容，又要推送到 Subversion 遠程服務器，則必須先向 Subversion 推送（`dcommit`），因為該操作會改變所提交的數據內容。

### 拉取最新進展 ###

如果要與其他開發者協作，總有那麼一天你推送完畢之後，其他人發現他們推送自己修改的時候（與你推送的內容）產生衝突。這些修改在你合併之前將一直被拒絕。在 `git svn` 裡這種情況形似：

	$ git svn dcommit
	Committing to file:///tmp/test-svn/trunk ...
	Merge conflict during commit: Your file or directory 'README.txt' is probably \
	out-of-date: resource out of date; try updating at /Users/schacon/libexec/git-\
	core/git-svn line 482

為瞭解決該問題，可以運行 `git svn rebase` ，它會拉取服務器上所有最新的改變，再次基礎上衍合你的修改：

	$ git svn rebase
	       M      README.txt
	r80 = ff829ab914e8775c7c025d741beb3d523ee30bc4 (trunk)
	First, rewinding head to replay your work on top of it...
	Applying: first user change

現在，你做出的修改都發生在服務器內容之後，所以可以順利的運行 `dcommit` ：

	$ git svn dcommit
	Committing to file:///tmp/test-svn/trunk ...
	       M      README.txt
	Committed r81
	       M      README.txt
	r81 = 456cbe6337abe49154db70106d1836bc1332deed (trunk)
	No changes between current HEAD and refs/remotes/trunk
	Resetting to the latest refs/remotes/trunk

需要牢記的一點是，Git 要求我們在推送之前先合併上游倉庫中最新的內容，而 `git svn` 只要求存在衝突的時候才這樣做。假如有人向一個文件推送了一些修改，這時你要向另一個文件推送一些修改，那麼 `dcommit` 將正常工作：

	$ git svn dcommit
	Committing to file:///tmp/test-svn/trunk ...
	       M      configure.ac
	Committed r84
	       M      autogen.sh
	r83 = 8aa54a74d452f82eee10076ab2584c1fc424853b (trunk)
	       M      configure.ac
	r84 = cdbac939211ccb18aa744e581e46563af5d962d0 (trunk)
	W: d2f23b80f67aaaa1f6f5aaef48fce3263ac71a92 and refs/remotes/trunk differ, \
	  using rebase:
	:100755 100755 efa5a59965fbbb5b2b0a12890f1b351bb5493c18 \
	  015e4c98c482f0fa71e4d5434338014530b37fa6 M   autogen.sh
	First, rewinding head to replay your work on top of it...
	Nothing to do.

這一點需要牢記，因為它的結果是推送之後項目處於一個不完整存在與任何主機上的狀態。如果做出的修改無法兼容但沒有產生衝突，則可能造成一些很難確診的難題。這和使用 Git 服務器是不同的——在 Git 世界裡，發佈之前，你可以在客戶端系統裡完整的測試項目的狀態，而在 SVN 永遠都沒法確保提交前後項目的狀態完全一樣。

及時還沒打算進行提交，你也應該用這個命令從 Subversion 服務器拉取最新修改。`sit svn fetch` 能獲取最新的數據，不過 `git svn rebase` 才會在獲取之後在本地進行更新 。

	$ git svn rebase
	       M      generate_descriptor_proto.sh
	r82 = bd16df9173e424c6f52c337ab6efa7f7643282f1 (trunk)
	First, rewinding head to replay your work on top of it...
	Fast-forwarded master to refs/remotes/trunk.

不時地運行一下 `git svn rebase` 可以確保你的代碼沒有過時。不過，運行該命令時需要確保工作目錄的整潔。如果在本地做了修改，則必須在運行 `git svn rebase` 之前或暫存工作，或暫時提交內容——否則，該命令會發現衍合的結果包含著衝突因而終止。

### Git 分支問題 ###

習慣了 Git 的工作流程以後，你可能會創建一些特性分支，完成相關的開發工作，然後合併他們。如果要用 git svn 向 Subversion 推送內容，那麼最好是每次用衍合來併入一個單一分支，而不是直接合併。使用衍合的原因是 Subversion 只有一個線性的歷史而不像 Git 那樣處理合併，所以 Git svn 在把快照轉換為 Subversion 的 commit 時只能包含第一個祖先。

假設分支歷史如下：創建一個 `experiment` 分支，進行兩次提交，然後合併到 `master` 。在 `dcommit` 的時候會得到如下輸出：

	$ git svn dcommit
	Committing to file:///tmp/test-svn/trunk ...
	       M      CHANGES.txt
	Committed r85
	       M      CHANGES.txt
	r85 = 4bfebeec434d156c36f2bcd18f4e3d97dc3269a2 (trunk)
	No changes between current HEAD and refs/remotes/trunk
	Resetting to the latest refs/remotes/trunk
	COPYING.txt: locally modified
	INSTALL.txt: locally modified
	       M      COPYING.txt
	       M      INSTALL.txt
	Committed r86
	       M      INSTALL.txt
	       M      COPYING.txt
	r86 = 2647f6b86ccfcaad4ec58c520e369ec81f7c283c (trunk)
	No changes between current HEAD and refs/remotes/trunk
	Resetting to the latest refs/remotes/trunk

在一個包含了合併歷史的分支上使用 `dcommit` 可以成功運行，不過在 Git 項目的歷史中，它沒有重寫你在 `experiment` 分支中的兩個 commit ——另一方面，這些改變卻出現在了 SVN 版本中同一個合併 commit 中。

在別人克隆該項目的時候，只能看到這個合併 commit 包含了所有發生過的修改；他們無法獲知修改的作者和時間等提交信息。

### Subversion 分支 ###

Subversion 的分支和 Git 中的不盡相同；避免過多的使用可能是最好方案。不過，用 git svn 創建和提交不同的 Subversion 分支仍是可行的。

#### 創建新的 SVN 分支 ####

要在 Subversion 中建立一個新分支，需要運行 `git svn branch [分支名]`
To create a new branch in Subversion, you run `git svn branch [branchname]`:

	$ git svn branch opera
	Copying file:///tmp/test-svn/trunk at r87 to file:///tmp/test-svn/branches/opera...
	Found possible branch point: file:///tmp/test-svn/trunk => \
	  file:///tmp/test-svn/branches/opera, 87
	Found branch parent: (opera) 1f6bfe471083cbca06ac8d4176f7ad4de0d62e5f
	Following parent with do_switch
	Successfully followed parent
	r89 = 9b6fe0b90c5c9adf9165f700897518dbc54a7cbf (opera)

相當於在 Subversion 中的 `svn copy trunk branches/opera` 命令並且對 Subversion 服務器進行了相關操作。值得提醒的是它沒有檢出和轉換到那個分支；如果現在進行提交，將提交到服務器上的 `trunk`， 而非 `opera`。

### 切換當前分支 ###

Git 通過搜尋提交歷史中 Subversion 分支的頭部來決定 dcommit 的目的地——而它應該只有一個，那就是當前分支歷史中最近一次包含 `git-svn-id` 的提交。

如果需要同時在多個分支上提交，可以通過導入 Subversion 上某個其他分支的 commit 來建立以該分支為 `dcommit` 目的地的本地分支。比如你想擁有一個並行維護的 `opera` 分支，可以運行

	$ git branch opera remotes/opera

然後，如果要把 `opera` 分支併入 `trunk` （本地的 `master` 分支），可以使用普通的 `git merge`。不過最好提供一條描述提交的信息（通過 `-m`），否則這次合併的記錄是 `Merge branch opera` ，而不是任何有用的東西。

記住，雖然使用了 `git merge` 來進行這次操作，並且合併過程可能比使用 Subversion 簡單一些（因為 Git 會自動找到適合的合併基礎），這並不是一次普通的 Git 合併提交。最終它將被推送回 commit 無法包含多個祖先的 Subversion 服務器上；因而在推送之後，它將變成一個包含了所有在其他分支上做出的改變的單一 commit。把一個分支合併到另一個分支以後，你沒法像在 Git 中那樣輕易的回到那個分支上繼續工作。提交時運行的 `dcommit` 命令擦除了全部有關哪個分支被併入的信息，因而以後的合併基礎計算將是不正確的—— dcommit 讓 `git merge` 的結果變得類似於 `git merge --squash`。不幸的是，我們沒有什麼好辦法來避免該情況—— Subversion 無法儲存這個信息，所以在使用它作為服務器的時候你將永遠為這個缺陷所困。為了不出現這種問題，在把本地分支（本例中的 `opera`）併入 trunk 以後應該立即將其刪除。

### 對應 Subversion 的命令 ###

`git svn` 工具集合了若幹個與 Subversion 類似的功能，對應的命令可以簡化向 Git 的轉化過程。下面這些命令能實現 Subversion 的這些功能。

#### SVN 風格的歷史 ####

習慣了 Subversion 的人可能想以 SVN 的風格顯示歷史，運行 `git svn log`  可以讓提交歷史顯示為 SVN 格式：

	$ git svn log
	------------------------------------------------------------------------
	r87 | schacon | 2009-05-02 16:07:37 -0700 (Sat, 02 May 2009) | 2 lines

	autogen change

	------------------------------------------------------------------------
	r86 | schacon | 2009-05-02 16:00:21 -0700 (Sat, 02 May 2009) | 2 lines

	Merge branch 'experiment'

	------------------------------------------------------------------------
	r85 | schacon | 2009-05-02 16:00:09 -0700 (Sat, 02 May 2009) | 2 lines
	
	updated the changelog

關於 `git svn log` ，有兩點需要注意。首先，它可以離線工作，不像 `svn log` 命令，需要向 Subversion 服務器索取數據。其次，它僅僅顯示已經提交到 Subversion 服務器上的 commit。在本地尚未 dcommit 的 Git 數據不會出現在這裡；其他人向 Subversion 服務器新提交的數據也不會顯示。等於說是顯示了最近已知 Subversion 服務器上的狀態。

#### SVN 日誌 ####

類似 `git svn log` 對 `git log` 的模擬，`svn annotate` 的等效命令是 `git svn blame [文件名]`。其輸出如下：

	$ git svn blame README.txt 
	 2   temporal Protocol Buffers - Google's data interchange format
	 2   temporal Copyright 2008 Google Inc.
	 2   temporal http://code.google.com/apis/protocolbuffers/
	 2   temporal 
	22   temporal C++ Installation - Unix
	22   temporal =======================
	 2   temporal 
	79    schacon Committing in git-svn.
	78    schacon 
	 2   temporal To build and install the C++ Protocol Buffer runtime and the Protocol
	 2   temporal Buffer compiler (protoc) execute the following:
	 2   temporal 

同樣，它不顯示本地的 Git 提交以及 Subversion 上後來更新的內容。

#### SVN 服務器信息 ####

還可以使用 `git svn info` 來獲取與運行 `svn info` 類似的信息：

	$ git svn info
	Path: .
	URL: https://schacon-test.googlecode.com/svn/trunk
	Repository Root: https://schacon-test.googlecode.com/svn
	Repository UUID: 4c93b258-373f-11de-be05-5f7a86268029
	Revision: 87
	Node Kind: directory
	Schedule: normal
	Last Changed Author: schacon
	Last Changed Rev: 87
	Last Changed Date: 2009-05-02 16:07:37 -0700 (Sat, 02 May 2009)

它與 `blame` 和 `log` 的相同點在於離線運行以及只更新到最後一次與 Subversion 服務器通信的狀態。

#### 略 Subversion 之所略 ####

假如克隆了一個包含了 `svn:ignore` 屬性的 Subversion 倉庫，就有必要建立對應的 `.gitignore` 文件來防止意外提交一些不應該提交的文件。`git svn` 有兩個有益於改善該問題的命令。第一個是 `git svn create-ignore`，它自動建立對應的 `.gitignore` 文件，以便下次提交的時候可以包含它。

第二個命令是 `git svn show-ignore`，它把需要放進 `.gitignore` 文件中的內容打印到標準輸出，方便我們把輸出重定向到項目的黑名單文件：

	$ git svn show-ignore > .git/info/exclude

這樣一來，避免了 `.gitignore` 對項目的干擾。如果你是一個 Subversion 團隊裡唯一的 Git 用戶，而其他隊友不喜歡項目包含 `.gitignore`，該方法是你的不二之選。

### Git-Svn 總結 ###

`git svn` 工具集在當前不得不使用 Subversion 服務器或者開發環境要求使用 Subversion 服務器的時候格外有用。不妨把它看成一個跛腳的 Git，然而，你還是有可能在轉換過程中碰到一些困惑你和合作者們的迷題。為了避免麻煩，試著遵守如下守則：

* 保持一個不包含由 `git merge` 生成的 commit 的線性提交歷史。將在主線分支外進行的開發通通衍合回主線；避免直接合併。
* 不要單獨建立和使用一個 Git 服務來搞合作。可以為了加速新開發者的克隆進程建立一個，但是不要向它提供任何不包含 `git-svn-id` 條目的內容。甚至可以添加一個 `pre-receive` 掛鉤來在每一個提交信息中查找 `git-svn-id` 並拒絕提交那些不包含它的 commit。

如果遵循這些守則，在 Subversion 上工作還可以接受。然而，如果能遷徙到真正的 Git 服務器，則能為團隊帶來更多好處。

## 遷移到 Git ##

如果在其他版本控制系統中保存了某項目的代碼而後決定轉而使用 Git，那麼該項目必須經歷某種形式的遷移。本節將介紹 Git 中包含的一些針對常見系統的導入腳本，並將展示編寫自定義的導入腳本的方法。

### 導入 ###

你將學習到如何從專業重量級的版本控制系統中導入數據—— Subversion 和 Perforce —— 因為據我所知這二者的用戶是（向 Git）轉換的主要群體，而且 Git 為此二者附帶了高質量的轉換工具。

### Subversion ###

讀過前一節有關 `git svn` 的內容以後，你應該能輕而易舉的根據其中的指導來 `git svn clone` 一個倉庫了；然後，停止 Subversion 的使用，向一個新 Git server 推送，並開始使用它。想保留歷史記錄，所畫的時間應該不過就是從 Subversion 服務器拉取數據的時間（可能要等上好一會就是了）。

然而，這樣的導入並不完美；而且還要花那麼多時間，不如乾脆一次把它做對！首當其衝的任務是作者信息。在 Subversion，每個提交者在都在主機上有一個用戶名，記錄在提交信息中。上節例子中多處顯示了 `schacon` ，比如 `blame` 的輸出以及 `git svn log`。如果想讓這條信息更好的映射到 Git 作者數據裡，則需要 從 Subversion 用戶名到 Git 作者的一個映射關係。建立一個叫做 `user.txt` 的文件，用如下格式表示映射關係：

	schacon = Scott Chacon <schacon@geemail.com>
	selse = Someo Nelse <selse@geemail.com>

通過該命令可以獲得 SVN 作者的列表：

	$ svn log --xml | grep author | sort -u | perl -pe 's/.>(.?)<./$1 = /'

它將輸出 XML 格式的日誌——你可以找到作者，建立一個單獨的列表，然後從 XML 中抽取出需要的信息。（顯而易見，本方法要求主機上安裝了`grep`，`sort` 和 `perl`.）然後把輸出重定向到 user.txt 文件，然後就可以在每一項的後面添加相應的 Git 用戶數據。

為 `git svn` 提供該文件可以然它更精確的映射作者數據。你還可以在 `clone` 或者 `init`後面添加 `--no-metadata` 來阻止 `git svn` 包含那些 Subversion 的附加信息。這樣 `import` 命令就變成了：

	$ git-svn clone http://my-project.googlecode.com/svn/ \
	      --authors-file=users.txt --no-metadata -s my_project

現在 `my_project` 目錄下導入的 Subversion 應該比原來整潔多了。原來的 commit 看上去是這樣：

	commit 37efa680e8473b615de980fa935944215428a35a
	Author: schacon <schacon@4c93b258-373f-11de-be05-5f7a86268029>
	Date:   Sun May 3 00:12:22 2009 +0000

	    fixed install - go to trunk

	    git-svn-id: https://my-project.googlecode.com/svn/trunk@94 4c93b258-373f-11de-
	    be05-5f7a86268029
現在是這樣：

	commit 03a8785f44c8ea5cdb0e8834b7c8e6c469be2ff2
	Author: Scott Chacon <schacon@geemail.com>
	Date:   Sun May 3 00:12:22 2009 +0000

	    fixed install - go to trunk

不僅作者一項乾淨了不少，`git-svn-id` 也就此消失了。

你還需要一點 `post-import（導入後）` 清理工作。最起碼的，應該清理一下 `git svn` 創建的那些怪異的索引結構。首先要移動標籤，把它們從奇怪的遠程分支變成實際的標籤，然後把剩下的分支移動到本地。

要把標籤變成合適的 Git 標籤，運行

	$ cp -Rf .git/refs/remotes/tags/* .git/refs/tags/
	$ rm -Rf .git/refs/remotes/tags

該命令將原本以 `tag/` 開頭的遠程分支的索引變成真正的（輕巧的）標籤。

接下來，把 `refs/remotes` 下面剩下的索引變成本地分支：

	$ cp -Rf .git/refs/remotes/* .git/refs/heads/
	$ rm -Rf .git/refs/remotes

現在所有的舊分支都變成真正的 Git 分支，所有的舊標籤也變成真正的 Git 標籤。最後一項工作就是把新建的 Git 服務器添加為遠程服務器並且向它推送。為了讓所有的分支和標籤都得到上傳，我們使用這條命令：

	$ git push origin --all

所有的分支和標籤現在都應該整齊乾淨的躺在新的 Git 服務器裡了。

### Perforce ###

你將瞭解到的下一個被導入的系統是 Perforce. Git 發行的時候同時也附帶了一個 Perforce 導入腳本，不過它是包含在源碼的 `contrib` 部分——而不像 `git svn` 那樣默認可用。運行它之前必須獲取 Git 的源碼，可以在 git.kernel.org 下載：

	$ git clone git://git.kernel.org/pub/scm/git/git.git
	$ cd git/contrib/fast-import

在這個 `fast-import` 目錄下，應該有一個叫做 `git-p4` 的 Python 可執行腳本。主機上必須裝有 Python 和 `p4` 工具該導入才能正常進行。例如，你要從 Perforce 公共代碼倉庫（譯註： Perforce Public Depot，Perforce 官方提供的代碼寄存服務）導入 Jam 工程。為了設定客戶端，我們要把 P4PORT 環境變量 export 到 Perforce 倉庫：

	$ export P4PORT=public.perforce.com:1666

運行 `git-p4 clone` 命令將從 Perforce 服務器導入 Jam 項目，我們需要給出倉庫和項目的路徑以及導入的目標路徑：

	$ git-p4 clone //public/jam/src@all /opt/p4import
	Importing from //public/jam/src@all into /opt/p4import
	Reinitialized existing Git repository in /opt/p4import/.git/
	Import destination: refs/remotes/p4/master
	Importing revision 4409 (100%)

現在去 `/opt/p4import` 目錄運行一下 `git log` ，就能看到導入的成果：

	$ git log -2
	commit 1fd4ec126171790efd2db83548b85b1bbbc07dc2
	Author: Perforce staff <support@perforce.com>
	Date:   Thu Aug 19 10:18:45 2004 -0800

	    Drop 'rc3' moniker of jam-2.5.  Folded rc2 and rc3 RELNOTES into
	    the main part of the document.  Built new tar/zip balls.

	    Only 16 months later.

	    [git-p4: depot-paths = "//public/jam/src/": change = 4409]

	commit ca8870db541a23ed867f38847eda65bf4363371d
	Author: Richard Geiger <rmg@perforce.com>
	Date:   Tue Apr 22 20:51:34 2003 -0800

	    Update derived jamgram.c

	    [git-p4: depot-paths = "//public/jam/src/": change = 3108]

每一個 commit 裡都有一個 `git-p4` 標識符。這個標識符可以保留，以防以後需要引用 Perforce 的修改版本號。然而，如果想刪除這些標識符，現在正是時候——在開啟新倉庫之前。可以通過 `git filter-branch` 來批量刪除這些標識符：

	$ git filter-branch --msg-filter '
	        sed -e "/^\[git-p4:/d"
	'
	Rewrite 1fd4ec126171790efd2db83548b85b1bbbc07dc2 (123/123)
	Ref 'refs/heads/master' was rewritten

現在運行一下 `git log`，你會發現這些 commit 的 SHA-1 校驗值都發生了改變，而那些 `git-p4` 字串則從提交信息裡消失了：

	$ git log -2
	commit 10a16d60cffca14d454a15c6164378f4082bc5b0
	Author: Perforce staff <support@perforce.com>
	Date:   Thu Aug 19 10:18:45 2004 -0800

	    Drop 'rc3' moniker of jam-2.5.  Folded rc2 and rc3 RELNOTES into
	    the main part of the document.  Built new tar/zip balls.

	    Only 16 months later.

	commit 2b6c6db311dd76c34c66ec1c40a49405e6b527b2
	Author: Richard Geiger <rmg@perforce.com>
	Date:   Tue Apr 22 20:51:34 2003 -0800

	    Update derived jamgram.c

至此導入已經完成，可以開始向新的 Git 服務器推送了。

### 自定導入腳本 ###

如果先前的系統不是 Subversion 或 Perforce 之一，先上網找一下有沒有與之對應的導入腳本——導入 CVS，Clear Case，Visual Source Safe，甚至存檔目錄的導入腳本已經存在。假如這些工具都不適用，或者使用的工具很少見，抑或你需要導入過程具有更多可制定性，則應該使用 `git fast-import`。該命令從標準輸入讀取簡單的指令來寫入具體的 Git 數據。這樣創建 Git 對象比運行純 Git 命令或者手動寫對象要簡單的多（更多相關內容見第九章）。通過它，你可以編寫一個導入腳本來從導入源讀取必要的信息，同時在標準輸出直接輸出相關指示。你可以運行該腳本並把它的輸出管道連接到 `git fast-import`。

下面演示一下如何編寫一個簡單的導入腳本。假設你在進行一項工作，並且按時通過把工作目錄複製為以時間戳 `back_YY_MM_DD` 命名的目錄來進行備份，現在你需要把它們導入 Git 。目錄結構如下：

	$ ls /opt/import_from
	back_2009_01_02
	back_2009_01_04
	back_2009_01_14
	back_2009_02_03
	current

為了導入到一個 Git 目錄，我們首先回顧一下 Git 儲存數據的方式。你可能還記得，Git 本質上是一個 commit 對象的鏈表，每一個對象指向一個內容的快照。而這裡需要做的工作就是告訴 `fast-import` 內容快照的位置，什麼樣的 commit 數據指向它們，以及它們的順序。我們採取一次處理一個快照的策略，為每一個內容目錄建立對應的 commit ，每一個 commit 與之前的建立鏈接。

正如在第七章 "Git 執行策略一例" 一節中一樣，我們將使用 Ruby 來編寫這個腳本，因為它是我日常使用的語言而且閱讀起來簡單一些。你可以用任何其他熟悉的語言來重寫這個例子——它僅需要把必要的信息打印到標準輸出而已。同時，如果你在使用 Windows，這意味著你要特別留意不要在換行的時候引入回車符（譯註：carriage returns，Windows換行時加入的符號，通常說的\r ）—— Git 的 fast-import 對僅使用換行符（LF）而非 Windows 的回車符（CRLF）要求非常嚴格。

首先，進入目標目錄並且找到所有子目錄，每一個子目錄將作為一個快照被導入為一個 commit。我們將依次進入每一個子目錄並打印所需的命令來導出它們。腳本的主循環大致是這樣：

	last_mark = nil

	# 循環遍歷所有目錄
	Dir.chdir(ARGV[0]) do
	  Dir.glob("*").each do |dir|
	    next if File.file?(dir)

	    # 進入目標目錄
	    Dir.chdir(dir) do 
	      last_mark = print_export(dir, last_mark)
	    end
	  end
	end

我們在每一個目錄裡運行 `print_export` ，它會取出上一個快照的索引和標記並返回本次快照的索引和標記；由此我們就可以正確的把二者連接起來。"標記（mark）" 是 `fast-import` 中對 commit 標識符的叫法；在創建 commit 的同時，我們逐一賦予一個標記以便以後在把它連接到其他 commit 時使用。因此，在 `print_export` 方法中要做的第一件事就是根據目錄名生成一個標記：

	mark = convert_dir_to_mark(dir)

實現該函數的方法是建立一個目錄的數組序列並使用數組的索引值作為標記，因為標記必須是一個整數。這個方法大致是這樣的：

	$marks = []
	def convert_dir_to_mark(dir)
	  if !$marks.include?(dir)
	    $marks << dir
	  end
	  ($marks.index(dir) + 1).to_s
	end

有了整數來代表每個 commit，我們現在需要提交附加信息中的日期。由於日期是用目錄名表示的，我們就從中解析出來。`print_export` 文件的下一行將是：

	date = convert_dir_to_date(dir)

而 `convert_dir_to_date` 則定義為

	def convert_dir_to_date(dir)
	  if dir == 'current'
	    return Time.now().to_i
	  else
	    dir = dir.gsub('back_', '')
	    (year, month, day) = dir.split('_')
	    return Time.local(year, month, day).to_i
	  end
	end

它為每個目錄返回一個整型值。提交附加信息裡最後一項所需的是提交者數據，我們在一個全局變量中直接定義之：

	$author = 'Scott Chacon <schacon@example.com>'

我們差不多可以開始為導入腳本輸出提交數據了。第一項信息指明我們定義的是一個 commit 對象以及它所在的分支，隨後是我們生成的標記，提交者信息以及提交備註，然後是前一個 commit 的索引，如果有的話。代碼大致這樣：

	# 打印導入所需的信息
	puts 'commit refs/heads/master'
	puts 'mark :' + mark
	puts "committer #{$author} #{date} -0700"
	export_data('imported from ' + dir)
	puts 'from :' + last_mark if last_mark

時區（-0700）處於簡化目的使用硬編碼。如果是從其他版本控制系統導入，則必須以變量的形式指明時區。
提交備註必須以特定格式給出：

	data (size)\n(contents)

該格式包含了單詞 data，所讀取數據的大小，一個換行符，最後是數據本身。由於隨後指明文件內容的時候要用到相同的格式，我們寫一個輔助方法，`export_data`：

	def export_data(string)
	  print "data #{string.size}\n#{string}"
	end

唯一剩下的就是每一個快照的內容了。這簡單的很，因為它們分別處於一個目錄——你可以輸出 `deleeall` 命令，隨後是目錄中每個文件的內容。Git 會正確的記錄每一個快照：

	puts 'deleteall'
	Dir.glob("**/*").each do |file| next if !File.file?(file)
	  inline_data(file)
	end

注意：由於很多系統把每次修訂看作一個 commit 到另一個 commit 的變化量，fast-import 也可以依據每次提交獲取一個命令來指出哪些文件被添加，刪除或者修改過，以及修改的內容。我們將需要計算快照之間的差別並且僅僅給出這項數據，不過該做法要複雜很多——還如不直接把所有數據丟給 Git 然它自己搞清楚。假如前面這個方法更適用於你的數據，參考 `fast-import` 的 man 幫助頁面來瞭解如何以這種方式提供數據。

列舉新文件內容或者指明帶有新內容的已修改文件的格式如下：

	M 644 inline path/to/file
	data (size)
	(file contents)

這裡，644 是權限模式（加入有可執行文件，則需要探測之並設定為 755），而 inline 說明我們在本行結束之後立即列出文件的內容。我們的 `inline_data` 方法大致是：

	def inline_data(file, code = 'M', mode = '644')
	  content = File.read(file)
	  puts "#{code} #{mode} inline #{file}"
	  export_data(content)
	end

我們重用了前面定義過的 `export_data`，因為這裡和指明提交註釋的格式如出一轍。

最後一項工作是返回當前的標記以便下次循環的使用。

	return mark

注意：如果你在用 Windows，一定記得添加一項額外的步驟。前面提過，Windows 使用 CRLF 作為換行字符而 Git fast-import 只接受 LF。為了繞開這個問題來滿足 git fast-import，你需要讓 ruby 用 LF 取代 CRLF：

	$stdout.binmode

搞定了。現在運行該腳本，你將得到如下內容：

	$ ruby import.rb /opt/import_from 
	commit refs/heads/master
	mark :1
	committer Scott Chacon <schacon@geemail.com> 1230883200 -0700
	data 29
	imported from back_2009_01_02deleteall
	M 644 inline file.rb
	data 12
	version two
	commit refs/heads/master
	mark :2
	committer Scott Chacon <schacon@geemail.com> 1231056000 -0700
	data 29
	imported from back_2009_01_04from :1
	deleteall
	M 644 inline file.rb
	data 14
	version three
	M 644 inline new.rb
	data 16
	new version one
	(...)

要運行導入腳本，在需要導入的目錄把該內容用管道定向到 `git fast-import`。你可以建立一個空目錄然後運行 `git init` 作為開頭，然後運行該腳本：

	$ git init
	Initialized empty Git repository in /opt/import_to/.git/
	$ ruby import.rb /opt/import_from | git fast-import
	git-fast-import statistics:
	---------------------------------------------------------------------
	Alloc'd objects:       5000
	Total objects:           18 (         1 duplicates                  )
	      blobs  :            7 (         1 duplicates          0 deltas)
	      trees  :            6 (         0 duplicates          1 deltas)
	      commits:            5 (         0 duplicates          0 deltas)
	      tags   :            0 (         0 duplicates          0 deltas)
	Total branches:           1 (         1 loads     )
	      marks:           1024 (         5 unique    )
	      atoms:              3
	Memory total:          2255 KiB
	       pools:          2098 KiB
	     objects:           156 KiB
	---------------------------------------------------------------------
	pack_report: getpagesize()            =       4096
	pack_report: core.packedGitWindowSize =   33554432
	pack_report: core.packedGitLimit      =  268435456
	pack_report: pack_used_ctr            =          9
	pack_report: pack_mmap_calls          =          5
	pack_report: pack_open_windows        =          1 /          1
	pack_report: pack_mapped              =       1356 /       1356
	---------------------------------------------------------------------

你會發現，在它成功執行完畢以後，會給出一堆有關已完成工作的數據。上例在一個分支導入了5次提交數據，包含了18個對象。現在可以運行 `git log` 來檢視新的歷史：

	$ git log -2
	commit 10bfe7d22ce15ee25b60a824c8982157ca593d41
	Author: Scott Chacon <schacon@example.com>
	Date:   Sun May 3 12:57:39 2009 -0700

	    imported from current

	commit 7e519590de754d079dd73b44d695a42c9d2df452
	Author: Scott Chacon <schacon@example.com>
	Date:   Tue Feb 3 01:00:00 2009 -0700

	    imported from back_2009_02_03

就它了——一個乾淨整潔的 Git 倉庫。需要注意的是此時沒有任何內容被檢出——剛開始當前目錄裡沒有任何文件。要獲取它們，你得轉到 `master` 分支的所在：

	$ ls
	$ git reset --hard master
	HEAD is now at 10bfe7d imported from current
	$ ls
	file.rb  lib

`fast-import` 還可以做更多——處理不同的文件模式，二進制文件，多重分支與合併，標籤，進展標識等等。一些更加複雜的實例可以在 Git 源碼的 `contib/fast-import` 目錄裡找到；其中較為出眾的是前面提過的 `git-p4` 腳本。

## 總結 ##

現在的你應該掌握了在 Subversion 上使用 Git 以及把幾乎任何先存倉庫無損失的導入為 Git 倉庫。下一章將介紹 Git 內部的原始數據格式，從而是使你能親手鍛造其中的每一個字節，如果必要的話。

