# 10. G코드 검증과 CAMotics 활용

## 1. 학습 목표

- [CAMotics](https://camotics.org/)를 사용해 G코드를 시각적으로 검증할 수 있다.
- AI가 제안한 절삭조건/공정이 실제로 안전한지 사전 탐지할 수 있다.
- **"신뢰하되 검증하라(Trust but Verify)"** 원칙을 실천적으로 체득한다.
- AI에게 G코드를 리뷰(리딩) 요청해 위험요소를 빠르게 점검하는 방법을 익힌다.

## 2. 왜 검증이 필수인가?

(08)에서 강조했듯, **AI는 절삭조건의 '참고값'을 제시할 뿐 '최종 안전 보장값'은 아닙니다.**

검증 단계는 **Human-in-the-Loop**의 가장 중요한 실행 지점입니다.

| 위험 요소 | 원인 | 검증 방법 |
|---|---|---|
| **공구-공작물 충돌** | Clearance/Retract 부족, Z 설정 오류 | CAMotics 3D 시뮬레이션 |
| **과도한 절삭부하** | DOC/Stepover/RPM-Feed 불균형 | 툴패스 밀도/칩형상 관찰 + 논리 검토 |
| **부품 이탈** | Contour 최종컷 시 Tabs 누락 | 마지막 패스 처리 확인 |
| **비효율/비논리 경로** | 공정순서 오류 | 공구경로 동선(Traverse) 확인 |
| **공구 파손 위험** | Plunge Feed 과도, Chip Load 비정상 | Plunge 동작/부하 우려 구간 확인 |

## 3. 실습 준비

- **툴:** [CAMotics](https://camotics.org/) (로컬 설치 권장, Windows .exe)
- **G코드:** (09)에서 내보낸 `bracket_L_AI_assisted.nc`
- **비교용(선택):** (06) 수동안 `bracket_L_manual.nc`
- **예상 소요시간:** 8~10분

## 4. Step 1. CAMotics 프로젝트 생성 (1분)

1. CAMotics 실행 → `New Project` (또는 File → New)
2. `Units` 확인: mm (G코드가 mm 단위인지 확인)
3. `Machine`는 기본값(3-axis Mill) 사용 (교육용 단순화)
4. G코드 불러오기: `Open GCode` → `bracket_L_AI_assisted.nc` 선택

## 5. Step 2. 시뮬레이션 환경 설정 (1~2분)

1. `Workpiece (Stock)` 설정: Kiri:Moto Setup의 Stock 크기와 대략 일치시켜 시각적 비교 용이하게
   - 예: L형 브래킷 외곽 +3mm X/Y/Z
2. `Tool` 확인: CAMotics가 G코드의 T코드/공구정보를 읽어오지만, 시각적 확인용으로 대략 Ø6mm End Mill 확인
3. 뷰(View)를 **Isometric(등각투상) + Orthographic** 전환하며 관찰하기 용이하게 설정

## 6. Step 3. 가공 시뮬레이션 실행 (3~4분) – 핵심 검증

`Simulate`(재생) 버튼으로 천천히 실행하며 **아래 항목을 순서대로 체크**하세요.

### 6.1 검증 체크리스트 (필수)

| 검증 항목 | 확인할 포인트 | Pass/Fail 기준 |
|---|---|---|
| **1. 초기 Clearance** | 공구가 첫 이동 시 Z가 안전하게 상승(Rapid)되는가? | Rapid Z가 Stock 상단+여유 이상으로 가는가? |
| **2. 공정 순서** | Operation 순서가 (07) 제안대로 진행되는가? (Pocket→Contour 등) | 의도한 순서와 일치 |
| **3. Pocket 가공** | 내부 Pocket이 의도 영역만 제거하는가? | 과도/미달 제거 없는지, 남는 섬(Island) 처리 적정 |
| **4. Plunge 동작** | 공구가 수직/경사 플런지하는가? 급격한 수직 plunging은 없는가? | 부드러운 진입, Plunge Feed 보수적 |
| **5. Contour(윤곽)** | 외곽 최종 패스에서 **Tabs 존재 여부** 확인 | 최종 컷 부품 이탈 위험 체크 |
| **6. 충돌 체크(Collision)** | 공구(Shank/Holder 가정 포함)와 Workpiece/Stock 경계에서 비정상 침투 없는가? | 절대 침투 또는 공구바디 충돌 의심 구간 확인 |
| **7. 남는 재료** | 의도하지 않은 잔여(Material Left) 영역 없는가? | Pocket 미처리, Step Over 간격 비정상 등 |
| **8. 공구 이송(Rapid/Feed)** | Feed 구간과 Rapid 구간이 논리적으로 구분되는가? | 비효율적 장거리 Rapid 과다 여부 |
| **9. 전체 동작 안정성** | 시뮬레이션 중 비정상적인 파고듦(Undercut/Overcut) 없는가? | 시각적으로 부자연스러운 구간 확인 |
| **10. 최종 형상** | 가공 완료 후 형상이 의도한 L형 브래킷과 유사한가? | 치수적/형상적 결함 여부 |

### 6.2 관찰 포인트 (교육적 유도)

- **천천히(Slow Motion)** 재생하며 의심 구간을 **일시정지(Pause)** → 토론 유도
- **다양한 뷰(Top/Front/ISO)**로 같은 구간을 반복 확인
- **수동안(`manual.nc`)과 AI보조안(`AI_assisted.nc`)을 비교**하며 관찰

## 7. Step 4. AI에게 G코드 리뷰 요청 (1~2분)

CAMotics에서 **일시정지한 의심 구간**의 G코드를 40~80줄 정도 복사해 AI에게 검토를 요청합니다.

### 7.1 G코드 리뷰 프롬프트 템플릿

```text
너는 CNC G코드 안전검토 리뷰어야.

아래 G코드를 가공 안전성 관점에서 검토해줘.

[GCODE_START]
(여기에 의심 구간 G코드 40~80줄 붙여넣기)
[GCODE_END]

[검토 항목]
- Tool collision(Z/Clearance/Retract)
- Excessive load(DOC/Feed/Plunge)
- Part detachment risk(Tabs/Final contour)
- Abnormal moves(G0/G1, rapid/feed)
- Other critical risks

[요청사항]
1. 위험도(High/Medium/Low) 분류
2. 의심 구간(가능하면 대략적 위치/특징 언급)
3. 구체적이고 실행 가능한 개선 제안(Actionable Suggestion)
4. 참고 의견 형태로 간결하게 (검증 우선순위: 시뮬레이션 CAMotics > AI 리뷰)
```

> **주의:** AI 리뷰는 보조 수단입니다. 최종 판단은 항상 CAMotics 시뮬레이션과 실제 가공 조건을 기준으로 합니다.

## 8. Step 5. 결과 정리 및 기록

검증에서 발견한 사항을 정리합니다(11단계 토론에 활용).

- 발견된 이상/위험 요소(예: 여유고 부족, 과도한 절삭부하 의심 구간)
- 수정 필요 여부 판단 → 필요 시 Kiri:Moto로 돌아가 파라미터 보수적 조정 후 재검증
- 수동안 vs AI보조안 차이점 메모

## 9. 핵심 교육 포인트

- **"검증 없는 적용 금지"**: AI 제안이 아무리 좋아 보여도, 시뮬레이션 검증은 필수
- **비판적 검토**: AI가 만든 G코드도 오류/비효율이 있을 수 있음을 전제
- **시각적 확인 습관**: 툴패스를 눈으로 보며 이상 구간을 찾는 습관이 안전의 시작

## 10. 핵심 takeaway

> **"AI가 제안한 모든 값은 검증 대상이다. 신뢰하되 반드시 검증하라."**

이 검증 단계는 본 세미나의 핵심 메시지인 **비판적 AI 리터러시**를 실천으로 옮기는 지점입니다.
