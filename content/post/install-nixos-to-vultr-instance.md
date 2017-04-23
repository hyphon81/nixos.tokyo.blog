+++
categories = ["NixOS", "Install", "ZFS", "VPS", "Vultr"]
date = "2017-04-23T23:02:49+09:00"
draft = true
tags = ["NixOS", "Install", "ZFS", "VPS", "Vultr"]
thumbnail = "images/logo_onwhite.png"
title = "Installed NixOS to Vultr Instance"
+++

<h2>NixOSをVultrのVPSにインストールしました。</h2>
<!--more-->

コストパフォーマンスが良く、東京リージョンのあるVPS
-- Vultr -- のインスタンスに、
NixOSをインストールしました。
その時のことについて、以下でまとめます。

<a href="https://www.vultr.com/?ref=7144485"><img src="https://www.vultr.com/media/banner_1.png" width="728" height="90"></a>

<h3>NixOSをインストール可能なサーバを借りることのできるところ</h3>

ざっと検索したところ、VPSやベアメタルサーバでNixOSを利用できるところが
次の3箇所見つかりました。

- <h4>ユーザがISOイメージをアップロードすることでNixOSが使用可能なところ</h4>
  - <a href="https://www.vultr.com/">Vultr</a>
  - <a href="https://www.onamae-server.com/vps/">お名前VPS</a>

- <h4>NixOSに対応しているところ</h4>
  - <a href="https://www.packet.net/">packet.net</a>

<a href="https://www.vultr.com/">Vultr</a>は3つの中で提示された性能と料金を見た場合のパフォーマンスが良い感じでした。

<a href="https://www.onamae-server.com/vps/">お名前VPS</a>は3つの中では唯一、日本語に対応している(というか、日本の会社が運営している)VPSになります。

<a href="https://www.packet.net/">packet.net</a>はベアメタルサーバのためか、
最安のプランの性能と料金が高めで、東京リージョンがUS/EUと比べて高めという、
まぁ、そんな感じです。Type-2AプランのARMサーバには魅かれます。

今回は、VultrのインスタンスにNixOSをインストールしました。

<h3>Vultrの料金</h3>

料金は1時間ごとに作成しているインスタンスに応じて課金されるようです。
月額にすると$2.5(だいたい￥300ぐらい)からのプランがあります。
これが、CPU 1 core、メモリ 512 MB、SSD 20 GBです。
ただし、これを書いている現在、$2.5のインスタンスはSold Outとなっています。
その1段階上の性能のインスタンスは月額$5でCPU 1 core、メモリ 1024 MB、SSD 25 GBとなっています。

個人的には、似たサービスとして<a href="https://www.conoha.jp/">Conoha</a>を対照に比較した場合、
Vultrの方が使い勝手が良く、コストパフォーマンスも良いと思いました。
他人に勧める場合の難点はサポートも英語であることくらいでしょうか。

ちなみに支払い方法は、クレジットカード、Paypal、Bitcoinが選べます。

<h3>Vultrのアカウント作成後にまずやったこと ~ ISOのアップロード ~</h3>

Vultrのアカウントは簡単に作成できると思います。

アカウント作成後にダッシュボードにアクセスして、
Servers -> ISO -> Add ISO の順番にクリックすれば、
ISOファイルのアップロードができます。

![operation01](/blog/images/vultr001.png)

NixOSのISOイメージファイルは、<a href="https://nixos.org/nixos/download.html">ここ</a>からダウンロードできます。
今回は、特にサーバ上でGUIを使うことは無いので、
"Minimal installation CD, 64-bit Intel/AMD"をダウンロードして使用しました。

VultrではISOイメージファイルのダウンロードを、
サーバ上のファイルのURLを指定することで行います。
なので、手元のPCにファイルを落としておく必要はないです。


<h3>インスタンスの作成</h3>

NixOSのISOファイルがアップロードできたら、インスタンスを作成します。
インスタンスの作成は右上の方に浮いてる"+"のボタンをクリックすればできます。

まず、特に海外を選ぶ理由が無ければServer LocationはTokyo Japanを選びましょう。

Server TypeはUpload ISOをクリックすれば、先程の操作でアップロードしたものを選ぶことができます。

Server Sizeはスペックと値段を考えて選びましょう。

Additional Featuresはよくわからなければチェックしなくても良いです。
一応、私はIPv6とPrivate Networkにチェックを入れておきました。
(この2つは特に追加料金とかは無いです。)

Server Hostname &amp; Labelは好きに選びましょう。
ただ、NixOSをインストールする都合上、
ここで設定するHostnameはあまり意味がないかもしれません。

