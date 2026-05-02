# YAMNet + 분류기 11종 소리 분류 과정

## 개요

기존에는 모든 소리를 우리 분류기(11개 카테고리)에 직접 넣었기 때문에,
대화 소리, 음악 등 관련 없는 소리도 11개 중 하나로 강제 분류되는 문제가 있었다.

이를 해결하기 위해 **YAMNet(521개 카테고리)을 1차 필터로 사용**하고,
통과된 소리만 2차로 처리하는 구조로 변경한다.

---

## 소리 분류 구조

```
마이크 녹음 (1초, 48kHz → 16kHz 리샘플링)
    ↓
━━━ 1차: YAMNet (521개 카테고리 분류) ━━━
    ↓
    ├── [차단] Speech, Music, Laughter 등 → 무시 (처리 안 함)
    │
    ├── [YAMNet 직접 매핑 - 7개]
    │   Bark(70)         → "개 짖음" 확정 → 서버 전송
    │   Baby cry(20)     → "아기 울음" 확정 → 서버 전송
    │   Water(282)       → "물 소리" 확정 → 서버 전송
    │   Knock(353)       → "노크 소리" 확정 → 서버 전송
    │   Doorbell(349)    → "초인종" 확정 → 서버 전송
    │   Shatter(437)     → "창문 깨지는 소리" 확정 → 서버 전송
    │   Ringtone(385)    → "휴대폰 벨소리" 확정 → 서버 전송
    │
    ├── [2차 분류기로 넘김 - 4개]
    │   Door(348), Beep(475), Siren(390) 등 관련 소리
    │       ↓
    │   ━━━ 2차: Hearo 분류기 ━━━
    │       → "도어락" / "인터폰" / "가전제품" / "사이렌" 판단
    │       → threshold(90%) 이상이면 서버 전송
    │
    └── [기타] 위 어디에도 해당 안 됨 → 무시
```

---

## YAMNet 521개 카테고리 분류 기준

### 차단 목록 (확실히 무관한 소리 - 무시)

| 인덱스 범위 | 분류 | 설명 | 예외 |
|------------|------|------|------|
| 0~18 | 사람 소리 | Speech, Laughter, Singing 등 | 없음 |
| 21~66 | 사람 소리 (계속) | Whimper, Sigh, Cough, Footsteps 등 | 없음 |
| 67~68 | 동물 (상위) | Animal, Domestic animals | 없음 |
| 74~131 | 동물 (개 짖음 외) | Cat, Horse, Bird, Insect 등 | 없음 |
| 132~276 | 음악 전체 | Music, Guitar, Piano, Pop, Jazz 등 | 없음 |
| 277~281 | 날씨 | Wind, Thunderstorm, Thunder | 없음 |
| 283~291 | 물 관련 (세부) | Rain, Waterfall, Ocean, Steam 등 | 282(Water) 는 통과 |
| 292~293 | 불 | Fire, Crackle | 없음 |
| 294~347 | 차량/교통 | Car, Truck, Train, Aircraft 등 | 없음 |
| 398~419 | 기계/도구 | Clock, Gears, Hammer, Drill 등 | 없음 |
| 431~434 | 나무 | Wood, Chop, Splinter, Crack | 없음 |
| 438~474 | 액체/기타 물리음 | Liquid, Splash, Bounce, Tear 등 | 없음 |
| 494~520 | 환경/효과음 | Silence, White noise, Reverb 등 | 없음 |

### 통과 목록 (우리 카테고리와 관련 있는 소리)

#### YAMNet 직접 매핑 (YAMNet 결과 → 우리 카테고리로 바로 변환)

| YAMNet 인덱스 | YAMNet 카테고리명 | → Hearo 카테고리 |
|--------------|------------------|-----------------|
| 19 | Crying, sobbing | 아기 울음 |
| 20 | Baby cry, infant cry | 아기 울음 |
| 69 | Dog | 개 짖음 |
| 70 | Bark | 개 짖음 |
| 71 | Yip | 개 짖음 |
| 73 | Bow-wow | 개 짖음 |
| 282 | Water | 물 소리 |
| 349 | Doorbell | 초인종 |
| 350 | Ding-dong | 초인종 |
| 353 | Knock | 노크 소리 |
| 354 | Tap | 노크 소리 |
| 384 | Telephone bell ringing | 휴대폰 벨소리 |
| 385 | Ringtone | 휴대폰 벨소리 |
| 435 | Glass | 창문 깨지는 소리 |
| 437 | Shatter | 창문 깨지는 소리 |
| 463 | Smash, crash | 창문 깨지는 소리 |
| 464 | Breaking | 창문 깨지는 소리 |

