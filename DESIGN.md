# PixelmonServer 설계 문서

> **상태**: v0.2 — 기술 조사 완료, Phase별 구체화
>
> 이 문서는 서버의 기준 진실(single source of truth)이다.
> 서버 구축 시 이 문서를 기준으로 작업한다.
> 상세 조사 자료는 `docs/RESEARCH.md` 참조.

---

## 1. 서버 컨셉

### 1.1 목표
- **친구 3~8명**과 함께 즐기는 PvE 중심 Pixelmon RPG 서버
- **최소한의 개발(틀만 잡기)**로 최대한의 컨텐츠 제공
- VPS가 남는 기간 동안의 즐길거리

### 1.2 핵심 원칙
- 협력 중심, 경쟁은 "보조 양념" 수준
- "빡센 MMO 서버"가 아니라 "잘 만든 Pixelmon RPG + 약간의 경쟁 요소"
- **Pixelmon 내장 기능을 최대한 활용**하고, 빈 틈만 사이드모드/플러그인으로 채운다
- 코딩은 최소화. 대부분 **JSON 설정 + NPC Editor GUI + 맵 빌딩**으로 해결

### 1.3 PvP 원칙
- 기본 월드: PvP 비활성
- PvP 허용: 전용 구역 또는 이벤트에서만
- PvP 패배 시: 아이템 드랍 ❌, 포켓몬 영구 손실 ❌
- PvP 보상: 소량 포인트/토큰 + 명예/칭호 위주 (직접 돈 파밍 수단 ❌)

---

## 2. 기술 스택

### 2.1 확정 구성

| 구성 요소 | 버전 | 역할 |
|-----------|------|------|
| Java | **21** (64-bit) | NeoForge 1.21.1 필수 요구사항 |
| Minecraft | **1.21.1** | Pixelmon 최신 지원 버전 |
| NeoForge | **21.1.172** | 모드 서버 로더 (모드팩 기준 버전) |
| Pixelmon | **9.3.14** | 포켓몬 모드 본체 (NeoForge 기반) |
| SpongeNeo | (버전 확인 필요) | 플러그인 로더 — **Phase 1에서 호환성 검증** |

### 2.2 VPS 요구사항

| 항목 | 최소 | 권장 |
|------|------|------|
| RAM | 6GB | 8GB |
| CPU | 2코어 | 4코어 |
| 저장소 | 10GB+ | 20GB+ |
| Java | JDK 21 (64-bit) | 동일 |

### 2.3 클라이언트 배포
- 친구 전원 NeoForge + Pixelmon 모드 클라이언트 필요
- **ATLauncher** 또는 **Modrinth**에서 "The Pixelmon Modpack" 설치 권장
- 클라이언트 RAM 최소 2GB 할당

### 2.4 핵심 리스크

| 리스크 | 심각도 | 대응 |
|--------|--------|------|
| NeoForge 21.1 + SpongeNeo + Pixelmon 9.3 호환성 | **높음** | Phase 1에서 반드시 검증 |
| Sponge 플러그인 생태계 부족 | **중간** | 사이드모드로 상당 부분 대체 가능 |
| VPS RAM 부족 | **중간** | 6GB 미만이면 서버 크래시/TPS 저하 |

### 2.5 대안 플랜 (SpongeNeo 호환 실패 시)

우선순위 순:
1. **옵션 A (권장)**: Pixelmon 사이드모드만으로 최대한 커버. 권한/편의는 서버 config + 바닐라 명령어 + OP 관리로 우회. 3~8명이면 충분.
2. **옵션 B**: Pixelmon 버전을 1.20.x로 낮춰서 SpongeForge 사용 (SpongeForge는 더 안정적)
3. **옵션 C**: Mohist 계열 (Forge + Bukkit 하이브리드) — NeoForge 지원 불확실하므로 최후 수단

---

## 3. Pixelmon 내장 기능 (활용 계획)

> Pixelmon이 자체적으로 제공하는 기능이 매우 많다.
> "최소 개발, 최대 컨텐츠"의 핵심이 바로 이것.

### 3.1 내장 기능 목록

