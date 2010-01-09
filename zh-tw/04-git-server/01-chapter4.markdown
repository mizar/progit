# 服務器上的 Git #

到目前為止，你應該已經學會了使用 Git 來完成日常的工作。然而，如果想與他人合作，還需要一個遠程的 Git 倉庫。儘管技術上可以從個人的倉庫裡推送和拉取改變，但是我們不鼓勵這樣做，因為一不留心就很容易弄混其他人的進度。另外，你也一定希望合作者們即使在自己不開機的時候也能從倉庫獲取數據——擁有一個更穩定的公共倉庫十分有用。因此，更好的合作方式是建立一個大家都可以訪問的共享倉庫，從那裡推送和拉取數據。我們將把這個倉庫稱為 "Git 服務器"；代理一個 Git 倉庫只需要花費很少的資源，幾乎從不需要整個服務器來支持它的運行。

架設一個 Git 服務器不難。第一步是選擇與服務器通訊的協議。本章的第一節將介紹可用的協議以及他們各自的優缺點。下面一節將介紹一些針對各個協議典型的設置以及如何在服務器上運行它們。最後，如果你不介意在其他人的服務器上保存你的代碼，又不想經歷自己架設和維護服務器的麻煩，我們將介紹幾個網絡上的倉庫託管服務。

如果你對架設自己的服務器沒興趣，可以跳到本章最後一節去看看如何創建一個代碼託管賬戶然後繼續下一章，我們會在那裡討論一個分佈式源碼控制環境的林林總總。

遠程倉庫通常只是一個 _純倉庫(bare repository)_ ——一個沒有當前工作目錄的倉庫。因為該倉庫只是一個合作媒介，所以不需要從一個處於已從硬盤上檢出狀態的快照；倉庫裡僅僅是 Git 的數據。簡單的說，純倉庫是你項目裡 `.git` 目錄的內容，別無他物。

## 協議 ##

Git 可以使用四種主要的協議來傳輸數據：本地傳輸，SSH 協議，Git 協議和 HTTP 協議。下面分別介紹一下他們以及你應該（或不應該）在怎樣的情形下使用他們。

值得注意的是除了 HTTP 協議之外，其他所有協議都要求在服務器端安裝並運行 Git 。

### 本地協議 ###

最基礎的就是 _本地協議(Local protocol)_ 了，遠程倉庫在該協議中就是硬盤上的另一個目錄。這常見於團隊每一個成員都對一個共享的文件系統(例如 NFS )擁有訪問權，抑或比較少見的多人共用同一台電腦的時候。後者不是很理想，因為你所有的代碼倉庫實例都儲存在同一台電腦裡，增加了災難性數據損失的可能性。

如果你使用一個共享的文件系統，就可以在一個本地倉庫裡克隆，推送和獲取。要從這樣的倉庫裡克隆或者將其作為遠程倉庫添加現有工程裡，可以用指向該倉庫的路徑作為URL。比如，克隆一個本地倉庫，可以用如下命令完成：

	$ git clone /opt/git/project.git

或者這樣：

	$ git clone file:///opt/git/project.git
如果你在URL的開頭明確的使用 `file://` ，那麼 Git 會以一種略微不同的方式運行。如果你只給出路徑，Git 會嘗試使用硬鏈接或者直接複製它需要的文件。如果使用了 `file://` ，Git會調用它平時通過網絡來傳輸數據的工序，而這種方式的效率相對很低。使用 `file://` 前綴的主要原因是當你需要一個不包含無關引用或對象的乾淨倉庫副本的時候——一般是從其他版本控制系統的導入之後或者類似的情形（參見第9章的維護任務）。我們這裡使用普通路徑，因為通常這樣總是更快。

要添加一個本地倉庫到現有 Git 工程，運行如下命令：

	$ git remote add local_proj /opt/git/project.git

然後就可以像在網絡上一樣向這個遠程倉庫推送和獲取數據了。

#### 優點 ####

基於文件倉庫的優點在於它的簡單，同時保留了現存文件的權限和網絡訪問權限。如果你的團隊已經有一個全體共享的文件系統，建立倉庫就十分容易了。你只需把一份純倉庫的副本放在大家能訪問的地方，然後像對其他共享目錄一樣設置讀寫權限就可以了。我們將在下一節「在服務器上部署 Git 」中討論如何為此導出一個純倉庫的副本。

這也是個從別人工作目錄裡獲取他工作成果的快捷方法。假如你和你的同事在一個項目中合作，他們想讓你檢出一些東西的時候，運行類似 `git pull /home/john/project` 通常會比他們推送到服務器，而你又從服務器獲取簡單得多。

#### 缺點 ####

這種方法的缺點是，與基本的網絡連接訪問相比，能從不同的位置訪問的共享權限難以架設。如果你想從家裡的筆記本電腦上推送，就要先掛載遠程硬盤，這和基於網絡連接的訪問相比更加困難和緩慢。

另一個很重要的問題是該方法不一定就是最快的，尤其是對於共享掛載的文件系統。本地倉庫只有在你對數據訪問速度快的時候才快。在同一個服務器上，如果二者同時允許 Git 訪問本地硬盤，通過 NFS 訪問倉庫通常會比 SSH 慢。

### SSH 協議 ###

Git 使用的傳輸協議中最常見的可能就是 SSH 了。這是因為大多數環境已經支持通過 SSH 對服務器的訪問——即使還沒有，也很容易架設。SSH 也是唯一一個同時便於讀和寫操作的網絡協議。另外兩個網絡協議（HTTP 和 Git）通常都是只讀的，所以雖然二者對大多數人都可用，但執行寫操作時還是需要 SSH。SSH 同時也是一個驗證授權的網絡協議；而因為其普遍性，通常也很容易架設和使用。

通過 SSH 克隆一個 Git 倉庫，你可以像下面這樣給出 ssh:// 的 URL：

	$ git clone ssh://user@server:project.git

或者不指明某個協議——這時 Git 會默認使用 SSH ：
	
	$ git clone user@server:project.git

也可以不指明用戶，Git 會默認使用你當前登錄的用戶。 

#### 優點 ####

使用 SSH 的好處有很多。首先，如果你想擁有對網絡倉庫的寫權限，基本上不可能不使用 SSH。其次，SSH 架設相對比較簡單—— SSH 守護進程很常見，很多網絡管理員都有一些使用經驗，而且很多操作系統都自帶了它或者相關的管理工具。再次，通過 SSH 進行訪問是安全的——所有數據傳輸都是加密和授權的。最後，類似 Git 和 本地協議，SSH 很高效，會在傳輸之前儘可能的壓縮數據。

#### 缺點 ####

