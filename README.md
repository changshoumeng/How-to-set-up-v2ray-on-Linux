# How-to-set-up-v2ray-on-Linux
This guide covers the installation of V2Ray on Linux systems, including the necessary software dependencies, configuration steps, and tips for optimizing its performance. 

## 下载v2ray
wget https://github.com/v2fly/v2ray-core/releases/latest/download/v2ray-linux-64.zip
unzip v2ray-linux-64.zip

## 配置config.json
    {
      "log": {
        "loglevel": "warning"
      },
      "inbounds": [
        {
          "tag": "socks",
          "port": 10808,
          "listen": "127.0.0.1",
          "protocol": "socks",
          "sniffing": {
            "enabled": true,
            "destOverride": [
              "http",
              "tls"
            ],
            "routeOnly": false
          },
          "settings": {
            "auth": "noauth",
            "udp": true,
            "allowTransparent": false
          }
        },
        {
          "tag": "http",
          "port": 10809,
          "listen": "127.0.0.1",
          "protocol": "http",
          "sniffing": {
            "enabled": true,
            "destOverride": [
              "http",
              "tls"
            ],
            "routeOnly": false
          },
          "settings": {
            "auth": "noauth",
            "udp": true,
            "allowTransparent": false
          }
        }
      ],
      "outbounds": [
        {
          "tag": "proxy",
          "protocol": "trojan",
          "settings": {
            "servers": [
              {
                "address": "iplc-hk-beta1.trojanwheel.com",
                "method": "chacha20",
                "ota": false,
                "password": "",
                "port": 5001,
                "level": 1
              }
            ]
          },
          "streamSettings": {
            "network": "tcp",
            "security": "tls",
            "tlsSettings": {
              "allowInsecure": false
            }
          },
          "mux": {
            "enabled": false,
            "concurrency": -1
          }
        },
        {
          "tag": "direct",
          "protocol": "freedom",
          "settings": {}
        },
        {
          "tag": "block",
          "protocol": "blackhole",
          "settings": {
            "response": {
              "type": "http"
            }
          }
        }
      ],
      "dns": {
        "hosts": {
          "dns.google": "8.8.8.8",
          "proxy.example.com": "127.0.0.1"
        },
        "servers": [
          {
            "address": "223.5.5.5",
            "domains": [
              "geosite:cn"
            ],
            "expectIPs": [
              "geoip:cn"
            ]
          },
          "1.1.1.1",
          "8.8.8.8",
          "https://dns.google/dns-query",
          {
            "address": "223.5.5.5",
            "domains": [
              "iplc-hk-beta1.trojanwheel.com"
            ]
          }
        ]
      },
      "routing": {
        "domainStrategy": "AsIs",
        "rules": [
          {
            "type": "field",
            "inboundTag": [
              "api"
            ],
            "outboundTag": "api"
          },
          {
            "type": "field",
            "port": "443",
            "network": "udp",
            "outboundTag": "block"
          },
          {
            "type": "field",
            "outboundTag": "block",
            "domain": [
              "geosite:category-ads-all"
            ]
          },
          {
            "type": "field",
            "outboundTag": "direct",
            "ip": [
              "geoip:private"
            ]
          },
          {
            "type": "field",
            "outboundTag": "direct",
            "domain": [
              "geosite:private"
            ]
          },
          {
            "type": "field",
            "port": "0-65535",
            "outboundTag": "proxy"
          }
        ]
      }
    }
## 运行v2ray
简单执行命令 ./v2ray即可

V2Ray 4.31.0 (V2Fly, a community-driven edition of V2Ray.) Custom (go1.15.2 linux/amd64)
A unified platform for anti-censorship.
2024/12/02 21:28:06 Using default config:  /mnt/tazssd/fs/kxue/tazv2ray/config.json
2024/12/02 21:28:06 [Info] v2ray.com/core/common/platform/ctlcmd: <v2ctl message> 
v2ctl> Read config:  /mnt/tazssd/fs/kxue/tazv2ray/config.json
2024/12/02 21:28:06 [Warning] v2ray.com/core: V2Ray 4.31.0 started

## 测试v2ray是否有效

curl --socks5 127.0.0.1:10808 http://www.google.com
curl --socks5-hostname 127.0.0.1:10808 http://www.google.com
curl --socks4 127.0.0.1:10808 http://www.google.com
curl --socks4a 127.0.0.1:10808 http://www.google.com
curl --http1.1 --proxy http://127.0.0.1:10809 http://www.google.com
curl --http2 --proxy http://127.0.0.1:10809 http://www.google.com
curl --proxy 127.0.0.1:10809 --tlsv1.2 http://www.google.com
curl --proxy 127.0.0.1:10809 --insecure http://www.google.com
curl --socks5 127.0.0.1:10808 https://www.google.com
curl --proxy http://127.0.0.1:10809 --data "test=1" http://www.google.com
curl --proxy 127.0.0.1:10809 --user-agent "Mozilla/5.0" http://www.google.com

## 参考其他
https://github.com/2dust/v2rayN/releases
https://github.com/XTLS/Xray-core/releases

## 应用这个代理服务到Docker容器里去

/mnt/tazssd/fs$ sudo cat  /etc/docker/daemon.json
{
  "proxies": {
    "http-proxy": "http://127.0.0.1:10809",
    "https-proxy": "http://127.0.0.1:10809",
    "no-proxy": "*.local,.example.com,127.0.0.0/8"
  }
}

sudo systemctl daemon-reload
sudo systemctl restart docker

/mnt/tazssd/fs$ docker info | grep -i proxy
 HTTP Proxy: http://127.0.0.1:10809
 HTTPS Proxy: http://127.0.0.1:10809
 No Proxy: *.local,.example.com,127.0.0.0/8

/mnt/tazssd/fs$ sudo docker pull hello-world
Using default tag: latest
latest: Pulling from library/hello-world
c1ec31eb5944: Pull complete 
Digest: sha256:305243c734571da2d100c8c8b3c3167a098cab6049c9a5b066b6021a60fcb966
Status: Downloaded newer image for hello-world:latest
docker.io/library/hello-world:latest




