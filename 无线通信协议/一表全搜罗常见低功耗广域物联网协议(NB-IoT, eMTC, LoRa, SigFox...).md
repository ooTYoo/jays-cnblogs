----
　　大家好，我是痞子衡，是正经搞技术的痞子。今天痞子衡给大家介绍的是**低功耗广域物联网协议**。  

　　上一篇痞子衡给大家搜罗了[短距离无线通信协议](http://www.cnblogs.com/henjay724/p/8598096.html)，它是物联网的基础，但它的应用距离比较短，对于长距离的物联网应用鞭长莫及。于是便有了长距离物联网协议，即低功耗广域网络LPWAN（Low Power Wide Area Network），专为低带宽、低功耗、远距离、大量连接的物联网应用而设计。今天痞子衡就用一张表为大家搜罗常见的低功耗广域物联网协议。  

### 低功耗广域物联网协议列表

<table><tbody>
    <tr>
        <th>协议名</th>
        <th>版本</th>
        <th>推出时间</th>
        <th>无线标准</th>
        <th>频段(Hz)</th>
        <th>速率(bps)</th>
        <th>距离(m)</th>
        <th>网络部署</th>
        <th>连接数量</th>
        <th>终端工作时间</th>
        <th>行业应用</th>
        <th>官方网站</th>
    </tr>
    <tr>
        <td>RMPA/OnRamp</td>
        <td>/</td>
        <td>2008</td>
        <td>/</td>
        <td>非授权频段2.4G</td>
        <td>624K</td>
        <td>3000K</td>
        <td>独立建网</td>
		<td>384000/扇区</td>
		<td></td>
		<td>智能路灯等</td>
		<td><a href="http://www.ingenu.com/">http://www.ingenu.com/</a></td>
    </tr>
    <tr>
        <td>SigFox</td>
        <td>/</td>
        <td>2009</td>
        <td>/</td>
        <td>非授权频段868M/902M/920M</td>
        <td>100</td>
        <td>50K</td>
        <td>独立建网</td>
		<td></td>
		<td>10年</td>
		<td></td>
		<td><a href="https://www.sigfox.com/">https://www.sigfox.com/</a></td>
    </tr>
    <tr>
        <td>LoRa</td>
        <td>/</td>
        <td>2013.08</td>
        <td>/</td>
        <td>非授权频段150M-1G</td>
        <td>0.3K-50K</td>
        <td>1K-20K</td>
        <td>独立建网</td>
		<td>200K-300K/hub</td>
		<td>3-10年</td>
		<td>智能电表、水表抄表、智能农业等</td>
		<td><a href="https://www.lora-alliance.org/">https://www.lora-alliance.org/</a><br>
		    <a href="https://www.semtech.com/">https://www.semtech.com/</a></td>
    </tr>
    <tr>
        <td rowspan="3">Weightless</td>
        <td>N</td>
        <td>2015.05</td>
        <td rowspan="3">/</td>
        <td rowspan="3">169/433/470/780/868/915/923MHz</td>
        <td></td>
        <td>5K</td>
        <td rowspan="3">独立建网</td>
		<td></td>
		<td>10年</td>
		<td rowspan="3"></td>
		<td rowspan="3"><a href="http://www.weightless.org/">http://www.weightless.org/</a></td>
    </tr>
    <tr>
        <td>P</td>
        <td></td>
        <td>200-100K</td>
        <td>2K</td>
        <td></td>
        <td>3-8年</td>
    </tr>
    <tr>
        <td>W</td>
        <td>2013.02</td>
        <td></td>
        <td>5K</td>
        <td></td>
        <td>3-5年</td>
    </tr>
    <tr>
        <td>ZETA</td>
        <td></td>
        <td>2013</td>
        <td>/</td>
        <td>433/470/500/787/868/915MHz</td>
        <td>100-50K</td>
        <td>2K-10K</td>
        <td></td>
		<td>10K-100K/站</td>
		<td>5-10年</td>
		<td>路灯、室内照明、智慧城等</td>
		<td><a href="http://www.zifisense.com/">http://www.zifisense.com/</a></td>
    </tr>
    <tr>
        <td>NB-IoT</td>
        <td>/</td>
        <td>2015.09</td>
        <td>3GPP R13</td>
        <td>运营商频段800M/900M</td>
        <td><100K</td>
        <td>15K</td>
        <td>复用现有蜂窝基站</td>
		<td>50K/cell</td>
		<td>10年</td>
		<td>智能电表、水表抄表、智能农业等</td>
		<td></td>
    </tr>
    <tr>
        <td>Wi-Fi HaLow</td>
        <td>/</td>
        <td>2016.09</td>
        <td>IEEE 802.11ah</td>
        <td>非授权频段900M</td>
        <td>150K - 347M</td>
        <td></td>
        <td>独立建网</td>
		<td></td>
		<td></td>
		<td>远端医疗照护监测、智能城市</td>
		<td></td>
    </tr>
    <tr>
        <td>eMTC/LTE-M</td>
        <td>/</td>
        <td>2016.10</td>
        <td>3GPP R13</td>
        <td>运营商频段</td>
        <td>1.4M</td>
        <td></td>
        <td>复用现有蜂窝基站</td>
		<td>50K/cell</td>
		<td>5-10年</td>
		<td></td>
		<td></td>
    </tr>
    <tr>
        <td>Telensa/Senaptics</td>
        <td></td>
        <td></td>
        <td>ETSI</td>
        <td></td>
        <td></td>
        <td></td>
        <td></td>
		<td></td>
		<td></td>
		<td>智慧照明、智慧城市等</td>
		<td><a href="https://www.telensa.com/">https://www.telensa.com/</a></td>
    </tr>
    <tr>
        <td>Nwave</td>
        <td></td>
        <td></td>
        <td></td>
        <td></td>
        <td></td>
        <td></td>
        <td></td>
		<td></td>
		<td></td>
		<td>智能停车等</td>
		<td><a href="https://www.nwave.io/">https://www.nwave.io/</a></td>
    </tr>
    <tr>
        <td>Amber Wireless</td>
        <td></td>
        <td></td>
        <td></td>
        <td></td>
        <td></td>
        <td></td>
        <td></td>
		<td></td>
		<td></td>
		<td></td>
		<td><a href="https://www.amber-wireless.de/">https://www.amber-wireless.de/</a></td>
    </tr>
</table>