| 기능 | 설명 | 활용 Phase |
|------|------|-----------|
| 포켓몬 스폰/포획/배틀 | 바이옴별 야생 스폰, 배틀 시스템 | Phase 1 (기본) |
| NPC 시스템 | 트레이너, 상점, 교환, 퀘스트 기버, 의사, 기술머신 NPC | Phase 3 |
| NPC Editor | 크리에이티브 모드에서 NPC 생성/편집 GUI | Phase 3 |
| 체육관 구조물 | Structure Block으로 타입별 체육관 프리셋 스폰 | Phase 3 |
| 배지 시스템 | 체육관 관장 격파 시 배지 자동 지급 | Phase 3 |
| 퀘스트 시스템 | JSON 기반 커스텀 퀘스트 (목표, 보상, 다단계) | Phase 3 |
| Max Raid Battles | Raid Den, 별 등급(1~5), raids.json으로 커스텀 | Phase 4 |
| 전설 스포너 | 글로벌 전설 포켓몬 스폰 (빈도/확률 config) | Phase 4 |
| 브리딩/교배 | Day Care NPC, 포켓몬 육성 | Phase 3 (자동) |
| PokeLoot | 보물 상자 시스템 | Phase 3 (탐험 보상) |

### 3.2 퀘스트 시스템 상세

**활성화**: `pixelmon.hocon` → `useExternalJSONFilesQuests=true`
**퀘스트 파일 위치**: `pixelmon/quests/` 폴더에 JSON 파일 생성
**리로드**: `/pokereload` 명령어 (서버 재시작 불필요)

퀘스트 타입:
- 🟡 Standard (1회 완료)
- 🔵 Repeatable (반복 가능)
- 🟢 Instantaneous (즉시 완료)
- 🔴 Legendary (전설 포켓몬용)
- 🟣 Dynamax (1회)

사용 가능한 목표(Objective) 유형:
- 포켓몬 잡기, NPC와 대화, 아이템 수집, 특정 장소 방문, 포켓몬 진화, 아이템 제작 등

**NPC 연동**: NPC Editor에서 퀘스트 기버 NPC 생성 → UUID 복사 → 퀘스트 JSON에 연결

### 3.3 체육관 시스템 상세

**프리셋 체육관**: Structure Block으로 즉시 생성
- 예: `pixelmon:gyms/fire/gym_fire`, `pixelmon:gyms/dragon/gym_dragon`
- 타입별 NPC 트레이너 + 관장 + Boss 트레이너(레벨+30, +40) 자동 포함

**커스텀 체육관**: NPC Editor로 직접 생성
- 관장 포켓몬/레벨/AI 설정
- 플레이어 최고 레벨 기준 자동 스케일링 옵션

**배지**: 관장 격파 시 해당 타입 배지 자동 지급

### 3.4 레이드/전설 시스템 상세

**Raid Den**: `raids.json`으로 바이옴별 등장 포켓몬/별 등급 설정
```
"4-Pikachu", "1,2-Diglett", "3,4,5-Dugtrio"
```

**전설 스포너**: `pixelmon.hocon`에서 독립 설정
- 스폰 빈도, 확률, 거리, 용량 제한 조절 가능
- 레이드와 별도 시스템

**3~8명 규모 현실적 운영**: 수동 스폰(`/pokespawn`) + 디스코드 공지가 자동화보다 효율적

---

## 4. 사이드모드 & 플러그인 계획

### 4.1 Pixelmon 사이드모드 (NeoForge에 직접 설치, SpongeNeo 불필요)

| 사이드모드 | 기능 | 용도 | 우선순위 | Phase |
|-----------|------|------|----------|-------|
| **Pixelmon Extras** | 40+ 관리 명령어 (pokeedit, ivs, hatch 등) | 운영자 필수 도구 | **필수** | 1 |
| **Pixelmon Broadcasts** | 전설/이벤트 스폰 시 서버 전체 알림 | 레이드/전설 알림 | 높음 | 1 |
| **PixelHunt Remastered** | 랜덤 포켓몬 사냥 이벤트 자동 진행 | 자동 이벤트 | 높음 | 4 |
| **Wonder Trade** | 랜덤 포켓몬 교환 시스템 | 재미 요소 | 중간 | 3 |
| **Pixelmon Auction** | 포켓몬 경매 시스템 | 플레이어 간 거래 | 중간 | 3 |
| **Gameshark** | 희귀 포켓몬 위치 추적 | 탐험 컨텐츠 | 낮음 | 선택 |

### 4.2 Sponge 플러그인 (SpongeNeo 필요 — Phase 1 검증 후 결정)

