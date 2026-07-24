---
name: proposal-writer
description: 한글(HWP/HWPX) 양식에 맞춘 제안서·보고서·예산근거 등 구조화 문서를 작성하는 범용 문서 작성 엔진. 특정 목차·기호에 고정되지 않고 양식에서 위계·글머리표를 추출해 처리한다. 파이프라인 - 양식 분석 → 위계 모델링 → 다중자료 근거화 → 미디어 배치 → 조립 → 자동 검증·수정. 트리거 - "제안서 작성", "양식대로 작성", "예산 근거 작성", "보고서를 한글로", "이 양식에 맞춰서", "제안서/보고서 꼭지 추가", "proposal", "HWPX 문서".
---

# Proposal Writer — 범용 HWPX 문서 작성 엔진

**이 스킬은 "특정 보고서를 잘 만드는 규칙"이 아니라, 아래 파이프라인을 수행하는 범용 엔진이다.**

```
양식 분석 → 위계 모델링 → 다중자료 근거화 → 미디어 배치 → 문서 조립 → 자동 검증·수정
```

특정 문서의 관습(예: 번호가 9까지 이어짐, `■/○/-/·` 기호, 특정 월·분야 명칭)을 규칙으로 굳히지 않는다.
그런 사례는 `tests/regression-cases.md`의 **회귀 테스트**로만 사용한다.

## 3대 불변 원칙

1. **양식 먼저.** 작성 전 양식(HWP/HWPX)을 요청한다. 없으면 임의 서식으로 만들지 않는다.
2. **서식 교차검증.** 결과물을 원본 양식·입력 목차와 대조하는 범용 불변조건(§7)을 통과해야 완성이다.
3. **출처 교차검증.** 조사 초안을 그대로 쓰지 않는다. 수치·출처·계산·귀속을 웹/제공자료로 반박 검증한다.

---

## 1) 목차가 아니라 '문서 위계'를 처리

번호 체계를 고정하지 말고, 양식·사용자 요구에서 위계를 추출한다. 표준 역할:

```
document_title · chapter · section · subsection · detail ·
group_heading · body · body_child · body_grandchild · caption · source
```

규칙:
- 사용자 제공 목차가 있으면 **최우선** 적용
- 없으면 양식의 기존 제목 위계를 추출
- `I. / 1. / 1.1. / 가. / 1) / ■` 등은 **역할이 아니라 표현 방식**으로 취급
- 자동번호와 텍스트 번호 중 **하나만** 선택(혼용 금지)
- 상위 항목 변경 시 하위 번호 재시작 규칙을 양식에서 추론
- 결과 제목 순서를 입력 목차와 **구조적으로 비교**

## 2) 글머리표를 기호가 아니라 역할로 관리

`■→○→-→·`를 고정하지 말고 **양식에서 실제 기호를 추출**한다.

```
styleRoles: {
  groupHeading:   { marker: <양식 추출>, charPr, paraPr },
  body:           { marker: <양식 추출>, charPr, paraPr },
  bodyChild:      { marker: <양식 추출>, charPr, paraPr },
  bodyGrandchild: { marker: <양식 추출>, charPr, paraPr },
  spacer:         { marker: null, paraPr: <비글머리표 스타일> }
}
```

필수:
- 빈 줄에는 글머리표 스타일 사용 금지(spacer 사용)
- 내용 없는 글머리표 문단 생성 금지
- 모든 본문을 같은 글머리표로 평탄화 금지(위계 보존)
- 하위 문단은 반드시 상위 문단 뒤에 배치
- 들여쓰기는 공백이 아니라 **문단 속성(paraPr)**으로 설정

## 3) 자료 통합을 '출처 범위'와 무관하게 일반화

특정 자료가 아니라 여러 참고자료의 공통 처리 구조:

```
sourceDocuments: [
  { path, type: "hwpx|hwp|pdf|docx|xlsx|image",
    label, priority, allowedUses: ["text","table","figure","data"] }
]
```

절차: ① 자료별 텍스트·표·그림·수치 추출 → ② 사용자 목차 × 출처 내용의 **근거 매트릭스** 작성 →
③ 중복 병합 → ④ 충돌 수치·설명 표시 → ⑤ 근거 부족 영역 탐지 → ⑥ 사용/미사용 자료 기록.
즉 "모든 자료 반영"이 아니라 **"제공된 모든 중요 자료의 활용 여부 검증"**.

## 4) 내용 충실도를 설정값으로 관리

문단 수를 고정하지 말고 목적에 따라 밀도를 정한다.

```
contentPolicy: {
  detailLevel: "brief|standard|detailed",
  minimumEvidencePerSection: 2,
  minimumConcreteExamples: 1,
  quantitativeEvidence: "when-available|required|optional",
  repetitionThreshold: 0.2,
  paragraphStyle: "outline|narrative|mixed"
}
```

