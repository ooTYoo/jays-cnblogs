----
　　大家好，我是痞子衡，是正经搞技术的痞子。今天痞子衡给大家介绍的是**J-Link仿真器版本变迁**。  

<table><tbody>
    <tr>
        <th style="width: 120px;">硬件版本</th>
        <th style="width: 300px;">主控芯片</th>
        <th style="width: 250px;">固件升级工具</th>
    </tr>
    <tr>
        <td>V7</td>
        <td rowspan="2">ARM7TDMI, 55MHz<br>
		                Atmel AT91SAM7S64</td>
        <td rowspan="2">官方AT91-ISP->SAM-PROG</td>
    </tr>
    <tr>
        <td>V8</td>
    </tr>
    <tr>
        <td>V9</td>
        <td>ARM Cortex-M3, 120MHz<br>
		    ST STM32F205RC</td>
        <td>官方Flash_Loader_Demonstrator<br>
		    第三方mcuisp/FlyMcu<br>
		    第三方STM32ISP</td>
    </tr>
    <tr>
        <td>V10</td>
        <td>ARM Cortex-M4F & M0, 204MHz<BR>
		    NXP LPC4322JBD144</td>
        <td>官方Flash Magic</td>
    </tr>
</table>

