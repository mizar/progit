# Починаючи #

Цей розділ присвячений знайомству з Git. Ми почнемо з поз’яснення принципів роботи систем контролю версій, потім перейдемо до того, як підняти Git на вашій системі та нарешті як почати з ним працювати. В кінці розділу ви вже  повинні розуміти, навіщо Git існує, чому ви повинні його використовувати та чому ви повинні виконати всі інсталяційні роботи.

## Про контроль версій ##

Що таке контроль версій, і навіщо це вам потрібно? Контроль версій - це система, яка записує всі зміни до файлу чи до групи файлів з часом, так що ви можете відновити будь яку версію. Приклади в цій книзі наводяться у вигляді файлів, що містять програмний код, та знаходяться у системі контролю версій, хоча контроль версій може бути застосований до будь-якого файлу на вашому комп’ютері.

Якщо ви є графічним чи веб дизайнером та бажаєте зберігати кожну версію зображення чи схеми (а певно що насправді ви цього хочете), Система Контролю Версій (СКВ) - це дуже розумна річ для цього. Вона дозволить вам повертати файли до попереднього стану, порівнювати зміни в часі, дивитися хто останній модифікував щось, що можливо спричинило проблеми, хто призвів до проблеми, і багато іншого. Використання СКВ також означає що якщо ви прикрутите якусь штуку, або втратите файли, ви можете легко відновитися. На додачу, ви матимете все це лише за дуже невеличку "переплату"

### Локальні системи контролю версій ###

Багато людей обирає методом контролю версій копіювання файлів в іншу директорію (можливо таку директорію, що містить часову мітку, якщо вони досить розумні). Цей підхід дуже поширений, оскільки він простий, проте він також надзвичайно схильний до помилок. Дуже легко забути, в якій директорії ви зараз є та випадково записати до некоретного файлу або закопіювати поверх файлів, чого ви не хотіли.

Для того, щоби подолати ці проблеми, програмісти досить давно придумали локальні СКВ які були простими базами даних, що зберігали всі зміни у файлах, що знаходилися під контролем ревізій (дивіться Малюнок 1-1).

Insert 18333fig0101.png

Малюнок 1-1. Діаграма локального контролю версій.

Одним із найпопулярніших інструментів СКВ була система  rcs, яка і досі постачається з багатьма комп'ютерами. 

### Централізовані системи контролю версій ###

The next major issue that people encounter is that they need to collaborate with developers on other systems. To deal with this problem, Centralized Version Control Systems (CVCSs) were developed. These systems, such as CVS, Subversion, and Perforce, have a single server that contains all the versioned files, and a number of clients that check out files from that central place. For many years, this has been the standard for version control (see Figure 1-2).

Вставити зображення 18333fig0102.png

Малюнок 1-2. Діаграма централізованого контролю версій.

This setup offers many advantages, especially over local VCSs. For example, everyone knows to a certain degree what everyone else on the project is doing. Administrators have fine-grained control over who can do what; and it’s far easier to administer a CVCS than it is to deal with local databases on every client.

However, this setup also has some serious downsides. The most obvious is the single point of failure that the centralized server represents. If that server goes down for an hour, then during that hour nobody can collaborate at all or save versioned changes to anything they’re working on. If the hard disk the central database is on becomes corrupted, and proper backups haven’t been kept, you lose absolutely everything—the entire history of the project except whatever single snapshots people happen to have on their local machines. Local VCS systems suffer from this same problem—whenever you have the entire history of the project in a single place, you risk losing everything.

### Розподілені системи контролю версій ###

Час перейти до розподілених систем контролю версій (РСКВ або DVCS). В РСКВ (таких як Git, Mercurial, Bazaar чи Darcs), клієнти не витягують останній знімок стану файлів; вони роблять повне дзеркало репозиторію. Так, якщо  помре сервер, або системи, в яких вони взаємодіють, будь який з клієнтськиї репозиторієв може бути зкопійований назад на сервер для відновлення. Кожен витяг (checkout) насправді є повним бекапом всіх даних (дивисть Малюнок 1-3).

Insert 18333fig0103.png

Малюнок 1-3. Діаграма розподіленої системи контролю версій.

Крім того, багато з таких систем працюють досить добре, маючи кілька віддалених репозиторієв з якими вони можуть працювати, отже ви может співпрацювати з різними групами людей різними шляхами одночасно на одному преокті. Це дозволяє встановити кілька різних типів робочих процесів що неможливо в централізованих системах, які є ієрархічними моделями.

## Коротка історія Git ##

Як і багато великих речей в житті, Git почався з невеликої кількості креативного знищення та жорсткого протистояння. Ядро Linux є проектом з відкритим кодом з дуже великим размахом. Більшість часу підтримки ядра Linux (1991-2002), зміни у програмне забезпечення розходилися у вигляді патчів та архівованих файлів. В 2002, для проекту ядра Linux почали використовувати пропрієтарну РСКВ, яка називалася BitKeeper.

