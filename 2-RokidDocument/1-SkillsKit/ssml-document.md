### 在您的技能中使用 SSML
若琪的文字转语音（Text to Speech，简称TTS，下同）会自动对标点符号做基本处理，比如在语句的`|` `，` `。`之后将会插入短停顿。能够满足基本的使用场景。

如果您需要对TTS进行诸如插入一段音频、改变语速、改变发音人等更加灵活的自定义，可以使用下述的语音合成标记语言（Speech Synthesis Markup Language， 简称SSML，下同）。

### 目录
- [在您的技能中使用 SSML](#%E5%9C%A8%E6%82%A8%E7%9A%84%E6%8A%80%E8%83%BD%E4%B8%AD%E4%BD%BF%E7%94%A8-ssml)
- [目录](#%E7%9B%AE%E5%BD%95)
- [如何在您技能的Response中使用SSML](#%E5%A6%82%E4%BD%95%E5%9C%A8%E6%82%A8%E6%8A%80%E8%83%BD%E7%9A%84response%E4%B8%AD%E4%BD%BF%E7%94%A8ssml)
- [目前可使用的SSML标签](#%E7%9B%AE%E5%89%8D%E5%8F%AF%E4%BD%BF%E7%94%A8%E7%9A%84ssml%E6%A0%87%E7%AD%BE)
  - [audio](#audio)
    - [将音频文件转换为Rokid可用的格式](#%E5%B0%86%E9%9F%B3%E9%A2%91%E6%96%87%E4%BB%B6%E8%BD%AC%E6%8D%A2%E4%B8%BArokid%E5%8F%AF%E7%94%A8%E7%9A%84%E6%A0%BC%E5%BC%8F)
  - [break](#break)
  - [p](#p)
  - [phoneme](#phoneme)
  - [prosody](#prosody)
  - [s](#s)
  - [say-as](#say-as)
  - [speak](#speak)
  - [sub](#sub)
  - [voice](#voice)
  - [word](#word)
- [特别说明](#%E7%89%B9%E5%88%AB%E8%AF%B4%E6%98%8E)
    

### 如何在您技能的Response中使用SSML
直接在返回的TTS中加入SSML格式，完整Voice Response协议请查看[Rokid 技能协议文档Voice部分](3-ApiReference/cloud-app-development-protocol_cn.md)。

```json
"item": {
    "tts": "<speak>Rokid TTS语音支持SSML。</speak>"
}
```

### 目前可使用的SSML标签

#### audio
在TTS语句中插入一段音频，可用做TTS回复的背景音，或者特定的音效回复。

| 参数 | 可选值 |
| :--- | :--- |
| src | 背景音仅支持通过URL提供的`.wav`文件，并且需要满足以下要求。<ul><li>WAV文件必须被放在一个可靠的HTTPS终端中，且HTTPS域名需要有合法的SSL证书。</li><li>WAV文件不能够含有个性化定制的内容以及敏感内容。</li><li>音频文件的长度需限制在90秒内。</li><li>比特率为16 bit，单声道 mono，采样率为24000 HZ。</li><li>WAV文件头的长度暂时只支持44字节，如果超出，则会出现爆音。</li></ul> 您需要使用[下方推荐的转换工具](https://www.ffmpeg.org/)将音频文件转换为符合上述要求的WAV文件。 |

像如下例子中一样，在TTS回复中用`speak`标签包裹`audio`标签后，您的回复将带有背景音。

```html
<speak>
    欢迎成为Rokid开发者。
    <audio src="http://s.rokidcdn.com/temp/rokid-ring.wav" /> 
    您会听到一段悦耳的铃声。 
</speak> 
```

注意：

- 单条response中最多只能插入5个音频文件
- 单条response中的所有音频长度加总不能超过90秒。

##### 将音频文件转换为Rokid可用的格式
您需要使用转换工具将需要插入的TTS中的音频转为WAV文件，16 bit，mono 24000HZ。推荐使用开源的命令行工具[FFmpeg](https://www.ffmpeg.org/)。下方的命令可以帮您将`<input-file>`转换为`audio`标签可用的WAV文件。

**MP3转WAV：**

```
ffmpeg -i <input-file.mp3> -acodec pcm_s16le -ac 1 -ar 24000 <output-file.wav>
```

**PCM转WAV：**

```
ffmpeg -f s16le -ar 24k -ac 1 -i <input-file.pcm> <output-file.wav>
```

**MP4转WAV：**

```
ffmpeg -i <input-file.mp4> -acodec pcm_s16le -ac 1 -ar 24000 <output-file.wav>
```

**WAV头部规整：**

如果wav格式符合以上格式，但在播放时最前面有爆音现象，是由于头部长度不是44字节，可以使用另一开源的命令行工具[sox](http://sox.sourceforge.net/)进行修正：
```
sox -r 24000 -e signed -b 16 -c 1 old.wav new.wav
```


#### break
在TTS语句中插入停顿，

| 参数 | 可选值 |
| :--- | :--- |
| strength | <ul><li>none: 无停顿。常用于消除默认停顿的部分。</li><li>x-weak: 无停顿。和<code>none</code>效果一致。</li><li>weak: 标准停顿。等同于逗号的效果。和<code>medium</code>效果一致。</li><li>medium: 标准停顿。</li><li>strong: 句子停顿。等同于句号以及<code>&lt;s&gt;</code>标签的效果。</li><li>x-strong: 段落停顿。等同于<code>&lt;p&gt;</code>标签的效果。</li></ul>|
| time | 具体停顿时长，秒的绝对值。上限10s，可以精确到小数点后1位。 |


```html
<speak>
    这句话之后会有3.3秒钟的停顿 <break time="3.3s"/> 
    然后才会继续说。
</speak>
```

#### p
表示一个段落。将会在段尾为TTS语音插入一个强停顿，等同于`<break strength="x-strong">`。

```html
<speak>                                         
    <p>这是第一段话。说完之后将会停顿一会。</p>       
    <p>这是第二段话。</p> 
</speak>    
```

#### phoneme
为TTS指定不同的发音。

| 参数 | 可选值 |
| :--- | :--- |
| alphabet | 指明音标规范 <ul><li>py: 拼音。</li></ul>|
| ph | 音标的具体序列。 |

```html
<speak>
    我的指甲又 <phoneme alphabet="py" ph="zhang3/chang2">长长</phoneme>了。 
</speak>   
```

#### prosody
对TTS的语速、声调、音量做调整。

| 参数 | 可选值 |
| :--- | :--- |
| rate | 语句的发音速度 <ul><li>枚举值：x-slow, slow, medium, fast, x-fast
| pitch | 语句的语调 <ul><li>枚举值：x-low, low, medium, high, x-high
| volume |  声音的高低 <ul><li>枚举值：silent, x-soft, soft, medium, loud, x-loud

```html
<speak>
    这句话是普通音量。
    <prosody volume="x-loud">第二句话增加了音量。</prosody>。
    接下来, <prosody rate="x-slow">语速会变慢</prosody>。
    我还可以把我的音调变高。 
    <prosody pitch="x-high"> 比如这样 </prosody>，
    但同样可以降低 <prosody pitch="low">变成这样</prosody>。
</speak>  
```

#### s
表示一个句子。将会在句后插入一个停顿。
等同于：

- 在句末加入句号`。`
- 插入一个停顿 `<break strength="strong"/>`

```html
<speak>
    <s>这是一个句子</s>
    <s>刚才你听到了停顿</s> 
    这个句子句末的句号将提供相同的停顿效果。
</speak> 
```


#### say-as
为单词或语句指定以何种方式进行发音。

| 参数 | 可选值 |
| :--- | :--- |
| interpret-as | <ul><li>cardinal, number: 按数值发音。比如“316”，将读作“三百一十六”。</li><li>digits: 按数字串发音。比如“666”，将读作“六六六”。</li><li>address: 将地址缩写读作该地址的全称（暂时仅支持英文）。</li></ul>|

```html
<speak>
    昨天逛超市花了<say-as interpret-as="cardinal">216</say-as>块。
    你的房号是 <say-as interpret-as="digits">666</say-as>。
</speak> 
```

#### speak
SSML的根元素。

```html
<speak>
    这句话是没有任何SSML语法的效果。
</speak> 
```

#### sub
将指定的单词或句子替换为特定的读音，需要搭配 `alias`参数来使用。

| 参数 | 可选值 |
| :--- | :--- |
| alias | 为指定的单词或句子替换读音。|

```html
<speak>
    今天的比分是4 <sub alias="比">：</sub>2。 
</speak> 
```

#### voice
使用不同的发音人进行发音。

| 参数 | 可选值 |
| :--- | :--- |
| name | <ul><li>normal: 正常若琪。</li><li>robot: 机器人。</li><li>sweet: 甜美女生。</li><li>crayon: 蜡笔小新。</li></ul>|

```html
<speak>
    接下来出场的是蜡笔小新，
    <voice name="crayon">大家好，我是蜡笔小新</voice>。 
</speak> 
```

#### word
使指定的词组不被拆分以保证不会有停顿。

```html
<speak>
    这双<word>球鞋</word>是C罗送给我的。 
</speak> 
```

### 特别说明

1. SSML语法不区分大小写。
2. 若一个句子中嵌套了多个SSML用法，效果将会叠加。比如对整句进行了加速和蜡笔小新的效果叠加，将会以蜡笔小新的声音快速读完该句话。
3. 若某部分的SSML出现了拼写错误，该部分会按照未添加任何SSML效果的语句来输出。比如这两个例子：`<word>球鞋</wrod>`、`<phoneme alphabet="py" ph="zhang3/canhg2">长长</phoneme>`。
4. 未在上述列出的用法暂不支持，若被您使用在句子中，若琪将按照普通语句输出。
5. 如果您在定义SSML时出现标签不完整的情况，比如：`<p>这是一个错误示例</p`、`<s>这是第二个错误示例>`，该句TTS将不会被输出，并返回`ssml syntax error`。



