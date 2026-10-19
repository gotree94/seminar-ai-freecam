# 08. AI로 공구와 절삭조건(Feeds & Speeds) 추천받기

## 1. 학습 목표

- AI에게 공구(Tool) 선정 방향과 절삭조건(Spindle RPM, Feed Rate, Depth of Cut, Stepover 등)을 제안받을 수 있다.
- **AI 추천값의 한계**를 인지하고, **비판적 검토(Critical Review)**하는 태도를 기른다.
- '참고값(Reference)'과 '적용값(Applied)'을 구분하여 안전하게 활용하는 방법을 익힌다.

## 2. Feeds & Speeds란?

절삭조건(Feeds & Speeds)은 가공성, 표면조도, 공구수명, 가공시간에 직접적인 영향을 줍니다.

| 항목 | 의미 | 단위(일반) |
|---|---|---|
| **Spindle Speed (RPM)** | 주축 회전수 | rev/min (RPM) |
| **Feed Rate (Feed)** | 이송속도 | mm/min 또는 mm/rev |
| **Depth of Cut (DOC, Ap)** | 절입깊이(축방향) | mm |
| **Stepover (Width of Cut, Ae)** | 피치/절삭폭(반경방향) | mm (또는 % of Tool Dia.) |
| **Chip Load (Feed per Tooth)** | 날당 이송량 | mm/tooth |
| **Cutting Speed (Vc)** | 절삭속도 | m/min |

## 3. 실습 준비

- LLM 세션 유지 (07과 연계 권장)
- L형 브래킷 STEP 파일 (참고용)
- 예상 소요시간: 7~8분

## 4. Step 1. 구체적인 조건 명시하기 (정확도 향상)

AI가 **일반론이 아닌 실용적 추천값**을 주도록 하려면, 가정을 명확히 제시해야 합니다.

### 4.1 AI에 전달할 조건

| 항목 | 권장 예시 | 이유 |
|---|---|---|
| **소재** | `Aluminum 6061-T6` | 절삭성 차이 큼 |
| **공작기계** | `3-axis vertical CNC mill, medium rigidity (hobby/educational)` | 강성(Rigidity)에 따라 권장 DOC 달라짐 |
| **공구** | `Ø6mm flat end mill, 2-flute, solid carbide (or HSS)` | Chip Load 계산에 필요 |
| **가공유형** | `Roughing (Pocket) + Light Finishing (Contour)` 구분 요청 | 전략별로 조건 다름 |
| **Coolant** | `Air/None (Dry) or Flood` (실습환경 가정) | 마찰/칩배출 영향 |
| **안전 우선순위** | `Safety > Tool Life > Cycle Time (교육용, 보수적 우선)` | 과도한 공격적 값 방지 |
| **단위** | `mm, mm/min, RPM` 명시 | 혼동 방지 |

## 5. Step 2. 프롬프트 작성 (3~4분)

### 5.1 프롬프트 템플릿 (Feeds & Speeds 추천)

```text
너는 CNC 밀링 절삭조건(Feeds & Speeds) 추천 보조야.

다음 조건을 바탕으로, 교육용(보수적·안전 우선)으로 추천해줘.

[가공 조건]
- 부품: L형 브래킷 (STEP 첨부, 3-axis milling)
- 소재: Aluminum 6061-T6
- 머신: 3-axis vertical mill, 중간 강성(rigidity) 가정 (교육/실습용)
- 공구: Ø6 mm Flat End Mill, 2-flute, Solid Carbide 가정
- 가공 구분: Roughing(Pocket) / Finishing(Contour) 각각 별도 제시
- Coolant: Dry (무절삭유) 가정
- 우선순위: Safety(안전) > Tool Life(공구수명) > Cycle Time(가공시간)
- 단위: mm, mm/min, RPM

[요청사항]
1. Roughing(Pocket), Finishing(Contour) 각각에 대해 아래 항목을 제시해줘
   - Spindle RPM
   - Feed Rate (mm/min)
   - Depth of Cut (DOC, Ap) [mm]
   - Stepover (Ae) [mm] (또는 Tool Dia. 대비 %로도 표시)
   - Chip Load (mm/tooth) (계산식 간단히 포함)
2. 추천 근거(Why)를 1~2줄씩 간단히 설명해줘
3. Chip Load 계산 근거도 간단히 보여줘 (공식: Feed = RPM × Flutes × ChipLoad)
4. 안전 마진(Safety Margin)을 고려한 '보수적(Conservative)' 값 위주로 추천해줘
5. "이 값은 참고값(Reference)일 뿐, 실제 기계/작업조건에 따라 조정 필요" 라는 경고를 반드시 포함해줘
6. 추천값이 너무 공격적이지 않은지 이유도 설명해줘

답변은 표 형식 위주로 간결하게 해줘.
```