В 2005, зв’язок між спільнотою розродників ядра Linux та комерційною компанією, яка розробляла BitKeeper розірвався, і статус безкоштовного існтрументу був відкликаний. Це підказало спільноті Linux розробників (та зокрема Лінусу Торвальдсу, творцю Linux) ідею створити власний інструмент, який би базувався на уроках отриманих під час використання BitKeeper. Ось деякі з цілей, які вони ставили перед новою системою:

* Швидкість

* Проста будова

* Сильна підтримка для нелінійної розробки (тисячі паралельних гілок-бренчів)

* Повністю розподілена

* Можливість ефективної підтримки великих проектів, таких як ядро Linux (швидкість та розмір даних)

Від часу народження в 2005, Git розвинувся та визрів і тепер він є легким в використанні при цьому зберігаючи всі свої стартові якості. Він є надзвичайно швидким, дуже ефективним для великих проектів, та має надзвичайну систему гілок для нелінійної розробки (Дивіться Розділ 3).

## Основи Git ##

Отже, що ж таке Git? Це важливо зрозуміти, бо якщо ви зрозумієте що є Git та фундаментальні основи його роботи, то наймовірніше, зможете ефективно його використовувати. Під час вивчення Git, намагайтеся очстити свій мозок від речей, які ви можливо знаєте по іншим СКВ, таким як Subversion чи Perforce; це допоможе уникнути конфузій під час користування інструментами. Git зберігає та розуміє інформацію зовсім по іншому ніж ці системи, незважаючи на те, що інтерфейс багато в чому подібний. Розуміння цих відмінностей допоможе уникнути конфузій під час користування.

### Знімки, а не відмінності ###

Найбільша відмінність між Git та будь якою іншою СКВ (включаючи Subversion та дрзів) - це те, яким чином Git розуміж дані. Концептуально, більшість інших систем зберігають інформацію як список змін до файлів. Ці системи (CVS, Subversion, Perforce, Bazaar та інші) оперують інформацією, яку вони зберігають як списко файлів, та змін які були зроблені з часом до кожного файлу, як показано на Малюнку 1-4.

Insert 18333fig0104.png

Малюнок 1-4. Ініші системи що зберігають дані як зміни до базової версії кожного файлу.

Git не розглядає набір своїх даних у такому вигляді. Натомість, Git розглядає свої дані більше як набір знімків невеличких файлових систем. Кожен раз, коли ви комітете, або зберігаєте стан вашого проекту в Git, він фотографує стан всіх ваших файлів в цей момент та збергіає посилання на цей знімок. Для ефективності, якщо файли не змінювалися, Git не зберігає їх - зберігається лише посилання на попередній ідентичний файл. Git розглядає дані приблизно так, як показано на Малюнку  1-5.

Insert 18333fig0105.png

Малюнок 1-5. Git зберігає дані як знімки проекту в часі.

Це важлива різниця між Git та майже всіма іншими СКВ. Це змусило Git переглянути майже кожен акспети контролю версій, які більшість систем успадкувала від поперенії поколінь. Це змусило Git бути більш подімним на мініатюрну файлову систему з потужними інструментами поверх неї ніж на просту СКВ. Ми побачимо деякі з переваг, які ви отримуєте коли розглядаєте ваші дані таким чином коли будемо досліджувати роботу з гілками в Git в розділі 3.

### Майже всі операції локальні ###

Більшість операцій в Git вимагають для роботи лише локальних файлів та ресурсів - взагалі не потребуючи ніякої інформації від інших комп’ютерів в мережі. Коли ви працюєте з ЦСКВ де більшість операцій мають витрачають додатковий час на мережеві затримки, цей аспект Git змусить вас подумати що боги швидкості обдарували Git невимовною силою. Оскільки ви маєте всю історію проекту у себе на машині, більшість операції є майже миттєвими.

Для прикладу, щоби переглянути історію проекту, Git не потребує ходити на сервер, витягувати історію та показувати її вам - він просто читає її прямо з вашої локальної бази даниї. Це означає, що ви побачити історію проекту майже миттєво. Якщо ви захочете побачити зміни, які були внесені між поточною версією файлу та цим же файлом місяць тому, Git гляне на файл станом місяць тому там підрахує зміни локально, замісто того аби просити це зробити віддалений сервер, чи витягувати стірий файл з віддаленого сервера та тільки тоді рахувати локально.

