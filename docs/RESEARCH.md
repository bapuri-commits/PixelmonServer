# 기술 조사 결과

> 작성일: 2026-02-09
> 목적: Phase 0 기술 조사. DESIGN.md에 반영된 결정의 근거 자료.

---

## 1. 서버 기반 — 현재 상황 대비 준비 사항

### Paper(BotTycoon) vs NeoForge(PixelmonServer) 비교

| 항목 | BotTycoon (Paper) | PixelmonServer (NeoForge) |
|------|-------------------|---------------------------|
| 서버 종류 | 바닐라 + 플러그인 | 모드 + 플러그인 (하이브리드) |
| 플러그인 생태계 | Bukkit/Spigot (수만 개) | Sponge (수백 개, 매우 작음) |
| 모드 호환 | 없음 (바닐라 기반) | NeoForge 모드 사용 가능 |
| 클라이언트 | 바닐라 접속 가능 | 모드 클라이언트 필수 |
| 개발 방식 | Java 플러그인 직접 작성 | 설정/JSON 편집 위주 |
| RAM 요구 | 2~4GB | 최소 6GB, 권장 8GB |

### 버전 호환성 매트릭스 (조사 기준)

| 구성 요소 | 버전 | 출처 |
|-----------|------|------|
| Minecraft | 1.21.1 | pixelmonmod.com |
| NeoForge | 21.1.200 | docs.neoforged.net |
| Pixelmon | 9.3.14 | reforged.gg |
| Java | 21 (64-bit) | NeoForge 공식 요구사항 |
| SpongeNeo | 버전 미확인 | spongepowered.org — 공식 지원은 존재하나 1.21.1 호환 여부 불확실 |

### SpongeNeo 상태

