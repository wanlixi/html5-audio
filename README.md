# html5-audio
展示html5提供的强大的音频控制API

****** 
![image](http://www.yuanxiaodie.com/piano/html5-audio.jpg)
线上demo：[模拟钢琴](http://www.yuanxiaodie.com/piano/)
首先需要浏览器支持HTML5 Web Audio API
HTML5 Web Audio API 一共提供了音阶 scale 0~9 的10个区间和音调 tone c~b 12个区间
这里我选择了scale 3~8，作为示例实现了一个简单模拟钢琴，并且下面带一首 我自己盲弹摸索的《两只老虎》，有兴趣可以按照我给出音谱弹奏，后续继续完善！
******


首先我们要声明下，这里所说的audio API和 <audio> 标签元素是两码事，而且提供了非常丰富的API，
HTML5 Audio API可以使我们无中生有创造出声音，也就是说如果我们掌握了音律知识和Audio API，就可以使用它和
硬件（电脑）组装出来一架 “ 钢琴 ” 。

HTML5 Web Audio API提供的功能有以下几点：
#### 1、对简单和复杂的声音进行混合。
#### 2、可以精确的控制声音的密度和节奏。
#### 3、内置淡入淡出，颗粒噪点，音调控制效果。
#### 4、可以灵活的处理声频的声道，例如拆分或合并。
#### 5、可以处理<audio> <video>媒体元素的音频源。
#### 6、使用MediaStream的getUserMedia（）方法处理现场输入的音频，例如变声。
#### 7、立体音效，支持多种3D游戏和沉浸式环境。
#### 8、利用卷积引擎，创建各类线性音效，例如小/大房间、大教堂、音乐厅、洞穴、隧道、走廊、森林、圆形剧场等。尤其适用于创建高质量的房间效果。
#### 9、高效的实时的时域和频分析，配合CSS3或Canvas或webGL可以实现音乐可视化支持。
#### 10、以及其他多种音频波形算法控制，几乎就是把一个音频编辑类软件搬到web上。

### 网页交互音效AudioContext，AudioParam等API介绍
``` 
  window.AudioContext = window.AudioContext || window.webkitAudioContext;
  var audioCtx = new AudioContext();
  var oscillator = audioCtx.createOscillator();
  var gainNode = audioCtx.createGain();
  oscillator.connect(gainNode);
  gainNode.connect(audioCtx.destination);
  oscillator.type = 'sine';
  oscillator.frequency.value = 196.00;
  gainNode.gain.setValueAtTime(0, audioCtx.currentTime);
  gainNode.gain.linearRampToValueAtTime(1, audioCtx.currentTime + 0.01);
  oscillator.start(audioCtx.currentTime);
  gainNode.gain.exponentialRampToValueAtTime(0.001, audioCtx.currentTime + 1);
  oscillator.stop(audioCtx.currentTime + 1); 
  ```
  
第一行：兼容老的webkit浏览器

第二行：创建一个音频的上下文，类似canvas里面的getContext，它包含了大量的属性和方法，如下
       AudioContext.createConstantSource(): 创建一个constantSourceNode对象，它持续的输出一个连续的单声道，这些样本都会有一个固定的相同的固定值
       AudioContext.createBufferSource(): 创建一个audioBufferSourceNode对象，它可以通过audioBuffer对象来播放和处理包含在内的音频数据，AudioBuffer可以通过AudioContext.createBuffer方法创建或者使用AudioContext.decodeAudioData方法解码音轨来创建。
       AudioContext.createMediaElementSource（）：创建一个MediaElementAudioSourceNode接口来关联HTMLMediaELement，这可以用来播放和处理<audio>和<video>标签元素的音频。
       AudioContext.createMediaStreamSource（）：创建一个MediaStreamAudioSourceNode接口来关联可能来自本地计算机麦克风或者其他来源的音频流MediaStream
       AudioContext.createMediaStreamDestination（）：创建一个MediaStreamAudioSourceNode接口来关联可能储存在本地或已发送至其他计算机的MediaStream音频
  [文档](http://www.zhangxinxu.com/wordpress/2017/06/html5-web-audio-api-js-ux-voice/)
  
第三行：createOscillator()四audioContext的一个方法，创建一个OscillatorNode，OscillatorNode表示一个周期性波形，比如一个正弦波，大家都知道声音的本质就是震动，是和波形紧密关联的，因此波形不一样，最终的声音也就不一样，而波还收频率影响，最终表现为不一样的音调高低，因此OscillatorNode有两个属性frequency和type来影响， *** Oscillator基本上是创建一个音调 ***

第四行：createGain（），创建了一个GainNode，它可以控制音频的总音量，它只有一个简单的只读属性gain完整书写GainNode.gain，本身没什么，但是GainNode.gain的返回值可是个大家伙，是个a-rate类型的[AudioParam](https://developer.mozilla.org/zh-CN/docs/Web/API/AudioParam)。AudioParam总共有2个类型，一个是a-rate还有一个是k-rate，前者AudioParam为音频信号每个样品帧的当前声音参数值；后者的AudioParam使用整个块（即128个样本帧）相同的初始音频参数值。AudioParam包含诸多属性和方法，都是与音量控制相关的。

第五行：音频和音量关联起来

第六行：gainNode.connect(audioCtx.destination) 这里的audioCtx.destination返回AudioDestinationNode对象，AudioDestinationNode接口表示音频图形在特定情况下的最终输出地址 – 通常为扬声器。话句话说就是和音频设备关联。

第七行： 7. oscillator.type = ‘sine’ 我们的声音波形指定为'sine'，也就是正弦波，还支持其他3种波形，为'square'方波，'triangle'三角波以及'sawtooth'锯齿波。形状如

<br> ![image](http://image.zhangxinxu.com/image/blog/201706/2017-06-10_223950.png)

第八行： oscillator.frequency.value = 196.00
设置波形的频率，实际上，可以理解为设置声音的音调，也就是设置最后的声音是“多瑞米发嗦啦西”其中的一个。数值越小，声音越低沉，越大提琴；数值越大，声音越清脆，越小提琴。

### 完整的八度音阶与频率的关系可以参考：[这里](http://www.cnblogs.com/cute/archive/2013/02/28/2937222.html)

demo页面中的音调完整范围如下：

[196.00, 220.00, 246.94, 261.63, 293.66, 329.63, 349.23, 392.00, 440.00, 493.88, 523.25, 587.33, 659.25, 698.46, 783.99, 880.00, 987.77, 1046.50]
每次鼠标hover我都会变换一个频率，这就是为什么每次经过按钮就会听到有规律的音调变化。

第九行：9. gainNode.gain.setValueAtTime(0, audioCtx.currentTime)
先来了解下audioCtx.currentTime，其值是以双精度浮点型数字返回硬件调用的秒数。只读，也就是说这个时间你是无法改变的，无论是暂停或者重置都不可以。当我们使用类似new AudioContext()创建AudioContext的时候，这个时间就可以从0开始走了，然后就像离弦的箭，一直不回头。

gainNode.gain这个前面介绍过了，返回的是AudioParam，所以这里实际上执行的是AudioParam.setValueAtTime()方法，此方法支持两个参数，一个是音量（范围0~1），一个时间，这里setValueAtTime(0, audioCtx.currentTime)表示当下就把音量设为0，也就是没声音，因为快速鼠标hover时候，之前声音可能还没播放结束，需要从0开始。

第十行： gainNode.gain.linearRampToValueAtTime(1, audioCtx.currentTime + 0.01)
linearRampToValueAtTime()方法表示音量在某时间线性变化到某值，这里linearRampToValueAtTime(1, audioCtx.currentTime + 0.01)实际上表示的是在0.01秒的时间内，声音音量线性从0到1。

第十一行： oscillator.start(audioCtx.currentTime)
开始出声啦……

第十二行： gainNode.gain.exponentialRampToValueAtTime(0.001, audioCtx.currentTime + 1)
exponentialRampToValueAtTime()方法表示音量在某时间指数变化到某值，这里的exponentialRampToValueAtTime(0.001, audioCtx.currentTime + 1)实际上表示的是在1.00秒的时间内，音量由之前的1以指数曲线的速度降到极低的0.001音量。


第十三行： oscillator.stop(audioCtx.currentTime + 1)
1秒后，声音关闭。

