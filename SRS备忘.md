## rtmp推流到SRS，转成webrtc协议
配置文件 rtmp2rtc.conf
配置文件中 listen字段就是rtmp监听的网卡与端口，比如下面的配置，则可以使用这个推流地址：`rtmp://127.0.0.1:11997/live/livestream`
http_api.listen字段在webrtc拉流中影响拉流地址的端口，比如对应的拉流地址为：`webrtc://192.168.56.43:11985/live/livestream`
配置了http_server则会启动一个nginx服务，端口号就是http_server.listen配置的端口号
在webrtc中这个http_server主要用于部署测试流的player，比如http://192.168.56.95:8078/players/rtc_player.html

```
listen              0.0.0.0:11997;
max_connections     1000;
daemon              off;
srs_log_tank        console;

# 配置了http_server则会启动一个nginx服务，端口号就是http_server.listen配置的端口号
# 在webrtc中这个http_server主要用于部署测试流的player，比如http://192.168.56.95:8078/players/rtc_player.html
http_server {
    enabled         on;
    listen          8078;
    dir             ./objs/nginx/html;
}

# 这里配置的端口号对应的是webrtc拉流地址里面的端口号，比如webrtc://192.168.56.43:11985/live/livestream
http_api {
    enabled         on;
    listen          11985;
}
stats {
    network         0;
}
rtc_server {
    enabled on;
    listen 8000; # UDP port
    # @see https://ossrs.net/lts/zh-cn/docs/v4/doc/webrtc#config-candidate
    candidate $CANDIDATE;
}

vhost __defaultVhost__ {
    rtc {
        enabled     on;
        # @see https://ossrs.net/lts/zh-cn/docs/v4/doc/webrtc#rtmp-to-rtc
        rtmp_to_rtc on;
        # @see https://ossrs.net/lts/zh-cn/docs/v4/doc/webrtc#rtc-to-rtmp
        rtc_to_rtmp on;
    }
    http_remux {
        enabled     on;
        mount       [vhost]/[app]/[stream].flv;
    }
}
```

启动脚本 windows srs-rtc.bat
==配置中，变量CANDIDATE的设置很重要。==
```
for /f "tokens=2*" %%i in ('REG QUERY "HKEY_LOCAL_MACHINE\Software\Microsoft\Windows\CurrentVersion\App Paths\srs\ins_dir"') do set srs_home=%%j

echo %srs_home%

set CANDIDATE=127.0.0.1

for %%I in ("%srs_home%") do set srs_disk=%%~dI

cd %srs_home%
@%srs_disk%

objs\srs.exe -c conf\rtmp2rtc.conf
cmd
```