## 6. Step 3. AI 응답 검토 – 가장 중요한 '비판적 검토' (3~4분)

08단계의 핵심은 맹신 금지, 비판적 수용입니다. 반드시 아래 항목을 짚고 넘어가세요.

### 6.1 타당성 검토 체크리스트

| 항목 | 확인 방법 | 비고 |
|---|---|---|
| Chip Load 합리성 | AI가 제시한 Chip Load이 일반적 범위(AL6061, 2F, Ø6 Carbide)에 부합하는가? (대략 0.01~0.05 mm/tooth 수준) | 터무니없이 높으면 즉시 의심 |
| RPM/Feed 일관성 | Feed = RPM × Flutes × Chip Load 공식이 일치하는가? (AI 답변의 계산식 확인) | 계산 오류 체크 |
| DOC/Stepover 현실성 | 교육용(중간강성) 가정에서 DOC가 너무 깊은가? (예: Ø6 Roughing DOC 2.0~3.0mm 보수적) | 강성 가정 명시한 점 활용 |
| Roughing/Finishing 구분 | Finishing이 Roughing보다 확실히 보수적으로 제안되었는가? | 논리성 확인 |
| 안전성 강조 | "참고값" 경고가 포함되었는가? | 누락 시 추가 질의 |
| 과도한 공격성 | Cycle Time 최적화보다 Safety 우선 요청이 반영되었는가? | 프롬프트 의도 확인 |

### 6.2 필수 추가 질의 (권장 1개)

> 이 추천값을 무조건 Kiri:Moto에 적용해도 안전하다고 생각하나요? 그렇지 않다면, 적용 전 반드시 확인/조정해야 할 점을 3가지 이내로 구체적으로 말해줘.

이 질문만으로도 AI는 본인 답변의 한계와 주의사항을 스스로 명시하게 되어, 교육 의도("신뢰하되 검증하라")가 자연스럽게 체득됩니다.

## 6. 안전 원칙 (반드시 강조)

1. **참고값(Reference), 적용값(Applied) 구분**
   AI 추천값은 **시작점(Starting Point)**이지 최종값이 아닙니다. 실제 공작기계 상태(진동, 툴 홀더, 클램핑, 냉각 등)를 모르는 AI가 최종 절삭조건을 보장할 수 없습니다.
2. **무조건 복사-붙여넣기 금지**
   "AI가 추천했으니 안전하다"는 논리는 위험합니다. 항상 보수적으로 조정하고, 다음 단계(10. CAMotics 시뮬레이션)에서 검증합니다.
3. **Trust but Verify (신뢰하되 검증하라)**
   이 원칙을 제조(AI+CAM) 맥락에 적용하는 것이 이번 세미나의 핵심 AI 리터러시입니다.

## 7. 교육적 포인트

- **절삭이론 몰라도 충분**: 계산식(Chip Load) 확인하게 유도하면, AI 답변의 논리적 일관성 체크 훈련이 됩니다.
- **위험 인식 훈련**: 공격적 값 vs 보수적 값 대화하면서 Trade-off(트레이드오프) 개념 자연스럽게 학습
- **책임성 강조**: AI가 스스로 한계/경고를 언급하도록 유도(6.2 질문)하는 게 가장 효과적

## 8. 핵심 takeaway

AI는 절삭조건의 '합리적 출발점'을 제시하는 데 유용하나, '최종 안전값'을 보장하지 않는다. 따라서 적용 전에는 반드시 시뮬레이션/검증이 필수다.

(08)에서 이 태도를 명확히 정립한 뒤, (09)에서 AI 제안을 실제 CAM에 적용하고, (10)에서 CAMotics로 검증하는 흐름이 자연스럽게 이어집니다.
