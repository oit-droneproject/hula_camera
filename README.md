# Hulaカメラ

## 前方カメラ

#### single_fly_Anticipatory_recognition(qr_id)

前方カメラでARマーカーを認識	

AR(QR)_id: QR ID [0-9]	{ result, x, y, z, yaw, qr_id }。x/y/zはドローンとQR間の距離、yawは角度、qr_idは認識したAR(QR)のID

#### AR_test.py
```python
import pyhula
import time

api = pyhula.UserApi()
if not api.connect():
    print("connect error")
else:
    print("connection to station by wifi")

print(f"battery={api.get_battery()}")

print("映像取得開始...")

while True:
    arry=api.single_fly_Anticipatory_recognition(0)
    print(arry)
```
#### 実施例
```bash
connect wifi
192.168.100.255 192.168.100.125
connection to station by wifi
battery=59
{'mode': 0, 'type': 2, 'x': 28, 'y': 82, 'z': 10, 'angle': 1925, 'result': True}
{'mode': 0, 'type': 2, 'x': 28, 'y': 82, 'z': 10, 'angle': 1925, 'result': True}
{'mode': 0, 'type': 2, 'x': 28, 'y': 82, 'z': 10, 'angle': 1925, 'result': True}
```
