# Hulaカメラ

Hulaドローンのカメラ機能（映像ストリームの取得・ARマーカー認識・色認識）を、`pyhula` から利用するためのサンプルです。

---

## 動画

前方（ジンバル）カメラの映像ストリーム（RTP）を有効にし、カメラ映像を表示します。

映像ストリームを有効にしたあと、映像表示用のメソッドを実行することで、カメラ映像を確認できます。

| メソッド                     | 引数           | 説明                                         |
| ------------------------ | ------------ | ------------------------------------------ |
| `Plane_cmd_swith_rtp(x)` | `x = {0, 1}` | 映像ストリームの送信を切り替える。`0`: 有効、`1`: 無効           |
| `single_fly_flip_rtp()`  | なし           | 映像ストリームのウィンドウを開く。SPS/PPS（デコード情報）を受信するために必要 |

### hula_camera01.py

```python
import pyhula
import time

api = pyhula.UserApi()
if not api.connect():
    print("connect error")
else:
    print("connection to station by wifi")

print(f"battery={api.get_battery()}")

# ストリームを有効化
api.Plane_cmd_swith_rtp(0)
api.single_fly_flip_rtp()

# SPS/PPSが届くまで待機
time.sleep(3)

while True:
    time.sleep(0.1)
```

`Ctrl+C` を押すと、プログラムを停止できます。

---

## カメラの向き

ジンバルカメラの向きを変更することもできます。

| メソッド                                 | 引数                            | 説明                                                                           |
| ------------------------------------ | ----------------------------- | ---------------------------------------------------------------------------- |
| `Plane_cmd_camera_angle(type, data)` | `type`（下記参照）、`data = 0〜90`（度） | ジンバルカメラのピッチ角を調整する。`type = 0`: 上向き絶対角度、`1`: 下向き絶対角度、`5`: 上向き相対角度、`6`: 下向き相対角度 |

### hula_camera02.py

```python
import pyhula
import time

api = pyhula.UserApi()
if not api.connect():
    print("connect error")
else:
    print("connection to station by wifi")

print(f"battery={api.get_battery()}")

# ストリームを有効化
api.Plane_cmd_swith_rtp(0)
api.single_fly_flip_rtp()

# SPS/PPSが届くまで待機
time.sleep(3)

print("映像取得開始...")
api.Plane_cmd_camera_angle(1, 90)

while True:
    time.sleep(0.1)
```

`Ctrl+C` を押すと、プログラムを停止できます。

---

## ARマーカー認識

前方カメラを使用して、ARマーカー（QRコード）を認識します。

| メソッド                                         | 引数                   | 説明                                                                                                                   |
| -------------------------------------------- | -------------------- | -------------------------------------------------------------------------------------------------------------------- |
| `single_fly_Anticipatory_recognition(qr_id)` | `qr_id = 0〜9`（QR ID） | 指定したARマーカーを認識する。戻り値は辞書 `{result, x, y, z, angle, ...}`。`x`、`y`、`z` はドローンとマーカーの位置関係、`angle` は角度（ヨー）、`result` は認識成否を表す |

数字・矢印・アルファベットのタグを認識したい場合は、`single_fly_AiIdentifies(mode)` も利用できます。

`mode` には、以下の値を指定します。

* `0〜9`: 数字
* `10〜13`: 矢印
* `65〜90`: A〜Z

### hula_camera03.py

```python
import pyhula
import time

api = pyhula.UserApi()
if not api.connect():
    print("connect error")
else:
    print("connection to station by wifi")

print(f"battery={api.get_battery()}")

while True:
    array = api.single_fly_Anticipatory_recognition(0)
    print(array)
```

### 実行例

```bash
connect wifi
192.168.100.255 192.168.100.125
connection to station by wifi
battery=59
{'mode': 0, 'type': 2, 'x': 28, 'y': 82, 'z': 10, 'angle': 1925, 'result': True}
{'mode': 0, 'type': 2, 'x': 28, 'y': 82, 'z': 10, 'angle': 1925, 'result': True}
{'mode': 0, 'type': 2, 'x': 28, 'y': 82, 'z': 10, 'angle': 1925, 'result': True}
...
```

`Ctrl+C` を押すと、プログラムを停止できます。

### ドローンとマーカーの位置関係

下図で使用しているタイルは、1辺が30 cmです。

