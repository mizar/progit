# 自定義Git #

到目前為止，我闡述了Git基本的運作機制和使用方式，介紹了Git提供的許多工具來幫助你簡單且有效地使用它。
在本章當中，我將會介紹Git的一些重要的配置方法和鉤子機制以滿足你自定義的要求，通過這些方法，它會和你、你的公司或團隊配合得天衣無縫。

## 配置Git ##

正如你在第一章見到的那樣，你能用`git config`配置Git，要做的第一件事就是設置名字和郵箱地址：

	$ git config --global user.name "John Doe"
	$ git config --global user.email johndoe@example.com

從現在開始，你會瞭解到一些更為有趣的設置選項，按照以上方式來自定義Git。

我會在這先過一遍第一章中提到的Git配置細節。Git使用一系列的配置文件來存儲你定義的偏好，它首先會查找`/etc/gitconfig`文件，該文件含有
對系統上所有用戶及他們所擁有的倉庫都生效的配置值（譯註：gitconfig是全局配置文件），
如果傳遞`--system`選項給`git config`命令，Git會讀寫這個文件。

接下來Git會查找每個用戶的`~/.gitconfig`文件，你能傳遞`--global`選項讓Git讀寫該文件。

最後Git會查找由用戶定義的各個庫中Git目錄下的配置文件（`.git/config`），該文件中的值只對屬主庫有效。
以上闡述的三層配置從一般到特殊層層推進，如果定義的值有衝突，以後面層中定義的為準，例如：在`.git/config`和`/etc/gitconfig`的較量中，
`.git/config`取得了勝利。雖然你也可以直接手動編輯這些配置文件，但是運行`git config`命令將會來得簡單些。

### 客戶端基本配置 ###

Git能夠識別的配置項被分為了兩大類：客戶端和服務器端，其中大部分基於你個人工作偏好，屬於客戶端配置。儘管有數不盡的選項，但我只闡述
其中經常使用或者會對你的工作流產生巨大影響的選項，如果你想觀察你當前的Git能識別的選項列表，請運行

	$ git config --help

`git config`的手冊頁（譯註：以man命令的顯示方式）非常細緻地羅列了所有可用的配置項。

#### core.editor ####

Git默認會調用你的環境變量editor定義的值作為文本編輯器，如果沒有定義的話，會調用Vi來創建和編輯提交以及標籤信息，
你可以使用`core.editor`改變默認編輯器：

	$ git config --global core.editor emacs

現在無論你的環境變量editor被定義成什麼，Git都會調用Emacs編輯信息。

#### commit.template ####

如果把此項指定為你系統上的一個文件，當你提交的時候，Git會默認使用該文件定義的內容。
例如：你創建了一個模板文件`$HOME/.gitmessage.txt`，它看起來像這樣：

	subject line

	what happened

	[ticket: X]

設置`commit.template`，當運行`git commit`時，Git會在你的編輯器中顯示以上的內容，
設置`commit.template`如下：

	$ git config --global commit.template $HOME/.gitmessage.txt
	$ git commit

然後當你提交時，在編輯器中顯示的提交信息如下：

	subject line

	what happened

	[ticket: X]
	# Please enter the commit message for your changes. Lines starting
	# with '#' will be ignored, and an empty message aborts the commit.
	# On branch master
	# Changes to be committed:
	#   (use "git reset HEAD <file>..." to unstage)
	#
	# modified:   lib/test.rb
	#
	~
	~
	".git/COMMIT_EDITMSG" 14L, 297C

如果你有特定的策略要運用在提交信息上，在系統上創建一個模板文件，設置Git默認使用它，這樣當提交時，你的策略每次都會被運用。

#### core.pager ####

core.pager指定Git運行諸如`log`、`diff`等所使用的分頁器，你能設置成用`more`或者任何你喜歡的分頁器（默認用的是`less`），
當然你也可以什麼都不用，設置空字符串：

	$ git config --global core.pager ''

這樣不管命令的輸出量多少，都會在一頁顯示所有內容。

#### user.signingkey ####

如果你要創建經簽署的含附註的標籤（正如第二章所述），那麼把你的GPG簽署密鑰設置為配置項會更好，設置密鑰ID如下：

	$ git config --global user.signingkey <gpg-key-id>

現在你能夠簽署標籤，從而不必每次運行`git tag`命令時定義密鑰：

	$ git tag -s <tag-name>

#### core.excludesfile ####

正如第二章所述，你能在項目庫的`.gitignore`文件裡頭用模式來定義那些無需納入Git管理的文件，這樣它們不會出現在未跟蹤列表，
也不會在你運行`git add`後被暫存。然而，如果你想用項目庫之外的文件來定義那些需被忽略的文件的話，用`core.excludesfile`
通知Git該文件所處的位置，文件內容和`.gitignore`類似。

#### help.autocorrect ####

該配置項只在Git 1.6.1及以上版本有效，假如你在Git 1.6中錯打了一條命令，會顯示：

	$ git com
	git: 'com' is not a git-command. See 'git --help'.

	Did you mean this?
	     commit

如果你把`help.autocorrect`設置成1（譯註：啟動自動修正），那麼在只有一個命令被模糊匹配到的情況下，Git會自動運行該命令。

### Git中的著色 ###

Git能夠為輸出到你終端的內容著色，以便你可以憑直觀進行快速、簡單地分析，有許多選項能供你使用以符合你的偏好。

#### color.ui ####

Git會按照你需要自動為大部分的輸出加上顏色，你能明確地規定哪些需要著色以及怎樣著色，設置`color.ui`為true來打開所有的默認終端著色。

	$ git config --global color.ui true

設置好以後，當輸出到終端時，Git會為之加上顏色。其他的參數還有false和always，false意味著不為輸出著色，而always則表明在任何情況下
都要著色，即使Git命令被重定向到文件或管道。Git 1.5.5版本引進了此項配置，如果你擁有的版本更老，你必須對顏色有關選項各自進行詳細地設置。

你會很少用到`color.ui = always`，在大多數情況下，如果你想在被重定向的輸出中插入顏色碼，你能傳遞`--color`標誌給Git命令來迫使
它這麼做，`color.ui = true`應該是你的首選。

#### `color.*` ####

想要具體到哪些命令輸出需要被著色以及怎樣著色或者Git的版本很老，你就要用到和具體命令有關的顏色配置選項，
它們都能被置為`true`、`false`或`always`：

	color.branch
	color.diff
	color.interactive
	color.status

除此之外，以上每個選項都有子選項，可以被用來覆蓋其父設置，以達到為輸出的各個部分著色的目的。
例如，讓diff輸出的改變信息以粗體、藍色前景和黑色背景的形式顯示：

	$ git config --global color.diff.meta 「blue black bold」

