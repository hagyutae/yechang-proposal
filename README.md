# yechang-proposal (예창패 사업계획서 플러그인)

[![Version](https://img.shields.io/badge/version-1.1.0-blue.svg)](https://github.com/hagyutae/yechang-proposal/releases)
[![Claude Code Plugin](https://img.shields.io/badge/Claude_Code-Plugin-orange.svg)](https://claude.ai)
[![License: MIT](https://img.shields.io/badge/License-MIT-green.svg)](LICENSE)

**예비창업패키지(예창패) 사업계획서 작성 전문** Claude Code 플러그인입니다.

2025년 예창패 공식 자료(모집공고, 주관기관 소개, 질의응답, 양식)가 내장되어 있어 **별도 파일 업로드 없이 바로 시작**할 수 있습니다. IT/AI 분야에 특화된 심사 기준으로 작성 및 검토를 지원합니다.

## 설치

```bash
# .plugin 파일 다운로드 후 설치
claude plugin install yechang-proposal-v1.1.plugin
```

또는 Releases 페이지에서 최신 `.plugin` 파일을 다운로드하여 설치하세요.

## 사용 흐름

```
/new [사업명]          → 프로젝트 시작 (2025년 내장 자료 자동 적용)
    ↓
/write [섹션번호]      → 섹션별 작성 (반복 가능)
    ↓
/research [URL|키워드] → 외부 자료 수집·분석하여 섹션에 반영
    ↓
/review [섹션번호]     → 심사 기준 기반 검토 및 피드백
    ↓
/write [섹션번호]      → 피드백 반영 수정
    ↓
/export word|pdf|notion → 최종 파일 출력
```

## 커맨드

| 커맨드 | 설명 |
|--------|------|
| `/new [사업명]` | 예창패 프로젝트 시작. 2025년 내장 자료 사용 또는 최신 자료 업로드 |
| `/write [섹션번호]` | 특정 섹션 작성 또는 수정. 반복 실행 시 해당 섹션만 업데이트 |
| `/review [섹션번호]` | 2025년 예창패 심사 기준으로 전체 또는 특정 섹션 검토 및 개선 피드백 |
| `/export word\|pdf\|notion` | 완성된 사업계획서를 Word, PDF 또는 Notion 페이지로 출력 |
| `/research [URL\|키워드]` | 외부 URL/검색으로 시장 데이터, 경쟁사, 기술 동향을 수집하여 섹션에 반영 |
| `/update [연도]` | 새로운 연도 예창패 자료로 기존 프로젝트 업데이트 |

## 주요 기능

- **내장 2025년 자료** — 모집공고, 주관기관 26개 상세 정보, 질의응답, 양식 구조가 플러그인에 포함
- **주관기관 전략 추천** — 아이템 분야·키워드 기반으로 특화분야 일치도·지역·선발 인원 등을 종합해 Top 3 추천
- **심사 기준 기반 작성** — 문제인식(30~40%), 실현가능성(25~35%), 성장전략(15~25%), 팀구성(10~20%) 배점에 맞춘 가이드
- **외부 자료 리서치** — URL 또는 키워드로 시장 규모, 경쟁사, 기술 동향 수집 → 출처와 함께 섹션에 반영
- **심사 검토** — 항목별 점수 예측, 60점 미만 탈락 기준 체크, 가점(AI대학원/경진대회/기후테크) 확인
- **Notion 내보내기** — 사업계획서를 Notion 페이지 또는 섹션별 데이터베이스로 출력하여 팀 협업 가능
- **15페이지 준수** — 분량 관리 및 페이지 초과 경고

## 2025년 예창패 기준

| 항목 | 내용 |
|------|------|
| 지원 대상 | 사업자등록 전 예비창업자 |
| 지원 규모 | 평균 0.5억원 (1단계 ~20백만원 + 2단계 ~40백만원) |
| 선정 인원 | 780명 (일반 660명 + 특화 120명) |
| 협약기간 | 약 8개월 |
| 평가 방식 | 서류 → 인큐베이팅 → 발표평가 (3단계) |
| 탈락 기준 | 60점 미만 |
| 가점 | AI대학원(2점), 창업경진대회(1점), 기후테크(1점), 최대 3점 |

## 업로드 파일 (모두 선택사항)

| 파일 | 설명 |
|------|------|
| 사업계획서 양식 | 매년 창업진흥원에서 배포하는 공식 양식 (HWP/PDF) |
| 주관기관 소개 자료 | 주관기관 목록·특화분야·선발 인원 등 안내 문서 |
| 모집공고 | 공식 모집공고 |
| 주요 질의응답(Q&A) | 지원 자격·유의사항 반영 |

파일을 업로드하지 않으면 내장 2025년 자료로 자동 진행됩니다.

## 플러그인 구조

```
yechang-proposal/
├── .claude-plugin/
│   └── plugin.json
├── commands/
│   ├── new.md            # 프로젝트 시작
│   ├── write.md          # 섹션 작성
│   ├── review.md         # 심사 검토
│   ├── export.md         # 파일 출력 (Word/PDF/Notion)
│   ├── research.md       # 외부 자료 리서치
│   └── update.md         # 자료 업데이트
├── skills/
│   └── yechang-guide/
│       ├── SKILL.md
│       └── references/
│           ├── evaluation-criteria.md
│           ├── section-templates.md
│           ├── yechang-guide.md
│           └── 2025-data/
│               ├── host-institutions.md   # 주관기관 26개 상세
│               ├── announcement.md        # 모집공고
│               ├── template.md            # 양식 구조
│               └── qna.md                # 질의응답
└── README.md
```

## 빌드

소스에서 직접 `.plugin` 파일을 빌드하려면:

```bash
cd yechang-proposal
zip -r ../yechang-proposal-v1.1.plugin . -x "*.DS_Store" ".git/*" "*.md~"
```

## 라이선스

MIT License — 자유롭게 사용·수정·배포 가능합니다.

## 기여

이슈나 PR을 환영합니다. 특히 다음과 같은 기여를 기다립니다:

- 새 연도(2026년~) 예창패 자료 업데이트
- 다른 정부지원사업(초기창업패키지, 창업도약패키지 등) 확장
- 섹션 작성 품질 개선 및 합격 사례 추가
