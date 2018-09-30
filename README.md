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
```