你能設置的顏色值如：normal、black、red、green、yellow、blue、magenta、cyan、white，
正如以上例子設置的粗體屬性，想要設置字體屬性的話，可以選擇如：bold、dim、ul、blink、reverse。

如果你想配置子選項的話，可以參考`git config`幫助頁。

### 外部的合併與比較工具 ###

雖然Git自己實現了diff,而且到目前為止你一直在使用它，但你能夠用一個外部的工具替代它，除此以外，你還能用一個圖形化的工具來合併和解決衝突
從而不必自己手動解決。有一個不錯且免費的工具可以被用來做比較和合併工作，它就是P4Merge（譯註：Perforce圖形化合併工具），我會展示它的安裝過程。

P4Merge可以在所有主流平台上運行，現在開始大膽嘗試吧。對於向你展示的例子，在Mac和Linux系統上，我會使用路徑名，
在Windows上，`/usr/local/bin`應該被改為你環境中的可執行路徑。

下載P4Merge：

	http://www.perforce.com/perforce/downloads/component.html

首先把你要運行的命令放入外部包裝腳本中，我會使用Mac系統上的路徑來指定該腳本的位置，在其他系統上，
它應該被放置在二進制文件`p4merge`所在的目錄中。創建一個merge包裝腳本，名字叫作`extMerge`，讓它帶參數調用`p4merge`二進制文件：

	$ cat /usr/local/bin/extMerge
	#!/bin/sh
	/Applications/p4merge.app/Contents/MacOS/p4merge $*

diff包裝腳本首先確定傳遞過來7個參數，隨後把其中2個傳遞給merge包裝腳本，默認情況下，Git傳遞以下參數給diff：

	path old-file old-hex old-mode new-file new-hex new-mode

