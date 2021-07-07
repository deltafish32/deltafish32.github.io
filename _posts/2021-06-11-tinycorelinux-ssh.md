---
title: TinyCoreLinux 로 SSH 서버 만들기
categories:
- Server
tags:
- TinyCoreLinux
- Linux
- SSH
- VNC
- RDP
---

# 개요
초경량 리눅스인 TinyCoreLinux 에 SSH 터널링 서버를 만드는 방법에 대한 자료를 정리한 문서입니다.



# 용도
보안되지 않는 서비스(주로 VNC)에 암호화 통신을 하도록 하거나, 방화벽 내부의 PC 에 원격 접속을 하기 위한 용도(RDP)로 사용할 수 있습니다.



# O/S 설치

## 다운로드
<http://www.tinycorelinux.net/downloads.html>

가장 단순한 Core, 여기에 GUI 를 포함한 TinyCore, 그리고 이것 저것 설치된 CorePlus 가 있습니다. 용도에 맞게 선택하면 됩니다.

Core 를 기준으로 설명합니다.


## 가상머신 설정
가상머신을 이용하는 경우, 12.0 기준 커널 5.10.3 버전에 맞는 설정을 선택하면 됩니다.

![tinycorelinux_vm1](https://user-images.githubusercontent.com/28891204/121616651-3eab7680-ca9e-11eb-8884-7ccf1eb14076.png)

코어 개수, 메모리 등 목적에 맞게 선택하면 됩니다.

![tinycorelinux_vm2](https://user-images.githubusercontent.com/28891204/121616685-4e2abf80-ca9e-11eb-8e53-0ffc2b7b0e22.png)

하드 디스크는 SCSI 가 아닌 **SATA 를 선택해야 정상 설치됩니다.** 그러나 VMware 에서는 가상머신 생성시 하드 디스크 설정이 제한되어 있습니다. 생성 후 바로 부팅하지 말고, Edit virtual machine settings 를 눌러 설정합니다.

![tinycorelinux_vm3](https://user-images.githubusercontent.com/28891204/121616683-4d922900-ca9e-11eb-8314-87894d6f1f75.png)

또한 SSH 서버로 활용하기 위해서는 Network Adapter 를 NAT 가 아닌 Bridged 로 설정하는 것이 좋습니다.

![tinycorelinux_vm4](https://user-images.githubusercontent.com/28891204/121616680-4cf99280-ca9e-11eb-8f20-16bef46fe54e.png)



## 부팅
다운받은 iso 파일을 통해 부팅합니다.
정상적으로 부팅 되었다면 아래와 같이 나올 것입니다.

``` bash
tc@box:~$
```

![tinycorelinux_boot](https://user-images.githubusercontent.com/28891204/121616678-4c60fc00-ca9e-11eb-9e52-d2746c5d1d42.png)

Windows PE 처럼 임시 환경으로 부팅되며, 여기서 작업한 모든 내용은 사라지게 됩니다. 일반적인 사용을 위해 설치를 해주어야 합니다.


## 설치
명령줄에 입력합니다.

``` bash
tce-load -wi tc-install
sudo tc-install.sh
```

설치 옵션이 많이 나옵니다. 아래 설정을 참고해주십시오.

``` bash
Install from [R]unning OS, from booted [C]drom, from [I]so file, or from [N]et.
(r/c/i/n): 
> c

Select Install type [F]rugal, [H]DD, [Z]ip. (f/h/z):
> f

Select Target for Installation of core

         1. Whole Disk
         2. Partition
		 
Enter selection ( 1 - 2 ) or (q)uit:
> 1

Would you like to install a bootloader?
...
Enter selection ( y, n ) or (q)uit:
> y

Install Extensions from this TCE/CDE Directory:
> 

Select Formatting Option for sda

         1. ext2
         2. ext3
         3. ext4
         4. vfat
		 
Enter selection ( 1 - 4 ) or (q)uit:
> 3

Enter space seperated boot options:
> 

Last chance to exit before destroying all data on sda
Continue (y/..)?
> y

Installation has completed
Press Enter key to continue.
> 

```

![tinycorelinux_done](https://user-images.githubusercontent.com/28891204/121616674-4bc86580-ca9e-11eb-8526-444fcf66d2bf.png)

설치가 완료되었습니다. 재부팅하면 설치된 O/S 로 부팅됩니다.

``` bash
sudo reboot
```



# SSH 설정
SSH 서버가 내장되어 있지 않으므로 따로 설치해야 합니다.

``` bash
tce-load -wi openssh
```

설정 기본값을 복사합니다.

``` bash
cd /usr/local/etc/ssh
sudo cp sshd_config.orig sshd_config
```

SSH 터널링을 위해 아래 설정을 변경해야 합니다. vi 혹은 nano 로 편집해줍니다.

``` bash
sudo vi sshd_config

...

AllowTcpForwarding yes
GatewayPorts yes
TCPKeepAlive yes
```

SSH 서비스를 시작합니다.

``` bash
sudo /usr/local/etc/init.d/openssh start
```

계정 암호를 설정합니다.

``` bash
passwd
```

재부팅시에 서비스를 시작하도록 설정합니다.
이상하게도, sudo echo 로는 정상적으로 추가되지 않습니다.
vi 혹은 nano 를 이용하여 맨 아랫줄에 추가해줍니다.

``` bash
sudo vi /opt/bootlocal.sh

...

/usr/local/etc/init.d/openssh start &
```

아래의 백업 설정을 반드시 해야, 재부팅 시에도 사용 가능합니다.



# 계정 생성
기본 계정인 tc 를 사용해도 되지만 이 계정을 통해 설정 변경이 가능하므로, 단순히 SSH 터널링을 위한 계정을 새로 만들어서 사용하는 것이 좋습니다.
여기서 계정명은 tunnel 로 했으며, 원하는 이름을 사용하시면 됩니다.

``` bash
sudo adduser tunnel
```



# 백업 설정
지정하지 않은 파일들은 재부팅시 삭제됩니다. 필요한 파일들은 따로 지정해주어야 합니다.

백업할 파일들을 지정합니다.

``` bash
sudo echo 'usr/local/etc/ssh' >> /opt/.filetool.lst
sudo echo 'etc' >> /opt/.filetool.lst
```

백업합니다.

``` bash
sudo filetool.sh -b
```

수정한 파일을 유지하고 싶을 때마다 실행해야 합니다.

- /opt/.filetool.lst 파일에 기록시 맨 앞의 / 는 생략해야 경고가 발생하지 않습니다.
- /etc 에 신규 계정, 비밀번호, 그룹 등의 설정이 저장되므로 추가했습니다.



# 해결되지 않은 문제
연결이 불완전하게 끊긴 경우, 다음 연결시 터널링에 실패하는 문제가 있습니다.
서버를 재부팅하면 해결이 됩니다. 재부팅하지 않고 해결할 수 있는 방법을 찾는 중입니다.



# 출처
- TinyCoreLinux - <https://iotbytes.wordpress.com/configure-ssh-server-on-microcore-tiny-linux/>
- SSH - <https://sando0.tistory.com/92>
