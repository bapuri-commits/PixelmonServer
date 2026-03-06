# PixelmonServer — VPS 배포 가이드

> ⚠️ **초안 문서** — 각 Stage 진행 시 실제 환경에 맞게 수정될 수 있음.

> Contabo VPS에서 Minecraft Pixelmon 서버를 systemd로 운영하는 절차.
> DevOps 학습 로드맵 Stage 11 (후순위)에서 사용.

---

## 아키텍처 개요

```
[클라이언트 (Minecraft)]  →  [VPS :25565]  →  [NeoForge + Pixelmon]
                                                      ↓
                                              [systemd 서비스 관리]
                                              [cron 백업 자동화]
```

---

## 사전 요구사항

- Java 21 (JDK, 64-bit)
- 최소 RAM: 6 GB (권장 8 GB)
- 디스크: 20 GB+

---

## 서버 스펙 (DESIGN.md 기준)

| 항목 | 값 |
|------|-----|
| Minecraft | 1.21.1 |
| NeoForge | 21.1.172 |
| Pixelmon | 9.3.14 |
| SpongeNeo | 호환성 검증 필요 |
| 최대 인원 | 3~8명 |

---

## Stage 11 — systemd 서비스 설정

### 서버 설치

```bash
# TODO: Stage 3에서 실행
# 1. Java 21 설치
# sudo apt install openjdk-21-jdk

# 2. 서버 디렉토리 생성
# sudo mkdir -p /opt/pixelmon
# cd /opt/pixelmon

# 3. NeoForge 설치
# java -jar neoforge-installer.jar --installServer

# 4. Pixelmon 모드 설치
# Pixelmon-9.3.14.jar → mods/ 폴더에 배치

# 5. eula.txt 동의
# echo "eula=true" > eula.txt

# 6. 서버 설정
# server.properties, pixelmon.hocon 등 configs/ 참조
```

### systemd 서비스 파일

```ini
# /etc/systemd/system/pixelmon.service
# TODO: Stage 3에서 작성
# [Unit]
# Description=Pixelmon Minecraft Server
# After=network.target
#
# [Service]
# User=minecraft
# WorkingDirectory=/opt/pixelmon
# ExecStart=/usr/bin/java -Xms4G -Xmx8G -jar neoforge-server.jar nogui
# Restart=on-failure
# RestartSec=10
#
# [Install]
# WantedBy=multi-user.target
```

### 서비스 관리 명령어

```bash
sudo systemctl daemon-reload
sudo systemctl enable pixelmon
sudo systemctl start pixelmon
sudo systemctl status pixelmon
journalctl -u pixelmon -f
```

---

## 방화벽

```bash
# Minecraft 기본 포트
sudo ufw allow 25565/tcp
```

---

## 백업 자동화

```bash
# TODO: Stage 3에서 작성
# /opt/pixelmon/scripts/backup.sh
# #!/bin/bash
# BACKUP_DIR="/opt/pixelmon/backups"
# DATE=$(date +%Y-%m-%d_%H%M)
# tar -czf "$BACKUP_DIR/world_$DATE.tar.gz" /opt/pixelmon/world/
# find "$BACKUP_DIR" -name "*.tar.gz" -mtime +7 -delete

# crontab:
# 0 4 * * * /opt/pixelmon/scripts/backup.sh >> /var/log/pixelmon-backup.log 2>&1
```

---

## 알려진 이슈

- SpongeNeo와 NeoForge 21.1.172 호환성 미검증
- `scripts/` 폴더가 DESIGN에 언급되었으나 아직 미생성
- 8GB RAM 할당 시 VPS 전체 24GB 중 1/3 사용 → 다른 서비스와 동시 운영 시 조정 필요
- 퀘스트 JSON (`configs/quests/`)은 Pixelmon 설정 형식에 맞는지 확인 필요
