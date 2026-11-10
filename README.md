# RHCE 랩 — 받는 사람용 README

## 사전 요건
- VirtualBox 7.x 설치: https://www.virtualbox.org/wiki/Downloads
- Vagrant 2.4+ 설치: https://developer.hashicorp.com/vagrant/downloads
- 여유 디스크 20GB / RAM 8GB 이상 (권장 16GB)

## 파일 구성 (이 폴더에 4개)
- control.box   — control 서버 이미지 (ansible/httpd 설치 완료)
- node.box      — 노드 이미지 (미니멀)
- Vagrantfile   — VM 6대 정의 (IP/호스트명/디스크 자동)
- README.md     — 이 문서

## 실행 (3줄)
```powershell
vagrant box add control control.box
vagrant box add node node.box
vagrant up
```
- 소요: 10~20분 (박스 임포트 + 6대 부팅)
- `vagrant status`로 control, node1~5 모두 running 확인

## 헤드리스 모드 (창 없이 실행)

기본 설정으로 `vagrant up`하면 VirtualBox GUI 창이 6개 뜹니다.
EDR(SentinelOne 등)이 없는 일반 PC에서는 창 없이 백그라운드로 실행할 수 있습니다.

**Vagrantfile에서 `vb.gui = true`를 `false`로 변경:**
```ruby
# control 블록
vb.gui = false   # ← true를 false로 변경

# node 블록
vb.gui = false   # ← true를 false로 변경
```

**전체 일괄 변경 (PowerShell):**
```powershell
(Get-Content .\Vagrantfile) -replace 'vb\.gui = true', 'vb.gui = false' | Set-Content .\Vagrantfile
```

변경 후:
```powershell
vagrant reload        # 실행 중이면 설정 재적용
# 또는
vagrant halt && vagrant up
```

> **주의**: PC에 SentinelOne 등 EDR이 설치되어 있으면 헤드리스 모드(VBoxHeadless.exe)가
> 강제 종료됩니다. 이 경우 `vb.gui = true`로 유지하세요 (창은 최소화해도 됩니다).

## 접속 / 계정
```powershell
vagrant ssh control     # control 접속 (vagrant 계정으로 접속됨)
vagrant ssh node1
vagrant halt            # 전체 종료
vagrant up              # 재기동
vagrant destroy -f      # 전체 삭제 (처음부터 다시)
```
- 앤서블 작업 계정: **admin / admin** (6대 공통, sudo 무암호)
  - `vagrant ssh control` 후 `su - admin` 으로 전환
- root도 SSH 접속 허용 (비번은 rocky)

## 네트워크
| VM | FQDN | IP | 연습용 디스크 |
|---|---|---|---|
| control | control.example.com | 172.17.29.10 | - |
| node1~3 | nodeX.example.com | 172.17.29.11~13 | 2GB |
| node4~5 | nodeX.example.com | 172.17.29.14~15 | 1GB |

- 내부망: 172.17.29.0/24 (PC↔VM, VM↔VM 통신)
- 인터넷: 각 VM NAT (dnf 등)

**배포 패키지에 포함된 것 (받는 사람이 별도 설정 불필요):**
- SSH 키: admin@control의 공개키가 node1~5의 admin 계정에 등록됨
- admin/admin 계정 (6대 공통), NOPASSWD sudo, /etc/hosts에 6대 FQDN 등록
- control: ansible 설치 + ~/ansible 작업 디렉터리

## 첫 실습 (control에서)
**SSH 키는 박스에 미리 배포되어 있습니다** (admin@control → admin@node1~5). 접속 테스트부터 바로 가능:
```bash
su - admin
cd ~/ansible
# 1) inventory/hosts 작성 (control + node1~5 FQDN)
# 2) ansible.cfg 작성 (remote_user=admin, host_key_checking=False, become)
# 3) 연결 확인
ansible nodes -m ping
```

## 문제해결
| 증상 | 해결 |
|---|---|
| vagrant up에서 storageattach 오류 | Vagrantfile의 "SATA"를 실제 컨트롤러 이름으로 수정 (`VBoxManage showvminfo <VM> --machinereadable \| findstr storagecontrollername`으로 확인) |
| 노드 hostname이 node1로 남아있음 | 첫 부팅 후 `sudo hostnamectl set-hostname nodeX.example.com` |
| PC에서 172.17.29.10 접근 안 됨 | Windows 방화벽/VPN 라우팅 확인 (`ping 172.17.29.10`) |
| 메모리 부족 | 안 쓰는 노드 종료: `vagrant halt node5` |
| VBoxHeadless 크래시 (0xc0000005) | EDR이 headless 모드를 차단 → Vagrantfile에서 `vb.gui = true`로 유지 |
