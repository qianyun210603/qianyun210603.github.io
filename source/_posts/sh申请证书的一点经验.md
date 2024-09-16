---
title: acme.sh申请SSL证书的一点经验
date: 2024-06-30T08:12:12+08:00
toc: true
tags: 
categories: 
- 其他
---

### acme.sh安装
```bash
curl https://get.acme.sh | sh -s email=business@kailinjt.com
alias  acme.sh='/root/.acme.sh/acme.sh'
```

### CA机构
- Let's Encrypt不晚于2024-06-29强制要求DNSSEC，否则报错`DNSSEC: DNSKEY Missing`。但鹅厂的DNSPod需要付费才能开启DNSSEC，故放弃。
  - 可以通过[VERISIGN&#x2122;//LABS](https://dnssec-debugger.verisignlabs.com/)查看域名的DNSSEC状态。
- 切换默认CA到ZeroSSL
    ```
    acme.sh --set-default-ca --server zerossl
    ```
- ZeroSSL需要登记下邮箱
    ```
    acme.sh --register-account -m qianyun6@sina.com --server zerossl
    ```

### 通过DNSPod认证获取证书
- 获取腾讯云SecretId和SecretKey
  
  参考[acme.sh 自动解析并申请证书](https://docs.dnspod.cn/dns/acme-sh/)

- 将获取的SecretId和SecretKey设为环境变量：
  ```
  export Tencent_SecretId="<Your SecretId>"
  export Tencent_SecretKey="<Your SecretKey>"
  ```

- 用acem.sh获取证书
  ```
  acme.sh --issue --dns dns_tencent -d <your domain>
  ```