SSH 的限制在於你不能通過它實現倉庫的匿名訪問。即使僅為讀取數據，人們也必須在能通過 SSH 訪問主機的前提下才能訪問倉庫，這使得 SSH 不利於開源的項目。如果你僅僅在公司網絡裡使用，SSH 可能是你唯一需要使用的協議。如果想允許對項目的匿名只讀訪問，那麼除了為自己推送而架設 SSH 協議之外，還需要其他協議來讓別人獲取數據。

### Git 協議 ###

接下來是 Git 協議。這是一個包含在 Git 軟件包中的特殊守護進程； 它會監聽一個提供類似於 SSH 服務的特定端口（9418），而無需任何授權。用 Git 協議運營倉庫，你需要創建 `git-export-daemon-ok` 文件——它是協議進程提供倉庫服務的必要條件——但除此之外該服務沒有什麼安全措施。要麼所有人都能克隆 Git 倉庫，要麼誰也不能。這也意味著該協議通常不能用來進行推送。你可以允許推送操作；然而由於沒有授權機制，一旦允許該操作，網絡上任何一個知道項目 URL 的人將都有推送權限。不用說，這是十分罕見的情況。

#### 優點 ####

Git 協議是現存最快的傳輸協議。如果你在提供一個有很大訪問量的公共項目，或者一個不需要對讀操作進行授權的龐大項目，架設一個 Git 守護進程來供應倉庫是個不錯的選擇。它使用與 SSH 協議相同的數據傳輸機制，但省去了加密和授權的開銷。

#### 缺點 ####

Git 協議消極的一面是缺少授權機制。用 Git 協議作為訪問項目的唯一方法通常是不可取的。一般做法是，同時提供 SSH 接口，讓幾個開發者擁有推送（寫）權限，其他人通過 `git://` 擁有只讀權限。
Git 協議可能也是最難架設的協議。它要求有單獨的守護進程，需要定製——我們將在本章的 「Gitosis」 一節詳細介紹它的架設——需要設定 `xinetd` 或類似的程序，而這些就沒那麼平易近人了。該協議還要求防火牆開放 9418 端口，而企業級防火牆一般不允許對這個非標準端口的訪問。大型企業級防火牆通常會封鎖這個少見的端口。

### HTTP/S 協議 ###

最後還有 HTTP 協議。HTTP 或 HTTPS 協議的優美之處在於架設的簡便性。基本上，
只需要把 Git 的純倉庫文件放在 HTTP 的文件根目錄下，配置一個特定的 `post-update` 掛鉤（hook），就搞定了（Git 掛鉤的細節見第七章）。從此，每個能訪問 Git 倉庫所在服務器上的 web 服務的人都可以進行克隆操作。下面的操作可以允許通過 HTTP 對倉庫進行讀取：

	$ cd /var/www/htdocs/
	$ git clone --bare /path/to/git_project gitproject.git
	$ cd gitproject.git
	$ mv hooks/post-update.sample hooks/post-update
	$ chmod a+x hooks/post-update

這樣就可以了。Git 附帶的 `post-update` 掛鉤會默認運行合適的命令（`git update-server-info`）來確保通過 HTTP 的獲取和克隆正常工作。這條命令在你用 SSH 向倉庫推送內容時運行；之後，其他人就可以用下面的命令來克隆倉庫：

	$ git clone http://example.com/gitproject.git

在本例中，我們使用了 Apache 設定中常用的 `/var/www/htdocs` 路徑，不過你可以使用任何靜態 web 服務——把純倉庫放在它的目錄裡就行了。 Git 的數據是以最基本的靜態文件的形式提供的（關於如何提供文件的詳情見第9章）。

通過HTTP進行推送操作也是可能的，不過這種做法不太常見並且牽扯到複雜的 WebDAV 設定。由於很少用到，本書將略過對該內容的討論。如果對 HTTP 推送協議感興趣，不妨在這個地址看一下操作方法：`http://www.kernel.org/pub/software/scm/git/docs/howto/setup-git-server-over-http.txt` 。通過 HTTP 推送的好處之一是你可以使用任何 WebDAV 服務器，不需要為 Git 設定特殊環境；所以如果主機提供商支持通過 WebDAV 更新網站內容，你也可以使用這項功能。

#### 優點 ####

使用 HTTP 協議的好處是易於架設。幾條必要的命令就可以讓全世界讀取到倉庫的內容。花費不過幾分鐘。HTTP 協議不會佔用過多服務器資源。因為它一般只用到靜態的 HTTP 服務提供所有的數據，普通的 Apache 服務器平均每秒能供應數千個文件——哪怕是讓一個小型的服務器超載都很難。

你也可以通過 HTTPS 提供只讀的倉庫，這意味著你可以加密傳輸內容；你甚至可以要求客戶端使用特定簽名的 SSL 證書。一般情況下，如果到了這一步，使用 SSH 公共密鑰可能是更簡單的方案；不過也存在一些特殊情況，這時通過 HTTPS 使用帶簽名的 SSL 證書或者其他基於 HTTP 的只讀連接授權方式是更好的解決方案。

HTTP 還有個額外的好處：HTTP 是一個如此常見的協議，以至於企業級防火牆通常都允許其端口的通信。

#### 缺點 ####

HTTP 協議的消極面在於，相對來說客戶端效率更低。克隆或者下載倉庫內容可能會花費更多時間，而且 HTTP 傳輸的體積和網絡開銷比其他任何一個協議都大。因為它沒有按需供應的能力——傳輸過程中沒有服務端的動態計算——因而 HTTP 協議經常會被稱為 _傻瓜(dumb)_ 協議。更多 HTTP 協議和其他協議效率上的差異見第九章。

## 在服務器部署 Git ##

開始架設 Git 服務器的時候，需要把一個現存的倉庫導出為新的純倉庫——不包含當前工作目錄的倉庫。方法非常直截了當。
把一個倉庫克隆為純倉庫，可以使用 clone 命令的 `--bare` 選項。純倉庫的目錄名以 `.git` 結尾， 如下：

	$ git clone --bare my_project my_project.git
	Initialized empty Git repository in /opt/projects/my_project.git/

該命令的輸出有點迷惑人。由於 `clone` 基本上等於 `git init` 加 `git fetch`，這裡出現的就是 `git init` 的輸出，它建立了一個空目錄。實際的對象轉換不會有任何輸出，不過確實發生了。現在在 `my_project.git` 中已經有了一份 Git 目錄數據的副本。

大體上相當於

	$ cp -Rf my_project/.git my_project.git

在配置文件中有幾個小改變；不過從效果角度講，克隆的內容是一樣的。它僅包含了 Git 目錄，沒有工作目錄，並且專門為之（譯註： Git 目錄）建立了一個單獨的目錄。

