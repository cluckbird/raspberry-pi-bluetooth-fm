# Bluetooth FM
使用树莓派发送FM广播（无线电），透过手机的蓝牙音频

## 准备
> 在使用前请先确保自己遵守自己所在地区的法律，发射国家允许的业余无线电频段，且请勿使用放大器

> 中国大陆和香港的FM频段是 87MHZ~108MHZ，其他地区可以自行 Google

- 一个树莓派（推荐3B+），运行 Raspberry Pi OS
- 一个能用的天线（不用也行，就是信号会很差）
- 如有金属外壳请先拆掉

## 开始

> 所有操作均需要 root 权限，如果你有懒癌可以直接切到 root 用户

### 安装组件

```bash
sudo apt update
sudo apt install git pulseaudio sox  libsox-fmt-all pulseaudio-module-bluetooth
```

### Clone Rpitx

> 如果你居住在中国大陆，请保证和 github 的网络连接

```bash
git clone https://github.com/F5OEO/rpitx
cd rpitx
sudo ./install.sh # 这里会询问你是否允许修改 CPU 配置，请同意
sudo reboot # 重新启动
```

### 启动 pulseaudio 服务

```bash
sudo sh -c "echo 'extra-arguments = --exit-idle-time=-1 --log-target=syslog' >> /etc/pulse/client.conf"
sudo pulseaudio --start &
```

### 透过蓝牙连接到手机

#### 修改蓝牙装置类型为车载音响

```bash
sudo vim /etc/bluetooth/main.conf

# 把 Class 的值修改为 0x200420
...
Class 0x200420
...

sudo reboot # 重启
```

#### 连接到手机
```bash
sudo hciconfig hci0 up
sudo bluetoothcli # 进入蓝牙CLI

discoverable on # 开启可被扫描
```
在手机蓝牙页面寻找装置：raspberrypi，配对并连接，共享音频（IOS和Windows装置可能无法使用）
```bash
exit # 退出蓝牙CLI
```

#### 寻找蓝牙声音输入设备
```bash
pacmd list | grep "a2dp_source"
```
可以找到一个叫 `bluez_source.XX_XX_XX_XX_XX_XX.a2dp_source` 的装置，请复制他的名字

![img](https://ci.cncn3.cn/b7cbe4f5adc5d111c40e2ff65be7cac2.png)

### 启动FM电台
> 请保持蓝牙音频内有输出，不然会断掉

- 蓝牙输入设备名：见上一条
- 发射频率：纯数字，符合你所在地区法律的无线电频段即可，单位 MHZ（我使用 87.2 mhz）

```bash
pacat --record -d 蓝牙输入装置名 | sox -t raw -b 16 -c 2 -v 1 -r 44100 -L -e signed-integer - -t wav - | sudo ./pifmrds -freq 发射频率 -audio -
```

## 完成

![img](https://ci.cncn3.cn/280da3794210ffcf59c1f88b6fd6b9d7.png)

只要你能看到这样的 output，且收音机可以收到正确的音频，那就成功了

### 遇到问题？

我也不确定我有没有漏掉步骤，欢迎提醒我或问我问题

TG: [@mumubird](https://t.me/mumubird)

Mail: [woodpackeraisser@gmail.com](mailto:woodpackeraisser@gmail.com)

### 有什么用吗

- 把收音机改造成蓝牙耳机，在短距离内的音质还是可以的
- 和邻居一起听歌
- ...

好吧我也觉得没有用，但是确实很好玩，且很有成就感，对于我这样的对无线电并不了解的后端开发（
