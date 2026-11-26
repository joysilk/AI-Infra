# 03-01. Linux

> 인프라 직무의 모든 것의 바닥. 클라우드 서버의 대부분이 Linux입니다. **Linux**는 서버에서 가장 많이 쓰는 운영체제(OS), **커널**은 그 핵심으로 하드웨어와 프로그램 사이를 중재하는 부분입니다.

## 1. 개념 설명

### 1-1. 파일 시스템과 기본 명령어
- 모든 것이 파일: 장치(`/dev`), 프로세스 정보(`/proc`)도 파일처럼 접근.
- 주요 디렉터리: `/etc`(설정), `/var/log`(로그), `/home`(사용자), `/usr/bin`(실행파일), `/tmp`(임시), `/proc`(프로세스·커널 정보).
- 핵심 명령어: `ls cd pwd cp mv rm mkdir find grep cat less head tail wc sort uniq cut awk sed xargs tee`
- **파이프(`|`)**: 앞 명령 출력을 뒤 명령 입력으로. **리다이렉션(`>`, `>>`, `2>`)**: 출력/에러를 파일로.
- 예: `grep ' 500 ' access.log | awk '{print $1}' | sort | uniq -c | sort -nr | head` → 500 에러를 낸 IP 상위 목록.

### 1-2. 사용자와 권한
- 사용자(user)/그룹(group)/기타(others)별로 읽기(r=4), 쓰기(w=2), 실행(x=1).
- `chmod 640 file` = 소유자 rw, 그룹 r, 기타 없음. `chown user:group file`.
- **sudo**: 일시적으로 관리자(root) 권한 사용. `/etc/sudoers`(`visudo`로 편집).
- 특수 권한: SUID, SGID, sticky bit(`/tmp`).
- **SSH 키 인증**: 비밀번호 대신 공개키/개인키 쌍. 서버에 공개키(`~/.ssh/authorized_keys`), 내가 개인키 보관.

### 1-3. 프로세스
- **프로세스**: 실행 중인 프로그램. PID(번호), PPID(부모). 상태: R(실행), S(대기), D(디스크 대기), Z(좀비).
- `ps aux`, `top/htop`, `kill -15 PID`(정상 종료 요청, SIGTERM), `kill -9 PID`(강제, SIGKILL), `nice/renice`, `lsof`, `strace`.
- **로드 애버리지**: 실행/대기 중인 작업 수의 평균. CPU 코어 수와 비교해 해석(`uptime`, `nproc`).
- 메모리: `free -h`, 캐시/버퍼는 필요 시 회수되므로 `available`을 볼 것. **OOM Killer**: 메모리 부족 시 커널이 프로세스를 강제 종료(`dmesg | grep -i oom`).

### 1-4. 서비스 관리 (systemd)
- **systemd**: 서비스 시작·정지·자동 시작을 관리하는 프로그램.
- `systemctl start|stop|restart|status|enable nginx`, 유닛 파일 `/etc/systemd/system/app.service`:
```ini
[Unit]
Description=My App
After=network.target
[Service]
User=app
ExecStart=/usr/bin/python3 /opt/app/main.py
Restart=on-failure
[Install]
WantedBy=multi-user.target
```
- 수정 후 `systemctl daemon-reload`.

### 1-5. 로그
- `journalctl -u nginx -f`(실시간), `journalctl --since "1 hour ago"`, `/var/log/syslog|messages|auth.log`.
- **logrotate**: 로그 파일이 커지지 않게 주기적으로 회전/압축.
- 시간·타임존 확인: `timedatectl`. 로그 시간이 UTC인지 KST인지 항상 확인.

### 1-6. 디스크·네트워크·성능
- `df -h`(용량), `du -sh *`(디렉터리 크기), `lsblk`, `mount`, `fstab`, LVM 개념, inode(`df -i`).
- `ip a`, `ip r`, `ss -tulnp`(열린 포트), `curl`, `dig`, `tcpdump`.
- 성능 도구: `vmstat`, `iostat`, `sar`, `pidstat`. "USE 방법": 자원별 사용률(Utilization)·포화(Saturation)·에러(Errors).