### 將純目錄轉移到服務器 ###

有了倉庫的純副本以後，剩下的就是把它放在服務器上並設定相關的協議。假設一個域名為 `git.example.com` 的服務器已經架設好，並可以通過 SSH 訪問，而你想把所有的 Git 倉庫儲存在 `/opt/git` 目錄下。只要把純倉庫複製上去：

	$ scp -r my_project.git user@git.example.com:/opt/git

現在，其他對該服務器具有 SSH 訪問權限並可以讀取 `/opt/git` 的用戶可以用以下命令克隆：

	$ git clone user@git.example.com:/opt/git/my_project.git

假如一個 SSH 用戶對 `/opt/git/my_project.git` 目錄有寫權限，他會自動具有推送權限。這時如果運行 `git init` 命令的時候加上 `--shared` 選項，Git 會自動對該倉庫加入可寫的組。

	$ ssh user@git.example.com
	$ cd /opt/git/my_project.git
	$ git init --bare --shared

可見選擇一個 Git 倉庫，創建一個純的版本，最後把它放在你和同事都有 SSH 訪問權的服務器上是多麼容易。現在已經可以開始在同一項目上密切合作了。

值得注意的是，這的的確確是架設一個少數人具有連接權的 Git 服務的全部——只要在服務器上加入可以用 SSH 接入的帳號，然後把純倉庫放在大家都有讀寫權限的地方。一切都做好了，無須更多。

下面的幾節中，你會瞭解如何擴展到更複雜的設定。這些內容包含如何避免為每一個用戶建立一個賬戶，給倉庫添加公共讀取權限，架設網頁界面，使用 Gitosis 工具等等。然而，只是和幾個人在一個不公開的項目上合作的話，僅僅是一個 SSH 服務器和純倉庫就足夠了，請牢記這一點。

### 小型安裝 ###

如果設備較少或者你只想在小型的開發團隊裡嘗試 Git ，那麼一切都很簡單。架設 Git 服務最複雜的方面之一在於賬戶管理。如果需要倉庫對特定的用戶可讀，而給另一部分用戶讀寫權限，那麼訪問和許可的安排就比較困難。

#### SSH 連接 ####

如果已經有了一個所有開發成員都可以用 SSH 訪問的服務器，架設第一個服務器將變得異常簡單，幾乎什麼都不用做（正如上節中介紹的那樣）。如果需要對倉庫進行更複雜的訪問控制，只要使用服務器操作系統的本地文件訪問許可機制就行了。

如果需要團隊裡的每個人都對倉庫有寫權限，又不能給每個人在服務器上建立賬戶，那麼提供 SSH 連接就是唯一的選擇了。我們假設用來共享倉庫的服務器已經安裝了 SSH 服務，而且你通過它訪問服務器。

有好幾個辦法可以讓團隊的每個人都有訪問權。第一個辦法是給每個人建立一個賬戶，直截了當但過於繁瑣。反覆的運行 `adduser` 並且給所有人設定臨時密碼可不是好玩的。

第二個辦法是在主機上建立一個 `git` 賬戶，讓每個需要寫權限的人發送一個 SSH 公鑰，然後將其加入 `git` 賬戶的 `~/.ssh/authorized_keys` 文件。這樣一來，所有人都將通過 `git` 賬戶訪問主機。這絲毫不會影響提交的數據——訪問主機用的身份不會影響commit的記錄。

另一個辦法是讓 SSH 服務器通過某個 LDAP 服務，或者其他已經設定好的集中授權機制，來進行授權。只要每個人都能獲得主機的 shell 訪問權，任何可用的 SSH 授權機制都能達到相同效果。

## 生成 SSH 公鑰 ##

話雖如此，大多數 Git 服務器使用 SSH 公鑰來授權。為了得到授權，系統中的每個沒有公鑰用戶都得生成一個新的。該過程在所有操作系統上都差不多。
首先，確定一下是否已經有一個公鑰了。SSH 公鑰默認儲存在賬戶的 `~/.ssh` 目錄。進入那裡並查看其內容，有沒有公鑰一目瞭然：

	$ cd ~/.ssh
	$ ls
	authorized_keys2  id_dsa       known_hosts
	config            id_dsa.pub

關鍵是看有沒有用 文件名 和 文件名.pub 來命名的一對文件，這個 文件名 通常是 `id_dsa` 或者 `id_rsa`。 `.pub` 文件是公鑰，另一個文件是密鑰。假如沒有這些文件（或者乾脆連 `.ssh` 目錄都沒有），你可以用 `ssh-keygen` 的程序來建立它們，該程序在 Linux/Mac 系統由 SSH 包提供， 在 Windows 上則包含在 MSysGit 包裡：

	$ ssh-keygen 
	Generating public/private rsa key pair.
	Enter file in which to save the key (/Users/schacon/.ssh/id_rsa): 
	Enter passphrase (empty for no passphrase): 
	Enter same passphrase again: 
	Your identification has been saved in /Users/schacon/.ssh/id_rsa.
	Your public key has been saved in /Users/schacon/.ssh/id_rsa.pub.
	The key fingerprint is:
	43:c5:5b:5f:b1:f1:50:43:ad:20:a6:92:6a:1f:9a:3a schacon@agadorlaptop.local

它先要求你確認保存公鑰的位置（`.ssh/id_rsa`），然後它會讓你重複一個密碼兩次，如果不想在使用公鑰的時候輸入密碼，可以留空。

現在，所有做過這一步的用戶都得把它們的公鑰給你或者 Git 服務器的管理者（假設 SSH 服務被設定為使用公鑰機制）。他們只需要複製 `.put` 文件的內容然後 e-email 之。公鑰的樣子大致如下：

	$ cat ~/.ssh/id_rsa.pub 
	ssh-rsa AAAAB3NzaC1yc2EAAAABIwAAAQEAklOUpkDHrfHY17SbrmTIpNLTGK9Tjom/BWDSU
	GPl+nafzlHDTYW7hdI4yZ5ew18JH4JW9jbhUFrviQzM7xlELEVf4h9lFX5QVkbPppSwg0cda3
	Pbv7kOdJ/MTyBlWXFCR+HAo3FXRitBqxiX1nKhXpHAZsMciLq8V6RjsNAQwdsdMFvSlVK/7XA
	t3FaoJoAsncM1Q9x5+3V0Ww68/eIFmb1zuUFljQJKprrX88XypNDvjYNby6vw/Pb0rwert/En
	mZ+AW4OZPnTPI89ZPmVMLuayrD2cE86Z/il8b+gw3r3+1nKatmIkjn2so1d01QraTlMqVSsbx
	NrRFi9wrf+M7Q== schacon@agadorlaptop.local

