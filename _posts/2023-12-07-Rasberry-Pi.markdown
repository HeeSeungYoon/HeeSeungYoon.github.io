---
layout: post
title: Rasberry Pi
date: 2023-12-07 09:00:00 +0900
description: # Add post description (optional)
img: # Add image post (optional)
tags: [Rasberry Pi, Setting] # add tag
---

![Rasberry Pi]({{site.baseurl}}/assets/img/rasberrypi/rasberrypi.jpg)

# Rasberry Pi 환경 설정

## 1. Rasberry Pi OS Image 설치

- Download Link : [Raspberry Pi OS with desktop and recommended software(64-bit)](https://downloads.raspberrypi.com/raspios_full_arm64/images/raspios_full_arm64-2023-12-06/2023-12-05-raspios-bookworm-arm64-full.img.xz?_gl=1*17s5gg3*_ga*ODgxNTQ0NjE3LjE3MDE5MDg4Mzk.*_ga_22FD70LWDS*MTcwMTkwODgzOS4xLjEuMTcwMTkwODk0MC4wLjAuMA..)

## 2. RasberryPi OS Imager 설치

- Download Link : [RasberryPi OS Imager for Window](https://downloads.raspberrypi.org/imager/imager_latest.exe)

## 3. RasberryPi SD Card에 OS Image 설치

![OS Imager]({{site.baseurl}}/assets/img/rasberrypi/OSImager.png)

- Rasberry Pi Device : Rasberry Pi 4
- 운영체제 : 사용자 정의 사용 -> 설치한 OS Image file
- 저장소 선택 : SD Card

## 4. Rasberry Pi Configuration

- VNC로 연결하기 위한 설정
    - Rasberry Pi menu -> Preference -> Rasberry Pi Configuration Click!!
    - Inference -> SSH, VNC Check!!

    ![Connection]({{site.baseurl}}/assets/img/rasberrypi/connection.png)

    - Terminal -> ifconfig : ip 확인

## 5. VNC Connection

- VNC에 ip를 입력하여 연결 생성
- VNC 연결 후 Display 설정
    - ```shell
        sudo raspi-config 
        ```
    - Display Options -> VNC Resolution -> Select!!
        
        ![Display]({{site.baseurl}}/assets/img/rasberrypi/display-option.png)
-   Reboot

## 6. Enternet Connection

- ```shell
    sudo apt install xrdp
    ```

- Window에서 네트워크 연결
    - Wi-Fi 연결 우클릭 -> 공유 탭 -> 인터넷 연결 공유 허용 (이더넷)
    ![network share]({{site.baseurl}}/assets/img/rasberrypi/network.png)
    
    - 이더넷 속성에서 IPv4 속성을 확인해보면 Window에서 Rasberry Pi로 IP 주소를 할당해줌

- Rasberry Pi Reboot
    - Terminal에서 ifconfig 명령으로 이더넷 주소 확인

# 7. 한글 설치

- locale 설정
    
    ![korean]({{site.baseurl}}/assets/img/rasberrypi/korean.png)


- ibus 설치
    ```bash
    sudo apt install ibus
    sudo apt install ibus-hangul
    sudo reboot
    ```
    한글 변경: shift+space

