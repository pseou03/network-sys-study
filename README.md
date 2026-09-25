# network-sys-study

## 🚀 AWS EC2 우분투 서버 구축 및 네트워크/보안 기초 실습

### 📌 1. 학습 개요
- **목표:** AWS EC2 인스턴스 생성, 아이패드 Terminus를 활용한 SSH 원격 접속, 그리고 네트워크 및 방화벽 기초 개념 정립
- **핵심 키워드:** LAN/MAC/PHY, cURL, 포트/소켓, 공인 IP와 `/32` 마스크, AWS 보안 그룹 vs UFW 방화벽

---

### 🛠️ 2. 주요 실습 및 트러블슈팅 과정

#### ① AWS 보안 그룹과 Public IP 구조
- 단일 서버에 할당되는 공인 IP는 주변 호스트를 둘 필요가 없으므로 `/32`(`255.255.255.255`) 마스크를 사용해 라우팅 경로를 단독 독점하는 구조를 이해함.
- 무차별 대입 공격(Brute Force)을 방어하기 위해 22번 포트(SSH)는 가급적 전체 허용(`0.0.0.0/0`)을 지양하고 내 공인 IP(`curl ifconfig.me`) 기반으로 설정하는 것의 중요성 학습.

#### ② 리눅스 소켓 통신 확인 (`ss -lnt`)
- `ss`(Socket Statistics) 명령어를 통해 시스템 부하 없이 현재 열려 있는 TCP 포트 상태(`LISTEN`)를 확인하는 법 실습.

#### ③ 리눅스 UFW 방화벽 설정 시 주의사항
- **실수 방지 순서:** 방화벽을 켤 때(`sudo ufw enable`) 규칙을 먼저 등록하지 않으면 원격 SSH 접속이 차단되어 서버에서 튕겨 나가는 사고 발생 가능.
- **안전한 실행 순서:**
  ```bash
  # 1. 내 IP를 SSH(22번) 허용 규칙으로 먼저 등록
  sudo ufw allow from <내_공인_IP> to any port 22 proto tcp

  # 2. 방화벽 활성화
  sudo ufw enable

  # 3. 상세 상태 확인
  sudo ufw status verbose
  ```

ubuntu@ip-172-31-4-69:~$ sudo ufw status verbose
Status: inactive

ubuntu@ip-172-31-4-69:~$ sudo ufw allow from 116.120.81.242 to any port 22 proto tcp
Rules updated

ubuntu@ip-172-31-4-69:~$ sudo ufw enable
Command may disrupt existing ssh connections. Proceed with operation (y|n)? y
Firewall is active and enabled on system startup

ubuntu@ip-172-31-4-69:~$ sudo ufw status verbose
Status: active
Logging: on (low)
Default: deny (incoming), allow (outgoing), disabled (routed)
New profiles: skip

To                         Action      From
--                         ------      ----
22/tcp                     ALLOW IN    116.120.81.242            