# WIFI密码破解

## 实验要求

利用有监听功能的wifi网卡和aircraft-ng完整完成一次对wifi AP的密码破解，详细记录过程，完成实验报告。

## 实验准备
 + virtualBox虚拟环境
 + 可监听网卡（Realtek Semiconductor Corp. RTL8187）
 + aircraft-ng系列工具包
## 实验步骤

### **1. 准备**

 + 对网卡的准备工作

    请务必在虚拟机里安装好网卡对应的驱动。部分网卡型号可能不被支持，请按照[参考网卡类型](https://c4pr1c3.github.io/cuc-mis/chap0x02/wifi_card_list.html)购买和使用。

    kali虚拟机请使用如下命令：
    ```bash
    sudo install networkname
    ```
 
    linux虚拟机请使用如下命令：
    ```bash
    opkg install networkname
    ```

    *注：特殊的网卡型号可能有不同的安装形式，在不同系统里的命令也不同。需要根据实际情况自行调整*

 + 对aircraft-ng工具包的准备

    **kali环境下往往自带aircraft-ng工具包，通过命令可直接调用，因此十分建议在kali虚拟机内完成此次实验。**

    *注：linux虚拟机只支持部分aircaft-ng包里的部分功能，比如airodump，但不能直接使用该工具包的全部功能。Debian等其他型号虚拟机可能未与预安装工具包，需要自行下载。*

 + 开启网卡监听
   
    使用如下命令找到新插入的可监听网卡
    ```bash
    (sudo) ifconfig
    ```
    
    通常情况下网卡名为wlan0，且无ip地址的分配。表明该网卡尚未连接至任何wifi，如下图所示：
    
    ![查看网卡初始情况](img/%E6%9F%A5%E7%9C%8B%E7%BD%91%E5%8D%A1.jpg)
    
    启动网卡驱动，并将网卡开启至监听模式
    ```bash
    sudo ifconfig wlan0 up
    sudo airmon-ng start wlan0
    sudo ifconfig wlan0 down      ###为了重启网卡。实际上通过物理方法直接插拔也行
    sudo iwconfig wlan0 mode monitor
    sudo ifconfig wlan0 up 
    ```
    
    将网卡监听模式正常启动后，系统会告知
    > monitor mode enable

    如图所示：![监听模式正常启动](img/%E7%9B%91%E5%90%AC%E6%A8%A1%E5%BC%8F%E6%AD%A3%E5%B8%B8%E5%90%AF%E5%8A%A8.jpg)

**2. 探测**

 + 用下列指令对周边wifi的信息进行探测
    ```bash
    sudo airodump-ng wlan0mon
    ```
    ![探测结果](img/%E7%BD%91%E5%8D%A1%E5%97%85%E6%8E%A2%E7%BB%93%E6%9E%9C.jpg)

    探测结果是随时变化的，一旦开启airodump的功能，就会持续进行探测。探测结果分两个表单显示，其中BSSID是mac地址，CH是频道，ESSID是wifi名，ENC是加密方式。

    这张图里的“wifi-virdy”是我自己搭建的局域网，“i-hefei”是合肥市政府的广域网，还有两个网络是邻居家的。就不乱搞了，玩玩自家WiFi就算了。

**3. 抓包**
    
   使用如下命令，对目标WiFi进行抓包，抓包的结果为可用于破解的IVS数据报文。

   ```bash
   sudo airodump-ng --ivs --bssid BSSIC –w longas -c CH wlan0
   ```
   这些报文会以longas-n.ivs的文件形式出现，其中n为抓包的次数。例如，第二次抓包的结果会存储在名为longas-02.ivs的文件中

**4. 攻击**

   为了获得破解所需的WPA2握手验证的整个完整数据包，我们将会发送一种称之为“Deauth”的数据包来将已经连接至无线路由器的合法无线客户端强制断开，此时，客户端就会自动重新连接无线路由器。

   此处需要新开一个shell：
   ```bash
   sudo aireplay-ng -0 n –a BSSIC -c STATION wlan0
   ```
   
   -0表示采用deauth攻击模式，n为攻击次数。BSSIC和STATION分别是路由器的mac地址和客户端的mac地址

   当抓包界面出现握手包（WPA handshake）时，我们就得到了包含WPA2握手验证的完整数据包了
   
   ![捕获握手包](img/%E6%8D%95%E8%8E%B7%E6%8F%A1%E6%89%8B%E5%8C%85.jpg)

**5. 破解**

 + 生成可能的密码字典
   
   利用kali自带的crunch工具生成字典，命令如下：
   
   ```bash
   crunch <min-len> <max-len> [<charset string>] [options]
   ```
   ![生成密码字典](img/%E7%94%9F%E6%88%90%E5%AF%86%E7%A0%81%E5%AD%97%E5%85%B8.jpg)
   
   用crunch生成密码不是必需的，因为毕竟自己还是知道密码的，不必穷举，过大的运算量会增加电脑负担，我的电脑性能比较差，别说破解了，生成字典的时候都会卡。建议电脑性能一般的朋友量力而行 : )

 + 对抓包的文件进行破解

   ```bash
   sudo aircrack-ng -w dict1.txt longas-02.ivs
   ```

 + 成功破解密码后的界面如图所示：
   
   ![key found](img/key%20found.jpg)
   key found后的[ ]里即为密码。标志着WiFi密码破解完成

## 参考文献

1. [移动互联网安全在线课本](https://c4pr1c3.github.io/cuc-mis/chap0x03/exp.html)
2. [Hacking Tutorials](https://www.hackingtutorials.org/wifi-hacking-tutorials/pixie-dust-attack-wps-in-kali-linux-with-reaver/)
3. [hack.lu](http://archive.hack.lu/2014/Hacklu2014_offline_bruteforce_attack_on_wps.pdf)