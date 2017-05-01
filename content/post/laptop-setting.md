+++
categories = []
date = "2017-05-02T05:19:21+09:00"
draft = true
tags = ["NixOS", "Install", "Nvidia", "CUDA", "bumblebee"]
thumbnail = ""
title = "Installed NixOS to Laptop that can use CUDA!"

+++

<h2>NixOSをノートPCにインストールしました。CUDAを使える形で。</h2>
<!--more-->

今までノートPCでは<a href="https://www.archlinux.org/">Arch Linux</a>を使っていましたが、
アップデートしたらEmacsの設定を読んでくれなくなってしまって、
結局どうしようもなくなったので、思い切ってNixOSに乗り換えました。

私の使用しているノートPCはNvidiaのGPUの載ったゲーミングノートなのですが、
CUDAのプログラミングがやりたくて、
その設定でいろいろ躓いたのでメモ程度に書き残しておきます。
いろいろなページを参考にしまくったので、
どこをどう参照したかがはっきりしませんすいません。

インストールは大体<a href="/blog/post/install-nixos-to-vultr-instance/">Vultrでやったとき</a>とあまり変わらないです。
(今回はZFSは使用しませんでした。あと、一応Windowsとデュアルブートにしてます。)

ノートPCへのNixOSのインストール後には、
サーバ環境ではやらなかったデスクトップ環境の設定をやったのですが、
現在NixOSのデスクトップ環境のデフォルト?になっているplasma5(KDE)を見て
気に入ったので、これを使うことにしました。

困ったのは、plasma5を有効にした状態でvideoDriversに"nvidia"が指定されている場合、
自分の場合はデスクトップ環境がOpenGLのライブラリをうまく読み込んでくれないようでした。

"Plasma is unable to start as it couldn't correctly use OpenGL 2. Please check that your graphic drivers are set up correctly."
とか言われて、デスクトップに何も表示されません。

別のデスクトップ環境であるXfceはこの状態でも起動しました。
CUDAのプログラムも動作したのですが、
glxinfoで情報がちゃんと取得できていないようでした。

そこで、最終的にplasma5 + intel videoDriver + bumblebeeの環境を
作ることにしました。

起動しなくなりました・・・

幸い既知の問題だったらしく、
<a href="https://github.com/Bumblebee-Project/Bumblebee/issues/764#issuecomment-234494238">ここ</a>を参考にカーネルパラメータを設定したら起動できました。

こうして起動するようになりましたが、
bumblebeeを単純に設定しただけだと、
optirunをsudoで実行しないとCUDAのプログラムが動かないという
状況になりました。

これはどうも、bumblebee側でnvidia_uvmのモジュールを読み込む権限がなく
sudoで実行した場合はnvidia_uvmがロードされてCUDAプログラムが実行可能になる
ということのようでした。

対策として、udevの設定を行うことでsudoなしでもoptirunで
CUDAを実行できるようにしました。

設定は以下のようになりました。

                { config, pkgsm ... }:

                let
                  ricty = pkgs.callPackage /home/hyphon81/workspace/TTF/ricty.nix {};
                in

                {
                   ...

                   boot.kernelParams = [
                     # bumblebee有効時に起動しない問題への対処
                     "acpi_osi=\"!Windows 2015\""
                   ];

                   # 日本語環境の設定
                   i18n = {
                     consoleFont = "Lat2-Terminus16";
                     consoleKeyMap = "jp106";
                     defaultLocale = "ja_JP.UTF-8";
                     inputMethod = {
                       enabled = "fcitx";
                       fcitx.engines = with pkgs.fcitx-engines; [
                         mozc
                         anthy
                       ];
                     };
                   };

                   # タイムゾーン設定
                   time.timeZone = "Asia/Tokyo";

                   # フォント設定
                   fonts = {
                     fonts = with pkgs; [
                       ricty 
                       ipafont
                       powerline-fonts
                       baekmuk-ttf
                       kochi-substitute
                       carlito
                     ];

                     fontconfig = { 
                       defaultFonts = {
                         monospace = [ 
                           "DejaVu Sans Mono for Powerline"
                           "IPAGothic"
                           "Baekmuk Dotum"
                         ];
                         serif = [ 
                           "DejaVu Serif"
                           "IPAPMincho"
                           "Baekmuk Batang"
                         ];
                         sansSerif = [
                           "DejaVu Sans"
                           "IPAPGothic"
                           "Baekmuk Dotum"
                         ];
                       };
                     };
                   };

                   # X11の設定
                   services.xserver.videoDrivers = [ "intel" ];
                   services.xserver.enable = true;
                   services.xserver.layout = "jp";

                   # bumblebeeの設定
                   hardware.bumblebee = {
                     enable = true;
                     driver = "nvidia";
                     group = "video";
                   };

                   # optirun実行時にsudoが必要な問題への対処
                   services.udev.extraRules = ''
                     # Load and unload nvidia-modeset module
                     ACTION=="add" DEVPATH=="/module/nvidia" SUBSYSTEM=="module" RUN+="${pkgs.kmod}/bin/modprobe nvidia-modeset"
                     ACTION=="remove" DEVPATH=="/module/nvidia" SUBSYSTEM=="module" RUN+="${pkgs.kmod}/bin/modprobe -r nvidia-modeset"

                     # Load and unload nvidia-uvm module
                     ACTION=="add" DEVPATH=="/module/nvidia" SUBSYSTEM=="module" RUN+="${pkgs.kmod}/bin/modprobe nvidia-uvm"
                     ACTION=="remove" DEVPATH=="/module/nvidia" SUBSYSTEM=="module" RUN+="${pkgs.kmod}/bin/modprobe -r nvidia-uvm"
                   '';

                   # 32bit OpenGLのサポート
                   hardware.opengl = {
                     driSupport32Bit = true;
                   };

                   # plasma5 + lightdm.
                   services.xserver.displayManager.lightdm = {
                     enable = true;
                   };
                   services.xserver.desktopManager.plasma5 = {
                     enable = true;
                   };

                   ...
                }

今回は以上です。