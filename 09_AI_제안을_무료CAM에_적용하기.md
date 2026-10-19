# 09. AI 제안을 무료 CAM에 적용하기 (AI → Kiri:Moto 파라미터 변환)

## 1. 학습 목표

- AI가 제안한 공정계획(07)과 절삭조건(08)을 [Kiri:Moto](https://www.kiri-moto.com/)의 실제 파라미터로 **정확하게 변환(Translation)**할 수 있다.
- '추상적 제안(Abstract Suggestion)'을 '구체적 설정(Concrete Parameters)'으로 옮기는 훈련을 한다.
- AI 제안의 누락 항목(여유고, 탭, 래피드 높이 등)을 스스로 식별·보완할 수 있다.

## 2. 학습 포인트: "번역(Translation) 훈련"

AI는 자연어/표 형태로 제안하지만, CAM 툴은 **정확한 숫자/드롭다운 값**을 요구합니다.

이번 단계의 핵심은 **"AI의 의도(Intent)를 훼손하지 않으면서, 툴이 요구하는 형식으로 옮기는 것"**입니다.

## 3. 실습 준비

- (07)에서 얻은 가공순서(Operation Sequence)
- (08)에서 얻은 절삭조건(Feeds & Speeds) – **보수적 값 우선 적용**
- Kiri:Moto에서 (06) 수동 실습 때 사용한 L형 브래킷 모델 로드
- 예상 소요시간: 10~12분

## 4. Step 1. 적용 전략 수립 (1분)

AI 제안 전체를 한 번에 복사하지 말고, **Operation 단위로 하나씩 적용·검토**하는 게 안전하고 교육적입니다.

**권장 순서:** (07)의 제안 순서 그대로 → 각 Operation 설정 시 (08)의 해당 구분(Roughing/Finishing) 조건 적용

## 5. Step 2. Setup 확인/조정 (1~2분)

Kiri:Moto `Setup` 탭에서 AI 제안과 일치하는지 확인합니다.

| 항목 | AI 제안(07) 확인 | Kiri:Moto 설정 | 비고 |
|---|---|---|---|
| **Work Zero** | AI가 제안한 원점 위치 | Setup → Origin | AI 근거가 타당하면 따르고, 아니라면 합리적 위치로 조정 |
| **Stock** | +X/Y/Z 여유 | Setup → Stock | 보수적으로 유지(+2~3mm 권장) |
| **Clearance (Safe Z)** | AI가 언급했는지 확인 | Processes/Global | 누락 시 **반드시 보완** (8장 참고) |

## 6. Step 3. Tools(공구) 반영 (1~2분)

`Tools` 탭에 (08) 가정과 일치하도록 설정합니다.

| 항목 | 값 | 비고 |
|---|---|---|
| Tool Type | Flat End Mill | AI 가정과 일치 |
| Diameter | Ø6 mm | 08 가정 유지 |
| Flutes | 2 | Chip Load 계산과 정합성 확인 |
| Length | 충분히 여유 | Z 충돌 방지(교육용 안전 우선) |

> **TIP:** 절삭조건(RPM/Feed/Doc)은 `Processes`에서 Operation별로 설정하므로, Tool은 주로 형상/치수 위주로 정의합니다.

## 7. Step 4. Processes(Operation) 단위 적용 (6~7분) – 핵심

(07)에서 제안된 Operation Sequence를 **하나씩** 생성합니다. 각 Operation마다 (08)의 해당 분류(Roughing/Finishing)를 매칭해 적용합니다.

### 7.1 Kiri:Moto 주요 파라미터 매핑표

| AI 제안 항목 | Kiri:Moto 해당 위치 | 설정 방식 | 비고 |
|---|---|---|---|
| **Process Type** (Pocket/Contour/Drill) | Processes → Type | 드롭다운 선택 | AI 제안 우선, 타당성 없으면 조정 |
| **Spindle RPM** | Processes → RPM/Spindle | 숫자 입력 | 08 Roughing/Finishing 구분 적용 |
| **Feed Rate** | Processes → Feed Rate (mm/min) | 숫자 입력 | 단위(mm/min) 확인 |
| **Depth of Cut (DOC/Ap)** | Processes → Depth/Step Down | mm 입력 | 한 번에 절입 깊이 |
| **Stepover (Ae)** | Processes → Step Over | mm 또는 % | %로 입력 가능 시 Tool Dia.%로도 확인 |
| **Clearance/Rapid Z** | Processes → Clearance/Rapid Height | mm | **AI가 누락 시 직접 보완 필수** (안전) |
| **Plunge Rate** | Processes → Plunge Feed | mm/min | Feed보다 보수적으로(보통 50% 이하) 권장 |
| **Tabs** | Processes → Tabs | On/Off | 부품 이탈 방지(Contour 최종 컷 시 고려) |
| **Direction (Climb/Conventional)** | Processes → Direction/Milling | 선택 | 일반적으로 Climb Milling 선호(AL6061) |
| **Stock to Leave** | Processes → Allowance/Leave | mm | Finishing 시 0.1~0.2mm 남기기 전략 등 |

### 7.2 Operation별 적용 가이드 (L형 브래킷 예시)

| Operation (07 제안) | Type | RPM/Feed | DOC/Stepover | 적용 시 체크포인트 |
|---|---|---|---|---|
| **1. Facing(선택)** | Face/Pocket | Roughing 기준(보수적) | 얕은 DOC, 넓은 Stepover | 불필요 시 생략 OK(교육시간 고려) |
| **2. Pocket(내부)** | Pocket | Roughing (08) | 보수적 DOC, Stepover 40~60% Dia. 권장 | **먼저 적용**(재료 제거량 많음) → 경로 안정성 확인 |
| **3. Contour/Outline** | Contour/Profile | Finishing (08) | 얕은 DOC, 작게 Stepover | **최종 형상**. Tabs 고려(부분 잔류)로 이탈 사고 예방 |
| **4. Drill(홀 유무)** | Drill/Hole | Drill 권장값 | Pecks/Peck Depth | 모델에 홀 없으면 SKIP |

> **주의:** AI가 제안한 Stepover % 표현을 mm로 환산할 때 (Ø6mm × 50% = 3.0mm) 오차 없이 입력했는지 확인하세요.

## 7. Step 5. 누락 항목 체크리스트 (스스로 보완)

AI 답변은 CAM 툴의 모든 세부 필드를 커버하지 않는 경우가 많습니다. **학생 스스로 보완**하게 유도하는 게 교육적입니다.

| 누락 가능 항목 | 권장값 (교육용, 안전우선) | 이유 |
|---|---|---|
| **Clearance (Rapid Z)** | 10~15 mm | 공구-클램프/작업물 충돌 방지 |
| **Retract Height** | 5~10 mm | 안전한 이송 |
| **Plunge Feed** | Feed Rate의 30~50% 수준 | 플런지 충돌/부하 완화 |
| **Tabs (Contour 최종컷)** | On, 2~3개, 폭 3~4mm, 두께 1~2mm | Contour 마지막 패스 시 부품 이탈 방지 |
| **Lead-in/Lead-out** | 기본값 유지 또는 소폭 | 교육용 단순화, 시간 고려 |

**유도 질문:** "AI 제안에는 없지만, Kiri:Moto에 꼭 설정해야 할 안전 관련 파라미터는 무엇이 있을까?" → 학생들이 스스로 찾아내게 하세요.

## 8. Step 6. 툴패스 생성 및 1차 검토 (2분)

1. `Generate`로 툴패스 계산
2. **시각적 점검** (Preview/Sim 뷰)
   - [ ] 가공 순서(Operation 순서)가 논리적인가? (07 제안과 일치)
   - [ ] 공구가 작업물/클램프 영역과 충돌할 여지는 없는가?
   - [ ] Clearance가 충분해 보이는가?
   - [ ] Pocket→Contour 순서가 의도대로 적용되었는가?
3. 의심되는 부분이 있으면 **파라미터를 보수적으로 조정**한 뒤 재생성

## 9. Step 7. G코드 내보내기

1. `Export → GCode (.nc/.gcode)` 저장
2. 파일명: `bracket_L_AI_assisted.nc` (수동안 `bracket_L_manual.nc`과 구분)

## 10. 핵심 교육 포인트

- **"적용은 수정과 동시"**: 한 번에 100% 정확히 옮기기보다, **의심스러운 항목은 보수적으로 조정**하면서 적용
- **AI 의도 보존**: 숫자만 기계적으로 복사하지 말고, **AI가 왜 그 값을 제안했는지(근거)**를 떠올리며 적용
- **능동적 보완 훈련**: 누락 항목(7장 체크리스트)을 학생 스스로 찾아내게 유도 → **비판적 사고력** 향상
- **안전 우선 원칙**: 애매하면 **보수적으로(Conservative)** 설정

## 11. 핵심 takeaway

> **AI+CAM 실습의 실질적 가치는 '제안을 이해하고, 툴 파라미터로 올바르게 번역·보완하는 능력'에 있다.**

이제 (09)에서 생성한 `AI_assisted.nc` G코드를 (10) CAMotics로 시뮬레이션하여, **AI 제안이 실제로 안전/타당한지 검증**하는 단계로 넘어갑니다.
