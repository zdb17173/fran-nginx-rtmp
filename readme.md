

# 安装

## 安装基础依赖
- sudo yum install gcc\*
- sudo yum install pcre-devel.x86_64
- sudo yum install zlib-devel.x86_64
- sudo yum install openssl-devel.x86_64

## 修改nginx_mod_h264_streaming-2.2.7代码
注释掉nginx_mod_h264_streaming-2.2.7/src/ngx_http_streaming_module.c文件中if(r->zero_in_uri){return NGX_DECLINED;}代码


## 编译配置nginx

```
sudo ./configure --prefix=/usr/local/nginx --add-module=../nginx_mod_h264_streaming-2.2.7 --add-module=../nginx-rtmp-module-1.2.1 --with-http_flv_module --with-http_mp4_module --with-http_stub_status_module --with-http_ssl_module --with-stream
```

将nginx-1.10.1中objs/Makefile中的CFLAGS= -pipe -O -W -Wall -Wpointer-arith -Wno-unused-parameter -Werror -g -D_LARGEFILE_SOURCE -DBUILDING_NGINX中的-Werror去掉

sudo make 
sudo make install


## 修改nginx config


record_suffix -%d-%b-%y-%T.flv;  ->   mystream-24-Apr-13-18:23:38.flv   http://pubs.opengroup.org/onlinepubs/009695399/functions/strftime.html


```
#user  nobody;
worker_processes  1;

error_log logs/error.log;

events {
    worker_connections  1024;
}


rtmp {
    server {
        listen 1935;
        chunk_size 4000;
		ping 10s;
		ping_timeout 5s;

		# rtmp输入，转码后转发到hls中进行ts切片录制
		application in{
			live on;
			record all; #原始输入流保存为flv
			record_path /tmp/rtmp;
			record_suffix -%d-%b-%y-%T.flv;
            record_unique on;

			exec_push /usr/local/ffmpeg/bin/ffmpeg -i rtmp://127.0.0.1/$app/$name -crf 32 -preset superfast -acodec libfaac -ar 24000 -b:a 96k -vcodec libx264 -r 25 -b:v 256k -f flv rtmp://127.0.0.1/hls/$name;
		}

		# rtmp输入，ts切片录制
	    application hls{
	        live on;
	        record all;
	        record_path /tmp/rtmp;
	        record_unique on;
	        hls on;
	        hls_path /tmp/rtmp;
	        hls_cleanup off;

	    }

	    log_format new '$connection $remote_addr $app-$name $args $flashver $swfurl $tcurl $pageurl $command $bytes_sent $bytes_received $time_local $session_time; $session_readable_time';
	    access_log logs/rtmp_access.log new;
	    access_log logs/rtmp_access.log;
	}
}

http {

    server {
    	listen 8080;
    	location /hls {
            # Serve HLS fragments
            types {
                application/vnd.apple.mpegurl m3u8;
                video/mp2t ts;
            }
            alias /tmp/rtmp;
            add_header Cache-Control no-cache;
        }
    }

}
```

## nginx启动停止重载

- 启动 sudo /usr/local/nginx/sbin/nginx
- 启动 sudo /usr/local/nginx/sbin/nginx -c /usr/local/nginx/conf/nginx.conf
- 停止 sudo /usr/local/nginx/sbin/nginx -s stop
- 重载配置 sudo /usr/local/nginx/sbin/nginx -s reload


# 直播地址

推流地址
- rtmp://ip:1935/in/111
- rtmp://ip:1935/hls/111

监看地址
- rtmp://ip/hls/111

直播地址
- http://ip:8080/hls/111.m3u8


# 推流

ffmpeg推流 视频源 H264 + ACC
` ffmpeg -i pujingyouyixunzhang.mp4 -vcodec libx264 -acodec copy -r 30 -f flv -s 1280x720 rtmp://ip:1935/hls/111 `



# 参考

nginx-rtmp-module `https://github.com/arut/nginx-rtmp-module`
nginx-rtmp-module wiki `https://github.com/arut/nginx-rtmp-module/wiki/Directives#exec_record_done`