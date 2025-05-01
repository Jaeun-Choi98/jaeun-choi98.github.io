---
layout: post
title: 'Linux init'
tags: 'linux'
date: 2023-10-11 00:00:00
---

리눅스를 사용하면서 systemd에 대해 정리하고자 합니다. 추가적으로, SysVinit에서 Upstart, 그리고 systemd로 이어지는 변화 과정을 정리하고, systemd의 구조와 특징을 중심으로 내용을 구성하였습니다.
또한 로그인 시 실행되는 초기화 파일에 대해서도 함께 정리하였습니다.

<br>
<br>

### **1. SysVinit**

---

#### **개요**

전통적인 리눅스 초기화 시스템으로, 가장 오래된 init 시스템입니다. 시스템이 부팅되면 `/sbin/init`이 PID 1로 실행되며, `/etc/inittab` 파일에 정의된 런레벨에 따라 스크립트를 실행합니다.

#### **주요 구성 요소**

- `/etc/inittab`: 런레벨 및 실행할 스크립트 정의
- `/etc/rc.d/`, `/etc/init.d/`: 런레벨별 스크립트 디렉토리
- `/etc/rcX.d/`: 런레벨 X에 해당하는 스크립트 실행 순서 지정 (`S`, `K` 접두어 사용)

#### **특징**

- 순차 실행 (병렬 처리 불가)
- 스크립트 기반 단순 구조
- 의존성 관리 미흡
- 런레벨 기반 구조 (0~6)

#### **한계**

- 병렬 처리가 불가능하여 부팅 속도가 느림
- 의존성 기반 처리 미흡 → 특정 서비스 실패 시 전체 영향을 줄 수 있음
- 복구/진단 도구 부족

---

<br>
<br>

### **2. Upstart**

---

#### **개요**

Ubuntu(6.10~14.10) 등에서 사용되었던 이벤트 기반 init 시스템으로, `SysVinit`의 단점을 보완하기 위해 개발되었습니다.

#### **주요 구성 요소**

- `/etc/init/` 디렉토리 내 `.conf` 파일들
- 각 파일은 서비스 정의와 실행 조건(이벤트 기반)을 설정

#### **특징**

- **이벤트 기반 실행**: 장치 연결, 네트워크 활성화 등 특정 이벤트 발생 시 서비스 시작
- **병렬 서비스 실행** 지원
- 기존 SysVinit 스크립트도 호환 가능

#### **한계**

- 복잡한 이벤트 관리 → 디버깅 어려움
- systemd에 비해 의존성 관리나 유닛 시스템이 불완전

---

<br>
<br>

### **3. systemd (현대 리눅스의 표준)**

---

#### **개요**

현대 리눅스 배포판(예: RHEL7+, CentOS7+, Ubuntu 15.04+, Debian 8+)에서 기본으로 채택한 init 시스템입니다. 단순한 초기화 시스템을 넘어서 **서비스 관리자, 로깅, 타이머, 소켓 활성화 등 통합 관리**를 목표로 합니다.

<br>

#### **구성 요소 요약**

| 구성 요소   | 역할                                  |
| ----------- | ------------------------------------- |
| `systemd`   | PID 1 프로세스, 초기화 및 서비스 관리 |
| `journald`  | 로깅 시스템                           |
| `logind`    | 사용자 세션 및 로그인 관리            |
| `networkd`  | 네트워크 설정 (선택적 사용)           |
| `timedated` | 시간/타임존 관리                      |

---

<br>

#### **systemd의 주요 특징**

**병렬 서비스 실행**

- 유닛 간 의존성을 기반으로 가능한 서비스는 동시에 실행
- 빠른 부팅 가능

**유닛 단위의 서비스 관리**

- 서비스(daemon), 타이머, 마운트 등 모두 "유닛(Unit)"으로 관리

**정밀한 의존성 관리**

- `After=`, `Requires=`, `Wants=`, `Conflicts=` 등 키워드를 통한 실행 순서와 관계 명시 가능

**소켓 및 DBus 기반 서비스 시작**

- 서비스가 필요할 때만 실행되는 **지연 시작(On-Demand)** 구조 가능

**통합 로깅**

- `journald`를 통한 바이너리 로그 → `journalctl`로 확인

**런레벨 대체: Target**

- SysVinit의 런레벨 개념을 `target`이라는 유닛으로 대체
  - 예: `multi-user.target`, `graphical.target`, `rescue.target`

---

<br>

#### **주요 유닛(Unit) 유형**

| 유닛 종류  | 설명                                     |
| ---------- | ---------------------------------------- |
| `.service` | 데몬, 백그라운드 서비스                  |
| `.socket`  | 소켓 활성화 (소켓 요청 시 서비스 실행)   |
| `.target`  | 여러 유닛을 그룹화, 런레벨처럼 사용 가능 |
| `.mount`   | 파일 시스템 마운트                       |
| `.timer`   | 특정 시간에 서비스 실행                  |
| `.path`    | 파일/디렉토리 변화 감지                  |
| `.device`  | 디바이스 활성화 시 트리거                |

---

<br>

#### **유닛 파일 구조 예시**

```
ini
복사편집
[Unit]
Description=Example Service
After=network.target

[Service]
ExecStart=/usr/bin/example
Restart=on-failure
User=nobody
Group=nogroup

[Install]
WantedBy=multi-user.target

```

- `[Unit]`: 유닛 설명, 실행 순서 지정
- `[Service]`: 실행 명령, 재시작 정책, 사용자
- `[Install]`: 어떤 target에 포함될지 지정

---

<br>

#### **유닛 파일 경로**

| 위치                   | 설명                              |
| ---------------------- | --------------------------------- |
| `/lib/systemd/system/` | 배포 시 설치된 기본 유닛          |
| `/etc/systemd/system/` | 사용자 정의 유닛 (우선순위 높음)  |
| `/run/systemd/system/` | 실행 중 생성된 임시 유닛 (휘발성) |

---

<br>

#### **주요 명령어**

| 명령어                                | 설명                   |
| ------------------------------------- | ---------------------- |
| `systemctl start <unit>`              | 유닛 실행              |
| `systemctl stop <unit>`               | 유닛 중지              |
| `systemctl restart <unit>`            | 유닛 재시작            |
| `systemctl enable <unit>`             | 부팅 시 자동 시작 등록 |
| `systemctl disable <unit>`            | 자동 시작 해제         |
| `systemctl status <unit>`             | 유닛 상태 확인         |
| `systemctl list-units --type=service` | 활성 서비스 목록       |
| `journalctl -u <unit>`                | 특정 유닛 로그 확인    |

<br>
<br>

### **4. 로그인 및 초기화 파일**

---

#### **로그인 시 실행되는 파일**

사용자가 로그인할 때 실행되는 초기화 파일은 다음과 같습니다.

1. `/etc/profile` : 시스템 전역 환경 변수 설정
2. `/etc/profile.d/*.sh` : 추가적인 환경 설정 스크립트 실행
3. `~/.profile` : 사용자의 로그인 셸 환경 설정
4. `~/.bash_profile` : 로그인 셸 환경 설정 (Bash로 로그인 될 때 실행)
5. `~/.bashrc` : 비로그인 셸 실행 시 환경 설정(Bash만)
6. `/etc/bashrc` : 전역적인 Bash 설정

기본적으로 `/etc/profile`과 `~/.profile`은 로그인할 때 실행되며, `~/.bashrc`는 새로운 Bash 셸이 실행될 때마다 실행됩니다.

환경 변수는 `/etc/profile` 또는 `~/.profile`에서 설정하는 것이 일반적입니다.
