你好，我是刘歧。

前面四节课，我们介绍了音视频与直播相关的基础知识，那么接下来我们就要进入实战阶段了。学完这个部分，音视频处理的常用工具怎么用，你就能心中有数了。

前面我们虽然了解了什么是直播，直播服务器可以用到哪些开源项目。但直播推流到底怎么实现并没有详细展开，所以这节课我们重点讲讲怎么基于FFmpeg推直播流。如果你的业务场景用FFmpeg不太方便，我还提供了另一个方法——桌面工具OBS推流。相信学完之后，你就能轻松搞定推流。

首先，我们做直播推流的前提是要有直播服务器接收直播流，所以需要我们自己建设一个流媒体服务器。我们可以根据上一节课提到的开源直播服务器的[官方文档](https://github.com/ossrs/srs/wiki/v5_CN_Home)部署直播服务器，也可以挖掘自己当前使用的直播服务平台的服务器接收直播流。为了方便演示，我使用快手的直播云服务来接收我推的直播流。界面如下：

![图片](https://static001.geekbang.org/resource/image/d4/0f/d478c987355cff9dc4f1a7fc849b930f.png?wh=2634x1616)

## FFmpeg推流

通常，推流服务器的管理界面会提供一个收流的RTMP服务器地址，还会提供一个直播流的流名称，也叫串流密钥。例如推流的RTMP服务器地址是rtmp://publish.x.com/live，串流密钥是stream，那么最后组成的推流地址就是rtmp://publish.x.com/live/stream。

如果使用FFmpeg推RTMP流的话，我们需要使用的输出格式为FLV，那么FFmpeg的输入就是-f flv rtmp://publish.x.com/live/stream。为了便于理解，我们自己模拟一个直播画面，然后推RTMP直播流，下面是完整的命令行。

```plain
ffmpeg -re -f lavfi -i testsrc=s=1280x720:r=25 -pix_fmt yuv420p -vcodec libx264 -f flv rtmp://publish.x.com/live/stream
```

这条命令行理解起来比较简单，FFmpeg分两大部分，一部分是输入部分，也就是-i与-i的参数以及它之前的部分，另一部分就是-i与-i参数后面的部分，为输出部分。仔细划分一下，这条命令行的输入部分是-re -f lavfi -i testsrc=s=1280x720:r=25，输出部分是-pix\_fmt yuv420p -vcodec libx264 -f flv rtmp://publish.x.com/live/stream。

### 命令行参数

输入部分的意思是使用FFmpeg的lavfi输入格式，也可以说输入的是lavfi设备。输入内容是testsrc，这个testsrc是输入lavfi格式的内容，lavfi的格式有很多内容，这些内容不是既有的内容，也不是某个文件，而是FFmpeg通过filter自己创建出来的。除了testsrc，还可以创建testsrc2、color、yuvall等图像内容（更多可创建的内容你可以查看[FFmpeg官方文档](https://ffmpeg.org/ffmpeg-filters.html#Video-Sources)）。输入的图像是25fps，也就是每秒钟会得到25帧图像，图像的宽是1280像素，高是720像素。

如果你足够细心的话，就会发现我还没有讲-re这个参数。它其实是控制获得图像频率的参数，用来控制输入包的读取速度，比如我们规定一秒钟输入25帧，它会把速度控制在25帧。因为在FFmpeg中数据是以最快的速度读完的，一般在高配的机器上读取速度会非常快。我们用FFmpeg自主生成的数据来模拟直播，就需要用-re来控制一下速度。但如果我们是读取摄像头还有RTSP等直播协议输入的数据，就可以不控制，因为对方输出也是控制频率的。

输出部分的意思是先把读取的图像像素点的颜色格式转成yuv420p格式，关于yuv420p格式，我们在前面的课程中是讲过的，这里为什么使用yuv420p呢？因为yuv420p在视频图像格式中是兼容性最好的，使用起来会比较稳定。

接下来编码器部分使用的视频编码器为libx264。libx264是一个第三方编码器，这里我们需要注意的是libx264的FFmpeg需要使用自由软件基金会的通用公共协议的License，也就是常说的GPL协议。作为开源软件发行版使用问题不大，如果商用的话可能需要考虑法律风险。

我们继续看输出部分，-f flv规定我们输出的封装格式为FLV，用-f指定封装格式以后，输出文件的文件名其实也不会有作用，因为FFmpeg会强制输出-f指定的输出格式。最后输出的文件是一个RTMP协议特征字符开头的URL，所以最终会将FLV格式的内容输出到FFmpeg RTMP协议内容中。

到这里，使用FFmpeg推流到RTMP服务器就可以执行了。我们点开快手，到我们自己的直播间看一下效果，从图中可以看到直播已经开始了，testsrc的画面已经出来了。

![图片](https://static001.geekbang.org/resource/image/ee/42/ee0c74eda1fd5c8846ed041166b1ee42.png?wh=4449x3057)

有些人觉得安装FFmpeg太麻烦了，并且使用命令行也比较繁琐，记不住参数。刚入门的时候使用FFmpeg可能确实有些麻烦，会觉得参数太多。其实用着用着找到窍门后，你就会发现**FFmpeg参数并不多，尤其是我们常用的能力部分，模块化做得很好**，不同的编码器都有自己的参数可以配置，如果我们集中精力只关注我们自己使用的模块部分，参数并不多也很容易记住。

当然有些人可能想要采集自己的摄像头、桌面等外设，这些通过FFmpeg的-devices参数可以得到相关的设备信息。通过[FFmpeg的设备相关的操作文档](https://ffmpeg.org/ffmpeg-devices.html)指引信息，我们也可以自己用FFmpeg获得摄像头、桌面等外设图像，通过编码推流到直播服务器上。支持的外设比较多，这里就不展开说了，具体内容你可以参考文档自己挖掘一下。

## 带界面的推流神器OBS

如果你确实觉得FFmpeg太难、太麻烦，还有一种方式也能帮助你完成推流。那就是**使用界面直播推流神器OBS，这种工具不用命令行，也可以帮你轻松搞定推流**。

OBS是个桌面应用程序，首先我们需要从OBS官方网站下载OBS并安装上，安装后打开的界面比较直观。

![图片](https://static001.geekbang.org/resource/image/df/b7/df5ca1ced76500a1yy4aef10ce768fb7.png?wh=1079x751)

OBS的功能非常强大，这里我们主要介绍抓取本地窗口的场景。首先我们基于场景添加一个来源，来源可以选择窗口采集。

![图片](https://static001.geekbang.org/resource/image/02/9a/024b2354a6f205283573140c6f10919a.png?wh=1432x874)

然后选择一个窗口，比如我正在本地用播放器播放一个MP4的视频，我可以通过OBS抓取这个播放MP4视频的播放器窗口，我们预计将这个播放器正在播放的内容以直播的方式推流到RTMP服务器上。

![图片](https://static001.geekbang.org/resource/image/5b/2d/5b65e7fc7cf66f3aa0e9767bf346102d.png?wh=819x912)

然后我们设置一下推流和编码器，点设置按钮。

![图片](https://static001.geekbang.org/resource/image/e0/c1/e071438ce009c8a90d44e198b0c95dc1.png?wh=1077x749)

进入设置窗口后选推流选项，填入RTMP服务器地址，推流密钥。

![图片](https://static001.geekbang.org/resource/image/26/5d/26d5a553231fca1a849930b788b0a05d.png?wh=981x748)

为了让推流更流畅，我们还需要进入视频选项里面，设置一下视频的编码参数。

![图片](https://static001.geekbang.org/resource/image/e2/ea/e2850d4ae8c2aa03ef309510d8f4deea.png?wh=981x748)

主要是分辨率和帧率，需要我们稍微调整一下，比如我的播放器播放的视频是720p，那我就可以选择720p对应的分辨率，关于分辨率我们在[第一节课](https://time.geekbang.org/column/article/543605)已经介绍过了，你可以一边回顾一边操作。

接下来我们设置一下音频编码参数。

![图片](https://static001.geekbang.org/resource/image/c7/8b/c792edf5536cd244fyycb9146fd4b88b.png?wh=981x748)

音频编码我们可以根据自己的需要进行设置，比如是否采集桌面声音，是否采集麦克风声音等，我们这里选择默认的麦克风输入声音即可。

当然，如果我们推流的PC机有GPU卡，或者我们想直接用CPU软编码的话，可以进入高级设置项设置详细参数。

![图片](https://static001.geekbang.org/resource/image/39/a1/391c7e0de8e59813fcf3b0079c197ea1.png?wh=977x744)

对应的可以设置的参数还是挺多的，比如我这里默认用的是x264编码器，那么可以设置码率是否为CBR，CBR主要是恒定码率，也就是在画面和视频图像运动场景比较复杂的时候，画质会显得模糊一些，如果用VBR的动态码率的话，在画质方面会根据图像运动场景等因素存储更多的参考帧相关的信息，码率会动态地拉升。

之后就需要设置关键帧间隔，这里我设置的1其实不太合理，一般我们设置2到5秒钟比较合理。关键帧间隔设置得合理的话，可以确保直播延迟低的同时画质更高，这个部分我们完全可以根据自己的考核标准操作。再往下看就是H.264编码标准里面的设置，例如快速编码，编码的profile设置，是否用低延迟模式等。

如果这些参数设置得不尽兴的话，了解了x264的更多详细参数后，你也可以自己选填更细节的参数。当然，如果想用GPU编码的话，也可以自由选择，比如我是苹果电脑，那么我可以选择使用videotoolbox进行视频编码，这样也可以节省一些我的CPU资源，电脑的风扇也会相对安静一些，编码性能大多数情况下会比CPU编码要高很多。

![图片](https://static001.geekbang.org/resource/image/8f/00/8f7ba8247fd55bac74c86f98668a4200.png?wh=1076x747)

推流之后，我们就能在直播间里看到自己的直播流内容了。

![图片](https://static001.geekbang.org/resource/image/b3/3a/b3571d654785b39b74f1c7d9baa9c93a.png?wh=4421x3219)

## 小结

到这里，我们可以在零基础的情况下做直播推流，推一些我们自己的内容到直播服务器上面了。无论是使用FFmpeg还是OBS，都能够很好地作为推流客户端进行推流，使用FFmpeg推流的话一条命令行就可以搞定。如果你不喜欢用鼠标点来点去的话，推荐你使用FFmpeg推流；而用OBS的话，不需要下载安装FFmpeg，不需要点开终端自己输入各种参数，也不用特意去记这些参数就能轻松搞定推流，深度挖掘的话，甚至还可以实现导播等功能。

更多OBS的高级功能和黑科技，还是需要我们自己耐心地去一点点挖掘。在这里我通过简单的介绍引领你入门，目的是**让你具备基础的直播推流能力**。但师父领进门，修行在个人，还是需要你自己动手实际操作起来，相信你会有不小的收获。

## 思考题

通过刚刚的操作，我们推流成功了，而且也能在播放器端看到推流成功后的直播流了，那么我们怎么确认这个直播流是不是我们设置好的1280x720的分辨率，帧率是不是25fps呢？期待在评论区看到你的答案，也欢迎你把这节课分享给需要的朋友，我们下节课再见！
<div><strong>精选留言（11）</strong></div><ul>
<li><span>Geek_e2e4e9</span> 👍（3） 💬（2）<p>ffmpeg安装太费劲了，分享个靠谱的文档：https:&#47;&#47;www.jianshu.com&#47;p&#47;663267e13769</p>2022-08-20</li><br/><li><span>村口大发</span> 👍（2） 💬（1）<p>请教一下老师，按照上面说的，试了一下ffmpeg推流，不过一直错误，难道是我参数有误？信息如下：xxx@xxx-MB4 FFmpeg % ffmpeg -re -f  -i &#47;Users&#47;allenchen&#47;workspace&#47;FFmpeg&#47;output.flv lavfi -i testsrc=s=1280x720:r=25 -pix_fmt yuv420p -vcodec libx264 -f flv rtmp:&#47;&#47;open-push.voip.yximgs.com&#47;gifshow&#47;kwai_actL_ol_act_9934790925_strL_origin?sign=633d0143-c210c0d7d91c2c1849c18bbeeac98244&amp;ks_fix_ts
输出内容：
[4] 32198
zsh: no matches found: rtmp:&#47;&#47;open-push.voip.yximgs.com&#47;gifshow&#47;kwai_actL_ol_act_9934790925_strL_origin?sign=633d0143-c210c0d7d91c2c1849c18bbeeac98244
[4]    exit 1     ffmpeg -re -f -i &#47;Users&#47;allenchen&#47;workspace&#47;FFmpeg&#47;output.flv lavfi -i      -
zsh: command not found: ks_fix_ts</p>2022-08-28</li><br/><li><span>ifelse</span> 👍（0） 💬（1）<p>老师，请问抖音，B站等平台提供的Pc直播工具，算是类似OBS的推流工具吗？</p>2023-12-23</li><br/><li><span>包美丽</span> 👍（0） 💬（1）<p>成功了！</p>2022-11-13</li><br/><li><span>Geek_3b601e</span> 👍（0） 💬（1）<p>大师兄 答案是什么，直播的过程中怎么看分辨率和帧率</p>2022-08-30</li><br/><li><span>晓风残月</span> 👍（0） 💬（1）<p>老师，我想做个远程控制Android平板，Android录屏用H264编码，服务器做转发（NAT穿越局限太大），但服务器只有5M带宽，还跑着别的项目，同一时刻最多有一个设备远程连接,您觉得我用什么传输协议好一些？RTMP还是RTSP或者是其他？麻烦告知一下</p>2022-08-12</li><br/><li><span>peter</span> 👍（0） 💬（1）<p>请教老师一个问题：
Q1：我对于“FFmpeg推流操作步骤”的理解是： Step1：手机上安装快手APP，创建一个直播间，该直播间会有文中提到的收流地址，Step2：在PC机上安装FFmpeg，安装后采用命令行来推送视频到直播间的收流地址。 我的理解是否对？</p>2022-08-08</li><br/><li><span>村口大发</span> 👍（1） 💬（0）<p>补充下上面推流失败的后续，经老师的指出，对推流命令中的目标地址url必须加上双引号，加上双引号之后还是报错，后来发现有参数的顺序错了，最终执行OK的命令如下：

ffmpeg -re -i &#47;Users&#47;allenchen&#47;workspace&#47;FFmpeg&#47;output.flv -f lavfi -i testsrc=s=1280x720:r=25 -pix_fmt yuv420p -vcodec libx264 -f flv &quot;rtmp:&#47;&#47;open-push.voip.yximgs.com&#47;gifshow&#47;kwai_actL_ol_act_9934790925_strL_origin?sign=633d0143-c210c0d7d91c2c1849c18bbeeac98244&amp;ks_fix_tsb&quot;

希望对其他同学有帮助</p>2022-08-28</li><br/><li><span>dog_brother</span> 👍（1） 💬（0）<p>之前基于nginx搭建过直播服务器</p>2022-08-03</li><br/><li><span>啊良梓是我</span> 👍（0） 💬（2）<p>云直播仅对媒体政务号和企业认证账号开放，其他账号请使用直播伴侣或快手App开播，并遵守快手直播规范</p>2023-12-06</li><br/><li><span>徐溪越</span> 👍（0） 💬（0）<p>ffplay -i
</p>2022-08-21</li><br/>
</ul>