| 플러그인 | 기능 | 용도 | 우선순위 |
|---------|------|------|----------|
| **LuckPerms** | 권한 관리 | OP/유저 권한 분리 | **필수** |
| **Nucleus** | Essentials 대체 (/spawn /home /tpa /warp) | 편의 명령어 | **필수** |
| **Economy Lite** 또는 **Total Economy** | 서버 화폐 시스템 | 경제 루프 | 높음 |
| **PixelmonEconomyBridge** | 포케달러 ↔ Sponge 경제 연동 | 화폐 통합 | 높음 |
| **GriefDefender** | 블록 보호 / Claim | 트롤 방지 | 높음 |

### 4.3 SpongeNeo 실패 시 대체 전략

SpongeNeo가 안 되면 Sponge 플러그인을 못 쓰므로:

| 기능 | Sponge 플러그인 | 대체 방안 |
|------|----------------|----------|
| 권한 관리 | LuckPerms | OP 등급으로 관리 (3~8명이면 충분) |
| /spawn /home /tpa | Nucleus | 바닐라 명령어 + 커맨드 블록 |
| 월드 보호 | GriefDefender | spawn-protection + 신뢰 기반 (소규모) |
| 경제 | Economy Lite | Pixelmon 내장 포케달러만 사용 |
| 화폐 연동 | EconomyBridge | 불필요 (포케달러 단일 화폐) |

---

## 5. 컨텐츠 설계

### 5.1 게임플레이 루프

```
야생 탐험 → 포켓몬 포획 → 트레이너 NPC 배틀 → 레벨업
     │
     ▼
퀘스트 수행 → 체육관 해금 → 체육관 도전 → 배지 획득
     │
     ▼
전체 클리어 → 레이드 참가 → 리그 도전 → 엔딩
     │
     ▼
(반복) PixelHunt 이벤트, Wonder Trade, 포켓몬 수집, PvP 토너먼트
```

### 5.2 컨텐츠 범위

| 컨텐츠 | 수량 | 구현 방식 | 난이도 |
|--------|------|----------|--------|
| 체육관 | 2~4개 | Structure Block 프리셋 + NPC Editor | ★★☆☆☆ |
| 퀘스트 | 5~7개 | JSON 파일 작성 | ★★☆☆☆ |
| 경제 | NPC 상점 | Pixelmon 내장 Shopkeeper NPC | ★☆☆☆☆ |
| 레이드 | 1~2종 | raids.json + 전설 스포너 config | ★★☆☆☆ |
| 리그 | 1개 | NPC Editor로 리그 관장 구성 | ★★☆☆☆ |
| 자동 이벤트 | PixelHunt | 사이드모드 설치 + config | ★☆☆☆☆ |
| PvP 토너먼트 | 선택 | 디스코드 공지 + 수동 진행 | ★☆☆☆☆ |

### 5.3 퀘스트 흐름 (구체)

| # | 퀘스트 | 목표 | 보상 | 해금 |
|---|--------|------|------|------|
| 1 | 첫 포획 | 야생 포켓몬 3마리 잡기 | 포케볼 20개 + 포케달러 | 상점 NPC 위치 안내 |
| 2 | 첫 도전 | 트레이너 NPC 3명 격파 | 고급 포케볼 + 힐링 아이템 | 체육관 1 위치 안내 |
| 3 | 불꽃의 배지 | 체육관 1 클리어 | 기술머신 + 포케달러 | 체육관 2 해금 |
| 4 | 물결의 배지 | 체육관 2 클리어 | 희귀 아이템 + 포케달러 | 레이드 참가 자격 |
| 5 | 전설의 도전 | 레이드 1회 참가 | 마스터볼 1개 | 리그 해금 (체육관 전체 클리어 시) |

> 퀘스트 내용은 Phase 3에서 확정. 위는 설계 예시.

### 5.4 체육관 설계 (확정)

| 체육관 | 타입 | 관장 레벨 기준 | Structure ID | 비고 |
|--------|------|--------------|-------------|------|
| 체육관 1 | 불꽃 | 플레이어 최고 레벨 | `pixelmon:gyms/fire/gym_fire` | 입문용 |
| 체육관 2 | 물 | 플레이어 최고 레벨 | `pixelmon:gyms/water/gym_water` | 중급 |
| 체육관 3 | 전기 | 플레이어 최고 레벨+5 | `pixelmon:gyms/electric/gym_electric` | 상급 |
| 체육관 4 | 드래곤 | 플레이어 최고 레벨+10 | `pixelmon:gyms/dragon/gym_dragon` | 고난이도 |

- **4개 확정**, Structure Block 프리셋 사용 (직접 빌딩 ❌)
- 프리셋에 NPC 트레이너 + 관장 + Boss 트레이너 자동 포함

### 5.5 경제 설계