關於在多個操作系統上設立相同 SSH 公鑰的教程，可以在 GitHub 有關 SSH 公鑰的嚮導中找到：`http://github.com/guides/providing-your-ssh-key`。

## 架設服務器 ##

現在我們過一邊服務器端架設 SSH 訪問的流程。本例將使用 `authorized_keys` 方法來給用戶授權。我們還將假定使用類似 Ubuntu 這樣的標準 Linux 發行版。首先，創建一個 'git' 用戶並為其創建一個 `.ssh` 目錄（譯註：在用戶的主目錄下）。

	$ sudo adduser git
	$ su git
	$ cd
	$ mkdir .ssh

接下來，把開發者的 SSH 公鑰添加到這個用戶的 `authorized_keys` 文件中。假設你通過 e-mail 收到了幾個公鑰並存到了臨時文件裡。重複一下，公鑰大致看起來是這個樣子：

	$ cat /tmp/id_rsa.john.pub
	ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABAQCB007n/ww+ouN4gSLKssMxXnBOvf9LGt4L
	ojG6rs6hPB09j9R/T17/x4lhJA0F3FR1rP6kYBRsWj2aThGw6HXLm9/5zytK6Ztg3RPKK+4k
	Yjh6541NYsnEAZuXz0jTTyAUfrtU3Z5E003C4oxOj6H0rfIF1kKI9MAQLMdpGW1GYEIgS9Ez
	Sdfd8AcCIicTDWbqLAcU4UpkaX8KyGlLwsNuuGztobF8m72ALC/nLF6JLtPofwFBlgc+myiv
	O7TCUSBdLQlgMVOFq1I2uPWQOkOWQAHukEOmfjy2jctxSDBQ220ymjaNsHT4kgtZg2AYYgPq
	dAv8JggJICUvax2T9va5 gsg-keypair

只要把它們加入 `authorized_keys` 文件（譯註：本例加入到了文件尾部）：

	$ cat /tmp/id_rsa.john.pub >> ~/.ssh/authorized_keys
	$ cat /tmp/id_rsa.josie.pub >> ~/.ssh/authorized_keys
	$ cat /tmp/id_rsa.jessica.pub >> ~/.ssh/authorized_keys

現在可以使用 `--bare` 選項運行 `git init` 來設定一個空倉庫，這會初始化一個不包含工作目錄的倉庫。

	$ cd /opt/git
	$ mkdir project.git
	$ cd project.git
	$ git --bare init

這時，Join，Josie 或者 Jessica 就可以把它加為遠程倉庫，推送一個分支，從而把第一個版本的工程上傳到倉庫裡了。值得注意的是，每次添加一個新項目都需要通過 shell 登入主機並創建一個純倉庫。我們不妨以 `gitserver` 作為 `git` 用戶和倉庫所在的主機名。如果你在網絡內部運行該主機，並且在 DNS 中設定 `gitserver` 指向該主機，那麼以下這些命令都是可用的：

	# 在 John 的電腦上
	$ cd myproject
	$ git init
	$ git add .
	$ git commit -m 'initial commit'
	$ git remote add origin git@gitserver:/opt/git/project.git
	$ git push origin master

這樣，其他人的克隆和推送也一樣變得很簡單：

	$ git clone git@gitserver:/opt/git/project.git
	$ vim README
	$ git commit -am 'fix for the README file'
	$ git push origin master

用這個方法可以很快捷的為少數幾個開發者架設一個可讀寫的 Git 服務。

作為一個額外的防範措施，你可以用 Git 自帶的 `git-shell` 簡單工具來把 `git` 用戶的活動限制在僅與 Git 相關。把它設為 `git` 用戶登入的 shell，那麼該用戶就不能擁有主機正常的 shell 訪問權。為了實現這一點，需要指明用戶的登入shell 是 `git-shell` ，而不是 bash 或者 csh。你可能得編輯 `/etc/passwd` 文件：

	$ sudo vim /etc/passwd

在文件末尾，你應該能找到類似這樣的行：

	git:x:1000:1000::/home/git:/bin/sh

把 `bin/sh` 改為 `/usr/bin/git-shell` （或者用 `which git-shell` 查看它的位置）。該行修改後的樣子如下：

	git:x:1000:1000::/home/git:/usr/bin/git-shell

現在 `git` 用戶只能用 SSH 連接來推送和獲取 Git 倉庫，而不能直接使用主機 shell。嘗試登錄的話，你會看到下面這樣的拒絕信息：

	$ ssh git@gitserver
	fatal: What do you think I am? A shell? （你以為我是個啥？shell嗎？)
	Connection to gitserver closed. （gitserver 連接已斷開。）

## 公共訪問 ##

匿名的讀取權限該怎麼實現呢？也許除了內部私有的項目之外，你還需要託管一些開源項目。抑或你使用一些自動化的服務器來進行編譯，或者一些經常變化的服務器群組，而又不想整天生成新的 SSH 密鑰——總之，你需要簡單的匿名讀取權限。

或許對小型的配置來說最簡單的辦法就是運行一個靜態 web 服務，把它的根目錄設定為 Git 倉庫所在的位置，然後開啟本章第一節提到的 `post-update` 掛鉤。這裡繼續使用之前的例子。假設倉庫處於 `/opt/git` 目錄，主機上運行著 Apache 服務。重申一下，任何 web 服務程序都可以達到相同效果；作為範例，我們將用一些基本的 Apache 設定來展示大體需要的步驟。

首先，開啟掛鉤：

	$ cd project.git
	$ mv hooks/post-update.sample hooks/post-update
	$ chmod a+x hooks/post-update

假如使用的 Git 版本小於 1.6，那 `mv` 命令可以省略—— Git 是從較晚的版本才開始在掛鉤實例的結尾添加 .sample 後綴名的。

`post-update` 掛鉤是做什麼的呢？其內容大致如下：

	$ cat .git/hooks/post-update 
	#!/bin/sh
	exec git-update-server-info

意思是當通過 SSH 向服務器推送時，Git 將運行這個命令來更新 HTTP 獲取所需的文件。

其次，在 Apache 配置文件中添加一個 VirtualHost 條目，把根文件（譯註： DocumentRoot 參數）設定為 Git 項目的根目錄。假定 DNS 服務已經配置好，會把 `.gitserver` 發送到任何你所在的主機來運行這些：

	<VirtualHost *:80>
	    ServerName git.gitserver
	    DocumentRoot /opt/git
	    <Directory /opt/git/>
	        Order allow, deny
	        allow from all
	    </Directory>
	</VirtualHost>

另外，需要把 `/opt/git` 目錄的 Unix 用戶組設定為 `www-data` ，這樣 web 服務才可以讀取倉庫內容，因為 Apache 運行 CGI 腳本的模塊（默認）使用的是該用戶：

	$ chgrp -R www-data /opt/git

