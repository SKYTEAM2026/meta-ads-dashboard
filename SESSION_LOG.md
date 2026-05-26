# Meta Ads Dashboard — 작업 세션 로그

> 작성일: 2026-05-26  
> 레포: https://github.com/SKYTEAM2026/meta-ads-dashboard  
> 배포: https://skyteam2026.github.io/meta-ads-dashboard/

---

## 1. 환경 세팅

### Git 설치 확인
```
git version 2.54.0.windows.1
```

### Git 전역 사용자 설정
```bash
git config --global user.email "sunghei812@gmail.com"
git config --global user.name "SKYTEAM2026"
```

---

## 2. GitHub 자동 배포 설정

### 프로젝트 폴더 구성
- 로컬 경로: `C:\Users\sungh\meta-ads-dashboard\`
- `meta-ads-dashboard.html` → `index.html`로 복사

### GitHub Actions 워크플로우
파일: `.github/workflows/deploy.yml`

```yaml
name: Deploy to GitHub Pages
on:
  push:
    branches:
      - main
permissions:
  contents: read
  pages: write
  id-token: write
jobs:
  deploy:
    environment:
      name: github-pages
      url: ${{ steps.deployment.outputs.page_url }}
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/configure-pages@v5
      - uses: actions/upload-pages-artifact@v3
        with:
          path: .
      - id: deployment
        uses: actions/deploy-pages@v4
```

### GitHub Pages 설정 (수동)
`https://github.com/SKYTEAM2026/meta-ads-dashboard/settings/pages`  
→ Source: **GitHub Actions** 선택

### 첫 push 이슈 해결
원격 레포에 이미 커밋이 있어 rejected 발생 → `git pull origin main --allow-unrelated-histories` 로 해결

---

## 3. 인플루언서 관리 대시보드

**파일:** `influencer-dashboard.html`  
**URL:** `https://skyteam2026.github.io/meta-ads-dashboard/influencer-dashboard.html`

### 기능
- **요약 카드**: 전체 인플루언서 수, 총 팔로워, 평균 참여율, 협찬 진행 수
- **테이블 컬럼**: 이름/@계정, 카테고리, 팔로워수, 게시물수, 참여율(시각 바), 광고 단가, 협찬 여부
- **추가**: 모달 폼으로 신규 인플루언서 등록
- **수정/삭제**: 행별 아이콘 버튼, 삭제 시 confirm 팝업
- **검색 + 필터**: 이름/@계정 검색, 협찬 여부·카테고리 필터
- **정렬**: 모든 컬럼 클릭 정렬 (오름/내림차순)
- **데이터 저장**: `localStorage` (새로고침 유지)
- **디자인**: 다크 테마 (`#0f0f13` 배경), 퍼플 계열 accent

---

## 4. Meta 광고 소재 등록 대시보드

**파일:** `ad-register.html`  
**URL:** `https://skyteam2026.github.io/meta-ads-dashboard/ad-register.html`  
**참고 사이트:** `https://dusk-ad-register.vercel.app/dashboard`

### 레이아웃
- 좌측 고정 사이드바 (220px) + 우측 메인 영역
- 상단 topbar: 페이지 타이틀, 브랜드 pill, 날짜, 설정 버튼, 소재 등록 버튼

### 네비게이션 탭
| 탭 | 설명 |
|---|---|
| 📊 대시보드 | 요약 통계 + 최근 소재 + 차트 |
| ➕ 소재 등록 | 광고 소재 등록 폼 |
| 📋 소재 목록 | 전체 소재 필터/검색 테이블 |
| 📡 캠페인 연동 | Meta API 실시간 캠페인 조회 |

---

## 5. 브랜드/제품 구조

계층형 2단계 구조 (회사 → 제품):

```
리칼지메디
  ├── 써마드
  └── 뉴베로

아치핀
  ├── 아치핀
  ├── 아큐스터
  └── 아치핀 맥스

포디온
  └── 포디온
```

### 구현 방식
```js
const BRAND_STRUCTURE = {
  '리칼지메디': { color: '#4f46e5', products: ['써마드', '뉴베로'] },
  '아치핀':     { color: '#10b981', products: ['아치핀', '아큐스터', '아치핀 맥스'] },
  '포디온':     { color: '#f59e0b', products: ['포디온'] },
};
```

- 폼의 `<select>`: `<optgroup>`으로 회사 → 제품 계층 표시
- 사이드바: 회사명 헤더 + 들여쓰기 제품 목록
- 필터: "리칼지메디 전체" 또는 개별 제품 선택 (`company:xxx` prefix)
- 테이블: "리칼지메디 · 써마드" 형식 표시

---

## 6. 소재 등록 폼