Це також означає, що існує досить мало операцій, які ви не зможете зробити, знаходячись офлайн, або без доступу до VPN. Якщо ви знаходитеся в літаку або в поїзді та бажаєте трохи попрацювати, ви можете комітити до того часу, як у вас з’явиться мережа для завантаження змін. Якщо ви прийшли додому та ваш VPN клієнт не працює, ви все одно можете працювати. В багатьох системах це неможливо або потребує надзвичайних зусиль. В Perforce, наприклад, ви не можете багато зробити без з’єднання з сервером, в Subversion та CVS ви можете редагувати файди, але не можете комітити зміни до вашої бази (оскільки ваша база офлайн). Це може виглядати як не дуже велика перевага, але ви здивуєтеся, наскільки велика різниця.

### Git Has Integrity ###

Everything in Git is check-summed before it is stored and is then referred to by that checksum. This means it’s impossible to change the contents of any file or directory without Git knowing about it. This functionality is built into Git at the lowest levels and is integral to its philosophy. You can’t lose information in transit or get file corruption without Git being able to detect it.

The mechanism that Git uses for this checksumming is called a SHA-1 hash. This is a 40-character string composed of hexadecimal characters (0–9 and a–f) and calculated based on the contents of a file or directory structure in Git. A SHA-1 hash looks something like this:

	24b9da6552252987aa493b52f8696cd6d3b00373

You will see these hash values all over the place in Git because it uses them so much. In fact, Git stores everything not by file name but in the Git database addressable by the hash value of its contents.

### Git Generally Only Adds Data ###

When you do actions in Git, nearly all of them only add data to the Git database. It is very difficult to get the system to do anything that is undoable or make it erase data in any way. As in any VCS, you can lose or mess up changes you haven’t committed yet; but after you commit a snapshot into Git, it is very difficult to lose, especially if you regularly push your database to another repository.

This makes using Git a joy because we know we can experiment without the danger of severely screwing things up. For a more in-depth look at how Git stores its data and how you can recover data that seems lost, see “Under the Covers” in Chapter 9.

### The Three States ###

Now, pay attention. This is the main thing to remember about Git if you want the rest of your learning process to go smoothly. Git has three main states that your files can reside in: committed, modified, and staged. Committed means that the data is safely stored in your local database. Modified means that you have changed the file but have not committed it to your database yet. Staged means that you have marked a modified file in its current version to go into your next commit snapshot.

This leads us to the three main sections of a Git project: the Git directory, the working directory, and the staging area.

Insert 18333fig0106.png 

Figure 1-6. Working directory, staging area, and git directory

The Git directory is where Git stores the metadata and object database for your project. This is the most important part of Git, and it is what is copied when you clone a repository from another computer.

The working directory is a single checkout of one version of the project. These files are pulled out of the compressed database in the Git directory and placed on disk for you to use or modify.

The staging area is a simple file, generally contained in your Git directory, that stores information about what will go into your next commit. It’s sometimes referred to as the index, but it’s becoming standard to refer to it as the staging area.

The basic Git workflow goes something like this:

1.	You modify files in your working directory.

2.	You stage the files, adding snapshots of them to your staging area.

3.	You do a commit, which takes the files as they are in the staging area and stores that snapshot permanently to your Git directory.

If a particular version of a file is in the git directory, it’t considered committed. If it’s modified but has been added to the staging area, it is staged. And if it was changed since it was checked out but has not been staged, it is modified. In Chapter 2, you’ll learn more about these states and how you can either take advantage of them or skip the staged part entirely.

## Installing Git ##

Let’s get into using some Git. First things first—you have to install it. You can get it a number of ways; the two major ones are to install it from source or to install an existing package for your platform.

### Installing from Source ###

If you can, it’s generally useful to install Git from source, because you’ll get the most recent version. Each version of Git tends to include useful UI enhancements, so getting the latest version is often the best route if you feel comfortable compiling software from source. It is also the case that many Linux distributions contain very old packages; so unless you’re on a very up-to-date distro or are using backports, installing from source may be the best bet.

To install Git, you need to have the following libraries that Git depends on: curl, zlib, openssl, expat, and libiconv. For example, if you’re on a system that has yum (such as Fedora) or apt-get (such as a Debian based system), you can use one of these commands to install all of the dependencies:

	$ yum install curl-devel expat-devel gettext-devel \

	  openssl-devel zlib-devel

	$ apt-get install curl-devel expat-devel gettext-devel \

	  openssl-devel zlib-devel

	

When you have all the necessary dependencies, you can go ahead and grab the latest snapshot from the Git web site:

	http://git-scm.com/download

	

Then, compile and install:

	$ tar -zxf git-1.6.0.5.tar.gz

	$ cd git-1.6.0.5

	$ make prefix=/usr/local all

	$ sudo make prefix=/usr/local install

After this is done, you can also get Git via Git itself for updates:

	$ git clone git://git.kernel.org/pub/scm/git/git.git

	

### Installing on Linux ###

