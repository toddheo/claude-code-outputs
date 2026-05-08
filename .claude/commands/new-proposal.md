# /new-proposal — 아임인(imin) 제안서 자동 생성 스킬

## 역할
이 스킬은 (주)티웨이브 아임인(imin) 서비스의 B2G/B2B 제안서를 신규 제작하는 워크플로우입니다.
`_proposal_template/` 디렉토리를 기반으로 대상 기관에 맞춘 GitBook 제안서를 생성하고 GitHub Pages에 배포합니다.

---

## 실행 방법
```
/new-proposal [대상기관명] [슬러그(영문)]
예: /new-proposal 고용노동부 employment
예: /new-proposal 신한은행 shinhan-bank
```
인수가 없으면 사용자에게 질문하세요.

---

## Step 1. 기본 정보 수집

사용자에게 다음을 확인하세요 (아직 파악되지 않은 경우만):

| 항목 | 설명 | 예시 |
|------|------|------|
| `{{대상기관}}` | 제출처 공식 기관명 | 고용노동부 청년고용정책과 |
| `{{슬러그}}` | GitHub Pages URL용 영문 경로 | employment |
| `{{담당부서}}` | 수신 부서/담당자 | 청년복지팀 |
| `{{제출일}}` | 제출 예정일 (YYYY년 MM월) | 2026년 6월 |
| `{{핵심페인포인트}}` | 이 기관이 겪는 핵심 문제 1~2줄 | 청년층 금융 부채 증가, 사회 초년생 금융 교육 부재 |
| `{{협업형태}}` | B2G / B2B / 파트너십 | B2G |
| `{{타겟유저}}` | 서비스 대상 최종 이용자 | 취업 준비 청년 (만 19~29세) |
| `{{협업RR}}` | 기관 역할 / 티웨이브 역할 요약 | 기관: 채널 제공·홍보 / 티웨이브: 플랫폼 운영 |

---

## Step 2. 디렉토리 생성

`_proposal_template/` 전체를 `{{슬러그}}_제안서_gitbook/` 으로 복사하세요.

```bash
cp -r _proposal_template/ {{슬러그}}_제안서_gitbook/
```

---

## Step 3. 각 파일 생성 규칙

아래 순서대로 파일을 작성하세요. 각 파일은 **`_proposal_template/`의 해당 파일을 기반**으로, 플레이스홀더를 실제 내용으로 교체합니다.

### 필수 작성 파일 목록

| 파일 | 핵심 작성 포인트 |
|------|----------------|
| `README.md` | 표지, Executive Summary, 5대 핵심 제안, 신뢰 지표 표 |
| `chapters/01_background.md` | 대상 기관·분야의 거시 환경 변화 + 아임인 도입 필요성 |
| `chapters/02_problem.md` | 이 기관의 핵심 페인포인트 3가지 (데이터 기반) |
| `chapters/03_current_limitations.md` | 현행 제도·경쟁 서비스의 구조적 한계 |
| `chapters/04_urgency.md` | 골든타임: 지금 도입하지 않으면 어떤 일이 벌어지는가 |
| `chapters/05_objectives.md` | 도입 목적 및 아임인의 전략적 포지셔닝 |
| `chapters/06_service_intro.md` | 아임인 서비스 소개 (스테이지: 5·7·9·11·13개월 홀수 순번계, 납입 10만~160만 원) |
| `chapters/07_cooperation.md` | 기관-티웨이브 협력 체계 및 R&R |
| `chapters/08_roadmap.md` | 단계별 추진 로드맵 (준비→파일럿→확대) |
| `chapters/09_effects.md` | 정량·정성 기대 효과 및 KPI |
| `chapters/10_company.md` | 제안사 소개 (수치: 누적 3조 863억, 60만 7,169명, 연체율 0.22%, 2026년 4월 기준) |
| `appendix/A_legal.md` | 법적·제도적 검토 (전자금융거래법, 개인정보보호법 등) |
| `appendix/B_simulation.md` | 재무 시뮬레이션 (참여 규모별 효과) |

---

## Step 4. 불변 사실 — 항상 이 수치를 사용할 것

아임인 공인 수치 (2026년 4월 기준):
- 누적 거래액: **3조 863억 원** (3,086,308,100,000원)
- 총 참여 인원: **60만 7,169명**
- 연체율: **0.22%** (제1금융권 수준)
- 서비스 연수: **8년 이상** (2016년~)
- 연간 매출: **116억 원** (2025년 기준)
- 주요 파트너: **신한은행, 케이뱅크(KBank)**
- 대표이사: **서재준**

스테이지 구성 (변경 불가):
- **5·7·9·11·13개월 홀수 순번계**
- 납입 범위: **월 10만 원 ~ 최대 160만 원**

금지 단어:
- ❌ 금융감독원(금감원)
- ❌ 규제 샌드박스

---

## Step 5. index.html 생성

`_proposal_template/index.html`을 복사 후 아래 항목 교체:

```javascript
// title, og:title, og:description 업데이트
// basePath: '/claude-code-outputs/{{슬러그}}-제안서/'
// name: '아임인(imin) {{대상기관}} 제안서'
```

OG 이미지는 `assets/imin_thumbnail.png` 절대 URL 사용:
```
https://toddheo.github.io/claude-code-outputs/{{슬러그}}-제안서/assets/imin_thumbnail.png
```

썸네일 파일을 `assets/` 폴더에 복사하세요:
```bash
cp 군장병_제안서_gitbook/assets/imin_thumbnail.png {{슬러그}}_제안서_gitbook/assets/
```

---

## Step 6. _sidebar.md 생성

챕터 수와 제목에 맞게 `_sidebar.md`를 작성하세요.
HTML 블록 없이 순수 마크다운 링크 목록만 사용하세요.

---

## Step 7. GitHub Pages 배포

```bash
# 1. main 브랜치에 커밋
git add {{슬러그}}_제안서_gitbook/
git commit -m "feat: {{대상기관}} 제안서 신규 추가"

# 2. gh-pages 워크트리에 복사 후 배포
git worktree add /tmp/gh-pages-new gh-pages
cp -r {{슬러그}}_제안서_gitbook/. /tmp/gh-pages-new/{{슬러그}}-제안서/
cd /tmp/gh-pages-new
git add {{슬러그}}-제안서/
git commit -m "deploy: {{대상기관}} 제안서 배포"
git push origin gh-pages

# 3. 배포 URL 확인
echo "https://toddheo.github.io/claude-code-outputs/{{슬러그}}-제안서/"
```

---

## Step 8. 완료 보고

완료 시 다음 형식으로 보고하세요:

```
✅ {{대상기관}} 제안서 생성 완료

📄 로컬 경로: {{슬러그}}_제안서_gitbook/
🌐 배포 URL: https://toddheo.github.io/claude-code-outputs/{{슬러그}}-제안서/
📑 총 챕터: X개 파일
```

---

## 품질 기준 (체크리스트)

- [ ] 금지 단어(금감원, 규제 샌드박스) 없음
- [ ] 아임인 공인 수치 정확히 사용 (2026년 4월 기준)
- [ ] 스테이지 5·7·9·11·13 홀수 순번계 반영
- [ ] 납입 범위 10만~160만 원 반영
- [ ] OG 썸네일 절대 URL 사용
- [ ] basePath 올바르게 설정
- [ ] .nojekyll 루트에 존재 확인
- [ ] 모든 챕터 링크 _sidebar.md에 포함
- [ ] 날짜 헤더 (`**[YYYY-MM-DD]**`) 각 파일 상단에 포함
- [ ] 대표이사 서재준 반영
