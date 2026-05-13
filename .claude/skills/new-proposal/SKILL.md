---
description: 아임인(imin) B2G/B2B 제안서를 brief 파일 기반으로 자동 생성하고 GitHub Pages에 배포합니다
argument-hint: [슬러그]
---

# /new-proposal — 아임인(imin) 제안서 자동 생성 스킬

슬러그: $ARGUMENTS

## 사용법
```
/new-proposal [슬러그]
예: /new-proposal park-golf-partnership   ← brief에 category: domestic
예: /new-proposal SBI-partnership         ← brief에 category: global
예: /new-proposal solana-partnership      ← brief에 category: web3
```

슬러그를 생략하면 `brief/` 폴더의 파일 목록을 보여주고 선택하게 하세요.

**카테고리 분류 (brief 파일 frontmatter에서 읽음):**
- `domestic` — 국내 B2B/B2G 제휴 (파크골프장, 프랜차이즈, 지자체 등)
- `global` — 해외 파트너십 (SBI, ZaloPay, 동남아·중화권 등)
- `web3` — Web3/블록체인 파트너십 (Movement, Solana, 스테이블코인 등)

---

## Step 0. 브리프 파일 확인

**반드시 먼저 브리프 파일을 읽으세요.**

```
brief/[슬러그].md           ← 핵심 컨텍스트 (slug + category + 내용 전체)
brief/_refs/[슬러그]/       ← 참고자료 (있으면 전부 읽기)
```

브리프 파일이 없으면 사용자에게 안내:
> "`brief/[슬러그].md` 파일이 없습니다. `brief/_template.md`를 복사해서 작성해 주세요."

브리프 frontmatter에서 **반드시 `category` 값을 확인**하세요.
`category`가 없거나 비어 있으면 사용자에게 domestic/global/web3 중 선택을 요청하세요.

브리프 파일의 **모든 필드**를 읽고 내용을 내재화한 뒤 Step 1로 진행하세요.
참고자료가 있으면 Read 툴로 전부 읽어 컨텍스트를 보강하세요.

---

## Step 1. 불명확한 항목 확인 (최소화)

브리프에서 비어 있거나 `{{...}}`로 남은 항목 중 **제안서 품질에 결정적인 항목만** 질문하세요.
모든 걸 묻지 말고, 추론 가능한 건 합리적으로 채우세요.

반드시 확인해야 하는 항목:
- 슬로건이 없으면 브리프 내용 기반으로 2~3안 제시 후 선택 요청
- 참고자료에서 수치를 못 찾은 페인포인트가 있으면 사용자에게 확인

---

## Step 2. 디렉토리 생성 및 파일 복사

```bash
cp -r _proposal_template/ [슬러그]_제안서_gitbook/
cp brief/_refs/[슬러그]/* [슬러그]_제안서_gitbook/  # 참고자료가 있으면
```

---

## Step 3. 파일 생성 순서 및 규칙

브리프 내용을 **모든 챕터에 일관되게** 반영하세요.
`_proposal_template/chapters/CHAPTERS_GUIDE.md`를 참조하여 각 챕터를 작성하세요.

### 작성 순서

1. `README.md` — 표지, Executive Summary, 5대 핵심 제안, 신뢰 지표
2. `chapters/01_background.md` ~ `chapters/16_cases.md`
3. `chapters/17_company.md` ~ `chapters/19_partnerships.md`
4. `appendix/A_legal.md`, `appendix/B_simulation.md`
5. `_sidebar.md` — 챕터 목록 정리
6. `index.html` — 플레이스홀더 교체

### 각 파일 필수 요소

- 파일 최상단: `**[오늘 날짜 YYYY-MM-DD]**`
- 제목: `# X.X 챕터 제목`
- 브리프의 페인포인트·협업 모델·기대 효과를 해당 챕터에 녹여낼 것
- 데이터 기반 논거 우선 (브리프 수치 → 공개 통계 → 논리적 추론 순)

---

## Step 4. index.html 플레이스홀더 교체

`_proposal_template/index.html`에서 교체할 항목:

| 플레이스홀더 | 교체 내용 |
|------------|----------|
| `{{제안서제목}}` | 브리프 기관명 기반 제목 |
| `{{카테고리}}` | 브리프 category 값 (domestic / global / web3) |
| `{{슬러그}}` | 브리프 slug 값 |
| `{{대상기관}}` | 브리프 대상기관 |

OG 이미지 URL:
```
https://toddheo.github.io/proposal/[카테고리]/[슬러그]/assets/imin_thumbnail.png
```

basePath:
```javascript
basePath: '/proposal/[카테고리]/[슬러그]/',
```

---

## Step 5. 불변 사실 — 항상 이 수치를 사용

| 항목 | 값 | 기준 |
|------|-----|------|
| 누적 거래액 | 3조 863억 원 | 2026년 4월 |
| 총 참여 인원 | 60만 7,169명 | 2026년 4월 |
| 연체율 | 0.22% | 제1금융권 수준 |
| 서비스 연수 | 8년 이상 | 2016년~ |
| 연간 매출 | 116억 원 | 2025년 |
| 주요 파트너 | 신한은행, 케이뱅크(KBank) | |
| 대표이사 | 서재준 | |

**스테이지:** 5·7·9·11·13개월 홀수 순번계 (3·6·12·18 절대 사용 금지)
**납입 범위:** 월 10만 원 ~ 최대 160만 원
**금지 단어:** 금융감독원(금감원), 규제 샌드박스

---

## Step 6. GitHub Pages 배포

배포 대상: `toddheo/proposal` 레포의 `gh-pages` 브랜치, `[카테고리]/[슬러그]/` 폴더

```bash
# main 브랜치 커밋
git add [슬러그]_제안서_gitbook/
git commit -m "feat: [대상기관] 제안서 신규 추가 ([카테고리])"

# proposal 레포 gh-pages 배포
PROPOSAL="/tmp/proposal-deploy"
# 이미 클론된 경우 pull, 아닌 경우 클론
if [ -d "$PROPOSAL/.git" ]; then
  cd $PROPOSAL && git checkout gh-pages && git pull origin gh-pages
else
  TOKEN=$(python -c "import subprocess; url=subprocess.check_output(['git','-C','c:/Users/티웨이브/Desktop/티웨이브/업무/96_안티그래비티/test','remote','get-url','origin']).decode().strip(); print(url.split('toddheo:')[1].split('@')[0])")
  git clone "https://toddheo:${TOKEN}@github.com/toddheo/proposal.git" $PROPOSAL
  cd $PROPOSAL && git checkout gh-pages
fi

mkdir -p "$PROPOSAL/[카테고리]/[슬러그]"
cp -r "c:/Users/티웨이브/Desktop/티웨이브/업무/96_안티그래비티/test/[슬러그]_제안서_gitbook/." "$PROPOSAL/[카테고리]/[슬러그]/"
cd $PROPOSAL
git add [카테고리]/[슬러그]/
git commit -m "deploy: [대상기관] 제안서 배포 ([카테고리]/[슬러그])"
git push origin gh-pages
```

---

## Step 7. 완료 보고

```
✅ [대상기관] 제안서 생성 완료

📄 로컬:  [슬러그]_제안서_gitbook/
🌐 URL:   https://toddheo.github.io/proposal/[카테고리]/[슬러그]/
📑 챕터:  총 XX개 파일
```

---

## 품질 체크리스트

- [ ] 브리프의 페인포인트가 1·2·3장에 일관되게 반영됨
- [ ] 브리프의 협업 R&R이 4·7장에 구체적으로 기술됨
- [ ] 브리프의 기대 효과가 5장 KPI로 연결됨
- [ ] 금지 단어(금감원, 규제 샌드박스) 없음
- [ ] 아임인 공인 수치 정확히 사용 (2026년 4월 기준)
- [ ] 스테이지 5·7·9·11·13 홀수 순번계만 사용
- [ ] 납입 범위 10만~160만 원 반영
- [ ] OG 썸네일 절대 URL 사용 (카테고리 포함)
- [ ] basePath 올바르게 설정 (카테고리/슬러그 포함)
- [ ] 각 파일 날짜 헤더 포함
- [ ] 대표이사 서재준 반영
