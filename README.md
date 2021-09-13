# ldc-hk-video
nodejs搭建wss服务,resp取流解析海康监控视频在浏览器上播放
​
首先先来看效果


 


摄像头是rtsp格式的，h5原生不支持这种格式,rtsp转rtmp，不过rtmp依赖falsh的支持，但是在chrome已经默认禁用、未来也会淘汰
所以最好方案是rtsp转化成一种类似http协议的方式,能够直接被h5识别
最终方案是nodejs搭建wss流 通过webSocket发送MPEG js解析MPEG不断绘制canvas
其中 Ffmpeg 负责解码 ,jsmpeg负责解码后逻辑,最终html连接wss浏览器播放

教程介绍
准备工作(完整源码会上传我的github,创作不易请顺手给个star,连接放到最后)

1.  Ffmpeg (FFmpeg是一套可以用来记录、转换数字音频、视频，并能将其转化为流的开源计算机程序。采用LGPL或GPL许可证。它提供了录制、转换以及流化音视频的完整解决方案。它包含了非常先进的音频/视频编解码库libavcodec，为了保证高可移植性和编解码质量，libavcodec里很多code都是从头开发的,FFmpeg在Linux平台下开发，但它同样也可以在其它操作系统环境中编译运行，包括Windows、Mac OS X等)
2.  Nodejs websocket WSS服务器 (nodejs有多强大不用介绍了,大前端必备)
3.  jsmpeg 运行解析程序  (MPEG（Moving Picture Experts Group，动态图像专家组）是ISO（International Standardization Organization，国际标准化组织）与IEC（International Electrotechnical Commission，国际电工委员会）于1988年成立的专门针对运动图像和语音压缩制定国际标准的组织,通过JS WEBGL进行对mpeg的解析)
4.  start.bat 启动程序 (基础bat脚本命令)

首先借助VLC Media Player 工具获取视频状态以及验证resp流是否能够播放

 **rtsp取流格式为:rtsp://用户名:密码@局域网:端口号/海康视频类型(h开头)/窗口/sub/av_stream** 


 **示例:rtsp://xxx:xxx@192.168.xxx.xxx:xxx/h264/ch1/sub/av_stream** 




                                               教程开始
安装完Ffmpeg之后配置你的环境变量 

这里以windows为例子

把ffmpeg下的bin配置到系统path变量里面，根据自己不同的路径配置



nodejs环境变量配置这里就不做演示

第一步运行jsmpeg  并且安装依赖  npm install --save ws



安装成功后启动wss服务 

8011是ffmpeg推送端口  

8012是webSocket wss端口

2个必须一起启动,其中断开视频也会断开,启动顺序 8012 => 8011

node websocket-relay.js supersecret 8011 8012



 ffmpeg -i "rtsp://admin:hxzh2019@192.168.11.85:554/h264/ch1/sub/av_stream" -q 0 -f mpegts -codec:v mpeg1video -s 1000x600 http://127.0.0.1:8011/supersecret/live1

ffmpeg 命令分别说明: 原始视频  导出尺寸  导出码率 导出帧频 和分辨率

相关参数请参看https://github.com/phoboslab/jsmpeg 文档



wss启动成功后 



 ffmpeg启动成功后



双击start.bat脚本命令

@echo off
SETLOCAL EnableExtensions
set EXE=nginx_sm_ws_rtsp.exe
FOR /F %%x IN ('tasklist /NH /FI "IMAGENAME eq %EXE%"') DO IF %%x == %EXE% goto FOUND
echo Now, running...
cd .\nginx\
start "" ".\%EXE%"
goto FINISH
:FOUND
echo Already running
:FINISH

rem timeout /t 1 /nobreak > NUL

start "" http://127.0.0.1:8081/

最终查看效果

 


​