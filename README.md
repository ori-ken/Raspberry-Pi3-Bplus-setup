# Raspberry-Pi3-Bplus-setup
Raspberry Pi 3 Model B+のセットアップメモです。購入した商品は[Raspberry Pi3 B+ コンプリートスターターキット(Standard, 16G)](https://www.amazon.co.jp/dp/B07CRY1JC1/)です。

## パワーオンまで
1. microSDを差し込む。  
1. USBマウスを接続する。  
1. USBキーボードを接続する。  
1. HDMIケーブルでテレビと本体を接続する。 
1. 電源ケーブルを本体に接続し、アダプタをコンセントに指す。

## 初期セットアップ
1. 言語設定
    1. Japaneseを選択
1. piユーザーのパスワード設定
1. wifi設定
1. OSアップデート
    1. スキップ可能

## ssh有効化
デフォルト設定ではssh接続できないため有効化する。(セキュリティ関係だそうです。詳細は[ここ。](https://www.raspberrypi.org/blog/a-security-update-for-raspbian-pixel/))
```
$ sudo touch /boot/ssh
$ sudo reboot
```
## rootパスワード設定
sudoではなくrootユーザで作業をする場合は、パスワードを設定する。
```
$ sudo passwd root
(パスワード入力する)
$ su -
# <- プロンプト変わればOK
```
## wifiへの固定IPアドレス割当て
ssh接続できてもIPアドレスがわからないと意味がないので、決まったIPアドレスが割当てられるように設定してする。
```
$ sudo vi /etc/dhcpcd.conf
(省略)
# 末尾に追記
interface wlan0
static ip_address=192.168.1.221/24      #割当てたいIPアドレス
static routers=192.168.1.1              #ネットワークのデフォルトゲートウェイ
static domain_name_servers=192.168.1.1  #ゲートウェイと同じで良いっぽい
# 追記ここまで。保存して終了
$ sudo reboot
```
## gitの設定
gitを利用するための細かな設定をする。設定後何かクローンしてみてからコミット＆プッシュして、プッシュできることを確認する。。
```
$ vi .gitconfig
以下を追加
[core]
	editor = vi

[user]
	name = "ori-ken"
	email = xxxxxxx@gmail.com
後々の自動化のためにユーザ名とパスワードを登録しておく。プッシュ時にパスワードを省略するため。
$ vi .netrc
以下を追加
machine github.com
login ori-ken
password xxxxxxx
```
## vimの導入
デフォルトのviは使いにくいのでvimを導入する。
```
バージョン確認
$ vi --version
(省略)
Small version without GUI. ...
(省略)

パッケージアンインストール
$ sudo apt-get --purge remove vim-common vim-tiny
パッケージインストール
$ sudo apt-get install vim

バージョン確認
$ vi --version
(省略)
Huge 版 without GUI. ...
(省略)

お好みで設定
$ cd
$ vi .vimrc
set ts=4
(以下略)
```
## bashrcの設定
細かい設定をする。
```
$ cd 
$ vi .bashrc
デフォルトのプロンプトはカレントディレクトリのパス階層が深くなると、そのフルパスが表示されるため見づらい。
カレントディレクトリ名だけ表示する。
#以下をコメントアウト
#    PS1='${debian_chroot:+($debian_chroot)}\[\033[01;32m\]\u@\h\[\033[00m\]:\[\033[01;34m\]\w \$\[\033[00m\] '
#以下を追加。”[pi@raspberrypi ~ ]$ ”のように表示される。
　　　　　　　　　　PS1='[\u@\h \W ]\$ '

lsエイリアスを変更して、カラー表示にする。
# enable color support of ls and also add handy aliases
if [ -x /usr/bin/dircolors ]; then
    test -r ~/.dircolors && eval "$(dircolors -b ~/.dircolors)" || eval "$(dircolors -b)"
    alias ls='ls --color=auto'　　　　#コメントアウトを外す。
    alias ll='ls -l'            #追加する。

```
## USBメモリを自動マウントする
開発用のディレクトリ/prodをUSBメモリからマウントする。購入した商品は[Samsung USBメモリ 32GB Fitタイプ 正規代理店保証品 MUF-32AB/EC](https://www.amazon.co.jp/gp/product/B07GYWPHY6/)です。
```
デフォルトのマウントは以下
$ df -h
ファイルシス   サイズ  使用  残り 使用% マウント位置
/dev/root         15G  4.0G  9.8G   29% /
devtmpfs         460M     0  460M    0% /dev
tmpfs            464M     0  464M    0% /dev/shm
tmpfs            464M   18M  446M    4% /run
tmpfs            5.0M  4.0K  5.0M    1% /run/lock
tmpfs            464M     0  464M    0% /sys/fs/cgroup
/dev/mmcblk0p1    43M   22M   21M   51% /boot
tmpfs             93M     0   93M    0% /run/user/1000

USBメモリを挿して以下を実行。/dev/sda1にマウントされていることを確認する。
$ sudo fdisk -l
(省略)
Disk /dev/sda: 29.9 GiB, 32080200192 bytes, 62656641 sectors
Units: sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
Disklabel type: dos
Disk identifier: 0xc3072e18

Device     Boot Start      End  Sectors  Size Id Type
/dev/sda1         224 62656640 62656417 29.9G  c W95 FAT32 (LBA)

パーティション情報の書込み。
$ sudo fdisk /dev/sda1
Command (m for help): d //削除。１〜４全部削除
Command (m for help): n //追加
Partition type
   p   primary (0 primary, 0 extended, 4 free)
   e   extended (container for logical partitions)
Select (default p): p
Partition number (1-4, default 1): 1
First sector (2048-62656416, default 2048): 2048
Last sector, +sectors or +size{K,M,G,T,P} (2048-62656416, default 62656416):

Created a new partition 1 of type 'Linux' and of size 29.9 GiB.

USBメモリをフォーマットを実行。
$ sudo mkfs.ext4 /dev/sda1
mke2fs 1.43.4 (31-Jan-2017)
/dev/sda1 contains a vfat file system labelled 'Samsung USB'
Proceed anyway? (y,N) y
Creating filesystem with 7831552 4k blocks and 1961712 inodes
proc            /proc           proc    defaults          0       0
Filesystem UUID: 80884d3b-5af4-41be-95f4-6b9090d49541
Superblock backups stored on blocks:
	32768, 98304, 163840, 229376, 294912, 819200, 884736, 1605632, 2654208,
	4096000

Allocating group tables: done
Writing inode tables: done
Creating journal (32768 blocks): done
Writing superblocks and filesystem accounting information: done

マウント実行。
$ sudo mkdir /prod
$ sudo mount /dev/sda1 /prod
$ df -ah
ファイルシス   サイズ  使用  残り 使用% マウント位置
/dev/root         15G  4.0G  9.8G   29% /
devtmpfs         460M     0  460M    0% /dev
tmpfs            464M     0  464M    0% /dev/shm
tmpfs            464M   18M  446M    4% /run
tmpfs            5.0M  4.0K  5.0M    1% /run/lock
tmpfs            464M     0  464M    0% /sys/fs/cgroup
/dev/mmcblk0p1    43M   22M   21M   51% /boot
tmpfs             93M     0   93M    0% /run/user/1000
/dev/sda1         30G   16K   30G    1% /prod

起動時の自動マウント設定。UUIDをメモる。
$ sudo blkid /dev/sda1
/dev/sda1: UUID="80884d3b-5af4-41be-95f4-6b9090d49541" TYPE="ext4" PARTUUID="c3072e18-01"

fstabに設定追加。fstabはバックアップしておくこと。
$ cd /etc
$ vi fstab
proc            /proc           proc    defaults          0       0
PARTUUID=fc7f0d83-01  /boot           vfat    defaults          0       2
PARTUUID=fc7f0d83-02  /               ext4    defaults,noatime  0       1
#以下追加
UUID=80884d3b-5af4-41be-95f4-6b9090d49541        /prod           ext4    defaults,nofail   0       0

cmdline.txtにディレイを仕込む。ラズパイのメモリーカードをMACなどにマウントして、cmdline.txtを編集する。
「rootwait」の前に「rootdelay=10」を追記する。
dwc_otg.lpm_enable=0 console=serial0,115200 console=tty1 root=PARTUUID=fc7f0d83-02 rootfstype=ext4 elevator=deadline fsck.repair=yes rootdelay=10 rootwait quiet splash plymouth.ignore-serial-consoles

最後に再起動して、dfコマンドでマウントされていることを確認する。
```

## cronの設定
cronを使うための設定
```
rsyslogの設定変更を行い、実行ログを出す。
# cd /etc
# vi rsyslog.conf
# First some standard log files.  Log by facility.
#
auth,authpriv.*         /var/log/auth.log
*.*;auth,authpriv.none      -/var/log/syslog
cron.*              /var/log/cron.log  <-先頭の#をとってコメントアウト解除
daemon.*            -/var/log/daemon.log
kern.*              -/var/log/kern.log
lpr.*               -/var/log/lpr.log
mail.*              -/var/log/mail.log
user.*              -/var/log/user.log

rsyslogの再起動
# systemctl restart rsyslog

crontabへのコマンド登録
$ crontab -e
# m h  dom mon dow   command
*/1 * * * * /prod/git/scraping-rbp/auto-upload >> /prod/var/log/auto-upload.log
```

## mecab-python3のインストール
機械学習の前段として、mecabをインストールする。
```
apt-getのアップデート＆アップグレード
$ sudo apt-get update
取得:1 http://archive.raspberrypi.org/debian stretch InRelease [25.4 kB]
取得:2 http://raspbian.raspberrypi.org/raspbian stretch InRelease [15.0 kB]
取得:3 http://raspbian.raspberrypi.org/raspbian stretch/main armhf Packages [11.7 MB]
取得:4 http://archive.raspberrypi.org/debian stretch/main armhf Packages [201 kB]
取得:5 http://archive.raspberrypi.org/debian stretch/ui armhf Packages [40.8 kB]
取得:6 http://raspbian.raspberrypi.org/raspbian stretch/contrib armhf Packages [56.9 kB]
取得:7 http://raspbian.raspberrypi.org/raspbian stretch/non-free armhf Packages [95.5 kB]
12.1 MB を 35秒 で取得しました (343 kB/s)
パッケージリストを読み込んでいます... 完了
$ sudo apt-get upgrade
（省略）
ca-certificates (20161130+nmu1+deb9u1) のトリガを処理しています ...
Updating certificates in /etc/ssl/certs...
0 added, 0 removed; done.
Running hooks in /etc/ca-certificates/update.d...
done.

mecab本体等のインストール
$ sudo apt-get install mecab libmecab-dev mecab-ipadic-utf8 python-mecab
(省略)
reading /usr/share/mecab/dic/juman/Demonstrative.csv ... 76
emitting double-array: 100% |###########################################|
reading /usr/share/mecab/dic/juman/matrix.def ... 1509x1509
emitting matrix      : 100% |###########################################|

done!

swigのインストール
$ sudo apt-get install swig
パッケージリストを読み込んでいます... 完了
依存関係ツリーを作成しています
状態情報を読み取っています... 完了
以下の追加パッケージがインストールされます:
  swig3.0
提案パッケージ:
  swig-doc swig-examples swig3.0-examples swig3.0-doc
以下のパッケージが新たにインストールされます:
  swig swig3.0
アップグレード: 0 個、新規インストール: 2 個、削除: 0 個、保留: 3 個。
1,510 kB のアーカイブを取得する必要があります。
この操作後に追加で 5,588 kB のディスク容量が消費されます。
続行しますか? [Y/n] Y
取得:1 http://ftp.tsukuba.wide.ad.jp/Linux/raspbian/raspbian stretch/main armhf swig3.0 armhf 3.0.10-1.1 [1,205 kB]
取得:2 http://ftp.tsukuba.wide.ad.jp/Linux/raspbian/raspbian stretch/main armhf swig armhf 3.0.10-1.1 [305 kB]
1,510 kB を 2秒 で取得しました (731 kB/s)
以前に未選択のパッケージ swig3.0 を選択しています。
(データベースを読み込んでいます ... 現在 117879 個のファイルとディレクトリがインストールされています。)
.../swig3.0_3.0.10-1.1_armhf.deb を展開する準備をしています ...
swig3.0 (3.0.10-1.1) を展開しています...
以前に未選択のパッケージ swig を選択しています。
.../swig_3.0.10-1.1_armhf.deb を展開する準備をしています ...
swig (3.0.10-1.1) を展開しています...
swig3.0 (3.0.10-1.1) を設定しています ...
man-db (2.7.6.1-2) のトリガを処理しています ...
swig (3.0.10-1.1) を設定しています ...

mecab-python3のインストール
$ sudo pip3 install mecab-python3
Collecting mecab-python3
  Using cached https://files.pythonhosted.org/packages/ac/48/295efe525df40cbc2173748eb869290e81a57e835bc41f6d3834fc5dad5f/mecab-python3-0.996.1.tar.gz
Building wheels for collected packages: mecab-python3
  Running setup.py bdist_wheel for mecab-python3 ... done
  Stored in directory: /root/.cache/pip/wheels/73/71/4f/63a79925b5e9bb38932043917cc60140beb8022ac14a952b1e
Successfully built mecab-python3
Installing collected packages: mecab-python3
Successfully installed mecab-python3-0.996.1

```
## fastTextのインストール
python3用のfastTextをインストールする。
```
githubからクローンする。
$ git clone https://github.com/facebookresearch/fastText.git
Cloning into 'fastText'...
remote: Enumerating objects: 2895, done.
remote: Total 2895 (delta 0), reused 0 (delta 0), pack-reused 2895
Receiving objects: 100% (2895/2895), 7.71 MiB | 3.00 MiB/s, done.
Resolving deltas: 100% (1828/1828), done.

rootでインストールする。
# cd fastText/
# pip3 install .
Processing /home/pi/git/fastText
Requirement already satisfied: numpy in /usr/lib/python3/dist-packages (from fasttext==0.8.22)
Requirement already satisfied: pybind11>=2.2 in /usr/local/lib/python3.5/dist-packages (from fasttext==0.8.22)
Requirement already satisfied: setuptools>=0.7.0 in /usr/lib/python3/dist-packages (from fasttext==0.8.22)
Installing collected packages: fasttext
  Running setup.py install for fasttext ... done
Successfully installed fasttext-0.8.22
$ $ pip3 list
<省略>
ExplorerHAT (0.4.2)
fasttext (0.8.22)
Flask (0.12.1)
<省略>
```