入力が済んだら、Servers Qtyが1であることや料金を確認しつつ、
Deploy Nowをクリックします。
これで、インスタンスが起動します。

<h3>NixOSインストールイメージのブート後の操作</h3>

作成したインスタンスのStatusがRunningになったら、
インスタンスのところをクリックして、右上にあるアイコンの列の
一番左のモニタのようなアイコンをクリックして、
コンソールを開きます。

![operation01](/blog/images/vultr002.png)

こんな画面が見られるハズです。

今回、ただインストールするだけでは面白くないので、
ZFS上にNixOSを入れてみました。

インストール時の操作の注意点として、
loadkeys jp106の設定後に日本語キーボードが上手く動作せず、
入力が上手くいかない場合があります。
この問題を回避するため、手元のPCからSSHで接続して
操作を行いました。
そのための準備として、次の操作を行います。

        [root@nixos:~]# chmod +w /etc/nixos/configuration.nix
        [root@nixos:~]# nano /etc/nixos/configuration.nix

起動したnanoエディタで以下の行を追加します。

                { config, pkgs, ... }:

                {
                  imports = [ <nixpkgs/nixos/modules/installer/cd-dvd/installation-cd-minimal.nix> ];
        ++追加    boot.supportedFilesystems = [ "zfs" ];
        ++追加    services.openssh.enable = true;
                }

入力に当たって、先に書いた問題が発生することがあります。
その場合は、おそらくデフォルトで"="が入力できないと思われるので、
"="以外を先に入力し、一旦Ctrl+X -> y で保存してプロンプトに戻った後で
loadkeys jp106を入力し、nanoで編集を行います。
(nanoである理由は、viだと":"が入力できなくなり面倒だから)

        [root@nixos:~]# loadkeys jp106
        [root@nixos:~]# nano /etc/nixos/configuration.nix

_この時"="は、Shift + 右Shiftの隣のキー"\_(アンダースコア)"にマップされてました。_

/etc/nixos/configuration.nixが無事に編集できたら、
nixos-rebuild switchを実行して設定を反映した後、
rootパスワードの設定とsshdの起動(なぜか起動してない)をします。

        [root@nixos:~]# nixos-rebuild switch

        ~~~

        [root@nixos:~]# passwd
        rootパスワードの設定

        ~~~

        [root@nixos:~]# systemctl start sshd

これで、SSHでログインができるようになりました。
接続先のIPアドレスはダッシュボードから確認できます。

        [hyphon81@mylaptop:~]$ ssh root@<接続先IP>
        The authenticity of host '<接続先IP> (<接続先IP>)' can't be established.
        ED25519 key fingerprint is SHA256:~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
        Are you sure you want to continue connecting (yes/no)? yes
        Warning: Permanently added '<接続先IP>' (ED25519) to the list of known hosts.
        Password:

        [root@nixos:~]#

ログインできたら、ディスクを確認します。

        [root@nixos:~]# lsblk
        NAME  MAJ:MIN RM   SIZE RO TYPE MOUNTPOINT
        loop0   7:0    0 287.8M  1 loop /nix/.ro-store
        sr0    11:0    1   315M  0 rom  /iso
        vda   253:0    0    25G  0 disk