由於你僅僅需要`old-file`和`new-file`參數，用diff包裝腳本來傳遞它們吧。

	$ cat /usr/local/bin/extDiff 
	#!/bin/sh
	[ $# -eq 7 ] && /usr/local/bin/extMerge "$2" "$5"

確認這兩個腳本是可執行的：

	$ sudo chmod +x /usr/local/bin/extMerge 
	$ sudo chmod +x /usr/local/bin/extDiff

現在來配置使用你自定義的比較和合併工具吧。這需要許多自定義設置：`merge.tool`通知Git使用哪個合併工具；
`mergetool.*.cmd`規定命令運行的方式；`mergetool.trustExitCode`會通知Git程序的退出是否指示合併操作成功；
`diff.external`通知Git用什麼命令做比較。因此，你能運行以下4條配置命令：

	$ git config --global merge.tool extMerge
	$ git config --global mergetool.extMerge.cmd \
	    'extMerge "$BASE" "$LOCAL" "$REMOTE" "$MERGED"'
	$ git config --global mergetool.trustExitCode false
	$ git config --global diff.external extDiff

或者直接編輯`~/.gitconfig`文件如下：

	[merge]
	  tool = extMerge
	[mergetool "extMerge"]
	  cmd = extMerge "$BASE" "$LOCAL" "$REMOTE" "$MERGED"
	  trustExitCode = false
	[diff]
	  external = extDiff

設置完畢後，運行diff命令：
	
	$ git diff 32d1776b1^ 32d1776b1

命令行居然沒有發現diff命令的輸出，其實，Git調用了剛剛設置的P4Merge，它看起來像圖7-1這樣：

Insert 18333fig0701.png 
Figure 7-1. P4Merge.

當你設法合併兩個分支，結果卻有衝突時，運行`git mergetool`，Git會調用P4Merge讓你通過圖形界面來解決衝突。

設置包裝腳本的好處是你能簡單地改變diff和merge工具，例如把`extDiff`和`extMerge`改成KDiff3，要做的僅僅是編輯`extMerge`腳本文件：

	$ cat /usr/local/bin/extMerge
	#!/bin/sh	
	/Applications/kdiff3.app/Contents/MacOS/kdiff3 $*

現在Git會使用KDiff3來做比較、合併和解決衝突。

Git預先設置了許多其他的合併和解決衝突的工具，而你不必設置cmd。可以把合併工具設置為：
kdiff3、opendiff、tkdiff、meld、xxdiff、emerge、vimdiff、gvimdiff。如果你不想用到KDiff3的所有功能，只是想用它來合併，
那麼kdiff3正符合你的要求，運行：

	$ git config --global merge.tool kdiff3

如果運行了以上命令，沒有設置`extMerge`和`extDiff`文件，Git會用KDiff3做合併，讓通常內設的比較工具來做比較。

### 格式化與空白 ###

格式化與空白是許多開發人員在協作時，特別是在跨平台情況下，遇到的令人頭疼的細小問題。
由於編輯器的不同或者Windows程序員在跨平台項目中的文件行尾加入了回車換行符，
一些細微的空格變化會不經意地進入大家合作的工作或提交的補丁中。不用怕，Git的一些配置選項會幫助你解決這些問題。

#### core.autocrlf ####

假如你正在Windows上寫程序，又或者你正在和其他人合作，他們在Windows上編程，而你卻在其他系統上，在這些情況下，你可能會遇到行尾結束符問題。
這是因為Windows使用回車和換行兩個字符來結束一行，而Mac和Linux只使用換行一個字符。
雖然這是小問題，但它會極大地擾亂跨平台協作。 

Git可以在你提交時自動地把行結束符CRLF轉換成LF，而在簽出代碼時把LF轉換成CRLF。用`core.autocrlf`來打開此項功能，
如果是在Windows系統上，把它設置成`true`，這樣當簽出代碼時，LF會被轉換成CRLF：

	$ git config --global core.autocrlf true

Linux或Mac系統使用LF作為行結束符，因此你不想Git在簽出文件時進行自動的轉換；當一個以CRLF為行結束符的文件不小心被引入時你肯定想進行修正，
把`core.autocrlf`設置成input來告訴Git在提交時把CRLF轉換成LF，簽出時不轉換：

	$ git config --global core.autocrlf input

這樣會在Windows系統上的簽出文件中保留CRLF，會在Mac和Linux系統上，包括倉庫中保留LF。

如果你是Windows程序員，且正在開發僅運行在Windows上的項目，可以設置`false`取消此功能，把回車符記錄在庫中：

	$ git config --global core.autocrlf false

#### core.whitespace ####

Git預先設置了一些選項來探測和修正空白問題，其4種主要選項中的2個默認被打開，另2個被關閉，你可以自由地打開或關閉它們。

默認被打開的2個選項是`trailing-space`和`space-before-tab`，`trailing-space`會查找每行結尾的空格，`space-before-tab`會查找每行開頭的製表符前的空格。

默認被關閉的2個選項是`indent-with-non-tab`和`cr-at-eol`，`indent-with-non-tab`會查找8個以上空格（非製表符）開頭的行，`cr-at-eol`讓Git知道行尾回車符是合法的。

設置`core.whitespace`，按照你的意圖來打開或關閉選項，選項以逗號分割。通過逗號分割的鏈中去掉選項或在選項前加`-`來關閉，例如，如果你想要打開除了`cr-at-eol`之外的所有選項：

	$ git config --global core.whitespace \
	    trailing-space,space-before-tab,indent-with-non-tab

當你運行`git diff`命令且為輸出著色時，Git會探測到這些問題，因此你也許在提交前能修復它們，當你用`git apply`打補丁時同樣也會從中受益。
如果正準備運用的補丁有特別的空白問題，你可以讓Git發警告：

	$ git apply --whitespace=warn <patch>

或者讓Git在打上補丁前自動修正此問題：

	$ git apply --whitespace=fix <patch>

這些選項也能運用於衍合。如果提交了有空白問題的文件但還沒推送到上流，你可以運行帶有`--whitespace=fix`選項的`rebase`來讓Git
在重寫補丁時自動修正它們。

### 服務器端配置 ###

Git服務器端的配置選項並不多，但仍有一些饒有生趣的選項值得你一看。

#### receive.fsckObjects ####

Git默認情況下不會在推送期間檢查所有對象的一致性。雖然會確認每個對象的有效性以及是否仍然匹配SHA-1檢驗和，但Git不會在每次推送時都檢查一致性。
對於Git來說，庫或推送的文件越大，這個操作代價就相對越高，每次推送會消耗更多時間，如果想在每次推送時Git都檢查一致性，設置
`receive.fsckObjects`為true來強迫它這麼做：

	$ git config --system receive.fsckObjects true

現在Git會在每次推送生效前檢查庫的完整性，確保有問題的客戶端沒有引入破壞性的數據。

#### receive.denyNonFastForwards ####

如果對已經被推送的提交歷史做衍合，繼而再推送，又或者以其它方式推送一個提交歷史至遠程分支，且該提交歷史沒在這個遠程分支中，這樣的推送會被拒絕。這通常是個很好的禁止策略，但有時你在做衍合併確定要更新遠程分支，可以在push命令後加`-f`標誌來強制更新。

要禁用這樣的強制更新功能，可以設置`receive.denyNonFastForwards`：

    $ git config --system receive.denyNonFastForwards true

稍後你會看到，用服務器端的接收鉤子也能達到同樣的目的。這個方法可以做更細緻的控制，例如：禁用特定的用戶做強制更新。

#### receive.denyDeletes ####

規避`denyNonFastForwards`策略的方法之一就是用戶刪除分支，然後推回新的引用。在更新的Git版本中（從1.6.1版本開始），把`receive.denyDeletes`設置為true：

    $ git config --system receive.denyDeletes true

這樣會在推送過程中阻止刪除分支和標籤 — 沒有用戶能夠這麼做。要刪除遠程分支，必須從服務器手動刪除引用文件。通過用戶訪問控制列表也能這麼做，
在本章結尾將會介紹這些有趣的方式。

## Git屬性 ##

一些設置項也能被運用於特定的路徑中，這樣，Git可以對一個特定的子目錄或子文件集運用那些設置項。這些設置項被稱為Git屬性，可以在你目錄中的`.gitattributes`文件內進行設置
（通常是你項目的根目錄），也可以當你不想讓這些屬性文件和項目文件一同提交時，在`.git/info/attributes`進行設置。

使用屬性，你可以對個別文件或目錄定義不同的合併策略，讓Git知道怎樣比較非文本文件，在你提交或簽出前讓Git過濾內容。你將在這部分瞭解到能在自己的項目中使用的屬性，以及一些實例。

### 二進制文件 ###

你可以用Git屬性讓其知道哪些是二進制文件（以防Git沒有識別出來），以及指示怎樣處理這些文件，這點很酷。例如，一些文本文件是由機器產生的，而且無法比較，而一些二進制文件可以比較 —
你將會瞭解到怎樣讓Git識別這些文件。

#### 識別二進制文件 ####

一些文件看起來像是文本文件，但其實是作為二進制數據被對待。例如，在Mac上的Xcode項目含有一個以`.pbxproj`結尾的文件，它是由記錄設置項的IDE寫到磁盤的JSON數據集（純文本javascript數據類型）。雖然技術上看它是由ASCII字符組成的文本文件，但你並不認為如此，因為它確實是一個輕量級數據庫 — 如果有2人改變了它，你通常無法合併和比較內容，只有機器才能進行識別和操作，於是，你想把它當成二進制文件。

讓Git把所有`pbxproj`文件當成二進制文件，在`.gitattributes`文件中設置如下：

    *.pbxproj -crlf -diff

現在，Git不會嘗試轉換和修正CRLF（回車換行）問題，也不會當你在項目中運行git show或git diff時，比較不同的內容。在Git 1.6及之後的版本中，可以用一個宏代替`-crlf -diff`：

    *.pbxproj binary

#### 比較二進制文件 ####

在Git 1.6及以上版本中，你能利用Git屬性來有效地比較二進制文件。可以設置Git把二進制數據轉換成文本格式，用通常的diff來比較。

這個特性很酷，而且鮮為人知，因此我會結合實例來講解。首先，要解決的是最令人頭疼的問題：對Word文檔進行版本控制。很多人對Word文檔又恨又愛，如果想對其進行版本控制，你可以把文件加入到Git庫中，每次修改後提交即可。但這樣做沒有一點實際意義，因為運行`git diff`命令後，你只能得到如下的結果：

    $ git diff
    diff --git a/chapter1.doc b/chapter1.doc
    index 88839c4..4afcb7c 100644
    Binary files a/chapter1.doc and b/chapter1.doc differ

你不能直接比較兩個不同版本的Word文件，除非進行手動掃瞄，不是嗎？Git屬性能很好地解決此問題，把下面的行加到`.gitattributes`文件：

    *.doc diff=word

當你要看比較結果時，如果文件擴展名是"doc"，Git會調用"word"過濾器。什麼是"word"過濾器呢？其實就是Git使用`strings` 程序，把Word文檔轉換成可讀的文本文件，之後再進行比較：

    $ git config diff.word.textconv strings

現在如果在兩個快照之間比較以`.doc`結尾的文件，Git會對這些文件運用"word"過濾器，在比較前把Word文件轉換成文本文件。

下面展示了一個實例，我把此書的第一章納入Git管理，在一個段落中加入了一些文本後保存，之後運行`git diff`命令，得到結果如下：

    $ git diff
    diff --git a/chapter1.doc b/chapter1.doc
    index c1c8a0a..b93c9e4 100644
    --- a/chapter1.doc
    +++ b/chapter1.doc
    @@ -8,7 +8,8 @@ re going to cover Version Control Systems (VCS) and Git basics
     re going to cover how to get it and set it up for the first time if you don
     t already have it on your system.
     In Chapter Two we will go over basic Git usage - how to use Git for the 80%
    -s going on, modify stuff and contribute changes. If the book spontaneously
    +s going on, modify stuff and contribute changes. If the book spontaneously
    +Let's see if this works.

Git 成功且簡潔地顯示出我增加的文本"Let's see if this works"。雖然有些瑕疵，在末尾顯示了一些隨機的內容，但確實可以比較了。如果你能找到或自己寫個Word到純文本的轉換器的話，效果可能會更好。 `strings`可以在大部分Mac和Linux系統上運行，所以它是處理二進制格式的第一選擇。

你還能用這個方法比較圖像文件。當比較時，對JPEG文件運用一個過濾器，它能提煉出EXIF信息 — 大部分圖像格式使用的元數據。如果你下載並安裝了`exiftool`程序，可以用它參照元數據把圖像轉換成文本。比較的不同結果將會用文本向你展示：

    $ echo '*.png diff=exif' >> .gitattributes
    $ git config diff.exif.textconv exiftool

如果在項目中替換了一個圖像文件，運行`git diff`命令的結果如下：

    diff --git a/image.png b/image.png
    index 88839c4..4afcb7c 100644
    --- a/image.png
    +++ b/image.png
    @@ -1,12 +1,12 @@
     ExifTool Version Number         : 7.74
    -File Size                       : 70 kB
    -File Modification Date/Time     : 2009:04:21 07:02:45-07:00
    +File Size                       : 94 kB
    +File Modification Date/Time     : 2009:04:21 07:02:43-07:00
     File Type                       : PNG
     MIME Type                       : image/png
    -Image Width                     : 1058
    -Image Height                    : 889
    +Image Width                     : 1056
    +Image Height                    : 827
     Bit Depth                       : 8
     Color Type                      : RGB with Alpha

你會發現文件的尺寸大小發生了改變。

### 關鍵字擴展 ###

使用SVN或CVS的開發人員經常要求關鍵字擴展。在Git中，你無法在一個文件被提交後修改它，因為Git會先對該文件計算校驗和。然而，你可以在簽出時注入文本，在提交前刪除它。Git屬性提供了2種方式這麼做。

首先，你能夠把blob的SHA-1校驗和自動注入文件的`$Id$`字段。如果在一個或多個文件上設置了此字段，當下次你簽出分支的時候，Git會用blob的SHA-1值替換那個字段。注意，這不是提交對象的SHA校驗和，而是blob本身的校驗和：

    $ echo '*.txt ident' >> .gitattributes
    $ echo '$Id$' > test.txt

下次簽出文件時，Git注入了blob的SHA值：

    $ rm text.txt
    $ git checkout -- text.txt
    $ cat test.txt
    $Id: 42812b7653c7b88933f8a9d6cad0ca16714b9bb3 $

然而，這樣的顯示結果沒有多大的實際意義。這個SHA的值相當地隨機，無法區分日期的前後，所以，如果你在CVS或Subversion中用過關鍵字替換，一定會包含一個日期值。

因此，你能寫自己的過濾器，在提交文件到暫存區或簽出文件時替換關鍵字。有2種過濾器，"clean"和"smudge"。在 `.gitattributes`文件中，你能對特定的路徑設置一個過濾器，然後設置處理文件的腳本，這些腳本會在文件簽出前（"smudge"，見圖 7-2）和提交到暫存區前（"clean"，見圖7-3）被調用。這些過濾器能夠做各種有趣的事。

Insert 18333fig0702.png
圖7-2. 簽出時，「smudge」過濾器被觸發。

Insert 18333fig0703.png
圖7-3. 提交到暫存區時，「clean」過濾器被觸發。

這裡舉一個簡單的例子：在暫存前，用`indent`（縮進）程序過濾所有C源代碼。在`.gitattributes`文件中設置"indent"過濾器過濾`*.c`文件：

    *.c     filter=indent

然後，通過以下配置，讓Git知道"indent"過濾器在遇到"smudge"和"clean"時分別該做什麼：

    $ git config --global filter.indent.clean indent
    $ git config --global filter.indent.smudge cat

於是，當你暫存`*.c`文件時，`indent`程序會被觸發，在把它們簽出之前，`cat`程序會被觸發。但`cat`程序在這裡沒什麼實際作用。這樣的組合，使C源代碼在暫存前被`indent`程序過濾，非常有效。

另一個例子是類似RCS的`$Date$`關鍵字擴展。為了演示，需要一個小腳本，接受文件名參數，得到項目的最新提交日期，最後把日期寫入該文件。下面用Ruby腳本來實現：

    #! /usr/bin/env ruby
    data = STDIN.read
    last_date = `git log --pretty=format:"%ad" -1`
    puts data.gsub('$Date$', '$Date: ' + last_date.to_s + '$')

該腳本從`git log`命令中得到最新提交日期，找到文件中的所有`$Date$`字符串，最後把該日期填充到`$Date$`字符串中 — 此腳本很簡單，你可以選擇你喜歡的編程語言來實現。把該腳本命名為`expand_date`，放到正確的路徑中，之後需要在Git中設置一個過濾器（`dater`），讓它在簽出文件時調用`expand_date`，在暫存文件時用Perl清除之：

    $ git config filter.dater.smudge expand_date
    $ git config filter.dater.clean 'perl -pe "s/\\\$Date[^\\\$]*\\\$/\\\$Date\\\$/"'

這個Perl小程序會刪除`$Date$`字符串裡多餘的字符，恢復`$Date$`原貌。到目前為止，你的過濾器已經設置完畢，可以開始測試了。打開一個文件，在文件中輸入`$Date$`關鍵字，然後設置Git屬性：

    $ echo '# $Date$' > date_test.txt
    $ echo 'date*.txt filter=dater' >> .gitattributes

如果暫存該文件，之後再簽出，你會發現關鍵字被替換了：

    $ git add date_test.txt .gitattributes
    $ git commit -m "Testing date expansion in Git"
    $ rm date_test.txt
    $ git checkout date_test.txt
    $ cat date_test.txt
    # $Date: Tue Apr 21 07:26:52 2009 -0700$

雖說這項技術對自定義應用來說很有用，但還是要小心，因為`.gitattributes`文件會隨著項目一起提交，而過濾器（例如：`dater`）不會，所以，過濾器不會在所有地方都生效。當你在設計這些過濾器時要注意，即使它們無法正常工作，也要讓整個項目運作下去。

### 導出倉庫 ###

Git屬性在導出項目歸檔時也能發揮作用。

#### export-ignore ####

當產生一個歸檔時，可以設置Git不導出某些文件和目錄。如果你不想在歸檔中包含一個子目錄或文件，但想他們納入項目的版本管理中，你能對應地設置`export-ignore`屬性。

例如，在`test/`子目錄中有一些測試文件，在項目的壓縮包中包含他們是沒有意義的。因此，可以增加下面這行到Git屬性文件中：

    test/ export-ignore

現在，當運行git archive來創建項目的壓縮包時，那個目錄不會在歸檔中出現。

#### export-subst ####

還能對歸檔做一些簡單的關鍵字替換。在第2章中已經可以看到，可以以`--pretty=format`形式的簡碼在任何文件中放入`$Format:$` 字符串。例如，如果想在項目中包含一個叫作`LAST_COMMIT`的文件，當運行`git archive`時，最後提交日期自動地注入進該文件，可以這樣設置：

    $ echo 'Last commit date: $Format:%cd$' > LAST_COMMIT
    $ echo "LAST_COMMIT export-subst" >> .gitattributes
    $ git add LAST_COMMIT .gitattributes
    $ git commit -am 'adding LAST_COMMIT file for archives'

運行`git archive`後，打開該文件，會發現其內容如下：

    $ cat LAST_COMMIT
    Last commit date: $Format:Tue Apr 21 08:38:48 2009 -0700$

### 合併策略 ###

通過Git屬性，還能對項目中的特定文件使用不同的合併策略。一個非常有用的選項就是，當一些特定文件發生衝突，Git不會嘗試合併他們，而使用你這邊的合併。


如果項目的一個分支有歧義或比較特別，但你想從該分支合併，而且需要忽略其中某些文件，這樣的合併策略是有用的。例如，你有一個數據庫設置文件database.xml，在2個分支中他們是不同的，你想合併一個分支到另一個，而不弄亂該數據庫文件，可以設置屬性如下：

    database.xml merge=ours

如果合併到另一個分支，database.xml文件不會有合併衝突，顯示如下：

    $ git merge topic
    Auto-merging database.xml
    Merge made by recursive.

這樣，database.xml會保持原樣。

## Git掛鉤 ##

和其他版本控制系統一樣，當某些重要事件發生時，Git可以調用自定義腳本。有兩組掛鉤：客戶端和服務器端。客戶端掛鉤用於客戶端的操作，如提交和合併。服務器端掛鉤用於Git服務器端的操作，如接收被推送的提交。你可以隨意地使用這些掛鉤，下面會講解其中一些。

### 安裝一個掛鉤 ###

掛鉤都被存儲在Git目錄下的`hooks`子目錄中，即大部分項目中的`.git/hooks`。Git默認會放置一些腳本樣本在這個目錄中，除了可以作為掛鉤使用，這些樣本本身是可以獨立使用的。所有的樣本都是shell腳本，其中一些還包含了Perl的腳本，不過，任何正確命名的可執行腳本都可以正常使用 — 可以用Ruby或Python，或其他。在Git 1.6版本之後，這些樣本名都是以.sample結尾，因此，你必須重新命名。在Git 1.6版本之前，這些樣本名都是正確的，但這些樣本不是可執行文件。


把一個正確命名且可執行的文件放入Git目錄下的`hooks`子目錄中，可以激活該掛鉤腳本，因此，之後他一直會被Git調用。隨後會講解主要的掛鉤腳本。

### 客戶端掛鉤 ###

有許多客戶端掛鉤，以下把他們分為：提交工作流掛鉤、電子郵件工作流掛鉤及其他客戶端掛鉤。

#### 提交工作流掛鉤 ####

有 4個掛鉤被用來處理提交的過程。`pre-commit`掛鉤在鍵入提交信息前運行，被用來檢查即將提交的快照，例如，檢查是否有東西被遺漏，確認測試是否運行，以及檢查代碼。當從該掛鉤返回非零值時，Git會放棄此次提交，但可以用`git commit --no-verify`來忽略。該掛鉤可以被用來檢查代碼錯誤（運行類似lint的程序），檢查尾部空白（默認掛鉤是這麼做的），檢查新方法（譯註：程序的函數）的說明。

`prepare-commit-msg`掛鉤在提交信息編輯器顯示之前，默認信息被創建之後運行。因此，可以有機會在提交作者看到默認信息前進行編輯。該掛鉤接收一些選項：擁有提交信息的文件路徑，提交類型，如果是一次修訂的話，提交的SHA-1校驗和。該掛鉤對通常的提交來說不是很有用，只在自動產生的默認提交信息的情況下有作用，如提交信息模板、合併、壓縮和修訂提交等。可以和提交模板配合使用，以編程的方式插入信息。

`commit-msg`掛鉤接收一個參數，此參數是包含最近提交信息的臨時文件的路徑。如果該掛鉤腳本以非零退出，Git會放棄提交，因此，可以用來在提交通過前驗證項目狀態或提交信息。本章上一小節已經展示了使用該掛鉤核對提交信息是否符合特定的模式。

`post-commit`掛鉤在整個提交過程完成後運行，他不會接收任何參數，但可以運行`git log -1 HEAD`來獲得最後的提交信息。總之，該掛鉤是作為通知之類使用的。

提交工作流的客戶端掛鉤腳本可以在任何工作流中使用，他們經常被用來實施某些策略，但值得注意的是，這些腳本在clone期間不會被傳送。可以在服務器端實施策略來拒絕不符合某些策略的推送，但這完全取決於開發者在客戶端使用這些腳本的情況。所以，這些腳本對開發者是有用的，由他們自己設置和維護，而且在任何時候都可以覆蓋或修改這些腳本。

#### E-mail工作流掛鉤 ####

有3個可用的客戶端掛鉤用於e-mail工作流。當運行`git am`命令時，會調用他們，因此，如果你沒有在工作流中用到此命令，可以跳過本節。如果你通過e-mail接收由`git format-patch`產生的補丁，這些掛鉤也許對你有用。

首先運行的是`applypatch-msg`掛鉤，他接收一個參數：包含被建議提交信息的臨時文件名。如果該腳本非零退出，Git會放棄此補丁。可以使用這個腳本確認提交信息是否被正確格式化，或讓腳本編輯信息以達到標準化。

下一個在`git am`運行期間調用是`pre-applypatch`掛鉤。該掛鉤不接收參數，在補丁被運用之後運行，因此，可以被用來在提交前檢查快照。你能用此腳本運行測試，檢查工作樹。如果有些什麼遺漏，或測試沒通過，腳本會以非零退出，放棄此次`git am`的運行，補丁不會被提交。

最後在`git am`運行期間調用的是`post-applypatch`掛鉤。你可以用他來通知一個小組或獲取的補丁的作者，但無法阻止打補丁的過程。

#### 其他客戶端掛鉤 ####

`pre- rebase`掛鉤在衍合前運行，腳本以非零退出可以中止衍合的過程。你可以使用這個掛鉤來禁止衍合已經推送的提交對象，Git的`pre- rebase`掛鉤樣本就是這麼做的。該樣本假定next是你定義的分支名，因此，你可能要修改樣本，把next改成你定義過且穩定的分支名。

在`git checkout`成功運行後，`post-checkout`掛鉤會被調用。他可以用來為你的項目環境設置合適的工作目錄。例如：放入大的二進制文件、自動產生的文檔或其他一切你不想納入版本控制的文件。

最後，在`merge`命令成功執行後，`post-merge`掛鉤會被調用。他可以用來在Git無法跟蹤的工作樹中恢複數據，諸如權限數據。該掛鉤同樣能夠驗證在Git控制之外的文件是否存在，因此，當工作樹改變時，你想這些文件可以被覆制。

### 服務器端掛鉤 ###

除了客戶端掛鉤，作為系統管理員，你還可以使用兩個服務器端的掛鉤對項目實施各種類型的策略。這些掛鉤腳本可以在提交對象推送到服務器前被調用，也可以在推送到服務器後被調用。推送到服務器前調用的掛鉤可以在任何時候以非零退出，拒絕推送，返回錯誤消息給客戶端，還可以如你所願設置足夠複雜的推送策略。

#### pre-receive和post-receive #### 

The first script to run when handling a push from a client is `pre-receive`. It takes a list of references that are being pushed from stdin; if it exits non-zero, none of them are accepted. You can use this hook to do things like make sure none of the updated references are non-fast-forwards; or to check that the user doing the pushing has create, delete, or push access or access to push updates to all the files they're modifying with the push.

The `post-receive` hook runs after the entire process is completed and can be used to update other services or notify users. It takes the same stdin data as the `pre-receive` hook. Examples include e-mailing a list, notifying a continuous integration server, or updating a ticket-tracking system — you can even parse the commit messages to see if any tickets need to be opened, modified, or closed. This script can't stop the push process, but the client doesn't disconnect until it has completed; so, be careful when you try to do anything that may take a long time.

#### update ####

The update script is very similar to the `pre-receive` script, except that it's run once for each branch the pusher is trying to update. If the pusher is trying to push to multiple branches, `pre-receive` runs only once, whereas update runs once per branch they're pushing to. Instead of reading from stdin, this script takes three arguments: the name of the reference (branch), the SHA-1 that reference pointed to before the push, and the SHA-1 the user is trying to push. If the update script exits non-zero, only that reference is rejected; other references can still be updated.

## An Example Git-Enforced Policy ##

In this section, you'll use what you've learned to establish a Git workflow that checks for a custom commit message format, enforces fast-forward-only pushes, and allows only certain users to modify certain subdirectories in a project. You'll build client scripts that help the developer know if their push will be rejected and server scripts that actually enforce the policies.

I used Ruby to write these, both because it's my preferred scripting language and because I feel it's the most pseudocode-looking of the scripting languages; thus you should be able to roughly follow the code even if you don't use Ruby. However, any language will work fine. All the sample hook scripts distributed with Git are in either Perl or Bash scripting, so you can also see plenty of examples of hooks in those languages by looking at the samples.

### Server-Side Hook ###

All the server-side work will go into the update file in your hooks directory. The update file runs once per branch being pushed and takes the reference being pushed to, the old revision where that branch was, and the new revision being pushed. You also have access to the user doing the pushing if the push is being run over SSH. If you've allowed everyone to connect with a single user (like "git") via public-key authentication, you may have to give that user a shell wrapper that determines which user is connecting based on the public key, and set an environment variable specifying that user. Here I assume the connecting user is in the `$USER` environment variable, so your update script begins by gathering all the information you need:

	#!/usr/bin/env ruby

	$refname = ARGV[0]
	$oldrev  = ARGV[1]
	$newrev  = ARGV[2]
	$user    = ENV['USER']

	puts "Enforcing Policies... \n(#{$refname}) (#{$oldrev[0,6]}) (#{$newrev[0,6]})"

Yes, I'm using global variables. Don't judge me — it's easier to demonstrate in this manner.

#### Enforcing a Specific Commit-Message Format ####

Your first challenge is to enforce that each commit message must adhere to a particular format. Just to have a target, assume that each message has to include a string that looks like "ref: 1234" because you want each commit to link to a work item in your ticketing system. You must look at each commit being pushed up, see if that string is in the commit message, and, if the string is absent from any of the commits, exit non-zero so the push is rejected.

You can get a list of the SHA-1 values of all the commits that are being pushed by taking the `$newrev` and `$oldrev` values and passing them to a Git plumbing command called `git rev-list`. This is basically the `git log` command, but by default it prints out only the SHA-1 values and no other information. So, to get a list of all the commit SHAs introduced between one commit SHA and another, you can run something like this:

	$ git rev-list 538c33..d14fc7
	d14fc7c847ab946ec39590d87783c69b031bdfb7
	9f585da4401b0a3999e84113824d15245c13f0be
	234071a1be950e2a8d078e6141f5cd20c1e61ad3
	dfa04c9ef3d5197182f13fb5b9b1fb7717d2222a
	17716ec0f1ff5c77eff40b7fe912f9f6cfd0e475

You can take that output, loop through each of those commit SHAs, grab the message for it, and test that message against a regular expression that looks for a pattern.

You have to figure out how to get the commit message from each of these commits to test. To get the raw commit data, you can use another plumbing command called `git cat-file`. I'll go over all these plumbing commands in detail in Chapter 9; but for now, here's what that command gives you:

	$ git cat-file commit ca82a6
	tree cfda3bf379e4f8dba8717dee55aab78aef7f4daf
	parent 085bb3bcb608e1e8451d4b2432f8ecbe6306e7e7
	author Scott Chacon <schacon@gmail.com> 1205815931 -0700
	committer Scott Chacon <schacon@gmail.com> 1240030591 -0700

	changed the version number

A simple way to get the commit message from a commit when you have the SHA-1 value is to go to the first blank line and take everything after that. You can do so with the `sed` command on Unix systems:

	$ git cat-file commit ca82a6 | sed '1,/^$/d'
	changed the version number

You can use that incantation to grab the commit message from each commit that is trying to be pushed and exit if you see anything that doesn't match. To exit the script and reject the push, exit non-zero. The whole method looks like this:

	$regex = /\[ref: (\d+)\]/

	# enforced custom commit message format
	def check_message_format
	  missed_revs = `git rev-list #{$oldrev}..#{$newrev}`.split("\n")
	  missed_revs.each do |rev|
	    message = `git cat-file commit #{rev} | sed '1,/^$/d'`
	    if !$regex.match(message)
	      puts "[POLICY] Your message is not formatted correctly"
	      exit 1
	    end
	  end
	end
	check_message_format

Putting that in your `update` script will reject updates that contain commits that have messages that don't adhere to your rule.

#### Enforcing a User-Based ACL System ####

Suppose you want to add a mechanism that uses an access control list (ACL) that specifies which users are allowed to push changes to which parts of your projects. Some people have full access, and others only have access to push changes to certain subdirectories or specific files. To enforce this, you'll write those rules to a file named `acl` that lives in your bare Git repository on the server. You'll have the `update` hook look at those rules, see what files are being introduced for all the commits being pushed, and determine whether the user doing the push has access to update all those files.

The first thing you'll do is write your ACL. Here you'll use a format very much like the CVS ACL mechanism: it uses a series of lines, where the first field is `avail` or `unavail`, the next field is a comma-delimited list of the users to which the rule applies, and the last field is the path to which the rule applies (blank meaning open access). All of these fields are delimited by a pipe (`|`) character.

In this case, you have a couple of administrators, some documentation writers with access to the `doc` directory, and one developer who only has access to the `lib` and `tests` directories, and your ACL file looks like this:

	avail|nickh,pjhyett,defunkt,tpw
	avail|usinclair,cdickens,ebronte|doc
	avail|schacon|lib
	avail|schacon|tests

You begin by reading this data into a structure that you can use. In this case, to keep the example simple, you'll only enforce the `avail` directives. Here is a method that gives you an associative array where the key is the user name and the value is an array of paths to which the user has write access:

	def get_acl_access_data(acl_file)
	  # read in ACL data
	  acl_file = File.read(acl_file).split("\n").reject { |line| line == '' }
	  access = {}
	  acl_file.each do |line|
	    avail, users, path = line.split('|')
	    next unless avail == 'avail'
	    users.split(',').each do |user|
	      access[user] ||= []
	      access[user] << path
	    end
	  end
	  access
	end

On the ACL file you looked at earlier, this `get_acl_access_data` method returns a data structure that looks like this:

	{"defunkt"=>[nil],
	 "tpw"=>[nil],
	 "nickh"=>[nil],
	 "pjhyett"=>[nil],
	 "schacon"=>["lib", "tests"],
	 "cdickens"=>["doc"],
	 "usinclair"=>["doc"],
	 "ebronte"=>["doc"]}

Now that you have the permissions sorted out, you need to determine what paths the commits being pushed have modified, so you can make sure the user who's pushing has access to all of them.

You can pretty easily see what files have been modified in a single commit with the `--name-only` option to the `git log` command (mentioned briefly in Chapter 2):

	$ git log -1 --name-only --pretty=format:'' 9f585d

	README
	lib/test.rb

If you use the ACL structure returned from the `get_acl_access_data` method and check it against the listed files in each of the commits, you can determine whether the user has access to push all of their commits:

	# only allows certain users to modify certain subdirectories in a project
	def check_directory_perms
	  access = get_acl_access_data('acl')

	  # see if anyone is trying to push something they can't
	  new_commits = `git rev-list #{$oldrev}..#{$newrev}`.split("\n")
	  new_commits.each do |rev|
	    files_modified = `git log -1 --name-only --pretty=format:'' #{rev}`.split("\n")
	    files_modified.each do |path|
	      next if path.size == 0
	      has_file_access = false
	      access[$user].each do |access_path|
	        if !access_path  # user has access to everything
	          || (path.index(access_path) == 0) # access to this path
	          has_file_access = true 
	        end
	      end
	      if !has_file_access
	        puts "[POLICY] You do not have access to push to #{path}"
	        exit 1
	      end
	    end
	  end  
	end

	check_directory_perms

Most of that should be easy to follow. You get a list of new commits being pushed to your server with `git rev-list`. Then, for each of those, you find which files are modified and make sure the user who's pushing has access to all the paths being modified. One Rubyism that may not be clear is `path.index(access_path) == 0`, which is true if path begins with `access_path` — this ensures that `access_path` is not just in one of the allowed paths, but an allowed path begins with each accessed path. 

Now your users can't push any commits with badly formed messages or with modified files outside of their designated paths.

#### Enforcing Fast-Forward-Only Pushes ####

The only thing left is to enforce fast-forward-only pushes. In Git versions 1.6 or newer, you can set the `receive.denyDeletes` and `receive.denyNonFastForwards` settings. But enforcing this with a hook will work in older versions of Git, and you can modify it to do so only for certain users or whatever else you come up with later.

The logic for checking this is to see if any commits are reachable from the older revision that aren't reachable from the newer one. If there are none, then it was a fast-forward push; otherwise, you deny it:

	# enforces fast-forward only pushes 
	def check_fast_forward
	  missed_refs = `git rev-list #{$newrev}..#{$oldrev}`
	  missed_ref_count = missed_refs.split("\n").size
	  if missed_ref_count > 0
	    puts "[POLICY] Cannot push a non fast-forward reference"
	    exit 1
	  end
	end

	check_fast_forward

Everything is set up. If you run `chmod u+x .git/hooks/update`, which is the file you into which you should have put all this code, and then try to push a non-fast-forwarded reference, you get something like this:

	$ git push -f origin master
	Counting objects: 5, done.
	Compressing objects: 100% (3/3), done.
	Writing objects: 100% (3/3), 323 bytes, done.
	Total 3 (delta 1), reused 0 (delta 0)
	Unpacking objects: 100% (3/3), done.
	Enforcing Policies... 
	(refs/heads/master) (8338c5) (c5b616)
	[POLICY] Cannot push a non-fast-forward reference
	error: hooks/update exited with error code 1
	error: hook declined to update refs/heads/master
	To git@gitserver:project.git
	 ! [remote rejected] master -> master (hook declined)
	error: failed to push some refs to 'git@gitserver:project.git'

There are a couple of interesting things here. First, you see this where the hook starts running.

	Enforcing Policies... 
	(refs/heads/master) (fb8c72) (c56860)

Notice that you printed that out to stdout at the very beginning of your update script. It's important to note that anything your script prints to stdout will be transferred to the client.

The next thing you'll notice is the error message.

	[POLICY] Cannot push a non fast-forward reference
	error: hooks/update exited with error code 1
	error: hook declined to update refs/heads/master

The first line was printed out by you, the other two were Git telling you that the update script exited non-zero and that is what is declining your push. Lastly, you have this:

	To git@gitserver:project.git
	 ! [remote rejected] master -> master (hook declined)
	error: failed to push some refs to 'git@gitserver:project.git'

You'll see a remote rejected message for each reference that your hook declined, and it tells you that it was declined specifically because of a hook failure.

Furthermore, if the ref marker isn't there in any of your commits, you'll see the error message you're printing out for that.

	[POLICY] Your message is not formatted correctly

Or if someone tries to edit a file they don't have access to and push a commit containing it, they will see something similar. For instance, if a documentation author tries to push a commit modifying something in the `lib` directory, they see

	[POLICY] You do not have access to push to lib/test.rb

That's all. From now on, as long as that `update` script is there and executable, your repository will never be rewound and will never have a commit message without your pattern in it, and your users will be sandboxed.

### Client-Side Hooks ###

The downside to this approach is the whining that will inevitably result when your users' commit pushes are rejected. Having their carefully crafted work rejected at the last minute can be extremely frustrating and confusing; and furthermore, they will have to edit their history to correct it, which isn't always for the faint of heart.

The answer to this dilemma is to provide some client-side hooks that users can use to notify them when they're doing something that the server is likely to reject. That way, they can correct any problems before committing and before those issues become more difficult to fix. Because hooks aren't transferred with a clone of a project, you must distribute these scripts some other way and then have your users copy them to their `.git/hooks` directory and make them executable. You can distribute these hooks within the project or in a separate project, but there is no way to set them up automatically.

To begin, you should check your commit message just before each commit is recorded, so you know the server won't reject your changes due to badly formatted commit messages. To do this, you can add the `commit-msg` hook. If you have it read the message from the file passed as the first argument and compare that to the pattern, you can force Git to abort the commit if there is no match:

	#!/usr/bin/env ruby
	message_file = ARGV[0]
	message = File.read(message_file)

	$regex = /\[ref: (\d+)\]/

	if !$regex.match(message)
	  puts "[POLICY] Your message is not formatted correctly"
	  exit 1
	end

If that script is in place (in `.git/hooks/commit-msg`) and executable, and you commit with a message that isn't properly formatted, you see this:

	$ git commit -am 'test'
	[POLICY] Your message is not formatted correctly

No commit was completed in that instance. However, if your message contains the proper pattern, Git allows you to commit:

	$ git commit -am 'test [ref: 132]'
	[master e05c914] test [ref: 132]
	 1 files changed, 1 insertions(+), 0 deletions(-)

Next, you want to make sure you aren't modifying files that are outside your ACL scope. If your project's `.git` directory contains a copy of the ACL file you used previously, then the following `pre-commit` script will enforce those constraints for you:

	#!/usr/bin/env ruby

	$user    = ENV['USER']

	# [ insert acl_access_data method from above ]

	# only allows certain users to modify certain subdirectories in a project
	def check_directory_perms
	  access = get_acl_access_data('.git/acl')

	  files_modified = `git diff-index --cached --name-only HEAD`.split("\n")
	  files_modified.each do |path|
	    next if path.size == 0
	    has_file_access = false
	    access[$user].each do |access_path|
	    if !access_path || (path.index(access_path) == 0)
	      has_file_access = true
	    end
	    if !has_file_access
	      puts "[POLICY] You do not have access to push to #{path}"
	      exit 1
	    end
	  end
	end

	check_directory_perms

This is roughly the same script as the server-side part, but with two important differences. First, the ACL file is in a different place, because this script runs from your working directory, not from your Git directory. You have to change the path to the ACL file from this

	access = get_acl_access_data('acl')

to this:

	access = get_acl_access_data('.git/acl')

The other important difference is the way you get a listing of the files that have been changed. Because the server-side method looks at the log of commits, and, at this point, the commit hasn't been recorded yet, you must get your file listing from the staging area instead. Instead of

	files_modified = `git log -1 --name-only --pretty=format:'' #{ref}`

you have to use

	files_modified = `git diff-index --cached --name-only HEAD`

But those are the only two differences — otherwise, the script works the same way. One caveat is that it expects you to be running locally as the same user you push as to the remote machine. If that is different, you must set the `$user` variable manually.

The last thing you have to do is check that you're not trying to push non-fast-forwarded references, but that is a bit less common. To get a reference that isn't a fast-forward, you either have to rebase past a commit you've already pushed up or try pushing a different local branch up to the same remote branch.

Because the server will tell you that you can't push a non-fast-forward anyway, and the hook prevents forced pushes, the only accidental thing you can try to catch is rebasing commits that have already been pushed.

Here is an example pre-rebase script that checks for that. It gets a list of all the commits you're about to rewrite and checks whether they exist in any of your remote references. If it sees one that is reachable from one of your remote references, it aborts the rebase:

	#!/usr/bin/env ruby

	base_branch = ARGV[0]
	if ARGV[1]
	  topic_branch = ARGV[1]
	else
	  topic_branch = "HEAD"
	end

	target_shas = `git rev-list #{base_branch}..#{topic_branch}`.split("\n")
	remote_refs = `git branch -r`.split("\n").map { |r| r.strip }

	target_shas.each do |sha|
	  remote_refs.each do |remote_ref|
	    shas_pushed = `git rev-list ^#{sha}^@ refs/remotes/#{remote_ref}`
	    if shas_pushed.split(「\n」).include?(sha)
	      puts "[POLICY] Commit #{sha} has already been pushed to #{remote_ref}"
	      exit 1
	    end
	  end
	end

This script uses a syntax that wasn't covered in the Revision Selection section of Chapter 6. You get a list of commits that have already been pushed up by running this:

	git rev-list ^#{sha}^@ refs/remotes/#{remote_ref}

The `SHA^@` syntax resolves to all the parents of that commit. You're looking for any commit that is reachable from the last commit on the remote and that isn't reachable from any parent of any of the SHAs you're trying to push up — meaning it's a fast-forward.

The main drawback to this approach is that it can be very slow and is often unnecessary — if you don't try to force the push with `-f`, the server will warn you and not accept the push. However, it's an interesting exercise and can in theory help you avoid a rebase that you might later have to go back and fix.

## Summary ##

You've covered most of the major ways that you can customize your Git client and server to best fit your workflow and projects. You've learned about all sorts of configuration settings, file-based attributes, and event hooks, and you've built an example policy-enforcing server. You should now be able to make Git fit nearly any workflow you can dream up.

