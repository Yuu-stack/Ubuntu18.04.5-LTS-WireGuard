# Ubuntu18.04.5-LTS-WireGuard  

`ubuntu-18.04.4-preinstalled-server-arm64+raspi3.img.xz`を使用しています.  
 https://ubuntu.com/download/raspberry-pi
 http://cdimage.ubuntu.com/releases/18.04/release/ubuntu-18.04.4-preinstalled-server-arm64+raspi3.img.xz
 
 
下記で local IP固定化済み.   
 https://github.com/Yuu-stack/Ubuntu18.04.5-LTS-InitialSetting/blob/master/README.md  
 
 下記コマンドを実行し現在の環境をメモしておく.  
 `$ uname -a && lsb_release -a`
 
    Linux ubuntu 5.3.0-1030-raspi2 #32~18.04.2-Ubuntu SMP Fri Jul 24 09:24:50 UTC 2020 aarch64 aarch64 aarch64 GNU/Linux
    No LSB modules are available.
    Distributor ID:	Ubuntu
    Description:	Ubuntu 18.04.5 LTS
    Release:	18.04
    Codename:	bionic
    

# このページでできる事  
01.サーバー側でのWireGuardのセットアップ  
02.クライアントPC側でのWireGuardのセットアップ  
03.サーバー側での設定ファイルの作成    
04.クライアントPC側での設定ファイルの作成  
05.動作確認  
06.終わりに  

# 01.サーバー側でのWireGuardのセットアップ

少し前までは 18.04 だとリポジトリを追加する必要がありましたが下記コマンドを実行すると現在以下の様に案内されます.    
> <s>$ sudo add-apt-repository ppa:wireguard/wireguard </s>  
> <s>$ sudo apt update </s>  
> <s>$ sudo apt install wireguard </s>  

> If you're reading this because you're following an online tutorial on  
> installing WireGuard, please contact the authors to request that they  
> simplify their instructions to simply.  
> 
>     `$ sudo apt install wireguard`  
> 
> This PPA is no longer required for WireGuard. Press CTRL+C, and do not  
> proceed with adding it.  

`$ sudo apt update && sudo apt install wireguard`

セットアップをしていきます.  

#サーバーのセットアップ  

`$ wg genkey | sudo tee server.key | wg pubkey > server.pub`  

これで秘密鍵と公開鍵がいっきにつくれます。  
wg genkeyもwg pubkeyも標準出力されますが、念のためファイル化  


    $ sudo cat server.key server.pub
    cPwR3oTYkXQMftAfyo7nfCCx/pKNOOc7OxETBaukL3U=
    LdUheSydosMTVpiCwQ6DhX9AL0WgS/B6my7268l20VE=


鍵はこんな感じになってます。これは設定ファイルでつかいます。  
念のため秘密鍵はパーミッションをおとしておきます  
`$ sudo chmod 600 server.key `  

# 02.クライアントPC側でのWireGuardのセットアップ  

つぎはクライアント 今回は Mac から接続します  
まずは Homebrew で WireGuard をインストール  

`$ brew install wireguard-tools`  

そしてサーバーと同じ様に鍵を作成  

`$ wg genkey | sudo tee client.key | wg pubkey > client.pub`


    $ cat client.key && cat client.pub
    cEliXXsM26Ng7YdEYsMzYdt/nBfRi0TaQ8+DmllhUk8=
    JB9uqhP3g9jYP+dripsztCEKTk+nCRPN7JwvK81kRSQ=

こちらも念の為パーミッションをおとしておきます.  

`$ sudo chmod 600 client.key`  

# 03.サーバー側での設定ファイルの作成  

サーバーにもどって、設定ファイルを用意します。
`/etc/wireguard/`に、今回は`wg0.conf`というファイル名でつくります.  
`wg0.conf`の部分はつくりたいインターフェイス名でも大丈夫です.  

`$ sudo vim /etc/wireguard/wg0.conf`
中身は  

    [Interface]                                                                    
    PrivateKey = cPwR3oTYkXQMftAfyo7nfCCx/pKNOOc7OxETBaukL3U=  //server側で作成した`server.key`
    Address = 192.168.1.1/24  //serverに設定するIPアドレス
    ListenPort = 51820        //サーバー側のポート
    SaveConfig = true
    PostUp = iptables -A FORWARD -o %i -j ACCEPT; iptables -t nat -A POSTROUTING -o     eth0 -j MASQUERADE; ip6tables -A FORWARD -o %i     ACCEPT; ip6tables -t nat -A     POSTROUTING -o eth0 -j MASQUERADE
    PostDown = iptables -D FORWARD -o %i -j ACCEPT; iptables -t nat -D POSTROUTING     -o eth0 -j MASQUERADE; ip6tables -D FORWARD -o %i -j ACCEPT; ip6tables -t nat -    D POSTROUTING -o eth0 -j MASQUERADE
    
    [Peer]
    PublicKey = JB9uqhP3g9jYP+dripsztCEKTk+nCRPN7JwvK81kRSQ=  //クライアント側で作成した`client.pub`
    AllowedIPs = 192.168.1.2/24                               //クライアントに設定するIPアドレス




