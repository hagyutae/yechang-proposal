---
description: 예창패 사업계획서를 Word, PDF 또는 Notion으로 출력
allowed-tools: Read, Write, Bash, mcp__e0bdf21f-a9d8-44b7-b3ea-0db81bfc44e6__notion-create-pages, mcp__e0bdf21f-a9d8-44b7-b3ea-0db81bfc44e6__notion-create-database, mcp__e0bdf21f-a9d8-44b7-b3ea-0db81bfc44e6__notion-fetch, mcp__e0bdf21f-a9d8-44b7-b3ea-0db81bfc44e6__notion-search, mcp__e0bdf21f-a9d8-44b7-b3ea-0db81bfc44e6__notion-update-page
argument-hint: [word | pdf | notion (생략 시 선택 요청)]
---

완성된 예비창업패키지 사업계획서를 Word(.docx), PDF 또는 Notion 페이지로 출력한다.

## Step 1: 프로젝트 파일 로드

워크스페이스에서 `bp-project.json`을 읽는다.
파일이 없으면: "먼저 `/new` 명령으로 예창패 프로젝트를 시작해 주세요."라고 안내하고 중단한다.

## Step 2: 출력 형식 결정

$ARGUMENTS가 `word` 또는 `docx`이면 Word로 출력한다.
$ARGUMENTS가 `pdf`이면 PDF로 출력한다.
$ARGUMENTS가 `notion`이면 Notion으로 출력한다.
$ARGUMENTS가 없으면 사용자에게 선택을 요청한다:

```
사업계획서를 어떤 형식으로 출력할까요?

1. 📄 Word (.docx) — K-Startup 제출용
2. 📋 PDF — 인쇄 및 공유용
3. 📝 Notion — Notion 페이지로 내보내기 (협업/편집용)
```

## Step 3: 미완성 섹션 확인

`status: empty`인 섹션이 있으면 사용자에게 알린다:
"아직 작성되지 않은 섹션이 있습니다: [목록]. 이대로 출력할까요, 아니면 먼저 작성하시겠어요?"
사용자 확인 후 진행한다.

## Step 4: 페이지 제한 확인

**⚠️ 주의:** 2025년 예창패는 **15페이지 이내(목차 제외)** 제한이 있습니다.
출력 후 페이지 수를 확인하세요. 초과하면 내용을 조정해 주세요.

---

## Step 5A: Word (.docx) 출력

python-docx가 없으면 먼저 설치한다:
```bash
pip install python-docx --break-system-packages -q
```

Bash에서 python-docx를 사용해 문서를 생성한다.
`bp-project.json`의 필드는 아래와 같이 매핑한다:
- `title` → 문서 제목
- `company_name` → 팀명/예정 법인명
- `applicant_name` → 신청자명
- `selected_host.name` → 신청 주관기관명
- `year` → 신청 연도
- `application_category` → 신청 분야
- `sections[].title` + `sections[].content` → 섹션별 내용

```bash
python3 << 'PYEOF'
from docx import Document
from docx.shared import Pt
from docx.enum.text import WD_ALIGN_PARAGRAPH
import json

with open('[bp-project.json 경로]', 'r', encoding='utf-8') as f:
    project = json.load(f)

doc = Document()

# 문서 제목
title_para = doc.add_heading(project.get('title', '예비창업패키지 사업계획서'), 0)
title_para.alignment = WD_ALIGN_PARAGRAPH.CENTER

# 기본 정보 테이블
info_table = doc.add_table(rows=5, cols=2)
info_table.style = 'Table Grid'
labels = ['신청자', '팀명', '신청 주관기관', '신청 연도', '신청 분야']
values = [
    project.get('applicant_name', ''),
    project.get('company_name', ''),
    project.get('selected_host', {}).get('name', ''),
    project.get('year', ''),
    project.get('application_category', '')
]
for i, (label, value) in enumerate(zip(labels, values)):
    info_table.cell(i, 0).text = label
    info_table.cell(i, 1).text = value

doc.add_paragraph()

# 섹션별 내용
for section in project.get('sections', []):
    content = section.get('content', '').strip()
    if content:
        doc.add_heading(section['title'], level=1)
        doc.add_paragraph(content)
        doc.add_paragraph()

# 저장
output_path = '[워크스페이스경로]/[사업명]-예창패-사업계획서-[연도].docx'
doc.save(output_path)
print(f'저장 완료: {output_path}')
PYEOF
```

---

## Step 5B: PDF 출력

Word 파일 생성 후 LibreOffice로 PDF 변환한다:
```bash
libreoffice --headless --convert-to pdf --outdir "[워크스페이스경로]" "[docx파일경로]" 2>&1
```

**HWP 출력 옵션:**
변환된 파일을 .hwp로 저장해야 하는 경우, LibreOffice에서 지원하는 범위 내에서 변환할 수 있습니다.
다만 Word 형식(docx)에서 HWP로의 변환은 포맷 호환성 손실이 있을 수 있으므로,
대부분의 경우 Word 또는 PDF로 제출하는 것을 권장합니다.

---

## Step 5C: Notion 출력

Notion MCP 커넥터를 사용하여 사업계획서를 Notion 페이지로 내보낸다.

### 5C-1. 출력 위치 결정

사용자에게 Notion 내보내기 위치를 물어본다:

```
Notion 어디에 사업계획서를 만들까요?

1. 🆕 새 페이지 — 워크스페이스 루트에 새 페이지 생성
2. 📁 기존 페이지 하위 — 특정 페이지 아래에 생성 (페이지 URL 또는 제목을 알려주세요)
3. 🗂️ 데이터베이스로 — 섹션별 진행 상태를 관리할 수 있는 데이터베이스로 생성
```

