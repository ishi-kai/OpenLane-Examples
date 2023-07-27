# OpenLane を誰でも簡単に!
このレポジトリは OpenLane の簡単なサンプルを誰でも簡単に
最後まで実行可能なようにする情報の蓄積を目的にしています。

現時点(2023年7月25日)では情報だけですが、将来的に独自のサンプルも提供します。

# Quick Start. とにかく実行したい!!
本情報は[ここ](https://note.com/akira_tsuchiya/n/nf770aa77b785)
の iic-osic-tools の情報をもとにしています。手順は先のページのほうが
絵付きで詳しく書いてあります。

また、dockerが無い/苦手のばあいは https://anaconda.org/litex-hub/openlane がおすすめです。

0. ターゲットは Linux/Mac/[Windows(WSL)](/windows.md)
1. Docker の準備
2. iic-osic-tools の 2023.06 を clone
```
$ git clone https://github.com/iic-jku/iic-osic-tools -b 2023.06 iic-osic-tools-2022.06
```
3. インストールおよび実行
```
$ cd iic-osic-tools-2023.06
$ DOCKER_TAG=2023.06 ./start_shell.sh
```
これで Docker のシェルが立ち上がります。GUI が必要なら ./start_x.sh を実行。
docker が `Got permission denied while trying to connect to the Docker daemon socke` と permission denied になる場合は /etc/group の docker に自分の名前を追加しましょう。[参考:ユーザをdockerグループに入れる](https://qiita.com/tifa2chan/items/9dc28a56efcfb50c7fbe)

4. ryos36 の caravel_user_project を clone して 42d1f1fb を chekout
efabless の caravel_user_project ではエラーになるので注意。

```
/foss/designs > git clone https://github.com/ryos36/caravel_user_project.git 
/foss/designs > git checkout 42d1f1fb
```
5. フローを実行
```
/foss/designs > export PDK=sky130B
/foss/designs > cd caravel_user_project
/foss/designs/caravel_user_project > cd openlane
/foss/designs/caravel_user_project/openlane > flow.tcl -design user_proj_example -tag my-test -overwrite
```
これで klayout で表示可能な GDSII 形式のレイアウトが出来るはず。

6. klayout で確認
Docker の外で signoff にある gds を klayout という GDSII を可視化するツールで見る。うまく Docker が起動していると ~/eda/designs に Docker の Image がマウントされているらしい。mount してない場合は docker cp でコピーする。

```
klayout ~/eda/designs/caravel_user_project/openlane/user_proj_example/runs/my-test/results/signoff/user_proj_example.klayout.gds
```
<img src="./images/klayout-counter.png" width="320px">


なお、caravel とつなげるためには user_proj_wrapper を作らないといけないみたいだが、それにはさらなる困難が待ち受けているらしい。

質問等は [ISHI 会](https://ishi-kai.org/) の discord へ。

以前の情報
- [iic-osic-tools の 2022.12 版](./iic-osic-tools-2022.12.md)

## iic-osic-tools の 2023.06 動かん
- efabless の caravel_user_project をコンパイルできない。最初の verilator の lint チェックで エラーになる。[ryos36](https://github.com/ryos36/caravel_user_project) 版を使えば解決する。
- /foss/examples/SKY130_SAR-ADC1 をコンパイルできない。STEP 45 までいってエラー。LVS で落ちちゃう。

# OpenLane って何？
![ChatGPT が語る OpenLane](./images/ChatGPTOpenLane.png)

[OpenROAD 版の OpenLane](https://github.com/The-OpenROAD-Project/OpenLane) が
本家のようだ。なぜか [efabless 版の OpenLane](https://github.com/efabless/OpenLane) がある(更新は止まっている)。

## OpenROAD
OpenROAD という[団体名](https://theopenroadproject.org/)でもあり、
そこが提供する[ツール](https://github.com/The-OpenROAD-Project/OpenROAD)でもある。更に、[フローのためのツール群: OpenROAD Flow](https://github.com/The-OpenROAD-Project/OpenROAD-flow-scripts)でもあるようだ。文脈によって使い分けられている!!

OpenLane は OpenROAD Flow のフローとは別のフローを提供するという試みのようで
OpenLane の中にツールとしての OpenROAD が含まれているようだ。

## OpenLane2 !!
ツールの寄せ集めじゃなくて、Nix ベースで、モジュールの形で
提供する次期バージョンのようだね。

https://github.com/efabless/openlane2

# その他ツールなど
## [caravel_user_project](https://github.com/efabless/caravel_user_project) って何？
efabless が提供する [Caravel](https://caravel-harness.readthedocs.io/en/latest/index.html) という RISC-V を含むデバッグシステムと
つなぐことが出来る OpenLane 用のプロジェクトの雛形。
どうやら Caravel の RISC-V は PicoRV32 で、I/F として
user_proj_wrapper という口が
(昔のXilinx 風に言うなら)パーシャルとして用意されている。

[Caravel 自身](https://github.com/efabless/caravel) は github で
提供されているものの RISC-V の部分は Verilog-HDL として見当たらなかった。
[caravel_pico](https://github.com/efabless/caravel_pico) なるものが
あるので、更に入れ子になっているのかも(未確認)。

### なんかおかしい
caravel_user_project の verilog/rtl/user_proj_example.v 。
verilator の lint でエラーになります。その上ビット幅にどうやら間違いがあるらしく、その後のビット幅チェックでも引っかかります。とりあえず修正してみました。
修正版は[こちら](https://github.com/ryos36/caravel_user_project)。
この修正で iic-osic-tools の 2023.06 (OpenLane 的には 2023.05) と一緒に動きました。本家にプルリクしてみた。

結局、OpenLane のバージョンに合わせた PDK と caravel_user_project が必要で
どの組み合わせで動くのかは自分で試すしかない!!
(のでこのドキュメントがあるわけです）。

## [caravel_user_project_analog](https://github.com/efabless/caravel_user_project_analog)
アナログ版の雛形かな？だれか試して!!

- https://github.com/ranan-usp/caravel_user_project_analog
同様の手順で iic-osic-tools の 2023.06( OpenLane の 2023.05 ) で
最後までフローが流れることを確認しました(2023/7/26)。結果の gds が
正しいかどうかは未確認。


## [iic-osic-tools](https://github.com/iic-jku/iic-osic-tools) とは？
Institute for Integrated Circuits (IIC) at the Johannes Kepler University Linz (JKU) が提供する OpenLane を含む色々ツール群をインストールした Docker コンテナ。
もとは efabless の [foss-asic-tools](https://github.com/efabless/foss-asic-tools) の fork。fork もとが tools の名称をつかっていたため、そのような名称になった模様。

## [foss-asic-tools](https://github.com/efabless/foss-asic-tools)
efabless が提供していた(過去形!!) OpenLane を含むいろいろツール群をインストールした Docker コンテナ。すでに更新は止まっている模様。

## efabless が提供する Docker コンテナ
https://hub.docker.com/u/efabless

efabless は [efabless/openlane](https://hub.docker.com/r/efabless/openlane) と
[efabless/openlane-tools](https://hub.docker.com/r/efabless/openlane-tools)
を現在 Docker コンテナとしてメンテナンスしている模様。
前者は OpenLane オンリー。後者は色々ツール入り。
おそらく OpenLane のレポジトリの Dockerfile が元ソース(未確認)。

また、OpenLane の環境から make の延長で Docker が起動するように設計されている模様。Docker 単体で使おうと思ったらどうつかってよいかわからなかった。

# 情報源
- [IIC-OSICによるオープンソース設計環境のセットアップ](https://note.com/akira_tsuchiya/n/nf770aa77b785)
- [OpenMPW入門](https://vlsi.jp/OpenMPW.html)