- **단일 화폐**: 포케달러 (Pixelmon 내장)
- **수입원**: 트레이너 배틀 보상, 퀘스트 보상
- **지출처**: NPC 상점 (포케볼, 힐링 아이템, 기술머신, 희귀 아이템)
- **복잡한 시스템 불필요**: 3~8명이므로 인플레이션 걱정 없음
- **플레이어 간 거래**: Wonder Trade + Pixelmon Auction으로 충분

---

## 6. 로드맵

### Phase 0 — 기획/설계 ✅
- [x] 서버 컨셉 확정
- [x] PvP 원칙 정의
- [x] 컨텐츠 범위 고정
- [x] 기술 스택 조사 (Pixelmon 9.3.14, NeoForge 21.1.200, SpongeNeo)
- [x] Pixelmon 내장 기능 조사 (NPC, 퀘스트, 체육관, 레이드)
- [x] 사이드모드/플러그인 목록 선정
- [x] Phase별 구체 작업 분해
- [ ] VPS 스펙 확인 (RAM 6GB 이상인지)

### Phase 1 — 서버 구동 + 호환성 검증

**목표**: 서버 접속 → 포켓몬 잡기 → 트레이너 NPC 배틀 가능

작업 (설치):
- [ ] VPS에 Java 21 (64-bit) 설치
- [ ] NeoForge 21.1.200 서버 설치 (`java -jar neoforge-installer.jar --installServer`)
- [ ] eula.txt 동의, server.properties 기본 설정
- [ ] Pixelmon 9.3.14 모드 파일을 `mods/` 에 배치
- [ ] 필수 사이드모드 설치: Pixelmon Extras, Pixelmon Broadcasts
- [ ] 서버 시작 → 포켓몬 스폰/포획/배틀 정상 동작 확인

작업 (SpongeNeo 검증):
- [ ] SpongeNeo 다운로드 (Pixelmon 9.3 + NeoForge 21.1 호환 버전 확인)
- [ ] `mods/` 에 배치 후 서버 시작
- [ ] Pixelmon과 충돌 없이 동작하는지 확인
- [ ] LuckPerms Sponge 버전 테스트 설치

분기:
- ✅ SpongeNeo 호환 → Phase 2로 (Sponge 플러그인 활용)
- ❌ SpongeNeo 비호환 → §2.5 대안 플랜 적용 (사이드모드 중심 전략)

작업 (클라이언트):
- [ ] 친구 배포용 클라이언트 설치 가이드 작성 (ATLauncher / Modrinth)
- [ ] 테스트 접속 확인

### Phase 2 — 기본 인프라

**목표**: 친구들과 안정적으로 접속, 기본 편의/보호 가동

SpongeNeo 사용 가능한 경우:
- [ ] LuckPerms 설치 + 권한 그룹 설정 (admin / default)
- [ ] Nucleus 설치 + 기본 명령어 활성화 (/spawn /home /tpa /warp)
- [ ] GriefDefender 또는 보호 플러그인 설치
- [ ] 스폰 지역 설정 + 보호

SpongeNeo 사용 불가한 경우:
- [ ] OP 등급으로 권한 관리
- [ ] 커맨드 블록으로 /spawn 구현
- [ ] spawn-protection 설정 (server.properties)
- [ ] 신뢰 기반 운영 (소규모이므로)

공통:
- [ ] 기본 월드 PvP 비활성 (`pixelmon.hocon` 또는 서버 설정)
- [ ] 서버 시작/백업 스크립트 작성 (`scripts/`)
- [ ] pixelmon.hocon 기본 튜닝 (스폰율, 전설 빈도 등)

### Phase 3 — 게임플레이 루프 구축

**목표**: 서버가 "게임"처럼 느껴짐. 포획→체육관→퀘스트 루프 작동

작업 (경제):
- [ ] 포케달러 기본 설정 확인 (Pixelmon 내장)
- [ ] (SpongeNeo 가능 시) Economy Lite + PixelmonEconomyBridge 설치
- [ ] NPC Shopkeeper 배치 (포케볼, 힐링, 기술머신 상점)
- [ ] 트레이너 배틀 보상 금액 밸런스 설정

작업 (체육관):
- [ ] 체육관 타입/수 확정 (2~4개)
- [ ] Structure Block 프리셋으로 체육관 건물 생성 또는 직접 빌딩
- [ ] NPC Editor로 관장 NPC 설정 (포켓몬, 레벨, AI)
- [ ] 관장 보상(배지 + 아이템) 연결
- [ ] 체육관 위치 warp 설정 (Nucleus) 또는 표지판 안내

