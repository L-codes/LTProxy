# LTPorxy

**LTProxy** 是一个 Linux 上 **Proxifier** 的透明代理链替代方案 (基于 `iptables + proxychains4 + ipt2socks` 实现)

> 还在为不断 `proxychains4 cmd` 而感到苦恼吗? :)

> 此工具仅限于安全研究和教学，用户承担因使用此工具而导致的所有法律和相关责任！ 作者不承担任何法律和相关责任！


## Version

0.0.3 - [版本修改日志](CHANGELOG.md)


## Features

* 支持代理链
* 支持规则路由
* 采用易读改的 yaml 格式单文件配置

## Dependencies
* [**proxychains-ng**] - https://github.com/rofl0r/proxychains-ng
* [**ipt2socks**] - https://github.com/zfl9/ipt2socks

## Usage
```ruby
0. 需安装好 proxychains4 和 ipt2socks 的环境
   $ command -v ruby proxychains4 ipt2socks  # 确保 shell 环境都能找到

1. 配置 ltproxy.yml 文件 (具体配置示例与语法参考下面的 "LTProxy Config File")
   配置文件会以下面顺序依次尝试读取配置文件
     './ltproxy.yml' -> '~/.ltproxy.yml' -> '/etc/ltproxy.yml'
   或启动时，通过 -f 参数指定配置文件
   # cp config_demo.yml /etc/ltproxy.yml 即可快速编辑开始

2. 启动 ltproxy 透明代理
  ./ltproxy start  # ltproxy 需要以 root 权限运行
                   # 并且初次运行会自动创建 ltproxy 用户给予 ipt2socks 使用

  # 如更新 ltproxy.yml 配置文件则需要重新启动，如下
  ./ltproxy restart # start 等价于 restart

3. 关闭 ltproxy 透明代理
  ./ltproxy stop
```

## LTProxy Config File

#### Format
- 规则和代理链的优先级自上往下
```yaml
# 语法格式
rules:
  - proxies:
      - <socks_type> <host> <port> [username] [password]
      - ...
    target:
      - <target> <ports>
      - ...

  - ...

# 字段解释
# proxies
    socks_type    代理的类型，目前支持 socks5/socks4/socks4a (最后一层代理目前仅支持 socks5)
    host          代理地址
    port          代理端口
    username      如需认证可添上
    password      如需认证可添上
    # proxy 一列，也可以直接用 direct 表示跳过不走透明代理链
# target
    target        需要透明代理的目标地址，支持格式：192.168.*.* 10.0.0.0/8 192.168.1.1 等
                  还支持别名 intranet extranet any，用于内网、外网和所有的选择
    ports         可选默认 any, 支持格式：22,8000-9000 等
```

#### Example
- 简单例子参考 `config_demo.yml` 文件
- 为了展示目前支持的所有功能，下面是一个复杂的例子 :)
- 建议每个项目目录下创建 `ltproxy.yml` 文件，即可保留使用的代理配置，并且 yaml 支持注释和标记引用等功能
```yaml
rules:
  # 本地 192.168.12.0/24 跳过不走透明代理
  - proxies:
      - direct
    target:
      - 192.168.12.*

  # 19.0.0.0/8、1.1.1.1、18.7.0.0/16、 172/168/10和内网网段的22/7000-9000端口走这个三层的代理链透明代理
  - proxies:
      - socks5 12.12.12.12 1080 twelve password
      - socks4 172.16.1.2 1088
      - socks5 19.128.1.2 8888 admin admin
    target:
      - 19.*.*.*
      - 1.1.1.1
      - intranet 22,7000-9000
      - 18.7.0.0/16

  # 其它外网的 IP 通过 socks5:21.21.21.21:1080 透明代理访问
  - proxies:
      - socks5 21.21.21.21 1080
    target:
      - extranet
```

## TODO

 * 使用 TPROXY 支持 UDP 透明代理

 * 考虑是否加入 uid/gid 等路由规则

> 目前太多任务，根据用户需求量调整优先级，TODO 的新功能设置了个闹钟，每 500 star 完成一个 TODO 任务! :)


## License

GPL 3.0
