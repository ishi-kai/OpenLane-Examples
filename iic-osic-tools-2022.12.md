# iic-osic-tools の 2022.12 版
この手順で今でも動きますが、OpenLane が古いので、より最新で
実績のあるものをつかったほうがいいでしょう。

0. ターゲットは Linux/Mac/[Windows(WSL)](/windows.md)
1. Docker の準備
2. iic-osic-tools の 2022.12 を clone
```
$ git clone https://github.com/iic-jku/iic-osic-tools -b 2022.12 iic-osic-tools-2022.12
```
3. インストールおよび実行
```
$ cd iic-osic-tools-2022.12
$ sudo DOCKER_TAG=2022.12 ./start_shell.sh
```
これで Docker のシェルが立ち上がります。GUI が必要なら sudo ./start_x.sh を実行。
4. caravel_user_project の mpw-9a を clone
```
/foss/designs > git clone https://github.com/efabless/caravel_user_project.git -b mpw-9a
```
5. フローを実行
```
/foss/designs > export PDK=sky130B
/foss/designs > cd caravel_user_project
/foss/designs/caravel_user_project > cd openlane
/foss/designs/caravel_user_project/openlane > flow.tcl -design user_proj_example -tag my-test -overwrite
```
これで klayout で表示可能な GDSII 形式のレイアウトが出来るはず。
