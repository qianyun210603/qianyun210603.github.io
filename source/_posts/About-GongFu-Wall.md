---
title: 关于减轻功夫墙（GFW）造成的不便的一些探讨
date: 2018-06-26 21:02:15
tags: GFW, gooreplacer, ss
---
### 使用gooreplacer重定向对Google的访问
　　众所周知，天朝无法访问任何Google的服务。但不幸很多外国网站，如码农必备的Stackoverflow等都会调用Google的API，这就导致了这些网站访问速度巨慢，功能不全等问题。在网上发现了有人开发的关于可以重定向Google API到国内镜像的插件，可以用于Firefox和Google Chrome上。特记录于此：
插件地址：https://github.com/jiacai2050/gooreplacer
如果只需要重定向Google的访问，可以复制如下Rules，保存为.gson格式后导入。
```gson
{        
    "createBy": "http://liujiacai.net/gooreplacer/",
    "createAt": "Tue Jan 10 2017 17:45:39 GMT+0000",
    "rules": {
        "ajax.googleapis.com": {
            "dstURL": "ajax.proxy.ustclug.org",
            "kind": "wildcard",
            "enable": true
        },
        "storage.googleapis.com": {
            "dstURL": "storage-googleapis.proxy.ustclug.org",
            "kind": "wildcard",
            "enable": true
        },
        "gerrit.googlesource.com": {
            "dstURL": "gerrit-googlesource.proxy.ustclug.org",
            "kind": "wildcard",
            "enable": true
        },
        "themes.googleusercontent.com": {
            "dstURL": "google-themes.proxy.ustclug.org",
            "kind": "wildcard",
            "enable": true
        },
        "platform.twitter.com/widgets.js": {
            "dstURL": "cdn.rawgit.com/jiacai2050/gooreplacer/gh-pages/proxy/widgets.js",
            "kind": "wildcard",
            "enable": true
        },
        "apis.google.com/js/api.js": {
            "dstURL": "cdn.rawgit.com/jiacai2050/gooreplacer/gh-pages/proxy/api.js",
            "kind": "wildcard",
            "enable": true
        },
        "apis.google.com/js/plusone.js": {
            "dstURL": "cdn.rawgit.com/jiacai2050/gooreplacer/gh-pages/proxy/plusone.js",
            "kind": "wildcard",
            "enable": true
        }
    }
}
```
目前该插件存在一点问题，就是配置是基于Cookies保存的，如果浏览器设置了隐私模式或清除了Cookies，配置会丢失。


### Use PAC list for ss
1. Install GenPAC (reference https://github.com/JinnLynn/GenPAC)
    ```bash
    pip install genpac
    ```
2. Download url list
    ```bash
    genpac --pac-proxy "SOCKS5 127.0.0.1:1080" --gfwlist-proxy="SOCKS5 127.0.0.1:1080" --output="mypac.pac"
    ```

3. Setup global proxy in system

![image](https://note.youdao.com/yws/public/resource/d7098784592b6d9090942f083ef9167b/xmlnote/WEBRESOURCEe496e589eb4c6a1a2889fb66a36ea602/530)