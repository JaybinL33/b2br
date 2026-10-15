# Born2beRoot 스피드런 — Rocky Linux 10.2 Minimal 신규 설치

## 1. Host · VirtualBox / Guest · 설치 GUI

```text
VM: b2br · RAM 4096 MB · vCPU 2 · EFI 끔 · VDI 30 GB
ISO: Rocky Linux 10.2 Minimal · 수동 설치 · Minimal Install
시간대: Asia/Seoul · Kdump 끔 · enp0s3 연결 · 일반 NAT/DHCP
포트 전달: ssh / TCP / Host 127.42.42.42:4242 / Guest IP 빈칸 / Guest Port 4242
호스트명: jungblee42 · 사용자: jungblee, 관리자(wheel), 비밀번호 입력
디스크: Custom · GPT · BIOS Boot 1 MiB · /boot ext4 2 GiB
VG: LVM · Encrypt 켬(암호 입력) · As large as possible
LV: root ext4 / 10 GiB · home ext4 /home 4 GiB · var_log ext4 /var/log 2 GiB · swap 4 GiB
Begin Installation → Reboot → ISO 제거 → LUKS 암호 입력 → jungblee 로그인
기본 유지: authselect local / pam_pwquality · SELinux Enforcing · sudo wheel/secure_path
```

## 2. Guest · jungblee · 계정

```bash
sudo hostnamectl set-hostname jungblee42
sudo groupadd user42
sudo groupadd sudo
sudo usermod -aG user42,sudo jungblee
```

## 3. Guest · jungblee · 비밀번호 정책

```bash
sudoedit /etc/login.defs
```

`/etc/login.defs` · 해당 세 항목

```text
PASS_MAX_DAYS 30
PASS_MIN_DAYS 2
PASS_WARN_AGE 7
```

```bash
sudo mkdir -p /etc/security/pwquality.conf.d
sudoedit /etc/security/pwquality.conf.d/90-local.conf
```

`/etc/security/pwquality.conf.d/90-local.conf`

```text
minlen = 10
ucredit = -1
lcredit = -1
dcredit = -1
ocredit = 0
maxrepeat = 3
usercheck = 1
difok = 7
enforcing = 1
enforce_for_root
```

```bash
passwd
sudo passwd root
sudo chage -M 30 -m 2 -W 7 root
sudo chage -M 30 -m 2 -W 7 jungblee
```

## 4. Guest · jungblee · sudo

```bash
sudo visudo -P -f /etc/sudoers.d/90-b2br
```

`/etc/sudoers.d/90-b2br`

```text
Defaults passwd_tries=3
Defaults badpass_message="Authentication failed. Please try again."
Defaults requiretty
Defaults log_input, log_output
Defaults iolog_dir="/var/log/sudo"
%sudo ALL=(ALL:ALL) ALL
```

```bash
sudo visudo -c
```

[sudo 그룹 새 로그인 검증·제출 기준 갱신](targets.md#sudo-그룹의-권한-연결과-평가-해석) · 문서 권장안, VM 적용·검증 전

## 5. Guest · jungblee · SSH / SELinux / firewalld

```bash
sudoedit /etc/ssh/sshd_config.d/00-b2br.conf
```

`/etc/ssh/sshd_config.d/00-b2br.conf`

```text
Port 4242
PermitRootLogin no
```

```bash
sudo dnf install policycoreutils-python-utils
sudo semanage port -a -t ssh_port_t -p tcp 4242
sudo systemctl enable --now firewalld
sudo firewall-cmd --permanent --zone=public --add-port=4242/tcp
sudo firewall-cmd --permanent --zone=public --remove-service=ssh
sudo firewall-cmd --permanent --zone=public --remove-service=cockpit
sudo firewall-cmd --permanent --zone=public --remove-service=dhcpv6-client
sudo firewall-cmd --reload
sudo systemctl enable sshd
sudo systemctl restart sshd
```

## 6. Guest · jungblee · monitoring.sh

```bash
vi monitoring.sh
```

`monitoring.sh`

현재 코드는 used/total·사용률에 available을 추가해 원문 예시와 가용량 문구를 함께 반영한다. 2026-10-15 KST 23:48 사용자 실행 결과에서 가용량 표시와 수동 방송 성공을 확인했다. 수정 후 설치본 전체 소스·bash -n 결과·해시는 첨부되지 않았다.

```bash
#!/bin/bash
export LC_ALL=C

architecture=$(uname -a)
physical=$(lscpu -p=SOCKET | grep -v '^#' | sort -u | wc -l)
virtual=$(lscpu -p=CPU | grep -v '^#' | wc -l)
memory=$(free -m | awk '$1 == "Mem:" {printf "%s/%sMiB (%.2f%%), available: %sMiB", $3, $2, $3 / $2 * 100, $7}')
disk=$(df -m --total -x tmpfs -x devtmpfs | awk '$1 == "total" {printf "%s/%sMiB (%.0f%%), available: %sMiB", $3, $2, $3 / $2 * 100, $4}')
cpu_load=$(vmstat 1 2 | awk 'END {printf "%.1f", 100 - $15}')
last_boot=$(uptime -s | cut -c 1-16)

if lsblk -rn -o TYPE | grep -qx lvm; then
    lvm_use=yes
else
    lvm_use=no
fi

connections=$(ss -Htn state established | wc -l)
users=$(who | awk '{print $1}' | sort -u | wc -l)
ip_address=$(ip -4 -o address show dev enp0s3 | awk '{print $4}' | cut -d / -f 1)
mac_address=$(cat /sys/class/net/enp0s3/address)
sudo_count=$(sudoreplay -d /var/log/sudo -l | wc -l)

{
    printf '#Architecture: %s\n' "$architecture"
    printf '#CPU physical : %s\n' "$physical"
    printf '#vCPU : %s\n' "$virtual"
    printf '#Memory Usage: %s\n' "$memory"
    printf '#Disk Usage: %s\n' "$disk"
    printf '#CPU load: %s%%\n' "$cpu_load"
    printf '#Last boot: %s\n' "$last_boot"
    printf '#LVM use: %s\n' "$lvm_use"
    printf '#Connections TCP : %s ESTABLISHED\n' "$connections"
    printf '#User log: %s\n' "$users"
    printf '#Network: IP %s (%s)\n' "$ip_address" "$mac_address"
    printf '#Sudo : %s cmd\n' "$sudo_count"
} | wall
```

```bash
sudo install -o root -g root -m 755 monitoring.sh /usr/local/sbin/monitoring.sh
sudo systemctl enable --now crond
sudo crontab -e
```

## 7. Guest · root crontab

현재 root crontab은 PATH 명시 줄 없이 `*/10`과 `@reboot` 두 예약을 사용하는 구성이다(사용자 적용 확인). cron이 제공하는 실행 환경의 PATH를 사용하며 `/bin/bash`와 스크립트는 절대경로로 호출한다. 스크립트 내부의 경로 없는 명령은 cron 환경의 PATH로 검색한다. 실제 cron PATH 값은 출력으로 확인하지 않았으므로 단정하지 않는다. sudo의 `secure_path`와는 별개이며 그 설정은 변경하지 않는다.

```cron
*/10 * * * * /bin/bash /usr/local/sbin/monitoring.sh
@reboot /bin/bash /usr/local/sbin/monitoring.sh
```
