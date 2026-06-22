---
title: "WSL Ubuntu에서 GUI 이용하기: XFCE + XRDP"
date: 2026-06-23
categories: [Development Environment]
tags: [WSL, XRDP, Ubuntu]
---

이 포스트에서는 윈도우에서 설치한 WSL 우분투를 GUI로 볼 수 있도록 설정하는 방법을 다룬다.  

실습 환경은 윈도우 11에 WSL로 우분투 26.04 LTS에서 진행했다.  


## 1. 접속 및 업데이트 진행
리눅스 키면 가장 기본적으로 하는 커맨드
```sh
wsl -d Ubuntu
sudo apt update
```

## 2. systemd 활성화
wsl 터미널에서 다음과 같이 입력해 systemd를 활성화한다.

```sh
sudo tee /etc/wsl.conf << 'EOF'
[boot]
systemd=true
EOF
```

입력하면 재부팅

```sh
sudo reboot
```

다음 입력해서 `systemd`가 출력으로 나오는지 확인
```sh
ps -p 1 -o comm=
```

## 3. XFCE와 XRDP 설치
다음 명령어로 설치해준다.

```sh
sudo apt install -y xfce4 xfce4-goodies xfce4-session xrdp xorgxrdp dbus-x11
```

다음 명령어로 xrdp와 관련된 인증서 권한 문제를 해소한다.
```sh
sudo adduser xrdp ssl-cert
```

## 4. XRDP 포트 변경(선택)
WSL의 GUI 접근은 원격 데스크탑 접속으로 이뤄지는데, 이 RDP의 기본 포트인 3389와 겹칠 수 있으므로 다른 포트로 바꾸는 것이 좋다. 이 포스트에서는 3390으로 바꾼다.  

다음과 같이 입력해 포트를 바꾼다.

```sh
sudo cp /etc/xrdp/xrdp.ini /etc/xrdp/xrdp.ini.bak
sudo sed -i 's/^port=.*/port=3390/' /etc/xrdp/xrdp.ini
```

다음 명령어를 이용해 포트가 3390으로 나오는지 확인
```sh
grep -n "^port=" /etc/xrdp/xrdp.ini
```

## 5. XRDP 로그인 시 XFCE가 뜨도록 설정
사용자 홈에 `.xsession` 파일 생성

```sh
echo "startxfce4" > ~/.xsession
chmod +x ~/.xsession
```

xrdp의 세션 시작 스크립트를 XFCE용으로 정리
```sh
sudo cp /etc/xrdp/startwm.sh /etc/xrdp/startwm.sh.bak
sudo tee /etc/xrdp/startwm.sh > /dev/null << 'EOF'
#!/bin/sh

unset DBUS_SESSION_BUS_ADDRESS
unset XDG_RUNTIME_DIR
unset WAYLAND_DISPLAY

if [ -r /etc/profile ]; then
  . /etc/profile
fi

exec startxfce4
EOF

sudo chmod +x /etc/xrdp/startwm.sh
```

## 6. XRDP 서비스 시작 및 자동 실행 등록
xrdp를 서비스에 등록해 자동실행될 수 있도록 한다.

```sh
sudo systemctl enable --now xrdp
sudo systemctl restart xrdp
```

정상적으로 진행되고 있다면 `systemd-sysv-install enable xrdp` 가 중간 출력으로 나온다. 이는 환경에 따라 나오지 않을 수도 있다.

다음 명령어로 xrdp가 running인지 확인. `active (running)`가 보이면 된다.

```sh
systemctl status xrdp --no-pager
```

마지막으로 다음 명령어를 통해 포트가 본인이 설정한 포트로 나오는지 확인. 아래 예시에서는 3390으로 했다.

```sh
ss -ltnp | grep 3390
```

## 7. 접속하기
윈도우에서 원격 데스크톱 연결을 실행한다.

![](/assets/img/posts/2026-06-23-wsl-gui-setup/1.png)

localhost:<포트번호> 지정하고 접속 시도

![](/assets/img/posts/2026-06-23-wsl-gui-setup/2.png)

xrdp 서비스가 실행되고 있다면 다음 경고가 나온다. \[예\]를 눌러서 진행하자.  

![](/assets/img/posts/2026-06-23-wsl-gui-setup/3.png)

그러면 다음과 같이 xrdp 로그인 창이 나온다. WSL username과 비밀번호를 입력 후 OK를 누른다.

![](/assets/img/posts/2026-06-23-wsl-gui-setup/4.png)

그런 다음 잠시 기다리면 GUI가 로드되며 필요한 작업을 진행할 수 있다.

![](/assets/img/posts/2026-06-23-wsl-gui-setup/5.png)

## 참고
XRDP 설정을 해줘도 접속하기 위해선 WSL이 부팅되어있어야 한다.  
윈도우 터미널에서 다음 명령어로 부팅 여부를 확인할 수 있다.  
```sh
wsl -l -v
```