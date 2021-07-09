---
title: SSH 비밀번호 없이 접속하기
categories:
- Server
tags:
- Linux
- SSH
- VMware
- ESXi
- Synology
- RSA
---

# 개요
SSH 사용시 비밀번호 없이 접속 가능하도록 설정하는 방법을 설명한 문서입니다.

일반적으로 SSH 연결시 비밀번호를 입력하여 연결하는데, 이 비밀번호가 유출된다면 어떤 기기에서든 연결이 가능한 문제가 있습니다.

이 방법을 이용하면 접속하는 기기 또는 기기에 저장된 키가 탈취되지 않는 한 안전하게 사용할 수 있습니다.

또한 자동화 스크립트를 통해 원격 서버를 관리하는 경우에도 유용하게 쓸 수 있습니다.



# 방법

## 키 생성
접속할 클라이언트에서 RSA 키를 생성합니다.

``` bash
ssh-keygen -t rsa
```

기본값으로 생성해도 되지만, 향상된 보안을 위해 4096비트로 생성할 수도 있습니다.

``` bash
ssh-keygen -t rsa -b 4096
```

저장할 경로, passphrase 를 묻는 옵션이 나오지만, 필요하지 않은 경우 Enter 를 눌러 넘어가도 됩니다.


## 키 복사
공개키를 서버에 복사합니다.

``` bash
ssh-copy-id [USERNAME]@[SERVER_IP] -p [PORT]

# ex)
ssh-copy-id charlie@192.168.0.123 -p 22
```

최초 서버에 연결시 fingerprint 확인 메시지가 출력됩니다. 중간자 공격 방지를 위해 확인하는 절차로, 문제 없다면 yes 를 눌러 진행합니다.


## 접속 테스트

접속을 테스트해봅니다. 설정이 잘 되었다면 비밀번호를 묻는 과정 없이 연결됩니다.

``` bash
ssh [USERNAME]@[SERVER_IP] -p [PORT]

# ex)
ssh charlie@192.168.0.123 -p 22
```


## 암호 접속 방지
더 이상 암호를 입력하여 접속할 필요가 없다고 판단되는 경우, 암호 접속을 차단할 수 있습니다.

**반드시 위 설정이 잘 적용되었는지 접속 테스트 후 진행하십시오.** 더 이상 연결이 불가능해질 수 있습니다.

``` bash
sudo vi /etc/ssh/sshd_config
```

아래 설정을 변경합니다.

```
PasswordAuthentication no
```

설정을 적용합니다.

``` bash
sudo systemctl restart ssh
```



# O/S 별 설정 설정 주의사항

## ESXi
ESXi 6.7 기준입니다.
ssh-copy-id 를 통해 복사 후 파일을 옮겨줘야 합니다.
계정 이름을 포함한 폴더를 만든 후 옮겨줍니다.

``` bash
mkdir /etc/ssh/keys-[USERNAME]
mv /.ssh/authorized_keys /etc/ssh/keys-[USERNAME]/

# ex)
mkdir /etc/ssh/keys-charlie
mv /.ssh/authorized_keys /etc/ssh/keys-charlie/
```

/etc/ssh/keys-[USERNAME]/authorized_keys 파일에 권한이 부족하면 접속시 비밀번호를 계속 묻게 됩니다. 충분한 권한을 줍니다.

``` bash
chmod 644 /etc/ssh/keys-[USERNAME]/authorized_keys

# ex)
chmod 644 /etc/ssh/keys-charlie/authorized_keys
```


## Synology
DSM 6.1 기준입니다.
권한이 과다한 경우 오히려 접속시 비밀번호를 계속 묻게 됩니다.

``` bash
sudo chmod 755 ~
sudo chmod 700 ~/.ssh
sudo chmod 644 ~/.ssh/authorized_keys
```



# 출처
- <https://dreamholic.tistory.com/111>
- <https://dreamholic.tistory.com/112>
- Synology - <https://stamler.ca/enable-passwordless-ssh-on-synology-dsm6/>
