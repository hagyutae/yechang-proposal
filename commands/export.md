---
description: 예창패 사업계획서를 Word 또는 PDF로 출력
allowed-tools: Read, Write, Bash
argument-hint: [word | pdf (생략 시 선택 요청)]
---

완성된 예비창업패키지 사업계획서를 Word(.docx) 또는 PDF 파일로 출력한다.

## Step 1: 프로젝트 파일 로드

워크스페이스에서 `bp-project.json`을 읽는다.
파일이 없으면: "먼저 `/new` 명령으로 예창패 프로젝트를 시작해 주세요."라고 안내하고 중단한다.

## Step 2: 출력 형식 결정

$ARGUMENTS가 `word` 또는 `docx`이면 Word로 출력한다.
$ARGUMENTS가 `pdf`이면 PDF로 출력한다.
$ARGUMENTS가 없으면 사용자에게 선택을 요청한다.

## Step 3: 미완성 섹션 확인

`status: empty`인 섹션이 있으면 사용자에게 알린다:
"아직 작성되지 않은 섹션이 있습니다: [목록]. 이대로 출력할까요, 아니면 먼저 작성하시겠어요?"
사용자 확인 후 진행한다.

## Step 4: 페이지 제한 확인

**⚠️ 주의:** 2025년 예창패는 **15페이지 이내(목차 제외)** 제한이 있습니다.
출력 후 페이지 수를 확인하세요. 초과하면 내용을 조정해 주세요.

## Step 5: 문서 생성

python-docx가 없으면 먼저 설치한다:
```bash
pip install python-docx --break-system-packages -q
```

### Word (.docx) 출력

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

### PDF 출력

Word 파일 생성 후 LibreOffice로 PDF 변환한다:
```bash
libreoffice --headless --convert-to pdf --outdir "[워크스페이스경로]" "[docx파일경로]" 2>&1
```

**HWP 출력 옵션:**
변환된 파일을 .hwp로 저장해야 하는 경우, LibreOffice에서 지원하는 범위 내에서 변환할 수 있습니다.
다만 Word 형식(docx)에서 HWP로의 변환은 포맷 호환성 손실이 있을 수 있으므로,
대부분의 경우 Word 또는 PDF로 제출하는 것을 권장합니다.

## Step 6: K-Startup 제출 안내

**K-Startup 포털에 제출할 때 주의사항:**
- 파일 크기 제한: 30MB 이하
- 생성된 파일이 30MB를 초과하면 이미지 품질을 낮추거나 불필요한 이미지를 제거해 주세요.

## Step 7: 완료 안내

출력 파일을 워크스페이스 폴더에서 열 수 있음을 안내한다.
파일명 형식: `[사업명]-예창패-사업계획서-[연도].docx` (또는 `.pdf`)

**페이지 수 확인 중요:** 반드시 15페이지 이내(목차 제외)를 준수하세요.
초과하면 `/write [섹션번호]`로 내용을 조정할 수 있습니다.

미작성 섹션이 있었다면 해당 섹션 목록을 다시 알려주고 `/write [섹션번호]`로 보완할 수 있음을 제안한다.