검사: 제목만 있고 본문 없는 절 / 근거 없이 선언만 반복 / 동일 표현 반복 / 지나치게 긴 단일 글머리표 /
수치만 있고 의미 설명 없음 / 근거자료 없는 확정 수치 / 요구 상세도보다 현저히 짧은 내용.

## 5) 그림을 '미디어 객체'로 관리

```
mediaItems: [
  { id, sourcePath, sourceDocument, caption, sourceText,
    targetSection, placement: "after|before|inline",
    boxStyle: "template|bordered|none", fitMode: "contain|width|original" }
]
```

규칙: 참고문서의 그림도 추출 / 그림+캡션+출처 함께 추출 / 관련 절에 **의미 기반** 배치 /
양식 그림박스 있으면 복제, 없으면 기본 박스 / 캡션·출처는 그림과 같은 컨테이너 /
그림 번호 문서 전체 연속 / **그림 수 = 캡션 수 = 매니페스트 참조 수** 상호 검증.

## 6) 양식 보존과 내용 확장을 분리 (명시 필수)

```
layoutPolicy: {
  mode: "strict-preserve|adaptive|content-first",
  pageGrowth: "forbid|minimize|allow",
  fontResize: "forbid|allow-with-limit",
  paragraphMerge: "forbid|allow",
  tableResize: "preserve|adaptive"
}
```

- `strict-preserve`: 원본 문단·표·쪽 흐름 우선(기존 문서 꼭지 삽입의 기본)
- `adaptive`: 양식 유지하되 필요한 문단·그림 추가
- `content-first`: 충실도 우선, 페이지 증가 허용

미지정 시 "양식 유지"와 "내용 대폭 보강"이 충돌하므로 반드시 선택(기본 `adaptive`).

## 7) 검증 = 범용 불변조건 (문서 제목·기호에 의존하지 않음)

- 입력 목차 순서 = 결과 제목 순서
- 제목 위계 역전 없음
- 자동번호·수동번호 중복 없음
- 빈 글머리표 0개 / 고아 하위 문단 0개
- 문단·표·그림 ID 중복(증가) 0개
- 그림 수 = 캡션 수 = 그림 매니페스트 참조 수
- 출처 필요한 그림의 출처 누락 0개
- ZIP·XML 무결성 통과 / `fill_hwpx.py check --strict` ok
- 사용자 지정 문체(개조식·서술식) 준수
- 요구 목차의 누락·추가·순서 변경 **보고**

## 8) 자동 수정 루프 (기본값)

```
분석 → 초안 → 조립 → 구조 검증 → 시각·문체 검증
                  ↑                        ↓
                  └──── 원인별 자동 수정 ────┘
```

실패 시 자동 조치: 빈 글머리표 삭제 / 번호 중복 → 번호 정책 재적용 /
그림 누락 → BinData·매니페스트 재등록 / 내용 부족 → 근거 매트릭스에서 보강 /
레이아웃 초과 → `layoutPolicy`에 따라 조정. **사용자 선택이 필요할 때만 중단.**

---

## 실행

1. AskUserQuestion으로 확인(있으면 건너뜀): 양식 파일 / 목차(또는 양식에서 추출) /
   참고자료(sourceDocuments) / 레이아웃 정책 / 상세도.
2. `Workflow({ scriptPath: '<이 스킬>/scripts/pipeline_general.js', args: {...} })`
   args: formPath, itemsDir, itemIdx, context, hierarchy|outline, styleRolesOverride,
   sourceDocuments, contentPolicy, mediaItems, layoutPolicy, verifyPolicy.
3. 불변조건(§7) 전부 통과 후 전달. 미통과 항목은 자동수정(§8), 불가한 것만 사용자에게 보고.

## HWPX 렌더링 함정 (엔진이 자동 처리 — 표현 방식과 무관하게 항상 적용)

| 함정 | 해결 |
|---|---|
| margin이 hh 네임스페이스 → 내어쓰기 미적용 | hc margin을 hp:switch(case −744/1000, default −1488/2000)로. 자식 순서 align→heading→breakSetting→autoSpacing→switch→border |
| fillBrush가 hh 네임스페이스 → 표 음영 미표시 | hc:fillBrush + hc:winBrush, borderFills itemCnt +1 |
| 이중 escape → `&apos;` 노출 | `<hp:t>`에 원문 1회만 escape |
| 중복 id → 한글 크래시 | 새 id는 문서 최대 id 초과에서 발급 |
| 섹션 제목 간격 없음 | 제목 앞 spacer(빈 줄) |
| 긴 URL 난잡 | 제목 줄 + URL 줄(회색, intent=0) 2단 |
| 이미지 미표시 | BinData + content.hpf 매니페스트 등록 |

## 스킬과 테스트 분리

세션 사례는 SKILL.md에 규칙으로 넣지 않고 `tests/regression-cases.md`에 회귀 테스트로 보관한다.

## 의존성

hwpx 스킬(build_hwpx.py, hwpx_helpers.py, 검증 스크립트), 파이프라인 scripts/pipeline_general.js.