重啟 Apache 之後，就可以通過項目的 URL 來克隆該目錄下的倉庫了。

	$ git clone http://git.gitserver/project.git

這一招可以讓你在幾分鐘內為相當數量的用戶架設好基於 HTTP 的讀取權限。另一個提供非授權訪問的簡單方法是開啟一個 Git 守護進程，不過這將要求該進程的常駐——下一節將是想走這條路的人準備的。

## 網頁界面 GitWeb ##

如今我們的項目已經有了讀寫和只讀的連接方式，也許應該再架設一個簡單的網頁界面使其更加可視化。為此，Git 自帶了一個叫做 GitWeb 的 CGI 腳本。你可以在類似 `http://git.kernel.org` 這樣的站點找到 GitWeb 的應用實例（見圖 4-1）。

Insert 18333fig0401.png 
Figure 4-1. 基於網頁的 GitWeb 用戶界面

如果想知道項目的 GitWeb 長什麼樣，Git 自帶了一個命令，可以在類似 `lighttpd` 或 `webrick` 這樣輕量級的服務器程序上打開一個臨時的實例。在 Linux 主機上通常都安裝了 `lighttpd` ，這時就可以在項目目錄裡輸入 `git instaweb` 來運行它。如果使用的是 Mac ，Leopard 預裝了 Ruby，所以 `webrick` 應該是最好的選擇。使用 lighttpd 以外的程序來啟用 `git instaweb`， 可以通過它的 `--httpd` 選項來實現。

	$ git instaweb --httpd=webrick
	[2009-02-21 10:02:21] INFO  WEBrick 1.3.1
	[2009-02-21 10:02:21] INFO  ruby 1.8.6 (2008-03-03) [universal-darwin9.0]

這會在 1234 端口開啟一個 HTTPD 服務，隨之在瀏覽器中顯示該頁。簡單的很。需要關閉服務的時候，只要使用相同命令的 `--stop` 選項就好了：

	$ git instaweb --httpd=webrick --stop

如果需要為團隊或者某個開源項目長期的運行 web 界面，那麼 CGI 腳本就要由正常的網頁服務來運行。一些 Linux 發行版可以通過 `apt` 或 `yum` 安裝一個叫做 `gitweb` 的軟件包，不妨首先嘗試一下。我們將快速的介紹一下手動安裝 GitWeb 的流程。首先，你需要 Git 的源碼，其中帶有 GitWeb，並能生成 CGI 腳本：

	$ git clone git://git.kernel.org/pub/scm/git/git.git
	$ cd git/
	$ make GITWEB_PROJECTROOT="/opt/git" \
	        prefix=/usr gitweb/gitweb.cgi
	$ sudo cp -Rf gitweb /var/www/

注意通過指定 `GITWEB_PROJECTROOT` 變量告訴編譯命令 Git 倉庫的位置。然後，讓 Apache 來提供腳本的 CGI，為此添加一個 VirtualHost：

	<VirtualHost *:80>
	    ServerName gitserver
	    DocumentRoot /var/www/gitweb
	    <Directory /var/www/gitweb>
	        Options ExecCGI +FollowSymLinks +SymLinksIfOwnerMatch
	        AllowOverride All
	        order allow,deny
	        Allow from all
	        AddHandler cgi-script cgi
	        DirectoryIndex gitweb.cgi
	    </Directory>
	</VirtualHost>

不難想像，GitWeb 可以使用任何兼容 CGI 的網頁服務來運行；如果偏向使用其他的（譯註：這裡指Apache 以外的服務），配置也不會很麻煩。現在，通過 `http://gitserver` 就可以在線訪問倉庫了，在 `http://git.server` 上還可以通過 HTTP 克隆和獲取倉庫的內容。
Again, GitWeb can be served with any CGI capable web server; if you prefer to use something else, it shouldn't be difficult to set up. At this point, you should be able to visit `http://gitserver/` to view your repositories online, and you can use `http://git.gitserver` to clone and fetch your repositories over HTTP.

## 權限管理器 Gitosis ##

把所有用戶的公鑰保存在 `authorized_keys` 文件的做法只能暫時奏效。當用戶數量到了幾百人的時候，它會變成一種痛苦。每一次都必須進入服務器的 shell，而且缺少對連接的限制——文件裡的每個人都對所有項目擁有讀寫權限。

現在，是時候向廣泛使用的軟件 Gitosis 求救了。Gitosis 簡單的說就是一套用來管理 `authorized_keys` 文件和實現簡單連接限制的腳本。最有意思的是，該軟件用來添加用戶和設定權限的界面不是網頁，而是一個特殊的 Git 倉庫。你只需要設定好某個項目；然後推送，Gitosis 就會隨之改變服務器設定，酷吧？

Gitosis 的安裝算不上傻瓜化，不過也不算太難。用 Linux 服務器架設起來最簡單——以下例子中的服務器使用 Ubuntu 8.10 系統。

Gitosis 需要使用部分 Python 工具，所以首先要安裝 Python 的 setuptools 包，在 Ubuntu 中名為 python-setuptools：

	$ apt-get install python-setuptools

接下來，從項目主頁克隆和安裝 Gitosis：

	$ git clone git://eagain.net/gitosis.git
	$ cd gitosis
	$ sudo python setup.py install

這會安裝幾個 Gitosis 用的可執行文件。現在，Gitosis 想把它的倉庫放在 `/home/git`，倒也可以。不過我們的倉庫已經建立在 `/opt/git` 了，這時可以創建一個文件連接，而不用從頭開始重新配置：

	$ ln -s /opt/git /home/git/repositories

Gitosis 將為我們管理公鑰，所以當前的文件需要刪除，以後再重新添加公鑰，並且讓 Gitosis 自動控制 `authorized_keys` 文件。現在，把 `authorized_keys`文件移走：

	$ mv /home/git/.ssh/authorized_keys /home/git/.ssh/ak.bak

然後恢復 'git' 用戶的 shell，假設之前把它改成了 `git-shell` 命令。其他人仍然不能通過它來登錄系統，不過這次有 Gitosis 幫我們實現。所以現在把 `/etc/passwd` 文件的這一行

	git:x:1000:1000::/home/git:/usr/bin/git-shell

恢復成:

	git:x:1000:1000::/home/git:/bin/sh

現在就可以初始化 Gitosis 了。需要通過自己的公鑰來運行 `gitosis-init`。如果公鑰不在服務器上，則必須複製一份：

	$ sudo -H -u git gitosis-init < /tmp/id_dsa.pub
	Initialized empty Git repository in /opt/git/gitosis-admin.git/
	Reinitialized existing Git repository in /opt/git/gitosis-admin.git/