vdaがインストール先のSSDのディスクになります。
ちなみに、UEFIは使用できないようなので、GPTは設定できません。
素直にfdiskを使いましょう。

        [root@nixos:~]# fdisk /dev/vda

        ~~~~~~~

        ~~bootパーティションの設定~~
        Command (m for help): n       <- nを入力
        Partition type
        p   primary (0 primary, 0 extended, 4 free)
        e   extended (container for logical partitions)
        Select (default p):           <- そのままEnter
        
        Using default response p.
        Partition number (1-4, default 1): <- そのままEnter
        First sector (2048-52428799, default 2048): <- そのままEnter
        Last sector, +sectors or +size{K,M,G,T,P} (2048-52428799, default 52428799): +512M <- +512Mを入力

        ~~swapパーティションの設定~~
        Command (m for help): n       <- nを入力
        Partition type
        p   primary (0 primary, 0 extended, 4 free)
        e   extended (container for logical partitions)
        Select (default p):           <- そのままEnter
        
        Using default response p.
        Partition number (1-4, default 2): <- そのままEnter
        First sector (1050624-52428799, default 1050624): <- そのままEnter
        Last sector, +sectors or +size{K,M,G,T,P} (1050624-52428799, default 52428799): +1G <- インスタンスのメモリのサイズに合わせて設定する

        Created a new partition 2 of type 'Linux' and of size 1 GiB.

        Command (m for help): t       <- tを入力
        Partition number (1,2, default 2): 2  <- 2を入力
        Partition type (type L to list all types): 82 <- 82を入力

        Changed type of partition 'Linux' to 'Linux swap / Solaris'。

        ~~zfsのパーティションの設定~~
        Command (m for help): n       <- nを入力
        Partition type
        p   primary (0 primary, 0 extended, 4 free)
        e   extended (container for logical partitions)
        Select (default p):           <- そのままEnter
        
        Using default response p.
        Partition number (1-4, default 3): <- そのままEnter
        First sector (3147776-52428799, default 3147776): <- そのままEnter
        Last sector, +sectors or +size{K,M,G,T,P} (3147776-52428799, default 52428799): <- そのままEnter

        Created a new partition 3 of type 'Linux' and of size 23.5 GiB.

        Command (m for help): t       <- tを入力
        Partition number (1,2, default 2): 3  <- 3を入力
        Partition type (type L to list all types): bf <- bfを入力

        Changed type of partition 'Linux' to 'Solaris'.

        ~~パーティションテーブルの書き込みとfdiskの終了~~
        Command (m for help): w <- wを入力
        The partition table has been altered.
        Calling ioctl() to re-read partition table.
        Syncing disks.


        [root@nixos:~]#

これで、パーティションの準備ができました。

        [root@nixos:~]# lsblk
        NAME   MAJ:MIN RM   SIZE RO TYPE MOUNTPOINT
        loop0    7:0    0 287.8M  1 loop /nix/.ro-store
        sr0     11:0    1   315M  0 rom  /iso
        vda    253:0    0    25G  0 disk 
        ├─vda1 253:1    0   512M  0 part 
        ├─vda2 253:2    0     1G  0 part
        └─vda3 253:3    0  23.5G  0 part

次にzpoolを作成し、zfsの設定をしていきます。

        [root@nixos:~]# zpool create -o ashift=12 -o altroot=/mnt rpool /dev/vda3
        
        [root@nixos:~]# zfs create -o mountpoint=none rpool/root
        
        [root@nixos:~]# zfs create -o mountpoint=legacy rpool/root/nixos

        [root@nixos:~]# zfs create -o mountpoint=legacy rpool/home

        [root@nixos:~]# zfs set compression=lz4 rpool/home <- ファイルシステムの圧縮がやりたい場合はお好みで設定

        [root@nixos:~]# zfs list
        NAME               USED  AVAIL  REFER  MOUNTPOINT
        rpool              636K  22.6G    96K  /mnt/rpool
        rpool/home          96K  22.6G    96K  legacy
        rpool/root         192K  22.6G    96K  none
        rpool/root/nixos    96K  22.6G    96K  legacy

インストールをするために、/mnt以下にファイルシステムをマウントします。

        [root@nixos:~]# mount -t zfs rpool/root/nixos /mnt

        [root@nixos:~]# mkdir /mnt/home

        [root@nixos:~]# mount -t zfs rpool/home /mnt/home

        [root@nixos:~]# mkfs.ext4 /dev/vda1
        mke2fs 1.43.4 (31-Jan-2017)
        Creating filesystem with 131072 4k blocks and 32768 inodes
        Filesystem UUID: 5b19f65e-84ce-4404-8211-2f978eebd9d7
        Superblock backups stored on blocks: 
                32768, 98304

        Allocating group tables: done                            
        Writing inode tables: done                            
        Creating journal (4096 blocks): done
        Writing superblocks and filesystem accounting information: done


        [root@nixos:~]# mkdir /mnt/boot

        [root@nixos:~]# mount /dev/vda1 /mnt/boot

        [root@nixos:~]# mkswap /dev/vda2
        Setting up swapspace version 1, size = 1024 MiB (1073737728 bytes)
        no label, UUID=55921a00-cc14-48f8-85d3-65684a57cd21

        [root@nixos:~]# swapon /dev/vda2

インストールするNixOSのconfigration.nixファイルを生成します。

        [root@nixos:~]# nixos-generate-config --root /mnt
        writing /mnt/etc/nixos/hardware-configuration.nix...
        writing /mnt/etc/nixos/configuration.nix...

hostidを確認しておきます。

        [root@nixos:~]# hostid
        8425e349

