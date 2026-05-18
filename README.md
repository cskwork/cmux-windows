> **cskwork fork 안내** — 이 저장소는 [`mkurman/cmux-windows`](https://github.com/mkurman/cmux-windows)의 공개 유지보수 fork입니다. 한국어 사용자를 위해 본 README는 한글로 제공하며, 영문 원본은 [README.en.md](./README.en.md)에 보존되어 있습니다. 원안은 [`manaflow-ai/cmux`](https://github.com/manaflow-ai/cmux) (macOS, Swift + libghostty, GPL-3.0). 상위 저작권 `Copyright (c) 2026 Mariusz Kurman`; MIT LICENSE는 원형 그대로 유지됩니다. Fork 시점: 커밋 `974b718`. 자세한 출처/라이선스는 [NOTICE.md](./NOTICE.md) 참고.

# cmux for Windows — 한국어 가이드

WPF + ConPTY 기반의 **Windows 네이티브 터미널 멀티플렉서**. AI 코딩 에이전트 (Claude Code, Codex, Cursor, Gemini CLI 등)와 함께 쓰도록 설계되었습니다. Electron 런타임을 쓰지 않는 진짜 네이티브 데스크톱 앱입니다.

## 한눈에 보기

| 무엇 | 어떻게 |
|---|---|
| 워크스페이스 + 서피스(탭) + 분할 패인 | `Ctrl+N` / `Ctrl+T` / `Ctrl+D` |
| AI 에이전트 OSC 알림 + 미확인 추적 | `Ctrl+I` / `Ctrl+Shift+U` |
| 세션 영구 보존 + 트랜스크립트 검색 | `Ctrl+Shift+V` (Session Vault) |
| 명령 히스토리 / 리플레이 | `Ctrl+Shift+L` / `Ctrl+Alt+H` |
| 스크립트 자동화용 CLI | `cmux notify`, `cmux workspace`, `cmux split` |

---

## AI 에이전트용 자동 설치 (PowerShell 한 블록)

### 어디서 실행하나요? (초급자 필독)

Windows에서 명령을 입력할 수 있는 "검은 창"은 여러 종류가 있고, **이 설치 블록은 그중 PowerShell 전용**입니다. cmd나 WSL에서는 동작하지 않습니다.

| 창 종류 | 어떻게 여는가 | 이 블록을 쓸 수 있나요? |
|---|---|---|
| **PowerShell** (또는 **Windows Terminal**) | 시작 메뉴(⊞ Win) → `PowerShell` 입력 → 클릭 | **예** — 이게 정답입니다 |
| **명령 프롬프트** (`cmd.exe`) | 시작 메뉴 → `cmd` 입력 | 아니오 — 명령 문법이 달라 동작하지 않습니다 |
| **WSL** (Ubuntu, Debian 등 리눅스 서브시스템) | 시작 메뉴 → `Ubuntu` 또는 `wsl` 입력 | 아니오 — WSL은 리눅스라서 Windows 전용 GUI 앱(`cmuxw.exe`)을 실행할 수 없습니다 |

**잘 모르겠다면 이렇게 하세요:**

1. 키보드 **⊞ Win 키**를 누른다.
2. `PowerShell` 이라고 그대로 친다.
3. 가장 위에 뜨는 결과(보통 **"Windows PowerShell"** 또는 **"PowerShell 7"**)를 클릭.
4. 파란색 또는 검은색 창이 열리면, 아래 블록 전체를 복사해서 그 창에 붙여넣고 **Enter**를 한 번 누르면 끝.

**관리자 권한은 필요 없습니다.** 평소 사용하는 일반 계정으로 PowerShell을 여세요. 필요한 것은 **Windows 10 또는 11**, 그리고 **인터넷 연결**뿐입니다. `winget`(Windows 패키지 매니저)이 시스템에 없다면 .NET 10 SDK와 git을 수동으로 설치하라는 안내 메시지가 표시됩니다.

**참고 — 왜 cmd나 WSL에서는 안 되나요?**

- `cmd.exe`는 아래 블록이 쓰는 `$ErrorActionPreference`, `Get-Command`, `[Environment]::SetEnvironmentVariable` 같은 PowerShell 전용 문법을 이해하지 못합니다.
- WSL은 리눅스 환경입니다. 반면 `cmuxw.exe`는 Windows의 화면 그리기 시스템(WPF)과 콘솔 시스템(ConPTY)을 직접 쓰는 **네이티브 Windows 앱**이라, 리눅스 안에서는 빌드해도 실행되지 않습니다. (WSLg가 깔려 있어도 WPF는 지원되지 않습니다.)

### 실행 블록

```powershell
# 0) 종료 시 즉시 중단
$ErrorActionPreference = 'Stop'

# 1) .NET 10 SDK 확인 (없으면 winget으로 설치)
$hasNet10 = $false
if (Get-Command dotnet -ErrorAction SilentlyContinue) {
  $hasNet10 = (dotnet --list-sdks 2>$null) -match '^10\.'
}
if (-not $hasNet10) {
  if (Get-Command winget -ErrorAction SilentlyContinue) {
    winget install --id Microsoft.DotNet.SDK.10 -e --accept-source-agreements --accept-package-agreements
    $env:PATH = "$env:ProgramFiles\dotnet;$env:PATH"
  } else {
    Write-Error "winget 없음. https://dotnet.microsoft.com/download 에서 .NET 10 SDK를 수동 설치하세요."
  }
}

# 2) git 확인
if (-not (Get-Command git -ErrorAction SilentlyContinue)) {
  if (Get-Command winget -ErrorAction SilentlyContinue) {
    winget install --id Git.Git -e --accept-source-agreements --accept-package-agreements
    $env:PATH = "$env:ProgramFiles\Git\cmd;$env:PATH"
  } else {
    Write-Error "git 없음. https://git-scm.com/download/win 에서 수동 설치하세요."
  }
}

# 3) 저장소 clone (cskwork fork)
$Root = Join-Path $env:USERPROFILE 'src\cmux-windows'
if (-not (Test-Path $Root)) {
  git clone https://github.com/cskwork/cmux-windows.git $Root
}
Set-Location $Root

# 4) 단일 파일 자기포함형 GUI 빌드 (대상 PC에 .NET 런타임 설치 불필요)
dotnet publish src\Cmux\Cmux.csproj -c Release -r win-x64 --self-contained true `
  /p:PublishSingleFile=true /p:PublishTrimmed=false -o publish\cmux-win-x64-single

# 5) CLI 빌드 + 현재 사용자 PATH에 등록
dotnet publish src\Cmux.Cli\Cmux.Cli.csproj -c Release -r win-x64 --self-contained true -o publish\cmux-cli
$CliDir = (Resolve-Path .\publish\cmux-cli).Path
$UserPath = [Environment]::GetEnvironmentVariable('PATH', 'User')
if ($UserPath -notlike "*$CliDir*") {
  [Environment]::SetEnvironmentVariable('PATH', "$UserPath;$CliDir", 'User')
  Write-Host "사용자 PATH에 $CliDir 추가됨. 새 셸에서 'cmux' 명령 사용 가능."
}

# 6) GUI 첫 실행
Start-Process .\publish\cmux-win-x64-single\cmuxw.exe
Write-Host "설치 완료. GUI: cmuxw.exe / CLI: cmux"
```

설치 끝. `cmuxw.exe`는 GUI, `cmux`는 CLI입니다.

### 수동 설치 (개발 디버그 모드)

```powershell
git clone https://github.com/cskwork/cmux-windows.git
cd cmux-windows
dotnet build Cmux.sln -c Debug
dotnet run --project src/Cmux/Cmux.csproj -c Debug
```

---

## 빌드 옵션 정리

| 시나리오 | 명령 | 산출물 |
|---|---|---|
| 개발 디버그 실행 | `dotnet run --project src/Cmux/Cmux.csproj -c Debug` | 즉시 실행 |
| 프레임워크 의존형 (작은 용량, 대상에 .NET 런타임 필요) | `dotnet publish src/Cmux/Cmux.csproj -c Release -r win-x64 --self-contained false -o publish/cmux-win-x64` | `publish/cmux-win-x64/cmuxw.exe` |
| 자기포함형 (런타임 포함) | `dotnet publish src/Cmux/Cmux.csproj -c Release -r win-x64 --self-contained true -o publish/cmux-win-x64-sc` | `publish/cmux-win-x64-sc/cmuxw.exe` |
| 단일 파일 자기포함형 (배포 친화) | `dotnet publish src/Cmux/Cmux.csproj -c Release -r win-x64 --self-contained true /p:PublishSingleFile=true /p:PublishTrimmed=false -o publish/cmux-win-x64-single` | `publish/cmux-win-x64-single/cmuxw.exe` |
| CLI 도구 | `dotnet publish src/Cmux.Cli/Cmux.Cli.csproj -c Release -r win-x64 --self-contained true -o publish/cmux-cli` | `publish/cmux-cli/cmux.exe` |

> WebView2 기반 기능을 쓰는 경우 대상 시스템에 [WebView2 Runtime](https://developer.microsoft.com/microsoft-edge/webview2/)이 필요할 수 있습니다.

---

## 처음 5분 사용 가이드

1. `cmuxw.exe` 실행
2. `Ctrl+N` — 작업 중인 저장소용 워크스페이스 생성
3. `Ctrl+T` — 서피스(탭) 추가
4. `Ctrl+D` / `Ctrl+Shift+D` — 패인 가로 / 세로 분할
5. `Ctrl+Shift+P` — 명령 팔레트
6. `Ctrl+Shift+L` — 명령 로그
7. `Ctrl+Shift+V` — Session Vault (트랜스크립트 보관함)
8. `Ctrl+,` — 설정 (테마 / 폰트 / 커서 조정)

---

## 단축키

### 워크스페이스

| 단축키 | 동작 |
|---|---|
| `Ctrl+N` | 새 워크스페이스 |
| `Ctrl+1..8` | 1~8번 워크스페이스로 이동 |
| `Ctrl+9` | 마지막 워크스페이스 |
| `Ctrl+Shift+W` | 워크스페이스 닫기 |
| `Ctrl+Shift+R` | 워크스페이스 이름 변경 |
| `Ctrl+B` | 사이드바 토글 |

### 서피스 (탭)

| 단축키 | 동작 |
|---|---|
| `Ctrl+T` | 새 서피스 |
| `Ctrl+W` | 서피스 닫기 |
| `Ctrl+Shift+]` / `Ctrl+Shift+[` | 다음 / 이전 서피스 |
| `Ctrl+Tab` / `Ctrl+Shift+Tab` | 서피스 순회 |

### 패인

| 단축키 | 동작 |
|---|---|
| `Ctrl+D` | 오른쪽 분할 |
| `Ctrl+Shift+D` | 아래쪽 분할 |
| `Ctrl+Alt+화살표` | 인접 패인으로 포커스 이동 |
| `Ctrl+Shift+Z` | 패인 확대 / 축소 토글 |

### 생산성

| 단축키 | 동작 |
|---|---|
| `Ctrl+Shift+P` | 명령 팔레트 |
| `Ctrl+Shift+F` | 검색 오버레이 |
| `Ctrl+Shift+L` | 명령 로그 |
| `Ctrl+Shift+V` | Session Vault |
| `Ctrl+Alt+H` | 명령 히스토리 선택기 |
| `Ctrl+,` | 설정 |

---

## CLI 사용법

`cmux` CLI는 named pipe로 동작 중인 `cmuxw` 프로세스에 명령을 보냅니다.

```powershell
# 에이전트 훅에서 알림 발송
cmux notify --title "Claude Code" --body "입력 대기 중"

# 워크스페이스 관리
cmux workspace list
cmux workspace create --name "My Project"
cmux workspace select --index 0

# 서피스 / 패인 제어
cmux surface create
cmux split right
cmux split down

# 상태 조회
cmux status
```

---

## AI 에이전트 통합 팁

- **Claude Code / Codex 등의 알림 훅**: 작업이 끝나거나 입력 대기에 들어갈 때 `cmux notify --title "..." --body "..."` 호출. 워크스페이스의 미확인 카운터가 올라가고 `Ctrl+Shift+U`로 바로 점프할 수 있습니다.
- **OSC 시퀀스 활용**: 셸 내부에서 `printf '\033]9;%s\007' "메시지"` 같은 OSC 9 / 99 / 777 시퀀스를 출력하면 cmux가 자동으로 알림으로 변환합니다 (별도 CLI 호출 불필요).
- **여러 에이전트 병렬 실행**: 에이전트별로 서피스(탭)를 분리하거나 패인을 나눠 동시에 모니터링.

---

## 아키텍처

```text
src/
  Cmux/         WPF 데스크톱 앱 (뷰, 컨트롤, 테마)
  Cmux.Core/    터미널 엔진, 모델, 서비스, 지속성, IPC
  Cmux.Cli/     자동화용 CLI 클라이언트
tests/
  Cmux.Tests/   유닛 테스트
```

---

## upstream 동기화

```powershell
git remote -v   # origin = cskwork, upstream = mkurman 이어야 함
git fetch upstream
git merge upstream/main
```

GitHub 웹 UI의 "Sync fork" 버튼도 동일하게 작동합니다.

---

## 라이선스 / 출처

- 라이선스: **MIT** (upstream `Copyright (c) 2026 Mariusz Kurman` 원형 보존)
- 자세한 출처 / 비주장 / 동기화 안내: [NOTICE.md](./NOTICE.md)
- 영문 원본 README: [README.en.md](./README.en.md)