這樣該公鑰的擁有者就能修改包含著 Gitosis 設置的那個 Git 倉庫了。然後手動將這個新的控制倉庫中的 	`post-update` 腳本加上執行權限。

	$ sudo chmod 755 /opt/git/gitosis-admin.git/hooks/post-update

萬事俱備了。如果設定過程沒出什麼差錯，現在可以試一下用初始化 Gitosis 公鑰的擁有者身份 SSH 進服務器。看到的結果應該和下面類似：

	$ ssh git@gitserver
	PTY allocation request failed on channel 0
	fatal: unrecognized command 'gitosis-serve schacon@quaternion'
	  Connection to gitserver closed.

說明 Gitosis 認出了該用戶的身份，但由於沒有運行任何 Git 命令所以它切斷了連接。所以，現在運行一個確切的 Git 命令——克隆 Gitosis 的控制倉庫：

	# 在自己的電腦上
	$ git clone git@gitserver:gitosis-admin.git

得到一個名為 `gitosis-admin` 的目錄，主要由兩部分組成：

	$ cd gitosis-admin
	$ find .
	./gitosis.conf
	./keydir
	./keydir/scott.pub

`gitosis.conf` 文件是用來設置用戶、倉庫和權限的控制文件。`keydir` 目錄則是保存所有具有訪問權限用戶公鑰的地方——每人一個。你 `keydir` 中的文件名（前例中的 `scott.pub`）應該有所不同—— Gitosis 從使用 `gitosis-init` 腳本導入的公鑰尾部的描述中獲取該名。

看一下 `gitosis.conf` 的內容，它應該只包含與剛剛克隆的 `gitosis-admin` 相關的信息：

	$ cat gitosis.conf 
	[gitosis]

	[group gitosis-admin]
	writable = gitosis-admin
	members = scott

它顯示用戶 `scott` ——初始化 Gitosis 公鑰的擁有者——是唯一能訪問 `gitosis-admin` 項目的人。

現在我們添加一個新的項目。我們將添加一個名為 `mobile` 的新節段，在這裡羅列手機開發團隊的開發者以及他們需要訪問權限的項目。由於 'scott' 是系統中的唯一用戶，我們把它加成唯一的用戶，從創建一個叫做 `iphone_project` 的新項目開始：

	[group mobile]
	writable = iphone_project
	members = scott

一旦修改了 `gitosis-admin` 項目的內容，只有提交並推送至服務器才能使之生效：

	$ git commit -am 'add iphone_project and mobile group'
	[master]: created 8962da8: "changed name"
	 1 files changed, 4 insertions(+), 0 deletions(-)
	$ git push
	Counting objects: 5, done.
	Compressing objects: 100% (2/2), done.
	Writing objects: 100% (3/3), 272 bytes, done.
	Total 3 (delta 1), reused 0 (delta 0)
	To git@gitserver:/opt/git/gitosis-admin.git
	   fb27aec..8962da8  master -> master

第一次向新工程 `iphone_project` 的推送需要在本地的版本中把服務器添加為一個 remote 然後推送。從此手動為新項目在服務器上創建純倉庫的麻煩就是歷史了—— Gitosis 會在第一次遇到推送的時候自動創建它們：

	$ git remote add origin git@gitserver:iphone_project.git
	$ git push origin master
	Initialized empty Git repository in /opt/git/iphone_project.git/
	Counting objects: 3, done.
	Writing objects: 100% (3/3), 230 bytes, done.
	Total 3 (delta 0), reused 0 (delta 0)
	To git@gitserver:iphone_project.git
	 * [new branch]      master -> master

注意到路徑被忽略了（加上它反而沒用），只有一個冒號加項目的名字—— Gitosis 會為你找到項目的位置。

要和朋友們共同在一個項目上共同工作，就得重新添加他們的公鑰。不過這次不用在服務器上一個一個手動添加到 `~/.ssh/authorized_keys` 文件末端，而是在 `keydir` 目錄為每一個公鑰添加一個文件。文件的命名將決定在 `gitosis.conf` 文件中用戶的稱呼。現在我們為 John，Josie 和 Jessica 添加公鑰：

	$ cp /tmp/id_rsa.john.pub keydir/john.pub
	$ cp /tmp/id_rsa.josie.pub keydir/josie.pub
	$ cp /tmp/id_rsa.jessica.pub keydir/jessica.pub

然後把他們都加進 'mobile' 團隊，讓他們對 `iphone_project` 具有讀寫權限：

	[group mobile]
	writable = iphone_project
	members = scott john josie jessica

如果你提交並推送這個修改，四個用戶將同時具有該項目的讀寫權限。

Gitosis 也具有簡單的訪問控制功能。如果想讓 John 只有讀權限，可以這樣做：

	[group mobile]
	writable = iphone_project
	members = scott josie jessica

	[group mobile_ro]
	readonly = iphone_project
	members = john

現在 John 可以克隆和獲取更新，但 Gitosis 不會允許他向項目推送任何內容。這樣的組可以有儘可能有隨意多個，每一個包含不同的用戶和項目。甚至可以指定某個組為成員，來繼承它所有的成員。

如果出現了什麼問題，把 `loglevel=DEBUG` 加入到 `[gitosis]` 部分或許有幫助（譯註：把日誌設置到調試級別，記錄更詳細的信息）。如果你一不小心搞錯了配置，失去了推送權限，可以手動修改服務器上的 `/home/git/.gitosis` 文件—— Gitosis 從該文件讀取信息。一次推送會把 `gitosis.conf` 保存在服務器上。如果你手動編輯該文件，它將在你下次向 `gitosis-admin` 推送之前它將保持原樣。

## Git 進程 ##

公共，非授權的只讀訪問要求我們在 HTTP 協議的基礎上使用 Git 協議。主因在於速度。Git 協議更為高效，進而比 HTTP 協議更迅速，所以它能節省很多時間。

重申一下，這一點只適用於非授權、只讀的訪問。如果在防火牆之外的服務器上，該服務的使用應該侷限於公諸於世的項目。假如是在防火牆之內，它也可以用於具有大量參與人員或者主機（長期整合資源或編譯的服務器）的只讀訪問的項目，可以省去為逐一添加 SSH 公鑰的麻煩。

無論哪種情況，Git 協議的設定都相對簡單。基本上，只要以長期守護進程的形式運行該命令：

	git daemon --reuseaddr --base-path=/opt/git/ /opt/git/

`--reuseaddr` 使得服務無須等到舊的連接嘗試過期以後再重啟，`--base-path` 選項使得克隆項目的時候不用給出完整的路徑，而最後面的路徑告訴 Git 進程導出倉庫的位置。假如有防火牆，則需要為該主機的 9418 端口打個允許通信的洞。