### 5C-2. 옵션 1 — 단일 Notion 페이지로 생성

`notion-create-pages`를 사용하여 사업계획서 전체를 하나의 Notion 페이지로 생성한다.

**페이지 구성:**
```
제목: [사업명] - 예창패 사업계획서 [연도]

내용 (Notion 마크다운):
# 일반현황
{sections[0].content}

# 창업아이템 개요(요약)
{sections[1].content}

# 1. 문제 인식(Problem)
{sections[2].content}

# 2. 실현 가능성(Solution)
{sections[3].content}
## 사업추진일정
{일정 테이블}
## 1단계 정부지원사업비 집행계획
{예산 테이블}
## 2단계 정부지원사업비 집행계획
{예산 테이블}

# 3. 성장전략(Scale-up)
{sections[4].content}

# 4. 팀 구성(Team)
{sections[5].content}
```

**Notion 마크다운 변환 규칙:**
- 각 섹션의 content를 Notion 마크다운 형식으로 변환
- 표(사업추진일정, 예산 등)는 Notion 테이블 마크다운으로 변환
- 글머리 기호, 번호 목록, 볼드/이탤릭 등 서식 보존
- 이미지 참조가 있으면 플레이스홀더로 표시: `[이미지: {설명}]`

**parent 지정:**
- 옵션 1(루트): parent 생략
- 옵션 2(기존 페이지 하위): `parent: { page_id: "사용자가 제공한 페이지 ID" }`

**properties 구성:**
```json
{
  "title": "[사업명] - 예창패 사업계획서 [연도]"
}
```

### 5C-3. 옵션 2 — 기존 페이지 하위에 생성

사용자가 Notion 페이지 URL이나 제목을 제공하면:
1. `notion-search`로 해당 페이지를 찾는다
2. `notion-fetch`로 페이지 ID를 확인한다
3. 해당 페이지를 parent로 지정하여 `notion-create-pages` 실행

### 5C-4. 옵션 3 — 데이터베이스로 생성

`notion-create-database`를 사용하여 섹션별 관리 데이터베이스를 생성한다.

**데이터베이스 스키마:**
```json
{
  "title": [{"text": {"content": "[사업명] - 예창패 사업계획서"}}],
  "properties": {
    "섹션": {
      "type": "title",
      "title": {}
    },
    "상태": {
      "type": "select",
      "select": {
        "options": [
          {"name": "미작성", "color": "gray"},
          {"name": "작성중", "color": "yellow"},
          {"name": "완료", "color": "green"},
          {"name": "검토필요", "color": "red"}
        ]
      }
    },
    "평가항목": {
      "type": "select",
      "select": {
        "options": [
          {"name": "문제인식", "color": "blue"},
          {"name": "실현가능성", "color": "purple"},
          {"name": "성장전략", "color": "orange"},
          {"name": "팀구성", "color": "green"},
          {"name": "일반", "color": "gray"}
        ]
      }
    },
    "배점비중": {
      "type": "rich_text",
      "rich_text": {}
    },
    "글자수": {
      "type": "number",
      "number": {"format": "number"}
    },
    "메모": {
      "type": "rich_text",
      "rich_text": {}
    }
  }
}
```

데이터베이스 생성 후 `notion-create-pages`로 각 섹션을 페이지로 추가한다:

```
섹션 1: 일반현황           | 상태: {status} | 평가항목: 일반      | 배점: -
섹션 2: 창업아이템 개요     | 상태: {status} | 평가항목: 일반      | 배점: -
섹션 3: 문제인식(Problem)  | 상태: {status} | 평가항목: 문제인식   | 배점: 30~40%
섹션 4: 실현가능성(Solution)| 상태: {status} | 평가항목: 실현가능성 | 배점: 25~35%
섹션 5: 성장전략(Scale-up) | 상태: {status} | 평가항목: 성장전략   | 배점: 15~25%
섹션 6: 팀구성(Team)       | 상태: {status} | 평가항목: 팀구성    | 배점: 10~20%
```

각 페이지의 content에 해당 섹션의 작성된 내용을 Notion 마크다운으로 넣는다.

### 5C-5. Notion 출력 완료

생성된 Notion 페이지/데이터베이스의 URL을 사용자에게 안내한다:

```
✅ Notion에 사업계획서가 생성되었습니다!

📝 페이지: [Notion URL]

Notion에서 직접 편집하거나 팀원과 공유할 수 있습니다.
변경 사항을 다시 프로젝트에 반영하려면 Notion에서 수정 후 `/update`를 사용하세요.
```

---

## Step 6: K-Startup 제출 안내 (Word/PDF인 경우)

**K-Startup 포털에 제출할 때 주의사항:**
- 파일 크기 제한: 30MB 이하
- 생성된 파일이 30MB를 초과하면 이미지 품질을 낮추거나 불필요한 이미지를 제거해 주세요.

## Step 7: 완료 안내

출력 파일을 워크스페이스 폴더에서 열 수 있음을 안내한다.
파일명 형식: `[사업명]-예창패-사업계획서-[연도].docx` (또는 `.pdf`)

**페이지 수 확인 중요:** 반드시 15페이지 이내(목차 제외)를 준수하세요.
초과하면 `/write [섹션번호]`로 내용을 조정할 수 있습니다.

미작성 섹션이 있었다면 해당 섹션 목록을 다시 알려주고 `/write [섹션번호]`로 보완할 수 있음을 제안한다.
