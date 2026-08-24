# Hulaカメラ

## 前方カメラ

#### single_fly_Anticipatory_recognition(qr_id)

前方カメラでQRコードを認識	

qr_id: QR ID [0-9]	{ result, x, y, z, yaw, qr_id }。x/y/zはドローンとQR間の距離、yawは角度、qr_idは認識したQRのID

#### camera_test.py
```python
import pyhula
import time
import cv2

api = pyhula.UserApi()
if not api.connect():
    print("connect error")
else:
    print("connection to station by wifi")

print(f"battery={api.get_battery()}")

# ストリーム有効化（この順番が重要）
api.Plane_cmd_swith_rtp(0)
api.single_fly_flip_rtp()  # これが必要


time.sleep(6)  # SPS/PPSが届くまで待つ

print("映像取得開始...")

while True:
    arry=api.single_fly_Anticipatory_recognition(0)
    print(arry)
```
