---
layout: post
title: JetSon Nano
date: 2023-11-17 00:00:00 +0900
description: # Add post description (optional)
img: jetson-nano.jpg 
tags: [Jetson Nano, Setting] # add tag
---

![JetSon Nano]({{site.baseurl}}/assets/img/jetson/jetson_nano.jpg)

# 1. JetSon Nano 환경 설정

## 1.1. [JetSon Nano Developer Kit SD Card Image 설치](https://developer.nvidia.com/embedded/l4t/r32_release_v6.1/jeston_nano/jetson-nano-jp46-sd-card-image.zip)(4GB)

## 1.2. [balenaEtcher](https://etcher.balena.io/#download-etcher) 설치, 실행하여 SD Card 포맷

## 1.3. JetSon Nano에 SD Card를 넣고 실행, Ubuntu OS 설치

## 1.4. Desktop Sharing

1. Window에서 VNC를 통해 JetSon Nano를 실행키시기 위해                                              /usr/share/glib-2.0/schemas/org.gnome.Vino.gschema.xml 파일 마지막에 아래 코드를 추가

```xml
<key name=’enabled’ type=’b’> 
	<summary>Enable remote access to the desktop</summary> 
	<description> If true, allows remote access to the desktop via the RFB protocol. 
	Users on remote machines may then connect to the desktop using a VNC viewer. 
	</description> 
	<default>true</default> 
</key>
```

2. Setting에서 desktop_sharing에서 다음과 같이 설정
![desktop sharing]({{site.baseurl}}/assets/img/jetson/desktop_sharing.png)

3. Setting에서 wired_connection의 IPv4 Setting Method를 Shared to other computer로 설정
![wired connection]({{site.baseurl}}/assets/img/jetson/wired_connection.png)

4. Terminal에서 ifconfig 명령어로 eth0 주소 또는 wlan0 주소 확인 

## 1.5. Window 환경으로 돌아와 [RealVNC Viewer](https://www.realvnc.com/en/connect/download/viewer/) 설치

## 1.6. VNC Viewer를 실행시켜 JetSon Nano 연결 생성

- JetSon Nano OS에서 확인한 wifi 주소 또는 ethernet 주소로 연결

## 1.7. 모니터 해상도 설정

- VNC Viewer 에서 보이는 JetSon Nano OS의 해상도를 설정
- /etc/X11/xorg.conf 파일의 마지막에 아래의 코드를 입력
![screen]({{site.baseurl}}/assets/img/jetson/screen.png)
- Virtual 값으로 해상도 설정

---

# 2. VSCode에서 JetSon Nano OS 연결

## 1. Extension에서 Remote_ssh 설치

![remote ssh]({{site.baseurl}}/assets/img/jetson/remote_ssh.png)

## 2. Remote 설정

1. VSCode의 왼쪽 아래에 Open a Remote Window 버튼 클릭
![remote]({{site.baseurl}}/assets/img/jetson/remote.png)

2. Connect to Host... 클릭
![remote]({{site.baseurl}}/assets/img/jetson/remote2.png)

3. 새로운 Host를 추가하기 위해 Configure SSH Hosts... 클릭
![remote]({{site.baseurl}}/assets/img/jetson/remote3.png)

4. config 파일에서 아래의 정보 입력
```
Host "Host ip address"
HostName "Host name"
User "JetSon Nano OS user id"
```

5. 추가한 Host로 SSH 연결