### 1-7. 셸 스크립트
```bash
#!/usr/bin/env bash
set -euo pipefail          # 에러/미정의 변수/파이프 실패 시 중단
LOG_DIR=/var/log/app
DAYS=${1:-7}
find "$LOG_DIR" -name '*.log' -mtime +"$DAYS" -print -delete
```
- 따옴표로 변수 감싸기, `shellcheck`로 점검, 함수·`trap`으로 정리 작업.

## 2. 공부 순서
1. 파일/디렉터리 명령어 → 2. 텍스트 처리(grep/awk/sed) → 3. 권한/사용자 → 4. 패키지 관리(apt/dnf) → 5. 프로세스/서비스(systemd) → 6. 로그/cron → 7. 디스크/LVM → 8. 네트워크 도구 → 9. 셸 스크립트 → 10. 성능 분석 → 11. 보안 하드닝(SSH, 방화벽, SELinux/AppArmor 개요).
- 기간: 집중 4주(0년차), 이후 평생 심화.

## 3. 실습 과제
1. VM에 Ubuntu 설치 → 일반 사용자 생성 → sudo 권한 부여 → 루트 SSH 로그인 차단.
2. nginx 설치 후 포트 80 확인(`ss -tulnp`), 방화벽(`ufw`) 허용, 접속 로그 분석(상위 IP/상태코드).
3. systemd로 Python 웹앱 서비스 등록, 강제 종료 시 자동 재시작 확인.
4. 디스크를 99%로 채워(`fallocate`) 장애를 만들고 원인(`du`)을 찾아 복구.
5. 메모리 누수 스크립트 작성→OOM 발생→`dmesg` 로 확인.
6. 백업 스크립트(압축+날짜+7일 보관)를 cron에 등록.
7. "서버 헬스체크 스크립트": CPU/메모리/디스크/상위 프로세스를 요약해 출력.
8. (심화) 장애 연습: 서비스가 안 뜨는 서버 5가지 시나리오(권한, 포트 충돌, 설정 오타, 디스크 가득, DNS)를 스스로 만들고 해결.

## 4. 추천 자료
- 무료(한글): 리눅스마스터 교재/블로그, 생활코딩 리눅스·셸, 위키독스 "리눅스 커맨드라인".
- 무료(영문): The Linux Command Line(William Shotts, 무료 PDF), Linux Journey(linuxjourney.com), OverTheWire Bandit(게임으로 명령어 학습), Brendan Gregg 성능 글/도구 지도.
- 유료/도서: 「리눅스 시스템 관리」류 서적, 「시스템 성능 분석과 최적화 (Systems Performance)」(심화), Udemy/인프런 리눅스 강의.
- 실습 환경: Multipass, UTM, VirtualBox, WSL2, AWS Free Tier EC2, Killercoda.

## 5. 면접 단골 질문
1. 리눅스 부팅 과정을 설명하세요. (BIOS/UEFI → 부트로더 GRUB → 커널 → init/systemd → 서비스)
2. 프로세스와 스레드의 차이는?
3. 좀비 프로세스란? 어떻게 처리하나? (부모가 wait 안 함 → 부모 종료/재시작)
4. `kill -9`와 `kill -15` 차이?
5. 서버가 느릴 때 어떻게 접근하나? (`uptime`/`top` → CPU/메모리/IO/네트워크 순 분리)
6. 디스크는 남았는데 파일을 못 만든다. 이유는? (inode 고갈, 쿼터, 읽기 전용 마운트)
7. 하드링크와 심볼릭 링크 차이?
8. `chmod 755`와 `644`의 의미?
9. 로드 애버리지 의미와 해석?
10. cron과 systemd timer 차이?
(모범 답변은 [06-3](06-3-기술면접-질문50.md) 참고)
