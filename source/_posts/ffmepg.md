ffmpeg 相关

# 参数相关

流程:输入--->处理-->输出

`-i` ===>input 输入

`-c` ===>codec 编解码器 ,告诉ffmepg接下来用什么编码器和解码器

​		`-c copy` ==>复制照抄输入文件原来编解码器所做的事情

> ​	老版本的ffmpeg:`-c copy`<====>`-vcodec copy -acodec copy`

​		`-c:v` 限定只处理视频画面

​		`-c:a` 限定只处理视频里的音频声音

​		`-c:s` 限定只处理字幕

> `-c:v libx264` 将编码格式换成h164
>
> `-c:a libmp3lame` 将编码格式换成mp3
>
> 
>
> 可以直接 `-c:v h264 -c:a mp3`

`-f ` ==>format 格式 ,强制输出什么格式,让ffmpeg自行挑选编解码器进行转码输出

`-f webm` 自己想转的容器格式有什么编码格式或编码器可以支持

## 转码思路

1. `ffprobe .\input.mp4` 查看文件的编码格式,或者直接查看文件属性

2. 确定自己的相关的容器或编码格式

3. `ffmpeg -formats`查询ffmpeg支持哪些容器格式

   demuxing 解封装,muxing 封装(转码)

   `ffmpeg -codecs`查询ffmpeg支持哪些编码格式及编码器名

   decoder 解码器名 encoder编码器名



## 高效转码,编码格式/容器格式

`ffprobe .\input.mp4 ` 用于查看文件的详细信息

一般画面的编码格式 h264

声音的编码格式 aac

容器格式: mp4/flv/mkv/avi==>封装格式



高效转码==>编码格式相同,容器格式不同

视频编码格式https://www.bilibili.com/read/cv4480903

<p>h264（又称mpeg-4 avc、mpeg-4 part 10）：mp4、flv、avi、mov、wmv、m4v、f4v、3gp、ts</p>

 mpeg4（不只一种，这里指mpeg-4 part 2、divx、xvid）：mp4、avi、mov、wmv、m4v、3gp、ts 

 h265（又称hevc、mpeg-h part 2）：mp4、avi、mov、ts 

 vp8：avi、wmv、ts、webm 

 vp9：mp4、avi、wmv、ts、webm 





