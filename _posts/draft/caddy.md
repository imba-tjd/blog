## cli

* 运行：caddy run [--watch]。非阻塞：start stop
* 重载配置：caddy reload
* 服务：Debian安装的支持systemctl，配置在/etc/caddy/Caddyfile。Win支持sc创建
  * 日志：journalctl -u caddy --no-pager | less +G
* 静态文件命令：caddy file-server --root ~/mysite --browse --domain example.com --listen :2015
* 反代命令：caddy reverse-proxy --from example.com --to localhost:9000
  * 默认转发所有Header，包括Host。可用--change-host-header更改Host
  * from默认为localhost。可设为:80

## Caddyfile

* 默认使用CWD下的。手动指定：--config Caddyfile
* 转换为json配置：caddy adapt --config Caddyfile
* 格式化：caddy fmt --overwrite
* VSC：Caddyfile Support

```
:2015

respond "Hello, world!"

:8080, :8081 { }

(snippet) { {args.0} }
import snippet ARG

@namedmatcher { 之后用在Matcher部分。若只有一个条件可以不加大括号
	method POST
	path /foo/* 还有path_regexp
    header Connection *Upgrade* 可多次指定。还有header_regexp
    query k=v
    remote_ip
}
@404-410 expression `{err.status_code} in [404, 410]`
@5xx expression `{err.status_code} >= 500 && {err.status_code} < 600`
expression {method}.startsWith("P")

@denied not remote_ip private_ranges
abort @denied
```

### 指令

* file_server 静态文件
* encode gzip 只会压缩文本。还支持zstd但没有浏览器支持
* root * /home/me/mysite
* reverse_proxy /api/* 127.0.0.1:9005
* header k v 操纵响应头：覆盖 增加(+因为某些头可以多次出现) 删除(-) 不存在则添加(?) 替换

#### 全局设定

* log：设定级别、输出到文件(包括设定rotated参数)、输出到socket。默认不会记录敏感信息头
* strict_sni_host on
* protocols 默认h1 h2 h3
* trusted_proxies static private_ranges
* timeouts 可设定四个阶段的超时时间

#### 操纵路径

* redir https://example.com{uri} 默认302
* rewrite /add /add/
* uri strip_prefix /app
* handle、handle_path /app/* {子指令} 类似于nginx的location，同一级可以有多个，按最长(具体)匹配。_path版相当于自带uri strip_prefix
* route {子指令} 处理HTTP handler chain，里面的内容会按书写顺序处理
* try_files {path} /index.html 用于SPA
* handle_errors { rewrite * /{err.status_code}.html file_server }

### 变量

* 环境变量：{$SITE_ADDRESS}
* host、hostport、method
* uri 是path加query
* labels.n 如对于example.com，则0是com，1是example
* remote_host、remote_port
* header.*

## 非常见指令

* template
* storage：支持s3 pg mysql redis
* metrics：要配合Prometheus
* php_fastcgi unix//run/php/php-version-fpm.sock

## 非内置模块

* cache-handler

## admin API

* 读取：localhost:2019/config/
* 设置：curl localhost:2019/load --json -d @caddy.json
* 修改了会保存，下次用--resume加载

## 自动HTTPS

* localhost会自签证书
* 自动Let's Encrypt：设置的域名能解析到本机IP，开放80和443。证书会放在$HOME里，Win下是%AppData%\Caddy
* 自动80转443
* 阻止：在地址前加http://、仅监听80、auto_https off

## 其它

* listener_address：也支持tcp ip unix。默认会bind所有IF和双栈，只在Caddy内部再按地址处理
