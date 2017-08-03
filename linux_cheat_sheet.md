---
title: Linux Cheat Sheet
---

# Linux共通チートシート
## シリアル通信

ターミナルでシリアル通信を監視するためには、cuを用いるのが便利である。インストールは``sudo apt-get install cu``でできる。

シリアル通信を監視するためには少なくともデバイス名とボーレート（通信速度、シリアル通信したいデバイス間で事前に共通の値を決めておく必要がある）の情報が必要である。現在コンピュータに接続されているデバイス一覧は``ls /dev/tty*``で確認できる。
例えばUSBデバイスなら``/dev/ttyUSB0``のように表示される。

接続先のデバイス名とボーレートがわかったら、以下のようにしてcuを実行する。ここでは、接続デバイスは/dev/ttyUSB0、ボーレートは57600と仮定する。
```bash
$ cu -l /dev/ttyUSB0 -s 57600
```

cuを使うとき、Permission deniedと表示されることがある。このときは以下のコマンドでデバイスのパーミッションを変更する。
```
$ sudo chmod o+wr /dev/ttyUSB0
```

シリアル通信の終了は``ctrl + c``で **できない** ので注意が必要である。

終了するためには``Enter``を押したのち``~.``と入力する。

参考: http://answers.ros.org/question/46790/failed-to-open-port-devttyusb0/

``dmesg|grep usb``でどのデバイスがttyUSB*に登録されているか確認