<img src="./image/camera2.jpg" alt="ドローンとARマーカーの位置関係" width="600">

---

## 映像表示とマーカー認識

カメラ映像を表示しながら、ARマーカーを認識することもできます。

### hula_camera04.py

```python
import pyhula
import time

api = pyhula.UserApi()
if not api.connect():
    print("connect error")
else:
    print("connection to station by wifi")

print(f"battery={api.get_battery()}")

# ストリームを有効化（この順番が重要）
api.Plane_cmd_swith_rtp(0)
api.single_fly_flip_rtp()

# SPS/PPSが届くまで待機
time.sleep(3)

print("映像取得開始...")

while True:
    array = api.single_fly_Anticipatory_recognition(0)
    print(array)
```

### 実行例

```bash
connect wifi
192.168.100.255 192.168.100.125
connection to station by wifi
battery=59
映像取得開始...
{'mode': 0, 'type': 2, 'x': 28, 'y': 82, 'z': 10, 'angle': 1925, 'result': True}
{'mode': 0, 'type': 2, 'x': 28, 'y': 82, 'z': 10, 'angle': 1925, 'result': True}
{'mode': 0, 'type': 2, 'x': 28, 'y': 82, 'z': 10, 'angle': 1925, 'result': True}
```

### カメラ映像

<img src="./image/camera1.png" alt="Hulaドローンのカメラ映像" width="600">

---

## 色認識

映像ストリームの現在のフレームから、代表色のRGB値を取得します。

| メソッド                    | 引数 | 説明                                                                                                     |
| ----------------------- | -- | ------------------------------------------------------------------------------------------------------ |
| `single_fly_getColor()` | なし | 現在の映像フレームから色を取得する。戻り値は辞書 `{r, g, b, state}`。`r`、`g`、`b` はRGB値、`state` は取得成否（`True`: 成功、`False`: 失敗）を表す |

### hula_camera05.py

```python
import pyhula
import time

api = pyhula.UserApi()
if not api.connect():
    print("connect error")
else:
    print("connection to station by wifi")

print(f"battery={api.get_battery()}")

# 以下のコメントアウトを外すと、映像を表示することも可能
# api.Plane_cmd_swith_rtp(0)
# api.single_fly_flip_rtp()

time.sleep(3)

while True:
    array = api.single_fly_getColor()
    print(array)
```

### 実行例

以下は、カメラの前に赤色の紙を置いた場合の実行例です。

取得されるRGB値は、用紙の色や照明などの周囲環境によって大きく変化します。

```bash
connect wifi
192.168.100.255 192.168.100.125
connection to station by wifi
battery=42
{'r': 128, 'g': 144, 'b': 176, 'state': True}
{'r': 128, 'g': 144, 'b': 176, 'state': True}
{'r': 128, 'g': 144, 'b': 176, 'state': True}
```

---

## 旋回動作を用いたマーカー探索

ドローンを10度ずつ旋回させながら、ID:0 のARマーカーを探索します。

マーカーを発見すると旋回を終了し、着陸します。

### hula_camera06.py

```python
import pyhula
import time

api = pyhula.UserApi()
if not api.connect():
    print("connect error")
else:
    print("connection to station by wifi")

print(f"battery={api.get_battery()}")

# ストリームを有効化（この順番が重要）
api.Plane_cmd_swith_rtp(0)
api.single_fly_flip_rtp()

# SPS/PPSが届くまで待機
time.sleep(3)

api.single_fly_takeoff()

print("映像取得開始...")

while True:
    api.single_fly_turnleft(10)

    array = api.single_fly_Anticipatory_recognition(0)
    print(array["result"])

    if array["result"]:
        print("マーカーを発見しました")
        break

api.single_fly_touchdown()
```

### 実行例

```bash
connect wifi
192.168.100.255 192.168.100.125
connection to station by wifi
battery=48
takeoff success
映像取得開始...
False
SFTurnLeftTP finish
False
SFTurnLeftTP finish
False
SFTurnLeftTP finish
False
SFTurnLeftTP finish
True
マーカーを発見しました
Touchdown not finish
```

ID:0 のマーカーがカメラ映像に映っていても、認識されるまで数秒かかる場合があります。

---

## 課題

ID:5 のARマーカーに向けてドローンを飛行させ、ドローンとマーカーの距離が1 m以内になったところで着陸させてください。