生成されたhardware-configuration.nixを確認しておきます。
妙なところはたぶん無いとおもいますが。

        [root@nixos:~]# cat /mnt/etc/nixos/hardware-configuration.nix 
        # Do not modify this file!  It was generated by ‘nixos-generate-config’
        # and may be overwritten by future invocations.  Please make changes
        # to /etc/nixos/configuration.nix instead.
        { config, lib, pkgs, ... }:

        {
          imports =
            [ <nixpkgs/nixos/modules/profiles/qemu-guest.nix>
            ];

          boot.initrd.availableKernelModules = [ "ata_piix" "uhci_hcd" "virtio_pci" "sr_mod" "virtio_blk" ];
          boot.kernelModules = [ ];
          boot.extraModulePackages = [ ];

          fileSystems."/" =
            { device = "rpool/root/nixos";
              fsType = "zfs";
            };

          fileSystems."/home" =
            { device = "rpool/home";
              fsType = "zfs";
            };

          fileSystems."/boot" =
            { device = "/dev/disk/by-uuid/<UUID>";
              fsType = "ext4";
            };

          swapDevices =
            [ { device = "/dev/disk/by-uuid/<UUID>"; }
            ];

          nix.maxJobs = lib.mkDefault 1;
        }

configuration.nixを編集します。

        [root@nixos:~]# nano /mnt/etc/nixos/configuration.nix

以下の設定は必ず必要です。

        # Use the GRUB 2 boot loader.
        boot.loader.grub.enable = true;
        boot.loader.grub.version = 2;
        boot.loader.grub.device = "/dev/vda"; <- bootデバイスの指定
        boot.zfs.devNodes = "/dev"; <- zpoolを/dev以下のデバイスから読み込むように指定

        networking.hostId = "8425e349"; <- hostidを設定

        services.openssh.enable = true; <- これがないと非常にやり辛い

あとはお好みで。

とりあえず、ロケールやタイムゾーン、ユーザの設定をしておけば良いと思います。

        # Select internationalisation properties.
        i18n = {
          consoleFont = "Lat2-Terminus16";
          consoleKeyMap = "jp106";
          defaultLocale = "en_US.UTF-8";
        };

        # Set your time zone.
        time.timeZone = "Asia/Tokyo";

        users.extraUsers.guest = {
          isNormalUser = true;
          createHome = true;
          extraGroups = [ "wheel" ];
          uid = 1000;
        };

いよいよインストールです。

        [root@nixos:~]# nixos-install

インストールの最後にrootパスワードの設定があります。

インストールができたら、Vultrのダッシュボード側からインスタンスをStopします。
必ずダッシュボード側からStopするのを忘れないでください。

インスタンスがStopしたら、ダッシュボードのServers -> インスタンスで
インスタンスの設定に入って、そこから、
Settings -> Custom ISOでRemove ISOのボタンが見えます。
これをクリックします。
OKをクリックすると、ISOがインスタンスからRemoveされ、
その後、勝手に再起動します。

すべての設定が上手くいっていれば、インスタンスのコンソールから
NixOSが起動しているのが確認できます。

        <<< Welcome to NixOS 17.03.944.7ad99e9fc8 (x86_64) - tty1 >>>


        nixos login: root
        Password:

        [root@nixos:~]# passwd guest
        Enter new UNIX password:
        Retype new UNIX password:
        passwd: password updated successfully

        [root@nixos:~]#

一般ユーザのパスワードを設定してSSHでアクセスする準備ができました。
アクセスの前に~/.ssh/known_hostsの中の、作成したインスタンスに対応する
IPの行を1度消すのをお忘れなく。

<h3>インストール完了後の設定について</h3>

一応、SSHのためのPublic Keyの設定とSSHのポートの変更ぐらいは
やっておいた方が良いでしょうか。

セキュリティと言えば、Vultrではアカウントの2段階認証の設定も可能なので、
こちらも設定した方が良いでしょう。

<h3>参考</h3>

- <a href="https://github.com/Tokyo-NixOS/Tokyo-NixOS-Meetup-Wiki/wiki/install">Tokyo-NixOS-Meetup-WikiのNixOSのインストールの項</a>
  - これを読めば自分の記事はハッキリいってあまり必要ないかも
  - というか、<a href="https://github.com/Tokyo-NixOS/Tokyo-NixOS-Meetup-Wiki/wiki/vultr">Vultrへのインストールについての項</a>もあります

- <a href="https://nixos.org/wiki/ZFS_on_NixOS">https://nixos.org/wiki/ZFS_on_NixOS</a>
  - NixOSをZFSにインストールする場合についても書かれています
  - ただ、上の方にNixOS wikiはシャットダウン中とあって、いずれなくなる模様
  