[Interface]はサーバーの設定で、[Peer]はクライアントの設定です。  
今回は WireGuard に192.168.1.0/24のサブネットを割り当てて、  
サーバーのインターフェイスに192.168.1.1、  
クライアントに192.168.1.2/24を割り当てることにする。   

PostUpとPostDownは WireGuard が起動しているあいだだけ、  
iptablesの IP マスカレードをする設定です。  
eth0は適宜じぶんの物理インターフェイスにかきかえてください。  

[Peer]のPublicKeyはクライアントの公開鍵です。  

念のため設定ファイル(wg0.conf)のパーミッションもおとしておく。

`$ sudo chmod 600 wg0.conf`

#つぎは WireGuard のネットワークから、  
インターネットへトラフィックをながす設定をします。  
Ubuntu はデフォルトでパケットのフォワーディングを禁止しているので、  
/etc/sysctl.confを編集しておく。  

`$ sudo vi /etc/sysctl.conf`  
該当のばしょのコメント化を解除する。  

    # Uncomment the next line to enable packet forwarding for IPv4
    net.ipv4.ip_forward=1
    # Uncomment the next line to enable packet forwarding for IPv6
    # Enabling this option disables Stateless Address Autoconfiguration
    # based on Router Advertisements for this host
    net.ipv6.conf.all.forwarding=1


そしてsysctlコマンドで変更を反映させます。

    $ sudo sysctl -p
    net.ipv4.ip_forward = 1
    net.ipv6.conf.all.forwarding = 1

iptablesまたはufwのファイアウォールをつかっている場合は、  
ポート番号の許可もしておきます。  

`$ sudo iptables -A INPUT -p udp --dport 51820 -j ACCEPT`

ufwの場合は  

    $ sudo ufw allow 51820/udp
    $ sudo ufw enable # 設定を有効化する
    $ sudo ufw status numbered # 現在の設定を確認する
    
あとは、じっさいに WireGuard を起動していきます。  
起動と停止には、wg-quickというラッパースクリプトを使用します.  
systemdのユニットファイルはwg-quick@インターフェイス名です.  

    $ sudo systemctl enable wg-quick@wg0
    $ sudo systemctl start wg-quick@wg0

ipコマンドでインターフェイス(wg0)が生成されていることを確認します.  

    $ ip a
    1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
        link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
        inet 127.0.0.1/8 scope host lo
           valid_lft forever preferred_lft forever
        inet6 ::1/128 scope host 
           valid_lft forever preferred_lft forever
    2: eth0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc fq_codel state UP group default qlen 1000
        link/ether b8:27:eb:c8:66:c8 brd ff:ff:ff:ff:ff:ff
        inet 192.168.10.170/24 brd 192.168.10.255 scope global eth0
           valid_lft forever preferred_lft forever
        inet6 fe80::ba27:ebff:fec8:66c8/64 scope link 
           valid_lft forever preferred_lft forever
    3: wlan0: <BROADCAST,MULTICAST> mtu 1500 qdisc noop state DOWN group default qlen 1000
        link/ether b8:27:eb:9d:33:9d brd ff:ff:ff:ff:ff:ff
    4: wg0: <POINTOPOINT,NOARP,UP,LOWER_UP> mtu 1420 qdisc noqueue state UNKNOWN group default qlen 1000
        link/none 
        inet 192.168.1.1/24 scope global wg0
           valid_lft forever preferred_lft forever
           

これでサーバー側は完了です。おつかれさまでした.  
…といいたいところだけど、以下のようなエラーがでることがある.  

> $ sudo systemctl start wg-quick@wg0
> Job for wg-quick@wg0.service failed because the control process exited with error code.
> See "systemctl status wg-quick@wg0.service" and "journalctl -xe" for details.

だいたいの場合、 WireGuard のカーネルモジュールがロードされていない。
以下のコマンドを実行してみる。

`$ sudo modprobe wireguard`  

エラーがでるようなら、いったん再起動  
これでおりました.  

App Store で WireGuard クライアントをダウンロードしてきて、接続してみる。  

Add Tunnel...から設定をかきこむ。  
`$ sudo vim wg0.conf`  

    [Interface]
    PrivateKey = cEliXXsM26Ng7YdEYsMzYdt/nBfRi0TaQ8+DmllhUk8=
    Address = 192.168.1.2/32
    DNS = 1.1.1.1
    
    [Peer]
    PublicKey = LdUheSydosMTVpiCwQ6DhX9AL0WgS/B6my7268l20VE=
    Endpoint = example.com:51820
    AllowedIPs = 0.0.0.0/0,::/0
    
サーバーとくらべると、だいぶシンプル.  

[Interface]のPublicKeyはさいしょのほうにつくった秘密鍵、  
[Peer]のPublicKeyはサーバーの公開鍵です  

DNSのところは Cloudflare の Public DNS を指定してるが、  
Google の8.8.8.8でもいいし、 LAN 内のやつでも大丈夫です.  

AllowedIPsを上記のようにすると、すべてのトラフィックをサーバーにながします  
もちろん10.0.0.0/24のように、特定の宛先を指定してもよいです  

保存をしてActivateボタンを押せば Handshake がおこなわれて接続されます!  
おつかれさまでしたー！