작업 (퀘스트):
- [ ] `pixelmon.hocon` → `useExternalJSONFilesQuests=true` 활성화
- [ ] 퀘스트 5~7개 JSON 파일 작성 (`pixelmon/quests/`)
- [ ] 퀘스트 기버 NPC 배치 (NPC Editor)
- [ ] 퀘스트 NPC UUID → JSON 연결
- [ ] `/pokereload`로 테스트 → 밸런스 조정

작업 (추가 사이드모드):
- [ ] Wonder Trade 설치 + config (선택)
- [ ] Pixelmon Auction 설치 + config (선택)

### Phase 4 — 이벤트/레이드 + 리그 (엔드 컨텐츠)

**목표**: 엔드 컨텐츠까지 플레이 가능. 서버 완성

작업 (레이드):
- [ ] `raids.json` 편집 — 바이옴별 레이드 포켓몬/별 등급 설정
- [ ] 전설 스포너 config 조정 (`pixelmon.hocon`)
- [ ] Pixelmon Broadcasts로 전설 스폰 알림 확인
- [ ] (선택) 특정 시간 수동 스폰 이벤트 (디스코드 공지 + `/pokespawn`)

작업 (리그):
- [ ] 리그 진입 조건 설계 (체육관 올클리어 + 특정 퀘스트 완료)
- [ ] 리그 NPC 구성 (NPC Editor — 최종 보스급 트레이너)
- [ ] 최종 보상 설정 (칭호, 상징 아이템, 마스터볼 등)

작업 (이벤트):
- [ ] PixelHunt Remastered 설치 + config (자동 사냥 이벤트)
- [ ] (선택) PvP 토너먼트 규칙 확정 (디스코드 공지 + 수동 진행)

---

## 7. 작업 유형 분류

### 7.1 개발(설정) 작업 — "손으로 하는 것"

| 작업 | 실제 내용 | 난이도 |
|------|----------|--------|
| 서버 설치 | NeoForge + Pixelmon + 사이드모드 설치 | ★☆☆☆☆ |
| SpongeNeo 검증 | 설치 + 호환성 테스트 | ★★★☆☆ |
| 플러그인 설정 | config 파일 편집 | ★★☆☆☆ |
| 퀘스트 JSON | JSON 파일 5~7개 작성 | ★★☆☆☆ |
| NPC 배치 | NPC Editor GUI로 생성/편집 | ★★☆☆☆ |
| 체육관 건물 | Structure Block 프리셋 또는 직접 빌딩 | ★★★☆☆ (빌딩) |
| 레이드 설정 | raids.json + pixelmon.hocon 편집 | ★★☆☆☆ |
| 서버 스크립트 | 시작/백업 쉘 스크립트 | ★☆☆☆☆ |

### 7.2 설계 작업 — "머리로 하는 것"

| 작업 | 결정해야 할 것 | 난이도 |
|------|--------------|--------|
| 체육관 구성 | 타입, 수, 난이도 곡선 | ★★★☆☆ |
| 퀘스트 흐름 | 해금 순서, 조건, 보상 밸런스 | ★★★☆☆ |
| 경제 밸런스 | 수입원/지출처 균형, 물가 설정 | ★★★☆☆ |
| 맵/동선 | 스폰, 체육관, 상점 위치 배치 | ★★★☆☆ |
| 레이드 밸런스 | 스폰 빈도, 보상 | ★★☆☆☆ |
| PvP 규칙 | 허용 구역, 보상 | ★☆☆☆☆ |

### 7.3 난이도 총평

- **코딩이 필요한 부분**: 거의 없음 (SpongeNeo 실패 시에도 사이드모드로 대체)
- **실제 난이도가 높은 것**: 게임 디자인 (밸런스, 진행감) + 맵 빌딩
- **BotTycoon 경험이 직접 도움**: 경제 밸런스, NPC 설정, 서버 운영 노하우

---

## 8. 참고

- BotTycoon(Paper API 서버) 운영 경험 있음 → 서버 운영/밸런스 노하우 활용
- 이 서버는 포트폴리오 대상이 아님 → 문서화보다 실용성 우선
- Pixelmon 공식 위키: https://pixelmonmod.com/wiki/
- Pixelmon 사이드모드: https://pixelmon-sidemods.com/
- Sponge 플러그인(Ore): https://ore.spongepowered.org/
- NeoForge 설치 가이드: https://docs.neoforged.net/user/docs/server/
