---
layout:     post
title:      使用 Docker 搭建 Alist 网盘挂载
subtitle:   使用 Docker 搭建 Alist 网盘挂载
date:       2024-05-11
author:     sundys
header-img: img/ns-sj.png
catalog: true
tags:
    - N1盒子
    - Docker
    - Alist
---



[Alist](https://alist.nn.ci/zh/) 一个支持多种存储的文件列表程序，使用 Gin 和 Solidjs。

## 1、安装

<table><tbody><tr><td><pre><span>1</span><br></pre></td><td><pre><span>docker run -d --restart=unless-stopped -v /etc/alist:/opt/alist/data -p 5244:5244 -e PUID=0 -e PGID=0 -e UMASK=022 --name="alist" xhofe/alist:latest</span><br></pre></td></tr></tbody></table>

## 2、生成管理员密码

3.25.0以上版本将密码改成加密方式存储的hash值，无法直接反算出密码，如果忘记了密码只能通过重新 `随机生成` 或者 `手动设置`

<table><tbody><tr><td><pre><span>1</span><br><span>2</span><br><span>3</span><br><span>4</span><br></pre></td><td><pre><span># 随机生成一个密码</span><br><span>docker exec -it alist ./alist admin random</span><br><span># 手动设置一个密码,`NEW_PASSWORD`是指你需要设置的密码</span><br><span>docker exec -it alist ./alist admin set NEW_PASSWORD</span><br></pre></td></tr></tbody></table>

## 3、配置 `Nginx`

## 3.1、生成证书

<table><tbody><tr><td><pre><span>1</span><br></pre></td><td><pre><span>certbot certonly --dns-cloudflare --dns-cloudflare-credentials ~/.secrets/certbot/cloudflare.ini -d alist.youhost.com --email 'email@youhost.com'</span><br></pre></td></tr></tbody></table>

## 3.2、将域名解析到服务器，利用 `Nginx` 代理请求 `127.0.0.1:5244`

<table><tbody><tr><td><pre><span>1</span><br><span>2</span><br><span>3</span><br><span>4</span><br><span>5</span><br><span>6</span><br><span>7</span><br><span>8</span><br><span>9</span><br><span>10</span><br><span>11</span><br><span>12</span><br><span>13</span><br><span>14</span><br><span>15</span><br><span>16</span><br><span>17</span><br><span>18</span><br><span>19</span><br><span>20</span><br><span>21</span><br><span>22</span><br><span>23</span><br><span>24</span><br><span>25</span><br><span>26</span><br><span>27</span><br><span>28</span><br><span>29</span><br><span>30</span><br><span>31</span><br><span>32</span><br><span>33</span><br><span>34</span><br><span>35</span><br><span>36</span><br><span>37</span><br><span>38</span><br><span>39</span><br><span>40</span><br><span>41</span><br><span>42</span><br><span>43</span><br><span>44</span><br><span>45</span><br><span>46</span><br><span>47</span><br><span>48</span><br><span>49</span><br><span>50</span><br><span>51</span><br><span>52</span><br><span>53</span><br><span>54</span><br><span>55</span><br><span>56</span><br><span>57</span><br><span>58</span><br><span>59</span><br><span>60</span><br><span>61</span><br><span>62</span><br><span>63</span><br><span>64</span><br><span>65</span><br><span>66</span><br><span>67</span><br><span>68</span><br><span>69</span><br><span>70</span><br><span>71</span><br><span>72</span><br><span>73</span><br><span>74</span><br><span>75</span><br><span>76</span><br><span>77</span><br><span>78</span><br><span>79</span><br><span>80</span><br><span>81</span><br><span>82</span><br><span>83</span><br><span>84</span><br><span>85</span><br><span>86</span><br><span>87</span><br><span>88</span><br><span>89</span><br></pre></td><td><pre><span>upstream alist {</span><br><span>  server 127.0.0.1:5244;</span><br><span>}</span><br><span></span><br><span>#HTTP 重定向到 HTTPS</span><br><span></span><br><span>server {</span><br><span>    listen 80;</span><br><span>    listen [::]:80;</span><br><span>    server_name alist.youhost.com;</span><br><span></span><br><span>    if ($host = alist.youhost.com) {</span><br><span>        return 301 https://$host$request_uri;</span><br><span>    }</span><br><span>    return 404;</span><br><span>}</span><br><span></span><br><span>server {</span><br><span></span><br><span>    listen 443 ssl http2;</span><br><span>    listen [::]:443 ssl http2;</span><br><span></span><br><span>    server_name alist.youhost.com; #你的域名，已经解析到服务器ip</span><br><span></span><br><span>  #HTTP 重定向到 HTTPS</span><br><span>    # if ($ssl_protocol = "") { return 301 https://$host$request_uri; }</span><br><span></span><br><span>  #判断 $host 如果不是设置好的域名，就返回403页面，可以禁止 IP 访问 Web。</span><br><span>    if ($host !~ (alist.youhost.com)$){</span><br><span>      return 403;</span><br><span>   }</span><br><span></span><br><span>  # IP 访问跳转至域名，将域名替换成自己的。就是判断 $host 如果不是域名结尾的，就重定向至该域名。</span><br><span>  #   if ($host !~ (youhost.com)$){</span><br><span>  #     rewrite ^ https://youhost.com$request_uri?;</span><br><span>  #  }</span><br><span></span><br><span>  # IP 访问跳转至域名，将 IP 和域名替换成自己的。判断是 IP 的，就重定向。</span><br><span>  #   if ($host ~ 192.168.1.1){</span><br><span>  #   rewrite ^ https://www.yourdomain.com$request_uri?;</span><br><span>  #  }</span><br><span></span><br><span></span><br><span>  #证书位置</span><br><span>    ssl_certificate      /etc/letsencrypt/live/alist.youhost.com/fullchain.pem; </span><br><span>    ssl_certificate_key  /etc/letsencrypt/live/alist.youhost.com/privkey.pem;</span><br><span></span><br><span>  #证书配置</span><br><span>    ssl_protocols TLSv1.1 TLSv1.2 TLSv1.3;  #安全链接可选的加密协议</span><br><span>    ssl_ciphers TLS13-AES-256-GCM-SHA384:TLS13-CHACHA20-POLY1305-SHA256:TLS13-AES-128-GCM-SHA256:TLS13-AES-128-CCM-8-SHA256:TLS13-AES-128-CCM-SHA256:EECDH+CHACHA20:EECDH+AES128:RSA+AES128:EECDH+AES256:RSA+AES256:EECDH+3DES:RSA+3DES:!MD5;  #加密算法</span><br><span>    ssl_prefer_server_ciphers on;  #表示优先使用服务端加密套件。默认开启</span><br><span>    ssl_session_timeout 10m;  #缓存有效期</span><br><span>    ssl_session_cache builtin:1000 shared:SSL:10m;</span><br><span>    ssl_buffer_size 1400;</span><br><span>    add_header Strict-Transport-Security max-age=15768000;</span><br><span>    ssl_stapling on;</span><br><span>    ssl_stapling_verify on;</span><br><span></span><br><span>    client_max_body_size 525m;</span><br><span></span><br><span>  #Nginx 默认的 client_max_body_size 配置大小为 1m，可能会导致你在 Halo 后台上传文件被 Nginx 限制，</span><br><span>  #所以此示例配置文件加上了 client_max_body_size 1024m; 这行配置。当然，1024m 可根据你的需要自行修改</span><br><span></span><br><span>  #日志</span><br><span>    access_log   /var/log/nginx/nginx.alist.access.log;</span><br><span>    error_log    /var/log/nginx/nginx.alist.error.log;</span><br><span></span><br><span>  #反代设置开始#</span><br><span>  location / {</span><br><span>    proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;</span><br><span>    proxy_set_header X-Forwarded-Proto $scheme;</span><br><span>    proxy_set_header Host $http_host;</span><br><span>    proxy_set_header X-Real-IP $remote_addr;</span><br><span>    proxy_set_header Range $http_range;</span><br><span>  proxy_set_header If-Range $http_if_range;</span><br><span>    proxy_redirect off;</span><br><span>    proxy_pass http://alist;</span><br><span>    # the max size of file to upload</span><br><span>    client_max_body_size 20000m;</span><br><span>  }</span><br><span>  #反代设置结束#</span><br><span></span><br><span>  #禁止访问的文件或目录</span><br><span>    location ~ /(\.user\.ini|\.ht|\.git|\.svn|\.project|LICENSE|README\.md) {</span><br><span>      deny all;</span><br><span>    }</span><br><span>  #禁止访问的文件或目录</span><br><span></span><br><span>}</span><br></pre></td></tr></tbody></table>

## 3.3、验证并重载 `nginx` 配置

<table><tbody><tr><td><pre><span>1</span><br></pre></td><td><pre><span>nginx -t</span><br></pre></td></tr></tbody></table>

<table><tbody><tr><td><pre><span>1</span><br></pre></td><td><pre><span>nginx -s reload</span><br></pre></td></tr></tbody></table>

## 4、更新

## 4.1、停止并移除旧容器

<table><tbody><tr><td><pre><span>1</span><br></pre></td><td><pre><span>docker stop alist &amp;&amp; docker rm alist</span><br></pre></td></tr></tbody></table>

## 4.2、拉取最新镜像

<table><tbody><tr><td><pre><span>1</span><br></pre></td><td><pre><span>docker pull xhofe/alist:latest</span><br></pre></td></tr></tbody></table>

## 4.3、重新安装

<table><tbody><tr><td><pre><span>1</span><br></pre></td><td><pre><span>docker run -d --restart=unless-stopped -v /etc/alist:/opt/alist/data -p 5244:5244 -e PUID=0 -e PGID=0 -e UMASK=022 --name="alist" xhofe/alist:latest</span><br></pre></td></tr></tbody></table>

## 5、设置

对照 [官方文档](https://alist.nn.ci/zh/) 进行设置。

___