# Proposal Writer (Claude / Cowork 스킬) — 범용 HWPX 문서 엔진

이 폴더가 Claude(Cowork) 스킬 본체입니다. 배포·설치 방법은 두 가지입니다.

## A. 그대로 설치 (권장)
동봉된 `proposal-writer.skill` 파일을 Claude 대화에 첨부 → 저장(설치).
(이 zip 안의 폴더를 다시 압축해 `.skill`로 만들어도 동일합니다: `zip -r proposal-writer.skill proposal-writer`)

## B. 내용 확인·수정용
- `SKILL.md` — 엔진 지침(8개 원칙·불변조건·HWPX 함정)
- `scripts/pipeline_general.js` — 7단계 파이프라인(양식분석→위계모델링→근거화→조사→검증→조립→불변조건→자동수정)
- `tests/regression-cases.md` — 회귀 테스트(세션 사례는 규칙이 아니라 여기 보관)

## 의존성
한글 조립·검증 도구인 **hwpx 스킬**이 함께 설치돼 있어야 합니다(build_hwpx.py, hwpx_helpers.py, fill_hwpx.py 등).

## 사용
양식(HWP/HWPX)·데이터·(선택)참고자료를 첨부하고 "이 양식으로 제안서 작성해줘"라고 요청하면
스킬이 트리거되어 8원칙 파이프라인을 수행합니다. 자세한 예시는 `proposal-writer_사용설명서.md` 참조.