- SpongeNeo는 NeoForge 플랫폼용 Sponge API 구현체로 **공식적으로 존재**한다.
- 그러나 Pixelmon 9.3 (NeoForge 21.1) 조합에서의 **안정성은 보장되지 않음**.
- GitHub 이슈(SpongePowered/Sponge#3887)에서 SpongeForge→NeoForge 전환 논의가 진행 중.
- **결론**: Phase 1에서 반드시 직접 테스트. 실패 시 대안 플랜 적용.

---

## 2. Pixelmon 모드 — 내장 기능 범위

Pixelmon은 단순한 "포켓몬 추가 모드"가 아니라, **서버 운영에 필요한 대부분의 시스템을 내장**하고 있다.

### 2.1 NPC 종류 (전부 내장)

| NPC | 역할 | 서버 활용 |
|-----|------|----------|
| Trainer | 배틀 상대 | 야생/체육관 트레이너 |
| Gym Leader | 체육관 관장 (배지 지급) | 체육관 시스템 핵심 |
| Shopkeeper | 아이템 판매 NPC | 경제 시스템 |
| Trader | 아이템 교환 NPC | 특수 교환 |
| Quest Giver | 퀘스트 부여 NPC | 퀘스트 시스템 |
| Doctor | 포켓몬 치료 | 편의 |
| Move Relearner | 기술 재학습 | 편의 |
| Move Tutor | 기술 교습 | 편의 |

출처: https://pixelmonmod.com/wiki/NPCs

### 2.2 퀘스트 시스템

**활성화**: `pixelmon.hocon` → `useExternalJSONFilesQuests=true`

퀘스트 타입:
- Standard (🟡) — 1회 완료
- Repeatable (🔵) — 반복 가능
- Instantaneous (🟢) — 즉시 완료
- Legendary (🔴) — 전설 포켓몬용
- Dynamax (🟣) — 1회

JSON 구조 핵심 필드:
```json
{
  "weight": 1,
  "abandonable": false,
  "repeatable": false,
  "color": [255, 255, 0],
  "activeStage": 0,
  "stages": [
    {
      "stage": 0,
      "nextStage": 1,
      "icon": { "type": "pokeball" },
      "objectives": ["..."],
      "actions": ["..."]
    }
  ]
}
```

사용 가능한 목표(Objective):
- 포켓몬 잡기, NPC와 대화, 아이템 수집
- 특정 장소 방문, 포켓몬 진화, 아이템 제작
- 포켓몬 배틀 승리, 포켓몬 치료 등

NPC 연동: NPC Editor에서 UUID 복사 → 퀘스트 JSON에 연결

출처: https://pixelmonmod.com/wiki/Quests/Creation

### 2.3 체육관 시스템

- Structure Block 프리셋: `pixelmon:gyms/[타입]/gym_[타입]`
  - 예: `pixelmon:gyms/fire/gym_fire`, `pixelmon:gyms/dragon/gym_dragon`
- NPC 자동 포함: 타입 전문 트레이너 + 관장 + Boss 트레이너(레벨+30, +40)
- 관장 격파 시 해당 타입 배지 자동 지급
- 레벨 스케일링: 플레이어 최고 레벨 기준 자동 조정

출처: https://pixelmonmod.com/wiki/Gym

### 2.4 레이드 시스템

- Max Raid Battles: Raid Den에서 진행
- `raids.json`: 바이옴별 등장 포켓몬/별 등급 지정
  - 형식: `"#,#,#-PokemonName"` (별 등급-이름)
  - 43개 바이옴 카테고리 지원
- 전설 포켓몬: 레이드와 별도의 글로벌 전설 스포너 존재
  - `pixelmon.hocon`에서 빈도/확률/거리/용량 조절

출처: https://pixelmonmod.com/wiki/Raids, https://pixelmonmod.com/wiki/Spawning

### 2.5 기타 내장 기능

- 브리딩/교배 (Day Care NPC)
- PokeLoot (보물 상자)
- External JSON files (NPC 데이터를 JSON으로 편집 가능)
- `/pokereload` (서버 재시작 없이 설정 리로드)

---

## 3. 사이드모드 — 44개 공식 등록

### 3.1 서버에 적합한 사이드모드

| 사이드모드 | 버전 | 기능 | SpongeNeo 필요? |
|-----------|------|------|----------------|
| Pixelmon Extras | 2.5.12 | 40+ 관리 명령어 (pokeedit, ivs, evs, hatch, breed, pc 등) | ❌ |
| Pixelmon Broadcasts | 0.6.1 | 전설/이벤트 스폰 서버 알림 | ❌ |
| PixelHunt Remastered | - | 랜덤 사냥 이벤트 자동 진행 + 보상 | ❌ |
| Wonder Trade | 5.2.0 | 랜덤 포켓몬 교환 (SQL DB 필요) | ❌ |
| Pixelmon Auction | 1.3.2 | 포켓몬 경매 | ❌ |
| Gameshark | 6.0.4 | 희귀 포켓몬 위치 추적 | ❌ |
| Pixelmon Friends | 2.3.1 | 다른 플레이어 포획/진화 추적 | ❌ |

전체 사이드모드 목록: https://pixelmonmod.com/wiki/Sidemods

### 3.2 Pixelmon Extras 명령어 분류

관리/유틸리티 중심으로 40+ 명령어 제공:
- **포켓몬 관리**: pokeclone, pokedel, pokeedit, pokeevolve, pokefaint, pokekill, pokerandom, pokereset
- **정보 조회**: dexcheck, hiddenpower, movelist, evs, ivs
- **브리딩**: breed, breeding, eggsteps, hatch
- **배틀/아이템**: bossbomb, redeemfossil, randomleg, tms
- **PC 관리**: pc, comptake, compsee, comptest, compsearch, compedit

출처: https://pixelmonmod.com/wiki/Pixelmon_Extras

---

## 4. Sponge 플러그인 생태계

### 4.1 핵심 플러그인 (Ore 저장소)

| 플러그인 | 역할 | 비고 |
|---------|------|------|
| LuckPerms | 권한 관리 | Sponge에서 유일한 유지보수 중인 권한 플러그인 |
| Nucleus | Essentials 대체 (/spawn /home /tpa /warp /jail) | 경제 미포함 — 별도 필요 |
| Economy Lite | 경량 경제 플러그인 | Nucleus 문서에서 추천 |
| Total Economy | 종합 경제 플러그인 | Nucleus 문서에서 추천 |
| PixelmonEconomyBridge | 포케달러 ↔ Sponge 경제 연동 | Sponge 전용 |

출처: https://ore.spongepowered.org/

### 4.2 생태계 한계

- Bukkit/Spigot 대비 **플러그인 수가 매우 적음**
- 1.21.1 + NeoForge 호환 여부는 **각 플러그인별로 개별 확인 필요**
- LuckPerms만 확실히 유지보수 중, 나머지는 업데이트 상태 불확실

---

## 5. 다른 서버 사례 — 설계 참고

### 5.1 TheNodeMC (대형 Pixelmon 서버)

구조:
- 경제 기반 진행 루프 (트레이너 배틀 → 화폐 → 상점)
- 포켓몬 티어 시스템 (수집 목표)
- 계절 이벤트 (Wintertide, Harvest, Halloween)
- 길드/직업 시스템 (장기 목표)
- GitBook 위키로 상세 가이드 제공

우리 서버에 적용 가능한 것:
- ✅ 경제 기반 진행 루프 (규모 축소)
- ✅ 계절/주간 이벤트 (PixelHunt으로 자동화)
- ❌ 길드/직업 (3~8명에게 과함)
- ❌ 상세 위키 (디스코드 핀으로 충분)

출처: https://wiki.thenodemc.com/, https://github.com/TheNodeMC/Pixelmon

### 5.2 소규모 서버의 일반적 구성

대부분의 소규모 Pixelmon 서버는:
1. Pixelmon 내장 기능 (NPC, 퀘스트, 체육관)을 그대로 활용
2. Pixelmon Extras + Broadcasts를 필수 사이드모드로 사용
3. 체육관은 프리셋 또는 직접 빌딩
4. 전설 포켓몬은 config 스폰 + 가끔 수동 이벤트
5. 경제는 포케달러 단일 화폐 + NPC 상점

→ 우리 서버의 방향과 정확히 일치한다.

---

## 6. 결론

> **이 서버의 핵심은 "Pixelmon이 이미 제공하는 걸 잘 조합하는 것"이다.**
>
> Pixelmon 자체가 NPC, 퀘스트, 체육관, 레이드, 전설 스폰, 상점, 브리딩을 전부 내장하고 있고,
> 그 위에 44개의 사이드모드가 있다.
> JSON 설정 + NPC Editor GUI + 맵 빌딩이 작업의 90%, Java 코드를 쓸 일은 거의 없다.
>
> 가장 중요한 선행 작업은 **Phase 1에서 SpongeNeo 호환성 검증**.
> 통과하면 수월, 안 되더라도 사이드모드만으로 상당 부분 커버 가능.