有幾個不同的辦法可以讓該進程長期駐留，取決於不同的操作系統。在 Ubuntu 主機上，可以用 Upstart 腳本來完成。於是，在下面這個文件

	/etc/event.d/local-git-daemon

加入該腳本內容：

	start on startup
	stop on shutdown
	exec /usr/bin/git daemon \
	    --user=git --group=git \
	    --reuseaddr \
	    --base-path=/opt/git/ \
	    /opt/git/
	respawn

出於安全考慮，強烈建議用一個對倉庫只有讀取權限的用戶身份來運行該進程——只需要簡單的新創建一個 `git-ro` 用戶（譯註：並將它對倉庫的權限設為只讀），用它來運行進程。為了簡化，下面我們將依舊使用運行了 Gitosis 的 'git' 用戶。

重啟主機的時候，Git 進程會自行啟動，一旦關閉了也會自行重啟。要不重啟就開啟它，可以運行這個命令：

	initctl start local-git-daemon

在其他系統上，或許應該使用 `xinetd`，`sysinit` 的一個腳本，或者其他的——只要能讓那個命令進程化和可監控。

然後，必須告訴 Gitosis 服務那些倉庫允許基於 Git 協議的非授權訪問。如果為每一個倉庫設立了自己的節段，就可以指定想讓 Git 進程給予可讀權限的倉庫。假如要允許通過 Git 協議訪問前面的 iphone 項目，可以把如下內容加到 `gitosis.conf` 文件的結尾：

	[repo iphone_project]
	daemon = yes

在提交和推送完成以後，運行中的進程將開始相應所有能訪問主機 9418 端口的人發來的項目請求。

假如不想使用 Gitosis，而又想架設一個 Git 協議進程，則必須為每一個想使用 Git 進程的項目運行如下命令：

	$ cd /path/to/project.git
	$ touch git-daemon-export-ok

該文件（譯註：指空文件 git-deamon-export-ok）告訴 Git 允許對該項目的非授權訪問。

Gitosis 還能控制 GitWeb 顯示哪些項目。首先，在 `/etc/gitweb.conf` 添加如下內容：

	$projects_list = "/home/git/gitosis/projects.list";
	$projectroot = "/home/git/repositories";
	$export_ok = "git-daemon-export-ok";
	@git_base_url_list = ('git://gitserver');

通過在 Gitosis 的設置文件裡添加或刪除 `gitweb` 設定，就能控制 GitWeb 允許用戶瀏覽哪些項目。比如，我們想讓 iphone 項目在 GitWeb 裡出現，把 `repo` 的設定改成下面的樣子：

	[repo iphone_project]
	daemon = yes
	gitweb = yes

如果現在提交和推送該項目，GitWeb 會自動開始展示我們的 iphone 項目。

## Git 託管服務 ##

如果不想經歷自己架設 Git 服務器的麻煩，網絡上有幾個專業的倉庫託管服務可供選擇。這樣做有幾大優點：託管賬戶的建立通常比較省時，方便項目的啟動，而且不涉及服務其的維護和監控。即使內部創建並運行了自己的服務器，為開源的代碼使用一個公共託管站點還是有好處——讓開源社區更方便的找到該項目並給予幫助。

目前，可供選擇的託管服務數量繁多，各有利弊。在 Git 官方 wiki 上的 Githosting 頁面有一個持續更新的託管服務列表：

	http://git.or.cz/gitwiki/GitHosting

由於本書無法全部一一介紹它們，而本人（譯註：指本書作者 Scott Chacon ）剛好在其中之一工作，我們將在這一節介紹一下在 GitHub 建立賬戶和開啟新項目的過程。為你提供一個使用託管服務的大致印象。

GitHub 是到目前為止最大的開源 Git 託管服務，並且是少數同時提供公共託管和私人託管服務的站點之一，所以你可以在一個站點同時保存開源和商業代碼。事實上，本書正是私下使用 GitHub 合寫的。（譯註：而本書的翻譯也是在 GitHub 上進行公共合作的）。

### GitHub ###

GitHub 和大多數的代碼託管站點在處理項目命名空間的方式上略有不同。GitHub 的設計更側重於用戶，而不是而不是全部基於項目。意謂本人在 GitHub 上託管一個 `grit` 項目的話，它將不會出現在 `github.com/grit`，而是在 `github.com/shacon/grit` （譯註：作者在 GitHub 上的用戶名是 shacon）。不存在所謂某個項目的官方版本，所以假如第一作者放棄了某個項目，它可以無縫轉移到其它用戶的旗下。

GitHub 同時也是一個向使用私有倉庫的用戶收取費用的商業公司，不過所有人都可以快捷的得到一個免費賬戶並且在上面託管任意多的開源項目。我們將快速介紹一下該過程。

### 建立賬戶 ###

第一個必要必要步驟是註冊一個免費的賬戶。訪問 Pricing and Signup （價格與註冊）頁面 `http://github.com/plans` 並點擊 Free acount （免費賬戶）的 "Sign Up（註冊）" 按鈕（見圖 4-2），進入註冊頁面。
The first thing you need to do is set up a free user account. If you visit the Pricing and Signup page at `http://github.com/plans` and click the "Sign Up" button on the Free account (see figure 4-2), you're taken to the signup page.

Insert 18333fig0402.png
Figure 4-2. GitHub 服務簡介頁面

這裡要求選擇一個系統中尚未存在的用戶名，提供一個與之相連的電郵地址，以及一個密碼（見圖 4-3）。

Insert 18333fig0403.png 
Figure 4-3. The GitHub user signup form

如果事先有準備，可以順便提供 SSH 公鑰。我們在前文中的"小型安裝" 一節介紹過生成新公鑰的方法。把生成的鑰匙對中的公鑰粘貼到 SSH Public Key （SSH 公鑰）文本框中。點擊 "explain ssh keys" 鏈接可以獲取在所有主流操作系統上完成該步驟的介紹。
點擊 "I agree，sign me up （同意條款，讓我註冊）" 按鈕就能進入新用戶的控制面板（見圖 4-4）。

Insert 18333fig0404.png 
Figure 4-4. GitHub 用戶面板

然後就可以建立新倉庫了。

### 建立新倉庫 ###

點擊用戶面板上倉庫旁邊的 "create a new one（新建）" 連接。進入 Create a New Repository （新建倉庫）表格（見圖 4-5）。

Insert 18333fig0405.png 
Figure 4-5. 在 GitHub 建立新倉庫

唯一必做的僅僅是提供一個項目名稱，當然也可以添加一點描述。搞定這些以後，點 "Create Repository（建立倉庫）" 按鈕。新倉庫就建立起來了（見圖4-6）。

