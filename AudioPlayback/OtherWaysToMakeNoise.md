###其它发出声响的方式

虽然MediaPlayer是主要的音频播放选项，特别是对MP3文件来说。构造其它种类的应用时存在其它的替代选项，特别是游戏以及自定义形式的流音频。

####SoundPool

SoundPool类名声在外的原因是加载多个声音的能力,并且以一种按优先顺序列出的方式，所以你的应用可以仅仅要求播放声音而SoundPool处理每个声音的开始，结束，以及播放时的混合。


这用一个样例来说明更有说服力。

假定你创建了一个第一人称视角的射击游戏。这个游戏能同时播放多个声音：

1.战场上的风声
2.浪拍打陆地的声音
3.靴子在沙地上的声音
4.角色在沙滩上跑动的是气喘嘘嘘的声音
5.中士在角色后的叫声
6.瞄准角色机枪的声音以及角色伙伴的声音
7.提供压制活力的战舰的爆炸声

等等。

原则上来说，SoundPool可以混合所有这些到单个音频流进行输出。你的游戏可能设置风和浪的声音作为
恒久不变的背景音，当角色移动的时候触发脚步声，随机地添加吼叫声，在实际游戏的时候绑定枪声。

实际上，你的智能手机会缺少处理所有音频而不损耗游戏帧率的CPU能力。所以，为了保持较高的帧率，你要告诉SoundPool同时最多播放两个声音。这就意味着当没有其他事情发生在这个游戏的时候，你会听到风声和浪声，但是在实际发生战斗的时候，这些声音就会被落下-用户可能没有在乎它们-所以这个游戏的速度保持的很好。


####AudioTrack

Java API最底层用于播放音频的是AudioTrack。它只要有两个职责：

1.它的首要职责是支持流音频，流以一种MediaPlayer不能处理的格式进来。虽然MediaPlayer能够处理RTSP，例如，它不能提供SIP。如果你要创建一个SIP客户端(可能是针对一个VOIP或者网络会议应用)，
你会需要把进来的数据流转换为PCM格式，然后把流交给一个AudioTrack实例进行播放。

2.它也可以用作“静态的”（对应流）音频字节，你已经预先解码成了PCM格式并且尽可能低延时地进行播放。例如，你可能使用这个作为游戏音（短促的尖声，子弹声，弹开的声音）。不同预解码数据成PCM并缓存结果，使用使用AudioTrack用于播放，你会使用最小的开销，最小化CPU对游戏过程和电池生命的影响。

####ToneGenerator

如果你想要你的手机听起来很像手机，你可以使用ToneGenerator去播放DTMF音调。换句话说，你可以通过一个常规的手机音模拟对应的按键音。这是被Android拨号器所使用的，例如，当用户使用屏幕上的键盘拨打电话的时候，作为一个音频增强。

注意这些会在手机的耳机，话筒或者连接耳机上播放出来。它们没有播放向外的调用流。原则上来说，你可能获取ToneGenerator通过扬声器去播放音调，这个音调足够响从而能被麦克风接起来，但是这不是一个推荐的实践。


