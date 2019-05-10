
# 安装ffmpeg

## 准备
```
yum install -y gcc-c++
yum install -y gcc
yum install -y m4

```

设置运行环境
```
vim ~/.bash_profile
PATH=$PATH:$HOME/bin:/usr/local/bin

sudo vim /etc/ld.so.conf 添加
/usr/local/bin/

# 修改后执行，否则修改不生效
sudo ldconfig
```


## 1. 安装autoconf
直接安装 yum install autoconf

编译安装
```
tar xvf autoconf-2.69.tar.xz
cd autoconf-2.69
./configure
make
make install
```

## 2. 安装automake

直接安装 yum install automake

编译安装
```
tar xvf automake-1.15.tar.xz
cd automake-1.15
./configure
make
make install
```

## 3. 安装yasm支持汇编优化（FFmpeg需要）
```
tar xvf yasm-1.3.0.tar.gz
cd yasm-1.3.0
./configure
make
make install
```

## 4. 安装MP3支持库LAME
cd lame-3.100
./configure
make
make install

## 5. 安装libtool（FAAC需要）
```
./configure
make
make install
yum install libtool
```

## 6. 安装fdk acc
```
cd mstorsjo-fdk-aac*
autoreconf -fiv
./configure --disable-shared --with-pic
#--with-pic 很重要，一定要带上，不然在编译FFmpeg时会报错
make
make install
```

## 7. 安装nasm
```
nasm-2.13.03.tar.gz
./configure
make
make install
```

## 8. 安装h264
```
./configure --enable-shared
make
make install
```

## 7. 编译ffmpeg
```
./configure --enable-version3 --enable-yasm --enable-libx264 --enable-gpl --enable-libfdk-aac --enable-nonfree --enable-pthreads
```


# 使用ffmpeg

## 视频转码
```
ffmpeg -i 74e36683-e375-4bb8-a26c-10b037dedb34.mp4 -vcodec libx264 -acodec libfdk_aac output.mp4
```
- -y即如果文件存在，直接覆盖写
- 设置视频分辨率 -s 320x180 
- -b:v 192000
- -minrate 192000
- -maxrate 192000
- -bufsize 192000
- 改变视频的帧率 -r 24
- -g 12
- -keyint_min 12
- -force_key_frames 1
-  设置视音频编码格式 -vcodec libx264
- -profile:v baseline 
- 设置视音频编码格式 -acodec libfdk_aac 
- -ac 2 
- 改变音频的采样率 -ar 44100 
- 改变视音频的码率 -b:a 64k 
- 设置输出格式 -f flv(mpegts/hls/mp4)
- 指定视音频不转码  -c copy  音频不转码 -c:a copy  视频不转码 –c:v copy
- 设置处理开始时间 -ss HH:MM:SS
- 设置处理结束时间 -to HH:MM:SS
- 设置关键帧间隔 -force_key_frames "expr:gte(t,n_forced*1)"


## 视频截取
```
ffmpeg –i input.flv –ss 00:10:00 –to 00:20:00 –c copy –f flv jiequ.flv –y
```


## 视频切片
```
ffmpeg -i fran.mp4 -c copy  -map 0 -f segment -segment_list playlist.m3u8 -segment_time 5 %04d.ts
```