#### 2차 분류기로 넘기는 소리 (도어락/인터폰/가전/사이렌 관련)

| YAMNet 인덱스 | YAMNet 카테고리명 | 관련 Hearo 카테고리 |
|--------------|------------------|-------------------|
| 348 | Door | 도어락 |
| 351 | Sliding door | 도어락 |
| 352 | Slam | 도어락 |
| 355 | Squeak | 도어락 |
| 356 | Cupboard open or close | 가전제품 |
| 357 | Drawer open or close | 가전제품 |
| 358 | Dishes, pots, and pans | 가전제품 |
| 359 | Cutlery, silverware | 가전제품 |
| 360 | Chopping (food) | 가전제품 |
| 361 | Frying (food) | 가전제품 |
| 362 | Microwave oven | 가전제품 |
| 363 | Blender | 가전제품 |
| 364 | Water tap, faucet | 물 소리 |
| 365 | Sink (filling or washing) | 물 소리 |
| 366 | Bathtub (filling or washing) | 물 소리 |
| 367 | Hair dryer | 가전제품 |
| 368 | Toilet flush | 가전제품 |
| 369 | Toothbrush | 가전제품 |
| 370 | Electric toothbrush | 가전제품 |
| 371 | Vacuum cleaner | 가전제품 |
| 373 | Keys jangling | 도어락 |
| 376 | Electric shaver, electric razor | 가전제품 |
| 382 | Alarm | 사이렌 |
| 383 | Telephone | 인터폰 |
| 386 | Telephone dialing, DTMF | 인터폰 |
| 387 | Dial tone | 인터폰 |
| 388 | Busy signal | 인터폰 |
| 389 | Alarm clock | 사이렌 |
| 390 | Siren | 사이렌 |
| 391 | Civil defense siren | 사이렌 |
| 392 | Buzzer | 인터폰 |
| 393 | Smoke detector, smoke alarm | 사이렌 |
| 394 | Fire alarm | 사이렌 |
| 395 | Foghorn | 사이렌 |
| 396 | Whistle | 사이렌 |
| 406 | Mechanical fan | 가전제품 |
| 407 | Air conditioning | 가전제품 |
| 420 | Explosion | 창문 깨지는 소리 |
| 428 | Burst, pop | 창문 깨지는 소리 |
| 430 | Boom | 창문 깨지는 소리 |
| 436 | Chink, clink | 창문 깨지는 소리 |
| 475 | Beep, bleep | 도어락 |
| 476 | Ping | 도어락 |
| 477 | Ding | 초인종 |
| 478 | Clang | 가전제품 |
| 479 | Squeal | 가전제품 |
| 484 | Sizzle | 가전제품 |
| 489 | Jingle, tinkle | 초인종 |
| 490 | Hum | 가전제품 |

---

## 왜 이 구조인가?

### 기존 문제점
1. 대화 소리가 "사이렌"으로 분류됨
2. 음악이 "휴대폰 벨소리"로 분류됨
3. 무음이 "창문 깨지는 소리 100%"로 분류됨
4. → softmax 특성상 항상 11개 중 하나로 강제 분류

### 해결 방법
1. YAMNet이 먼저 소리의 종류를 판단 (521개 중)
2. 확실히 무관한 소리(대화, 음악 등)는 1차에서 차단
3. 관련 소리만 2차로 넘겨서 정확한 분류 수행
4. 차단 범위를 넓게 잡아서 한국 가정음이 누락되지 않도록 함

### YAMNet vs 우리 분류기 역할 분담
| 항목 | YAMNet | 우리 분류기 |
|------|--------|------------|
| 학습 데이터 | 수백만 개 (AudioSet) | ~300개 (직접 수집) |
| 카테고리 수 | 521개 | 11개 |
| 강점 | 범용 소리 구분 | 한국 가정음 특화 |
| 담당 카테고리 | 7개 (직접 매핑) | 4개 (도어락, 인터폰, 가전, 사이렌) |