If you want to install Git on Linux via a binary installer, you can generally do so through the basic package-management tool that comes with your distribution. If you’re on Fedora, you can use yum:

	$ yum install git-core

Or if you’re on a Debian-based distribution like Ubuntu, try apt-get:

	$ apt-get instal git-core

### Installing on Mac ###

There are two easy ways to install Git on a Mac. The easiest is to use the graphical Git installer, which you can download from the Google Code page (see Figure 1-7):

http://code.google.com/p/git-osx-installer

Insert 18333fig0107.png 

Figure 1-7. Git OS X installer

The other major way is to install Git via MacPorts (http://www.macports.org). If you have MacPorts installed, install Git via

	$ sudo port install git-core +svn +doc +bash_completion +gitweb

You don’t have to add all the extras, but you’ll probably want to include +svn in case you ever have to use Git with Subversion repositories (see Chapter 8).

### Installing on Windows ###

Installing Git on Windows is very easy. The msysGit project has one of the easier installation procedures. Simply download the installer exe file from the Google Code page, and run it:

	http://code.google.com/p/msysgit

After it’s installed, you have both a command-line version (including an SSH client that will come in handy later) and the standard GUI.

## First-Time Git Setup ##

Now that you have Git on your system, you’ll want to do a few things to customize your Git environment. You should have to do these things only once; they’ll stick around between upgrades. You can also change them at any time by running through the commands again.

Git comes with a tool called git config that lets you get and set configuration variables that control all aspects of how Git looks and operates. These variables can be stored in three different places:

*	`/etc/gitconfig` file: Contains values for every user on the system and all their repositories. If you pass the option` --system` to `git config`, it reads and writes from this file specifically. 

*	`~/.gitconfig` file: Specific to your user. You can make Git read and write to this file specifically by passing the `--global` option. 

*	config file in the git directory (that is, `.git/config`) of whatever repository you’re currently using: Specific to that single repository. Each level overrides values in the previous level, so values in `.git/config` trump those in `/etc/sysconfig`.

On Windows systems, Git looks for the `.gitconfig` file in the `$HOME` directory (C:\Documents and Settings\$USER for most people). It also still looks for /etc/gitconfig, although it’s relative to the MSys root, which is wherever you decide to install Git on your Windows system when you run the installer.

### Your Identity ###

The first thing you should do when you install Git is to set your user name and e-mail address. This is important because every Git commit uses this information, and it’s immutably baked into the commits you pass around:

	$ git config --global user.name "John Doe"

	$ git config --global user.email johndoe@example.com

Again, you need to do this only once if you pass the `--global` option, because then Git will always use that information for anything you do on that system. If you want to override this with a different name or e-mail address for specific projects, you can run the command without the `--global` option when you’re in that project.

### Your Editor ###

Now that your identity is set up, you can configure the default text editor that will be used when Git needs you to type in a message. By default, Git uses your system’s default editor, which is generally Vi or Vim. If you want to use a different text editor, such as Emacs, you can do the following:

	$ git config --global core.editor emacs

	

### Your Diff Tool ###

Another useful option you may want to configure is the default diff tool to use to resolve merge conflicts. Say you want to use vimdiff:

	$ git config --global merge.tool vimdiff

Git accepts kdiff3, tkdiff, meld, xxdiff, emerge, vimdiff, gvimdiff, ecmerge, and opendiff as valid merge tools. You can also set up a custom tool; see Chapter 7 for more information about doing that.

### Checking Your Settings ###

If you want to check your settings, you can use the `git config --list` command to list all the settings Git can find at that point:

	$ git config --list

	user.name=Scott Chacon

	user.email=schacon@gmail.com

	color.status=auto

	color.branch=auto

	color.interactive=auto

	color.diff=auto

	...

You may see keys more than once, because Git reads the same key from different files (`/etc/gitconfig` and `~/.gitconfig`, for example). In this case, Git uses the last value for each unique key it sees.

You can also check what Git thinks a specific key’s value is by typing `git config {key}`:

	$ git config user.name

	Scott Chacon

## Getting Help ##

If you ever need help while using Git, there are three ways to get the manual page (manpage) help for any of the Git commands:

	$ git help <verb>

	$ git <verb> --help

	$ man git-<verb>

For example, you can get the manpage help for the config command by running

	$ git help config

These commands are nice because you can access them anywhere, even offline.

If the manpages and this book aren’t enough and you need in-person help, you can try the `#git` or `#github` channel on the Freenode IRC server (irc.freenode.net). These channels are regularly filled with hundreds of people who are all very knowledgeable about Git and are often willing to help.

## Summary ##

You should have a basic understanding of what Git is and how it’s different from the CVCS you may have been using. You should also now have a working version of Git on your system that’s set up with your personal identity. It’s now time to learn some Git basics.
