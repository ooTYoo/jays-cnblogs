----

　　大家好，我是痞子衡，是正经搞技术的痞子。本系列痞子衡给大家介绍的是**语音处理工具JaysPySPEECH诞生**。  

　　智能语音交互市场近年来发展迅速，其典型的应用之一智能音箱产品如今已走入千家万户，深受大家喜爱。智能音箱产品的核心就是语音处理，包括音频采集、语音识别（ASR）、自然语言处理（NLP）、文语合成（TTS）、音频播放五大部分。目前除了音频采集和播放必须在嵌入式端实现外，其余三部分一般都在云端处理（嵌入式端通过有线(USB)或无线(Wifi/BLE)将音频数据发送到云端）。痞子衡对语音处理一直比较感兴趣，最近在玩Python也注意到Python里有很多语音处理库，因此打算从零开始写一个基于Python的语音处理工具，这个语音处理工具我们暂且叫她JaysPySPEECH，初步计划为JaysPySPEECH设计4大功能：wav音频录制，语音识别，文语合成，音频播放，第一个稳定正式版v1.0.0效果如下：  

<img src="http://odox9r8vg.bkt.clouddn.com/image/cnblogs/JaysPySPEECH_v1.0.0_overview.PNG" style="zoom:100%" />

> * JaysPySPEECH的github: https://github.com/JayHeng/JaysPySPEECH.git  

### 设计篇（持续更新中...）
> [语音处理工具JaysPySPEECH诞生记（1）- 环境搭建(Python2.7.14 + PyAudio0.2.11 + Matplotlib2.2.3 + SpeechRecognition3.8.1 + pyttsx3 2.7)](https://www.cnblogs.com/henjay724/p/9542690.html)  
> [语音处理工具JaysPySPEECH诞生记（3）- 音频显示实现(Matplotlib, NumPy1.15.0)](https://www.cnblogs.com/henjay724/p/9644637.html)  
> [语音处理工具JaysPySPEECH诞生记（5）- 语音识别实现(SpeechRecognition, PocketSphinx0.1.15)](https://www.cnblogs.com/henjay724/p/9576670.html)  
> [语音处理工具JaysPySPEECH诞生记（6）- 文语合成实现(pyttsx3, eSpeak1.48.04)](https://www.cnblogs.com/henjay724/p/9590032.html)  

> [语音处理工具JaysPySPEECH诞生记（2）- 界面构建(wxFormBuilder3.8.0)]()  
> [语音处理工具JaysPySPEECH诞生记（4）- 音频录播实现(PyAudio)]()  
> [语音处理工具JaysPySPEECH诞生记（7）- 深度开发]()  
