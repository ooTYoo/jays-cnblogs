----
　　大家好，我是痞子衡，是正经搞技术的痞子。今天痞子衡给大家介绍的是**常见短距离无线通信协议**。  

　　短距离无线通信是物联网的基础，随着物联网IoT的火热发展，各种短距离无线通信协议也是层出不穷，这些协议标准各有优缺以及应用场合。各种协议并存的现象将长期存在，没有人能够解决无线短距离互联标准不统一的问题，因为行业发展太快而标准跟不上，短期内还看不到一统江湖的短距离无线标准。今天痞子衡就用一张表为大家搜罗常见的短距离无线标准协议。  

### 短距离无线通信协议列表

<table><tbody>
    <tr>
        <th>协议名</th>
        <th>版本</th>
        <th>推出时间</th>
        <th>无线标准</th>
        <th>电磁波频段(Hz)</th>
        <th>速率(bps)</th>
        <th>距离(m)</th>
        <th>行业应用</th>
        <th>官方网站</th>
    </tr>
    <tr>
        <td rowspan="3">IrDA</td>
        <td>1.0(SIR)</td>
        <td>1994</td>
        <td rowspan="3">/</td>
        <td rowspan="3">波长介于760nm~1mm的红外线</td>
        <td>115200</td>
        <td rowspan="3">0.1 - 2</td>
        <td rowspan="3">家电遥控器</td>
		<td rowspan="3"><a href="http://www.irda.org/">http://www.irda.org/</a></td>
    </tr>
    <tr>
        <td>1.1(FIR)</td>
        <td>1996</td>
        <td>4M</td>
    </tr>
    <tr>
        <td>VFIR</td>
        <td>/</td>
        <td>16M</td>
    </tr>
    <tr>
        <td>HomeRF</td>
        <td>/</td>
        <td>1997</td>
        <td>N/A，结合DECT和WLAN技术</td>
        <td>2.4G</td>
        <td>2FSK - 1M<br>
		    4FSK - 2M</td>
        <td>100</td>
        <td>物联网、智能家居</td>
		<td></td>
    </tr>
    <tr>
        <td>RFID - Mifare</td>
        <td>/</td>
        <td>1998</td>
        <td>ISO/IEC 14443A</td>
        <td>13.56M<br>
		    波长介于1mm~1m的微波（下同）</td>
        <td>106K</td>
        <td>0.1</td>
        <td>门禁系统、公交卡</td>
		<td><a href="https://www.mifare.net/">https://www.mifare.net/</a></td>
    </tr>
    <tr>
        <td rowspan="2">Wi-Fi</td>
        <td>/</td>
        <td>1999.09</td>
        <td>IEEE 802.11b</td>
        <td rowspan="2">2.4G</td>
        <td>11M</td>
        <td rowspan="2">100</td>
        <td rowspan="2">手持设备、公众场合联网</td>
		<td rowspan="2"><a href="https://www.wi-fi.org/">https://www.wi-fi.org/</a></td>
    </tr>
    <tr>
        <td>/</td>
        <td>2001.11</td>
        <td>IEEE 802.11g</td>
        <td>54M</td>
    </tr>
    <tr>
        <td>Z-wave</td>
        <td>/</td>
        <td>2001</td>
        <td>/</td>
        <td>868M/915M</td>
        <td>9.6K/40K</td>
        <td>30</td>
        <td>照明商业控制</td>
		<td><a href="http://www.z-wave.com/">http://www.z-wave.com/</a></td>
    </tr>
    <tr>
        <td>UWB</td>
        <td>/</td>
        <td>2002.02</td>
        <td>IEEE802.15.3a（MBOA/DS-UWB）</td>
        <td>3.1 - 10.6G<br>
		    纳秒至微秒级的非正弦波窄脉冲</td>
        <td>110M</td>
        <td>10</td>
        <td>精确测距室内定位</td>
		<td><a href="http://www.wimedia.org/">http://www.wimedia.org/</a></td>
    </tr>
    <tr>
        <td rowspan="2">Bluetooth Classic</td>
        <td>1.1</td>
        <td>2002.06</td>
        <td rowspan="4">IEEE 802.15.1</td>
        <td rowspan="4">2.4G</td>
        <td>720K</td>
        <td rowspan="4">Class4: 300<br>
		                Class3: 100<br>
		                Class2: 10<br>
						Class1: 1</td>
        <td rowspan="4">可穿戴设备、物联网</td>
		<td rowspan="4"><a href="https://www.bluetooth.com/">https://www.bluetooth.com/</a></td>
    </tr>
    <tr>
        <td>5.0</td>
        <td>2016.12</td>
        <td>50M</td>
    </tr>
    <tr>
        <td rowspan="2">BLE</td>
        <td>4.0</td>
        <td>2010.06</td>
        <td>1M</td>
    </tr>
    <tr>
        <td>5.0</td>
        <td>2016.12</td>
        <td>2M</td>
    </tr>
    <tr>
        <td>NFC</td>
        <td>/</td>
        <td>2003.03</td>
        <td>ISO/IEC 18092<br>
		    ECMA 340<br>
			ETSI TS 102 190</td>
        <td>13.56M</td>
        <td>106K<br>
		    212K<br>
			424K</td>
        <td>0.1</td>
        <td>身份识别、移动支付</td>
		<td><a href="https://nfc-forum.org/">https://nfc-forum.org/</a></td>
    </tr>
    <tr>
        <td rowspan="2">ZigBee</td>
        <td>1.0</td>
        <td>2004.12</td>
        <td rowspan="2">IEEE 802.15.4</td>
        <td rowspan="2">2.4G<br>
		                915M<br>
		                868M</td>
        <td rowspan="2">250K<br>
		                40K<br>
						20K</td>
        <td rowspan="2">100</td>
        <td rowspan="2">工业自动化、智能楼宇、智能家居</td>
		<td rowspan="2"><a href="http://www.zigbee.org/">http://www.zigbee.org/</a></td>
    </tr>
    <tr>
        <td>3.0</td>
        <td>2016.05</td>
    </tr>
    <tr>
        <td>LiFi</td>
        <td>1.0</td>
        <td>2011</td>
        <td>IEEE 802.15.7 VLC</td>
        <td>波长介于380nm～780nm的可见光</td>
        <td>96M</td>
        <td>10</td>
        <td>实验室研究阶段</td>
		<td><a href="https://purelifi.com/">https://purelifi.com/</a></td>
    </tr>
    <tr>
        <td>Thread</td>
        <td>/</td>
        <td>2014.07</td>
        <td>IEEE 802.15.4</td>
        <td>2.4G</td>
        <td>250K</td>
        <td></td>
        <td>物联网、智能家居</td>
		<td><a href="http://threadgroup.org/">http://threadgroup.org/</a></td>
    </tr>
</table>


