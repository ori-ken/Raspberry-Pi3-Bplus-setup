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

