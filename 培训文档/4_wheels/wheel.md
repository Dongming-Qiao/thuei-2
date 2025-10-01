# 匹配輪子和麥輪運動控制

## 開始讓車車跑起來吧！！！

### 這次的培訓主要是讓大家來實現thuei-1/MecanumControl/Car_Forward_Demo.py這個文件來確認輪子的編號

- 先把文件中第26行的
```
chassis = mecanum.MecanumChassis(wheel_init_dir=[1, 1, 1, 1], wheel_init_map=[2, 4, 1, 3])
```
改成
```
chassis = mecanum.MecanumChassis(wheel_init_dir=[1, 1, 1, 1], wheel_init_map=[1, 2, 3, 4])
```
- 馬達轉向配置
```
wheel_init_dir=[1,1,1,1]
```
1是正轉（順時針）-1是反轉（逆時針）

- 輪子編號對應馬達編號
```
wheel_init_map=[1, 2, 3, 4]
```
1-左前輪、2-右前輪、3-左後輪、4-右後輪

![修改文檔](輪子編號.jpg)

- 注意要確認輪子的接綫順序和方向是不是正確




## I2C
I2C(Inter-Integrated Circuit)序列通訊匯流排(一種串列通訊協定)

簡單來説：只用兩條綫就能讓多個IC晶片彼此通訊👇

那爲什麽我們要用i2c？

- 如果要接很多感測器或者馬達驅動器，傳統的方法每一個都要接很多綫會很複雜
- 而i2c只需要2根綫就好：串列資料綫（SDA）和串列時鐘綫（SCL）
- 分爲主節點Master和從節點Slave
- Master(香橙派)負責去發問、傳資料、決定時鐘
- Slave(麥輪的馬達驅動器)等著被叫地址，然後回答

簡單的流程
- Master發出start的訊號，表示「我要講話了」
- Master呼叫其中一個Slave的地址
- 被呼叫到的Slave回應
- 開始傳送資料
- 傳送完畢，Master發出Stop的訊號，通訊結束

想多了解i2c的原理可以參考:https://zh.wikipedia.org/wiki/I%C2%B2C


## 如果運行上面的Car_Forward_Demo.py出現問題的話 請查看I2C

### 如何查看I2C編號
```
i2cdetect -l
```
然後就會列出系統裏面所有的I2C bus

### 如何查看I2C地址
```
sudo i2cdetect -y -a -r 7
```
命令掃描I2C總綫上的所有設備的地址
- -y表示不提示確認
- -a表示顯示所有地址（包括保留地址）
- -r表示使用快速模式
- 樹莓派默認編號1 香橙派默認編號7

![i2c地址](i2c地址.jpg)

### 請記得修改thuei-1/HiwonderSDK/Sonar.py裏面的第34行
```
self.i2c=7
```
![Sonar](修改sonar.jpg)


## with 寫法（語法糖）
```
try...finally的用法：
f = open("data.txt", "r")
try:
    content = f.read()
finally:
    f.close()  
```
確保不管發生什麼，最後一定關閉

## 使用signal模塊捕獲SIGINT信號（MotorControlDemo.py）
- SIGINT(Signal Interrupt)
在Linux或Unix系統中，按下Ctrl+C的時候，系統會給正在運行的程式發送一個SIGINT，預設行爲是立刻退出
- 如果直接Ctrl+C終端程式的話，馬達可能會卡在保持輸出的狀態，也就是説車子可能會亂跑
- 所以要攔截SIGINT，在程式退出前釋放資源（停止馬達轉動、關閉GPIO等等）
- MotorControlDemo.py中的這一部分
```
start = True

def Stop(signum, frame):
    global start
    start = False
    print('关闭中...')
    MotorStop()  # 停止所有電機

signal.signal(signal.SIGINT, Stop)
```
就是用signal.signal捕獲SIGINT來關閉所有馬達（電機）【安全地退出】
- MotorControlDemo.py用來測試馬達可不可以正常運轉、速度控制有沒有反應



## 常見問題
- 如果遇到 [Errno 5] Input/output error 請檢查I2C連接和電池電量
- 如果輪子轉的方向不對 請修改wheel_init_dir
- 如果小車不按照計劃的方向走 請檢查馬達編號wheel_init_map



## 使用SMBus類創建I2C總綫對象，__i2c參數指定總綫編號
```python
from smbus import SMBus
    with SMBus(__i2c) as bus
```
I2C編號=Linux裏面注冊的I2C控制器的順序
