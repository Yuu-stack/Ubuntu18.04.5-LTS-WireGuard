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

少し前までは 18.04 だとリポジトリを追加する必要がありましたが下記コマンドを実行すると現在はしたの様に案内されます.    
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

