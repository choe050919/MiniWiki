# 개발자 가이드 (Developer Guide)

이 문서는 **Mini Wiki Dark**의 코드 구조 및 내부 동작 원리에 대해 설명합니다.

## 📂 프로젝트 구조

```text
.
├── index.html      # 앱의 진입점, 레이아웃 구조 (Topbar, Main, Sidebars)
├── style.css       # 전체 스타일링, CSS 변수(테마), 반응형 처리
├── script.js       # 핵심 로직 (상태 관리, 라우팅, 스토리지, 이벤트 처리)
├── TagManager.js   # 문서 태그 관리 로직 및 API
├── SearchEngine.js # 통합 검색 엔진 로직 및 API
├── recentEdits.js  # 최근 편집 문서 목록 관리 로직 및 API
└── README.md       # 사용자 매뉴얼
```

## 🏗 아키텍처 및 상태 관리

이 앱은 바닐라 자바스크립트로 작성된 **SPA(Single Page Application)** 형태입니다.  
`script.js` 내의 `state` 객체가 앱의 모든 상태를 관리하며, 변경 사항은 즉시 `localStorage`에 동기화됩니다.

### 1\. 주요 상수 (Storage Keys)

  * `miniWikiDocs`: 문서 데이터 객체 저장.
  * `miniWikiHistory`: 문서 수정 내역 배열 저장.
  * `miniWikiVisited`: 최근 방문 기록 저장.
  * `miniWikiPinned`: 사이드바에 고정된 문서 목록 저장.

### 2\. State 객체 구조

```javascript
let state = {
  current: "Home",       // 현재 보고 있는 문서의 제목 (String)
  pages: {               // 모든 문서 데이터 (Object)
    "Home": "# Home\n 내용...",
    "AnotherPage": "..."
  },
  tags: {                // 모든 태그와 각 태그의 참조 횟수 (Object: tagName -> count)
    "개발": 5,
    "문서": 3
  },
  tagPages: {            // 태그별 문서 목록 (Object: tagName -> Array of pageNames)
    "개발": ["페이지A", "페이지B"],
    "문서": ["페이지A", "페이지C"]
  },
  recentEdits: ["Home", "AnotherPage"], // 최근 편집된 문서 목록 (Array)
  mode: "view",          // 현재 UI 모드 (view | edit | list | history | historyDetail | search | tags)
  historyPage: null,     // 히스토리 모드에서 보고 있는 대상 문서명
  historyIdx: null       // 히스토리 상세 보기 시 선택된 인덱스
};
```

### 3\. 모드 시스템 (`setMode`)

UI는 `state.mode`에 따라 렌더링이 달라집니다.

  * **view**: 마크다운 렌더링 화면 (`previewEl` 표시, `editorEl` 숨김).
  * **edit**: 텍스트 에디터 화면 (`editorEl` 표시).
  * **list**: 전체 문서 목록 렌더링 (태그 필터링 포함).
  * **history**: 특정 문서의 수정 이력 목록.
  * **historyDetail**: 특정 시점의 문서 내용 미리보기 및 복원 기능.
  * **search**: 통합 검색 결과 표시.
  * **tags**: 태그 목록 및 태그별 문서 목록 표시 (선택 사항, `list` 모드에 통합될 수도 있음).

## 🧩 새로운 모듈 및 확장된 아키텍처

최근 업데이트를 통해 코드의 응집도를 높이고 유지보수를 용이하게 하기 위해 핵심 기능들이 모듈화되었습니다. `script.js`는 이제 이러한 모듈들을 통합하고 앱의 전반적인 흐름을 제어하는 역할을 합니다.

### 1\. 모듈화된 구조

*   **`TagManager.js`**: 태그 시스템과 관련된 모든 로직을 관리합니다. 문서 저장 시 자동으로 태그를 파싱하고 업데이트하는 기능이 포함됩니다.
    *   주요 API:
        *   `updateTagsForPage(pageName, content)`: 특정 문서의 내용을 기반으로 태그를 파싱하고 `state.tags`, `state.tagPages`를 업데이트합니다.
        *   `getAllTags()`: 현재 시스템의 모든 태그와 그 참조 횟수를 반환합니다.
        *   `getPagesForTag(tagName)`: 특정 태그에 연결된 문서 목록을 반환합니다.
*   **`SearchEngine.js`**: 통합 검색 기능을 담당합니다. 문서 내용과 제목을 인덱싱하고 검색 쿼리에 대한 관련성 높은 결과를 반환합니다.
    *   주요 API:
        *   `indexPage(pageName, content)`: 특정 문서를 검색 인덱스에 추가하거나 업데이트합니다.
        *   `search(query)`: 주어진 쿼리에 대해 문서 제목과 내용에서 검색하고, 관련성 점수와 함께 결과를 반환합니다.
*   **`recentEdits.js`**: 최근 편집된 문서 목록을 관리합니다.
    *   주요 API:
        *   `addRecentEdit(pageName)`: 문서가 편집될 때마다 최근 편집 목록에 추가합니다.
        *   `getRecentEdits()`: 최근 편집된 문서 목록을 반환합니다.

### 2\. 확장된 상태 관리

`state` 객체에 다음 필드들이 추가되어 새로운 모듈들에 의해 관리됩니다:

*   `state.tags`: 모든 태그와 각 태그의 참조 횟수를 저장합니다.
*   `state.tagPages`: 각 태그에 연결된 문서 목록을 저장합니다.
*   `state.recentEdits`: 사용자가 최근에 편집한 문서 목록을 시간 순서대로 저장합니다.

## 🧩 주요 기능 구현 상세

### 사이드바 (Sidebars)

  *   **좌측 (Sidebar Left)**:
      *   **전체 탭**: `state.pages` 키를 기반으로 가나다순(`alpha`) 또는 최근 방문순(`recent`) 정렬.
      *   **고정 탭**: `pinned` 배열을 기반으로 렌더링하며, HTML5 Drag & Drop API를 사용하여 순서 변경 가능.
      *   **최근 편집 탭**: `state.recentEdits` 배열을 기반으로 최근에 편집된 문서 목록을 표시합니다.
  *   **우측 (Sidebar Right)**:
      *   **목차 (TOC)**: 렌더링된 HTML의 `h1`\~`h6` 태그를 파싱하여 동적으로 생성. 나무위키 스타일의 넘버링 로직 포함.
      *   **백링크 (Backlinks)**: 전체 문서를 순회하며 정규식(`\[.*?\]\(현재문서명\)`)을 통해 역참조 링크를 탐색.

### 히스토리 (History)

  * 문서 저장(`Ctrl+S` 또는 버튼 클릭) 시 `addHistory()` 함수가 호출됩니다.
  * `history` 배열에 `{ page, time, content }` 객체를 추가합니다.
  * 최대 **100개**의 최신 기록만 유지하도록 슬라이싱 처리되어 있습니다.

### 마크다운 렌더링

  * `marked.parse(text)`를 사용하여 HTML로 변환합니다.
  * **내부 링크 처리**: 렌더링 후 `attachInternalLinkHandlers()`를 통해 `<a>` 태그를 가로채서, 외부 링크가 아닌 경우 SPA 내부 라우팅으로 처리합니다.

### 태그 시스템 (Tag System)

문서에 태그를 추가, 편집, 삭제하고 태그별로 문서를 필터링할 수 있는 기능입니다.

*   **태그 파싱**: 문서 저장 시 `TagManager.js` 모듈을 통해 문서 내용에서 `[#태그명]` 형식의 문자열을 자동으로 파싱하여 태그로 등록합니다.
*   **상태 관리**: `state.tags` (태그별 참조 횟수) 및 `state.tagPages` (태그에 연결된 문서 목록) 두 가지 주요 상태 객체가 관리됩니다.
*   **동작 방식**: 문서가 저장될 때마다 `TagManager.updateTagsForPage()` 함수가 호출되어 해당 문서의 태그 정보가 업데이트됩니다. 이를 통해 태그 목록과 태그별 문서 매핑이 항상 최신 상태를 유지합니다.

### 통합 검색 (Integrated Search)

문서 제목과 내용에서 키워드를 검색할 수 있는 기능입니다.

*   **인덱싱**: `SearchEngine.js` 모듈은 문서가 저장되거나 수정될 때 `SearchEngine.indexPage()` 함수를 통해 문서의 제목과 내용을 검색 가능한 인덱스에 추가합니다.
*   **검색 쿼리**: 사용자가 검색 쿼리를 입력하면 `SearchEngine.search()` 함수를 사용하여 일치하는 문서를 찾습니다.
*   **결과 정렬**: 검색 결과는 쿼리와의 관련성(relevance score)에 따라 정렬되어 사용자에게 가장 적합한 정보를 먼저 제공합니다.

### 최근 편집 문서 목록 (Recent Edits)

사이드바에 최근 편집된 문서 목록을 표시하는 기능입니다.

*   **추가 로직**: 문서가 성공적으로 저장될 때마다 `recentEdits.js` 모듈의 `addRecentEdit()` 함수가 호출되어 해당 문서가 `state.recentEdits` 배열의 맨 앞에 추가됩니다.
*   **목록 유지**: 이 배열은 최신 편집 순서로 문서를 관리하며, 사이드바의 "최근 편집 탭"에서 이 목록을 활용하여 UI를 렌더링합니다.

## 🎨 스타일링 (Theming)

`style.css`의 `:root` 가상 클래스에서 CSS 변수를 정의하여 다크 모드를 기본으로 설정했습니다.
`.light` 클래스가 `<html>` 태그에 붙으면 변수 값이 재정의되어 라이트 모드로 전환됩니다.

## ⚠️ 개발 시 유의사항

1.  **로컬 스토리지 용량**: 브라우저마다 다르지만 보통 5MB\~10MB 제한이 있습니다. 이미지를 Base64로 직접 넣는 것은 권장하지 않습니다.
2.  **동시성**: 동기적으로 작동하므로 대량의 데이터 처리 시 UI 블로킹이 발생할 수 있습니다 (현재 구조상 큰 문제는 없음).
3.  **보안**: `marked.js`를 사용하나 별도의 Sanitize(소독) 과정이 포함되어 있지 않습니다. 로컬 전용 앱이라 XSS 위험은 낮으나, 외부 데이터를 Import 할 때는 주의가 필요합니다.