### 입력 필드
| 필드 | 내용 |
|---|---|
| 브랜드/제품 | optgroup 드롭다운 |
| 캠페인 목표 | 인지도/트래픽/전환/리타게팅/앱설치/리드 |
| 소재명 | 텍스트 |
| 소재 유형 | 이미지 / 동영상 / 카루셀 (시각 선택) |
| 노출 플랫폼 | Facebook 피드 / Instagram 피드 / Instagram 스토리 / Audience Network (멀티 선택) |
| 헤드라인 | 40자 제한, 글자수 카운터 |
| 본문 텍스트 | 125자 제한, 글자수 카운터 |
| CTA 버튼 | 드롭다운 |
| 랜딩 URL | URL 입력 |
| 일 예산 | 숫자 (원) |
| 상태 | 검토중/승인/집행중/일시정지/반려 |
| 집행 기간 | 시작일 ~ 종료일 |
| 메모 | 텍스트 |

### Instagram 피드 미리보기
- 우측 고정 패널에 실시간 반영 (헤드라인, 본문, CTA 연동)

---

## 7. Meta Ads API 연동

### 보안 원칙
- Access Token을 코드에 절대 하드코딩하지 않음 (레포가 Public이므로)
- 모든 인증 정보는 **브라우저 localStorage에만 저장** (`meta_api_settings_v1` 키)
- GitHub 코드에 토큰 없음

### 설정 구조 (localStorage)
```json
{
  "token": "EAAxxxxxxxxxx",
  "version": "v21.0",
  "accounts": [
    { "name": "리칼지메디", "id": "act_XXXXXXXXXX" },
    { "name": "아치핀",     "id": "act_YYYYYYYYYY" },
    { "name": "포디온",     "id": "act_ZZZZZZZZZZ" }
  ],
  "activeIdx": 0
}
```

### API 호출
```
GET https://graph.facebook.com/v21.0/{act_ID}/campaigns
  ?fields=id,name,status,effective_status,objective,
          daily_budget,lifetime_budget,buying_type,
          start_time,stop_time,created_time
  &limit=50
  &access_token={token}
```

### 다계정 선택 UI
- 캠페인 탭 상단에 계정 탭 최대 3개 표시
- 탭 클릭 시 해당 계정으로 전환, 캠페인 데이터 초기화 후 재조회
- 연결 상태 표시: 사이드바 도트 + 상단 배너 (연결됨/오류/준비)

### 캠페인 목록 기능
- 캠페인 요약 카드: 전체/집행중/일시정지/총 일예산
- 필터: 캠페인명 검색, 상태 필터, 목표 필터
- 테이블 컬럼: 캠페인명+ID, 목표(한국어), 구매유형, 일예산, 총예산, 상태, 집행기간
- 페이지네이션: 50개씩 로드, 다음 페이지 버튼
- 연결 테스트: 계정명·통화 반환으로 토큰 유효성 검증

---

## 8. localStorage 키 목록

| 키 | 내용 |
|---|---|
| `meta_ad_v2` | 등록된 광고 소재 목록 |
| `meta_api_settings_v1` | API 토큰 + 계정 목록 + 활성 계정 인덱스 |

---

## 9. 파일 구조

```
meta-ads-dashboard/
├── .github/
│   └── workflows/
│       └── deploy.yml          # GitHub Actions 자동 배포
├── index.html                  # 메인 페이지 (meta-ads-dashboard.html 복사본)
├── ad-register.html            # Meta 광고 소재 등록 대시보드 (메인)
├── influencer-dashboard.html   # 인플루언서 관리 대시보드
└── SESSION_LOG.md              # 이 파일
```

---

## 10. Git 커밋 히스토리

```
ef43652  Add multi-account selector for Meta Ads API
10b0547  Add Meta Ads API integration: campaign list with settings modal
48aa4dd  Update brand structure: 리칼지메디/아치핀/포디온 hierarchy
f07016a  Add Meta ad creative registration dashboard
601c7c5  Add influencer management dashboard
b55ae43  Merge branch 'main' of https://github.com/SKYTEAM2026/meta-ads-dashboard
745c1ab  Initial commit: meta-ads-dashboard with GitHub Pages deploy
1f7a2d3  Add files via upload
```

---

## 11. 향후 작업 아이디어 (미구현)

- [ ] 캠페인별 Ad Set / Ad 목록 드릴다운
- [ ] Insights API 연동 (노출수, 클릭수, CTR, 지출 비용)
- [ ] 소재별 성과 연결 (소재 목록 ↔ 캠페인 매핑)
- [ ] 데이터 CSV 내보내기
- [ ] 캠페인 상태 변경 (ACTIVE ↔ PAUSED) — `ads_management` 권한 필요
