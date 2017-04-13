1. 安装, 配置
参考 https://trex-tgn.cisco.com/trex/doc/trex_manual.html#_obtaining_the_trex_package

```
$ yum install -y pciutils
$ wget --no-cache http://trex-tgn.cisco.com/trex/release/latest
$ tar xf latest
$ cd trex-v2.23/
$ sudo ./dpdk_setup_ports.py -s  查看网卡pci(需要至少两块网卡)
$ cp cfg/simple_cfg.yaml /etc/trex_cfg.yaml
$ cat /etc/trex_cfg.yaml
- port_limit : 2
  version : 2
  #List of interfaces. Change to suit your setup. Use ./dpdk_setup_ports.py -s to see available options
  interfaces : ["03:00.0","03:00.1"]
  port_info :  # Port IPs. Change to suit your needs. In case of loopback, you can leave as is.
  - ip : 1.1.1.1  # trex server网卡IP
    default_gw : 2.2.2.2  # 被测机器IP, trex在发包前会arp获取mac
  - ip : 2.2.2.2
    default_gw : 1.1.1.1
```

2. 启动trex server(stateless mode)

```
$ cd trex-v2.23/
$ ./t-re-x -i
```

3. 检测链路

```
$ cd trex-v2.23/
$ ./trex-console  # 进入trex console
$ trex> service  # 进入service模式
$ trex> ping  # ping trex server
$ trex> arp  # arp被测机器
$ trex> service --off  # 退出service模式
```

4. 准备报文

trex使用scapy脚本导入,修改原始报文
```
$ cd trex-v2.23/
$ ls frag  # frag.pcap是原始报文, frag.py是scapy脚本(在trex-v2.23/stl目录下有很多不同测试场景的样例)
frag.pcap  frag.py
$ cat frag/frag.py
# -*- coding: utf-8
import os
from trex_stl_lib.api import *

# PCAP profile
class STLPcap(object):

    def __init__ (self, pcap_file):
        self.pcap_file = pcap_file

    def get_streams (self, direction = 0, ipg_usec = 10.0, loop_count = 1, **kwargs):  # loop_count可指定发包个数

        profile = STLProfile.load_pcap(self.pcap_file, ipg_usec = ipg_usec, loop_count = loop_count)

        return profile.get_streams()



# dynamic load - used for trex console or simulator
def register():
    # get file relative to profile dir
    return STLPcap(os.path.join(os.path.dirname(__file__), 'frag.pcap'))  # frag.pcap与frag.py在同一个目录

$ ./stl-sim -f frag/frag.py --pkt  # 查看报文内容
```

5. 发包

```
$ cd trex-v2.23/
$ ./trex-console  # 进入trex console
$ trex> start -f frag/frag.py --port 0  # 从port0发包
```