Insert 18333fig0406.png 
Figure 4-6. GitHub 項目頭信息

由於還沒有提交代碼，GitHub 會展示如何創建一個新項目，如何推送一個現存項目，以及如何從一個公共的 Subversion 倉庫導入項目（譯註：這簡直是公開挖 google code 和 sourceforge 的牆角）（見圖 4-7）。

Insert 18333fig0407.png 
Figure 4-7. 新倉庫指南

該指南和本書前文中的介紹類似。要把一個非 Git 項目變成 Git 項目，運行

	$ git init
	$ git add .
	$ git commit -m 'initial commit'

一旦擁有一個本地 Git 倉庫，把 GitHub 添加為遠程倉庫並推送 master 分支：

	$ git remote add origin git@github.com:testinguser/iphone_project.git
	$ git push origin master

這時該項目就託管在 GitHub 上了。你可以把它的 URL 發給每個希望分享該工程的人。本例的 URL 是 `http://github.com/testinguser/iphone_project`。你將在項目頁面的頭部發現有兩個 Git URL（見圖 4-8）。

Insert 18333fig0408.png 
Figure 4-8. 項目開頭的公共 URL 和私有 URL 。

Public Clone URL（公共克隆 URL）是一個公開的，只讀的 Git URL，任何人都可以通過它克隆該項目。可以隨意的散播這個 URL，發步到個人網站之類的地方。

Your Clone URL（私用克隆 URL）是一個給予 SSH 的讀寫 URL，只有使用與上傳的 SSH 公鑰對應的密鑰來連接時，才能通過它進行讀寫操作。其他用戶訪問項目頁面的時候看不到該URL——只有公共的那個。

### 從 Subversion 中導入項目 ###

如果想把某個公共 Subversion 項目導入 Git，GitHub 可以幫忙。在指南的最後有一個指嚮導入 Subversion 頁面的鏈接。點擊它，可以得到一個表格，它包含著有關導入流程的信息以及一個用來粘貼公共 Subversion 項目連接的文本框（見圖 4-9）。

Insert 18333fig0409.png 
Figure 4-9. Subversion 導入界面

如果項目很大，採用非標準結構，或者是私有的，那麼該流程將不適用。在第七章，你將瞭解到手動導入複雜工程的方法。

### 開始合作 ###

現在把團隊裡其他的人也加進來。如果 John，Josie 和 Jessica 都在 GitHub 註冊了賬戶，要給他們向倉庫推送的訪問權，可以把它們加為項目合作者。這樣他們的公鑰就能用來向倉庫推送了。

點擊項目頁面上方的 "edit（編輯）" 按鈕或者頂部的 Admin （管理）標籤進入項目管理頁面（見圖 4-10）。

Insert 18333fig0410.png 
Figure 4-10. GitHub 管理頁面

為了給另一個用戶添加項目的寫權限，點擊 "Add another collaborator（添加另一個合作者）" 鏈接。一個新文本框會出現，用來輸入用戶名。在輸入用戶名的同時將會跳出一個幫助提示，顯示出可能匹配的用戶名。找到正確的用戶名以後，點 Add （添加）按鈕，把它變成該項目的合作者（見圖 4-11）。

Insert 18333fig0411.png 
Figure 4-11. 為項目添加合作者

添加完合作者以後，就可以在 Repository Collaborators （倉庫合作者）區域看到他們的列表（見圖 4-12）。

Insert 18333fig0412.png 
Figure 4-12. 項目合作者列表

如果需要取消某人的訪問權，點擊 "revoke （撤銷）"，他的推送權限就被刪除了。在未來的項目中，可以通過複製現存項目的權限設定來得到相同的合作者群組。

### 項目頁面 ###

在推送或從 Subversion 導入項目之後，你會得到一個類似圖 4-13 的項目主頁。

Insert 18333fig0413.png 
Figure 4-13. GitHub 項目主頁

其他人訪問你的項目時，他們會看到該頁面。它包含了該項目不同方面的標籤。Commits 標籤將按時間展示逆序的 commit 列表，與 `git log` 命令的輸出類似。Network 標籤展示所有 fork 了該項目並做出貢獻的用戶的關係圖。Downloads 標籤允許你上傳項目的二進制文件，並提供了指向該項目所有標記過的位置的 tar/zip 打包下載連接。Wiki 標籤提供了一個用來撰寫文檔或其他項目相關信息的 wiki。Graphs 標籤包含了一些可視化的項目信息與數據。剛開始進入的 Source 標籤頁面列出了項目的主目錄；並且在下方自動展示 README 文件的內容（如果該文件存在的話）。該標籤還包含了最近一次提交的相關信息。

### 派生（forking）項目 ###

如果想向一個自己沒有推送權限的項目貢獻代碼，GitHub 提倡使用派生（forking）。在你發現一個感興趣的項目，打算在上面 Hack 一把的時候，可以點擊頁面上方的 "fork（派生）" 按鈕，GitHub 會為你的用戶複製一份該項目，這樣你就可以向它推送內容了。

使用這個辦法，項目維護者不用操心為了推送權限把其他人加為合作者的麻煩。大家可以派生一個項目副本並進行推送，而後項目的主要維護者可以把這些副本添加為遠程倉庫，從中拉取更新的內容進行合併。

要派生一個項目，到該項目的頁面（本例中是 mojombo/chronic）點擊上面的 "fork" 按鈕（見圖 4-14）。

Insert 18333fig0414.png 
Figure 4-14. 點擊 "fork" 按鈕來獲得任意項目的可寫副本

幾秒鐘以後，你將進入新建的項目頁面，顯示出該項目是派生自另一個項目的副本（見圖 4-15）。

Insert 18333fig0415.png 
Figure 4-15. 你派生的項目副本

### GitHub 小節 ###

GitHub 就介紹這麼多，不過意識到做到這些是多麼快捷十分重要。不過幾分鐘的時間，你就能創建一個賬戶，添加一個新的項目並開始推送。如果你的項目是開源的，它還同時獲得了對龐大的開發者社區的可視性，社區成員可能會派生它並做出貢獻。退一萬步講，這至少是個快速開始嘗試 Git 的好辦法。

## 小節 ##

幾個不同的方案可以讓你獲得遠程 Git 倉庫來與其他人合作或分享你的成果。

運行自己的服務器意味著更多的控制權以及在防火牆內部操作的可能性，然而這樣的服務器通常需要投入一定的時間來架設和維護。如果把數據放在託管服務上，假設和維護變得十分簡單；然而，你不得不把代碼保存在別人的服務器上，很多公司不允許這種做法。

使用哪個方案或哪種方案的組合對你和你的團隊更合適，應該不是一個太難的決定。

