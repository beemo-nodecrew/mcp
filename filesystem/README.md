# Filesystem MCP 서버

파일시스템 작업을 위한 모델 컨텍스트 프로토콜(MCP)을 구현한 Node.js 서버입니다.

## 특징

- 파일 읽기/쓰기
- 디렉토리 생성/목록 조회/삭제
- 파일/디렉토리 이동
- 파일 검색
- 파일 메타데이터 조회

**참고**: 이 서버는 `args`를 통해 지정된 디렉토리 내에서만 작업을 허용합니다.

## API

### 리소스

- `file://system`: 파일 시스템 작업 인터페이스

### 도구

- **read_file**

  - 파일의 전체 내용 읽기
  - 입력: `path` (문자열)
  - UTF-8 인코딩으로 파일 전체 내용을 읽습니다

- **read_multiple_files**

  - 여러 파일을 동시에 읽기
  - 입력: `paths` (문자열 배열)
  - 일부 파일 읽기에 실패해도 전체 작업은 계속됩니다

- **write_file**

  - 새 파일 생성 또는 기존 파일 덮어쓰기 (이 기능 사용 시 주의하세요)
  - 입력:
    - `path` (문자열): 파일 위치
    - `content` (문자열): 파일 내용

- **edit_file**

  - 고급 패턴 매칭과 포맷팅을 사용한 선택적 편집
  - 기능:
    - 라인 기반 및 다중 라인 내용 매칭
    - 들여쓰기 보존과 함께 공백 정규화
    - 올바른 위치 지정으로 여러 편집 동시 수행
    - 들여쓰기 스타일 감지 및 보존
    - Git 스타일의 차이점 출력 및 컨텍스트 제공
    - 드라이 런 모드로 변경 사항 미리보기
  - 입력:
    - `path` (문자열): 편집할 파일
    - `edits` (배열): 편집 작업 목록
      - `oldText` (문자열): 검색할 텍스트 (부분 문자열일 수 있음)
      - `newText` (문자열): 대체할 텍스트
    - `dryRun` (불리언): 실제 적용 없이 변경 사항 미리보기 (기본값: false)
  - 드라이 런의 경우 자세한 차이점과 매칭 정보를 반환하고, 그렇지 않으면 변경 사항을 적용합니다
  - 모범 사례: 항상 먼저 dryRun을 사용하여 변경 사항을 적용하기 전에 미리 확인하세요

- **create_directory**

  - 새 디렉토리 생성 또는 존재 확인
  - 입력: `path` (문자열)
  - 필요한 경우 부모 디렉토리도 생성합니다
  - 디렉토리가 이미 있으면 조용히 성공으로 처리합니다

- **list_directory**

  - 디렉토리 내용을 [FILE] 또는 [DIR] 접두사와 함께 나열
  - 입력: `path` (문자열)

- **move_file**

  - 파일 및 디렉토리 이동 또는 이름 변경
  - 입력:
    - `source` (문자열)
    - `destination` (문자열)
  - 대상이 이미 있으면 실패합니다

- **search_files**

  - 파일/디렉토리를 재귀적으로 검색
  - 입력:
    - `path` (문자열): 시작 디렉토리
    - `pattern` (문자열): 검색 패턴
    - `excludePatterns` (문자열[]): 제외할 패턴. Glob 형식이 지원됩니다.
  - 대소문자를 구분하지 않는 매칭
  - 일치하는 항목의 전체 경로를 반환합니다

- **get_file_info**

  - 자세한 파일/디렉토리 메타데이터 가져오기
  - 입력: `path` (문자열)
  - 반환:
    - 크기
    - 생성 시간
    - 수정 시간
    - 접근 시간
    - 유형 (파일/디렉토리)
    - 권한

- **list_allowed_directories**
  - 서버가 접근할 수 있는 모든 디렉토리 나열
  - 입력 불필요
  - 반환:
    - 이 서버가 읽기/쓰기할 수 있는 디렉토리들

## Claude Desktop에서 사용하기

`claude_desktop_config.json`에 다음을 추가하세요:

참고: 서버에 샌드박스 디렉토리를 제공하려면 `/projects`에 마운트하세요. `ro` 플래그를 추가하면 서버에서 해당 디렉토리를 읽기 전용으로 설정합니다.

### Docker

참고: 모든 디렉토리는 기본적으로 `/projects`에 마운트되어야 합니다.

```json
{
  "mcpServers": {
    "filesystem": {
      "command": "docker",
      "args": [
        "run",
        "-i",
        "--rm",
        "--mount",
        "type=bind,src=/Users/username/Desktop,dst=/projects/Desktop",
        "--mount",
        "type=bind,src=/path/to/other/allowed/dir,dst=/projects/other/allowed/dir,ro",
        "--mount",
        "type=bind,src=/path/to/file.txt,dst=/projects/path/to/file.txt",
        "mcp/filesystem",
        "/projects"
      ]
    }
  }
}
```

### NPX

```json
{
  "mcpServers": {
    "filesystem": {
      "command": "npx",
      "args": [
        "-y",
        "@modelcontextprotocol/server-filesystem",
        "/Users/username/Desktop",
        "/path/to/other/allowed/dir"
      ]
    }
  }
}
```

## 빌드

Docker 빌드:

```bash
docker build -t mcp/filesystem -f Dockerfile .
```

## 라이선스

이 MCP 서버는 MIT 라이선스에 따라 제공됩니다. 이는 MIT 라이선스의 약관에 따라 소프트웨어를 자유롭게 사용, 수정 및 배포할 수 있음을 의미합니다. 자세한 내용은 프로젝트 저장소의 LICENSE 파일을 참조하세요.
