# 📦 사내 RAG 챗봇 시스템 - 설치 가이드

**버전**: 1.1  
**작성일**: 2024-01-15  
**대상**: 시스템 관리자, DevOps  

---

## 📑 목차

1. [사전 준비사항](#1-사전-준비사항)
2. [Phase 1: Windows Server 환경 구축](#2-phase-1-windows-server-환경-구축)
3. [Phase 2: Docker 설치 및 설정](#3-phase-2-docker-설치-및-설정)
4. [Phase 3: 컨테이너 배포](#4-phase-3-컨테이너-배포)
5. [Phase 4: n8n 워크플로우 설정](#5-phase-4-n8n-워크플로우-설정)
6. [Phase 5: 세션 정리 자동화](#6-phase-5-세션-정리-자동화)
7. [Phase 6: 테스트 및 검증](#7-phase-6-테스트-및-검증)
8. [트러블슈팅](#8-트러블슈팅)
9. [백업 및 복구](#9-백업-및-복구)

---

## 1. 사전 준비사항

### 1.1 하드웨어 요구사항

시나리오 2 (권장 사양) 기준:

| 구성요소 | 최소 사양 | 권장 사양 |
|---------|----------|----------|
| **CPU** | 8코어 | 16코어 (AMD EPYC 7313P / Intel Xeon Silver 4314) |
| **RAM** | 32GB | 64GB DDR4 ECC |
| **SSD** | 250GB | 500GB NVMe Gen3 |
| **HDD** | - | 2TB SATA (백업용) |
| **네트워크** | 1Gbps | 1Gbps |

### 1.2 소프트웨어 준비

| 소프트웨어 | 버전 | 다운로드 링크 |
|----------|------|-------------|
| **Windows Server 2022** 또는<br>**Windows 10/11 Pro** | 최신 | Microsoft 공식 사이트 |
| **Docker Desktop** | v24.x 이상 | https://www.docker.com/products/docker-desktop |
| **Git** (선택) | 최신 | https://git-scm.com/downloads |
| **VS Code** (선택) | 최신 | https://code.visualstudio.com/ |

### 1.3 네트워크 준비

```yaml
서버 IP 설정:
  - 고정 IP 주소: 192.168.x.x (예: 192.168.1.100)
  - 서브넷 마스크: 255.255.255.0
  - 게이트웨이: 192.168.x.1
  - DNS: 8.8.8.8, 8.8.4.4

포트 개방 확인:
  - 5678 (n8n 웹 인터페이스)
  - 6333 (Qdrant, 로컬만)
  - 11434 (Ollama, 로컬만)
```

### 1.4 필요한 파일 목록

다음 파일들을 준비하세요:

- [ ] `docker-compose.yml` (Phase 3에서 생성)
- [ ] `admin-upload-workflow.json` (n8n 워크플로우)
- [ ] `user-chat-workflow.json` (n8n 워크플로우)
- [ ] `admin-history-workflow.json` (n8n 워크플로우)
- [ ] `cleanup-sessions.ps1` (세션 정리 스크립트)
- [ ] `admin.html` (관리자 웹 페이지)
- [ ] `chat.html` (사용자 챗봇 페이지)

---

## 2. Phase 1: Windows Server 환경 구축

### 2.1 Windows 업데이트

**Step 1: Windows Update 실행**

```powershell
# PowerShell (관리자 권한)
# Windows 업데이트 확인
Get-WindowsUpdate

# 업데이트 설치
Install-WindowsUpdate -AcceptAll -AutoReboot
```

**또는 GUI 방법:**
1. `설정` → `업데이트 및 보안` → `Windows Update`
2. `업데이트 확인` 클릭
3. 모든 업데이트 설치

### 2.2 방화벽 규칙 설정

```powershell
# PowerShell (관리자 권한)

# n8n 포트 (5678) - 사내망만 허용
New-NetFirewallRule -DisplayName "n8n Web Access" `
    -Direction Inbound `
    -LocalPort 5678 `
    -Protocol TCP `
    -RemoteAddress 192.168.0.0/16,10.0.0.0/8,172.16.0.0/12 `
    -Action Allow

# Qdrant 포트 (6333) - 로컬만 허용
New-NetFirewallRule -DisplayName "Qdrant API" `
    -Direction Inbound `
    -LocalPort 6333 `
    -Protocol TCP `
    -RemoteAddress 127.0.0.1 `
    -Action Allow

# Ollama 포트 (11434) - 로컬만 허용
New-NetFirewallRule -DisplayName "Ollama API" `
    -Direction Inbound `
    -LocalPort 11434 `
    -Protocol TCP `
    -RemoteAddress 127.0.0.1 `
    -Action Allow

Write-Host "✅ 방화벽 규칙 설정 완료"
```

### 2.3 폴더 구조 생성

```powershell
# PowerShell (관리자 권한)

# 기본 디렉토리 생성
$basePath = "C:\n8n-data"

$folders = @(
    "$basePath\uploads",
    "$basePath\sessions",
    "$basePath\history",
    "$basePath\config",
    "$basePath\logs",
    "C:\n8n-scripts"
)

foreach ($folder in $folders) {
    if (!(Test-Path $folder)) {
        New-Item -Path $folder -ItemType Directory -Force
        Write-Host "✅ Created: $folder"
    }
}

Write-Host "✅ 폴더 구조 생성 완료"
```

### 2.4 고정 IP 설정

**GUI 방법:**
1. `제어판` → `네트워크 및 인터넷` → `네트워크 연결`
2. 이더넷 어댑터 우클릭 → `속성`
3. `인터넷 프로토콜 버전 4 (TCP/IPv4)` 선택 → `속성`
4. 다음 설정 입력:
   ```
   IP 주소: 192.168.1.100
   서브넷 마스크: 255.255.255.0
   기본 게이트웨이: 192.168.1.1
   기본 설정 DNS: 8.8.8.8
   보조 DNS: 8.8.4.4
   ```

**PowerShell 방법:**
```powershell
# PowerShell (관리자 권한)
New-NetIPAddress -InterfaceAlias "이더넷" `
    -IPAddress 192.168.1.100 `
    -PrefixLength 24 `
    -DefaultGateway 192.168.1.1

Set-DnsClientServerAddress -InterfaceAlias "이더넷" `
    -ServerAddresses 8.8.8.8,8.8.4.4
```

---

## 3. Phase 2: Docker 설치 및 설정

### 3.1 Docker Desktop 설치

**Step 1: Docker Desktop 다운로드**
1. https://www.docker.com/products/docker-desktop 접속
2. `Download for Windows` 클릭
3. 설치 파일 실행 (`Docker Desktop Installer.exe`)

**Step 2: 설치 진행**
1. 설치 마법사 따라가기
2. `Use WSL 2 instead of Hyper-V` 옵션 체크 (권장)
3. 설치 완료 후 재부팅

**Step 3: Docker Desktop 실행**
1. Docker Desktop 아이콘 실행
2. 서비스 약관 동의
3. 로그인 (선택사항, 개인 계정 또는 건너뛰기)

### 3.2 Docker 설정 확인

```powershell
# PowerShell
# Docker 버전 확인
docker --version
# 출력 예시: Docker version 24.0.7, build afdd53b

# Docker Compose 버전 확인
docker-compose --version
# 출력 예시: Docker Compose version v2.23.0

# Docker 상태 확인
docker info
```

### 3.3 Docker Desktop 리소스 할당

1. Docker Desktop 실행
2. 설정 (톱니바퀴 아이콘) 클릭
3. `Resources` → `Advanced` 선택
4. 리소스 할당:
   ```
   CPUs: 12 (전체의 75%)
   Memory: 48GB (전체의 75%)
   Swap: 4GB
   Disk image size: 200GB
   ```
5. `Apply & Restart` 클릭

---

## 4. Phase 3: 컨테이너 배포

### 4.1 Docker Compose 파일 생성

**파일 위치**: `C:\n8n-project\docker-compose.yml`

```yaml
version: '3.8'

services:
  # n8n - 워크플로우 엔진
  n8n:
    image: n8nio/n8n:latest
    container_name: n8n
    restart: unless-stopped
    ports:
      - "5678:5678"
    environment:
      - N8N_HOST=192.168.1.100
      - N8N_PORT=5678
      - N8N_PROTOCOL=http
      - WEBHOOK_URL=http://192.168.1.100:5678/
      - GENERIC_TIMEZONE=Asia/Seoul
      - TZ=Asia/Seoul
    volumes:
      - C:\n8n-data:/home/node/.n8n
    networks:
      - rag-network

  # Ollama - LLM 실행 엔진
  ollama:
    image: ollama/ollama:latest
    container_name: ollama
    restart: unless-stopped
    ports:
      - "11434:11434"
    volumes:
      - ollama-data:/root/.ollama
    networks:
      - rag-network

  # Qdrant - 벡터 데이터베이스
  qdrant:
    image: qdrant/qdrant:latest
    container_name: qdrant
    restart: unless-stopped
    ports:
      - "6333:6333"
      - "6334:6334"
    volumes:
      - qdrant-data:/qdrant/storage
    networks:
      - rag-network

networks:
  rag-network:
    driver: bridge

volumes:
  ollama-data:
  qdrant-data:
```

### 4.2 컨테이너 실행

```powershell
# PowerShell
cd C:\n8n-project

# 컨테이너 시작
docker-compose up -d

# 출력 예시:
# [+] Running 4/4
#  ✔ Network rag-network    Created
#  ✔ Container n8n          Started
#  ✔ Container ollama       Started
#  ✔ Container qdrant       Started

# 컨테이너 상태 확인
docker-compose ps

# 출력 예시:
# NAME      IMAGE                    STATUS
# n8n       n8nio/n8n:latest        Up 30 seconds
# ollama    ollama/ollama:latest    Up 30 seconds
# qdrant    qdrant/qdrant:latest    Up 30 seconds
```

### 4.3 Ollama 모델 다운로드

```powershell
# PowerShell

# exaone3.5 7B 모델 다운로드 (약 4.5GB)
docker exec -it ollama ollama pull exaone3.5

# 출력 예시:
# pulling manifest
# pulling 8934d96d3f08... 100%
# pulling 8c17c2ebb0ea... 100%
# verifying sha256 digest
# writing manifest
# success

# nomic-embed-text 모델 다운로드 (약 274MB)
docker exec -it ollama ollama pull nomic-embed-text

# 모델 다운로드 확인
docker exec -it ollama ollama list

# 출력 예시:
# NAME                    ID              SIZE
# exaone3.5:latest        abc123def456    4.5 GB
# nomic-embed-text:latest xyz789uvw012    274 MB
```

### 4.4 Qdrant 컬렉션 생성

**방법 1: PowerShell (REST API)**

```powershell
# PowerShell

# Qdrant에 documents 컬렉션 생성
$body = @{
    vectors = @{
        size = 768
        distance = "Cosine"
    }
} | ConvertTo-Json

Invoke-RestMethod -Uri "http://localhost:6333/collections/documents" `
    -Method Put `
    -ContentType "application/json" `
    -Body $body

Write-Host "✅ Qdrant 컬렉션 생성 완료"
```

**방법 2: 웹 브라우저 (Qdrant 대시보드)**

1. 브라우저에서 `http://localhost:6333/dashboard` 접속
2. 좌측 메뉴 `Collections` 클릭
3. `Create Collection` 버튼 클릭
4. 다음 설정 입력:
   ```
   Collection name: documents
   Vector size: 768
   Distance: Cosine
   ```
5. `Create` 버튼 클릭

### 4.5 접속 확인

| 서비스 | URL | 확인 방법 |
|--------|-----|----------|
| **n8n** | http://192.168.1.100:5678 | 웹 브라우저 접속, 초기 계정 생성 화면 확인 |
| **Qdrant** | http://localhost:6333/dashboard | 대시보드 접속, `documents` 컬렉션 확인 |
| **Ollama** | http://localhost:11434 | PowerShell: `curl http://localhost:11434` |

---

## 5. Phase 4: n8n 워크플로우 설정

### 5.1 n8n 초기 설정

**Step 1: n8n 접속**
1. 브라우저에서 `http://192.168.1.100:5678` 접속
2. 계정 생성:
   ```
   Email: admin@company.com
   Password: [강력한 비밀번호]
   ```

**Step 2: n8n 기본 설정**
1. 좌측 상단 메뉴 → `Settings`
2. `General` → `Timezone` → `Asia/Seoul` 선택
3. `Save` 클릭

### 5.2 워크플로우 임포트

**Step 1: 관리자 문서 업로드 워크플로우**

1. n8n 메인 화면에서 `+` 버튼 클릭
2. 우측 상단 `⋯` (메뉴) → `Import from File...` 선택
3. `admin-upload-workflow.json` 파일 선택
4. 워크플로우 이름: `Admin Document Upload`
5. `Save` 버튼 클릭 (Ctrl+S)
6. 우측 상단 `Active` 토글 활성화

**Step 2: 사용자 챗봇 워크플로우**

1. 동일한 방법으로 `user-chat-workflow.json` 임포트
2. 워크플로우 이름: `User Chatbot (with Session)`
3. 저장 및 활성화

**Step 3: 관리자 히스토리 워크플로우**

1. `admin-history-workflow.json` 임포트
2. 워크플로우 이름: `Admin History Viewer`
3. 저장 및 활성화

### 5.3 Webhook URL 확인

각 워크플로우에서 Webhook URL을 확인하세요:

```
관리자 문서 업로드: http://192.168.1.100:5678/webhook/admin-upload
사용자 챗봇: http://192.168.1.100:5678/webhook/chat
관리자 히스토리: http://192.168.1.100:5678/webhook/admin-history
```

### 5.4 환경 변수 설정 (워크플로우 내)

각 워크플로우의 HTTP Request 노드에서 URL 확인:

```yaml
Ollama API:
  - URL: http://ollama:11434/api/embeddings
  - URL: http://ollama:11434/api/generate

Qdrant API:
  - URL: http://qdrant:6333/collections/documents/points
  - URL: http://qdrant:6333/collections/documents/points/search
```

**주의**: 컨테이너 간 통신은 서비스명(ollama, qdrant)을 사용합니다!

---

## 6. Phase 5: 세션 정리 자동화

### 6.1 세션 정리 스크립트 생성

**파일 위치**: `C:\n8n-scripts\cleanup-sessions.ps1`

```powershell
# cleanup-sessions.ps1
# 작업 스케줄러로 매시간 실행

$sessionDir = "C:\n8n-data\sessions"
$maxAge = (Get-Date).AddHours(-1)  # 1시간 이상 된 세션 삭제
$logFile = "C:\n8n-data\logs\session_cleanup.log"

# 로그 함수
function Write-Log {
    param($Message)
    $timestamp = Get-Date -Format "yyyy-MM-dd HH:mm:ss"
    $logMessage = "[$timestamp] $Message"
    Write-Host $logMessage
    Add-Content -Path $logFile -Value $logMessage
}

Write-Log "=== Session cleanup started ==="

if (Test-Path $sessionDir) {
    $deletedCount = 0
    $errorCount = 0
    
    Get-ChildItem $sessionDir -Filter "*.json" | ForEach-Object {
        try {
            $session = Get-Content $_.FullName | ConvertFrom-Json
            $lastAccess = [DateTime]$session.last_access
            
            if ($lastAccess -lt $maxAge) {
                Remove-Item $_.FullName -Force
                Write-Log "Deleted expired session: $($_.Name)"
                $deletedCount++
            }
        } catch {
            # 손상된 파일 삭제
            Remove-Item $_.FullName -Force
            Write-Log "Deleted corrupted session: $($_.Name)"
            $errorCount++
        }
    }
    
    $remainingSessions = (Get-ChildItem $sessionDir -Filter "*.json").Count
    Write-Log "Summary: Deleted=$deletedCount, Errors=$errorCount, Active=$remainingSessions"
} else {
    Write-Log "ERROR: Session directory not found: $sessionDir"
}

Write-Log "=== Session cleanup completed ==="
```

### 6.2 작업 스케줄러 등록

```powershell
# PowerShell (관리자 권한)

# 작업 스케줄러 작업 생성
$action = New-ScheduledTaskAction -Execute "PowerShell.exe" `
    -Argument "-ExecutionPolicy Bypass -File C:\n8n-scripts\cleanup-sessions.ps1"

$trigger = New-ScheduledTaskTrigger -Once -At (Get-Date) `
    -RepetitionInterval (New-TimeSpan -Hours 1) `
    -RepetitionDuration ([TimeSpan]::MaxValue)

$principal = New-ScheduledTaskPrincipal -UserId "SYSTEM" `
    -LogonType ServiceAccount -RunLevel Highest

Register-ScheduledTask -TaskName "n8n-SessionCleanup" `
    -Action $action `
    -Trigger $trigger `
    -Principal $principal `
    -Description "n8n 챗봇 세션 정리 (1시간마다)"

Write-Host "✅ 작업 스케줄러 등록 완료"

# 작업 확인
Get-ScheduledTask -TaskName "n8n-SessionCleanup"
```

### 6.3 스크립트 수동 실행 (테스트)

```powershell
# PowerShell (관리자 권한)
C:\n8n-scripts\cleanup-sessions.ps1

# 로그 확인
Get-Content C:\n8n-data\logs\session_cleanup.log -Tail 20
```

---

## 7. Phase 6: 테스트 및 검증

### 7.1 시스템 상태 확인

```powershell
# PowerShell

# Docker 컨테이너 상태
docker-compose ps

# 예상 출력:
# NAME      STATUS
# n8n       Up 2 hours
# ollama    Up 2 hours
# qdrant    Up 2 hours

# 컨테이너 로그 확인
docker logs n8n --tail 50
docker logs ollama --tail 50
docker logs qdrant --tail 50

# 포트 리스닝 확인
netstat -an | Select-String "5678|6333|11434"
```

### 7.2 문서 업로드 테스트

**Step 1: 테스트 문서 준비**
- 샘플 PDF 파일 (10페이지 이내)
- 파일명: `test_document.pdf`

**Step 2: Postman 또는 curl로 테스트**

```powershell
# PowerShell (curl 사용)
$url = "http://192.168.1.100:5678/webhook/admin-upload"
$filePath = "C:\test_document.pdf"

curl.exe -X POST $url `
    -F "file=@$filePath"

# 예상 응답:
# {
#   "status": "success",
#   "doc_id": "doc_1705123456789",
#   "filename": "test_document.pdf",
#   "chunks": 23
# }
```

**Step 3: Qdrant 확인**
1. http://localhost:6333/dashboard 접속
2. `documents` 컬렉션 클릭
3. Points 수 확인 (23개 이상이어야 함)

### 7.3 챗봇 질의응답 테스트

```powershell
# PowerShell
$url = "http://192.168.1.100:5678/webhook/chat"
$body = @{
    question = "테스트 문서에 대해 설명해주세요"
    session_id = "test_session_001"
} | ConvertTo-Json

$response = Invoke-RestMethod -Uri $url `
    -Method Post `
    -ContentType "application/json" `
    -Body $body

# 응답 확인
$response | ConvertTo-Json -Depth 10
```

### 7.4 세션 관리 테스트

**Step 1: 첫 질문**
```powershell
$body = @{
    question = "2024년 휴가 정책은?"
    session_id = "test_session_002"
} | ConvertTo-Json

Invoke-RestMethod -Uri $url -Method Post -ContentType "application/json" -Body $body
```

**Step 2: 후속 질문**
```powershell
$body = @{
    question = "그럼 신청은 어떻게 해?"
    session_id = "test_session_002"  # 동일한 세션 ID
} | ConvertTo-Json

Invoke-RestMethod -Uri $url -Method Post -ContentType "application/json" -Body $body
```

**Step 3: 세션 파일 확인**
```powershell
Get-ChildItem C:\n8n-data\sessions\test_session_002.json
Get-Content C:\n8n-data\sessions\test_session_002.json | ConvertFrom-Json
```

### 7.5 성능 테스트

```powershell
# PowerShell - 응답 시간 측정
$stopwatch = [System.Diagnostics.Stopwatch]::StartNew()

$body = @{
    question = "회사 정책에 대해 알려주세요"
    session_id = "perf_test_001"
} | ConvertTo-Json

$response = Invoke-RestMethod -Uri $url -Method Post -ContentType "application/json" -Body $body

$stopwatch.Stop()
Write-Host "응답 시간: $($stopwatch.ElapsedMilliseconds)ms"
# 목표: 5,000-8,000ms (5-8초)
```

### 7.6 체크리스트

설치 완료 확인:

- [ ] Docker 컨테이너 3개 모두 실행 중 (n8n, ollama, qdrant)
- [ ] Ollama 모델 2개 다운로드 완료 (exaone3.5, nomic-embed-text)
- [ ] Qdrant `documents` 컬렉션 생성 완료
- [ ] n8n 워크플로우 3개 임포트 및 활성화
- [ ] 세션 정리 스크립트 작업 스케줄러 등록
- [ ] 문서 업로드 테스트 성공
- [ ] 챗봇 질의응답 테스트 성공
- [ ] 세션 관리 테스트 성공 (후속 질문)
- [ ] 평균 응답 시간 10초 이하

---

## 8. 트러블슈팅

### 8.1 Docker 컨테이너 시작 실패

**증상**: `docker-compose up -d` 실행 시 컨테이너가 시작되지 않음

**해결 방법**:
```powershell
# 로그 확인
docker-compose logs

# 특정 컨테이너 로그
docker logs n8n
docker logs ollama
docker logs qdrant

# 컨테이너 재시작
docker-compose restart

# 완전 재배포
docker-compose down
docker-compose up -d
```

### 8.2 Ollama 모델 다운로드 실패

**증상**: `ollama pull exaone3.5` 명령이 실패하거나 멈춤

**해결 방법**:
```powershell
# Ollama 컨테이너 재시작
docker restart ollama

# 디스크 공간 확인
Get-PSDrive C

# 다시 시도
docker exec -it ollama ollama pull exaone3.5
```

### 8.3 n8n Webhook 접속 불가

**증상**: 브라우저에서 `http://192.168.1.100:5678` 접속 안 됨

**해결 방법**:
```powershell
# 방화벽 규칙 확인
Get-NetFirewallRule -DisplayName "n8n Web Access"

# 포트 리스닝 확인
netstat -an | Select-String "5678"

# n8n 컨테이너 재시작
docker restart n8n

# n8n 로그 확인
docker logs n8n --tail 100
```

### 8.4 Qdrant 검색 결과 없음

**증상**: 챗봇이 "해당 정보를 찾을 수 없습니다" 계속 응답

**해결 방법**:
```powershell
# Qdrant 컬렉션 확인
Invoke-RestMethod -Uri "http://localhost:6333/collections/documents"

# Points 수 확인 (0개면 문서 업로드 필요)
# 출력 예시:
# {
#   "result": {
#     "vectors_count": 230,
#     "points_count": 230
#   }
# }

# 문서 재업로드 필요 시
# 1. 테스트 문서로 다시 업로드
# 2. Qdrant 대시보드에서 Points 수 확인
```

### 8.5 세션 정리 스크립트 실행 안 됨

**증상**: 오래된 세션 파일이 자동 삭제되지 않음

**해결 방법**:
```powershell
# 작업 스케줄러 상태 확인
Get-ScheduledTask -TaskName "n8n-SessionCleanup"

# 작업 수동 실행
Start-ScheduledTask -TaskName "n8n-SessionCleanup"

# 로그 확인
Get-Content C:\n8n-data\logs\session_cleanup.log -Tail 20

# 작업 재등록 (필요시)
Unregister-ScheduledTask -TaskName "n8n-SessionCleanup" -Confirm:$false
# (6.2 작업 스케줄러 등록 스크립트 재실행)
```

### 8.6 응답 속도 너무 느림 (15초 이상)

**원인**: CPU 부하 또는 동시 요청 많음

**해결 방법**:
```powershell
# CPU 사용률 확인
Get-Counter '\Processor(_Total)\% Processor Time'

# RAM 사용률 확인
Get-Counter '\Memory\Available MBytes'

# Docker 리소스 제한 조정
# Docker Desktop → Settings → Resources
# CPU: 14 → 16 증가
# Memory: 48GB → 56GB 증가
```

### 8.7 일반적인 오류 메시지

| 오류 메시지 | 원인 | 해결 방법 |
|------------|------|----------|
| `Connection refused` | 서비스 미실행 | 컨테이너 재시작 |
| `404 Not Found` | 잘못된 URL | Webhook URL 확인 |
| `500 Internal Server Error` | 워크플로우 오류 | n8n 로그 확인 |
| `Out of memory` | RAM 부족 | Docker 메모리 할당 증가 |
| `Disk full` | 디스크 공간 부족 | 오래된 로그/세션 삭제 |

---

## 9. 백업 및 복구

### 9.1 백업 스크립트

**파일 위치**: `C:\n8n-scripts\backup-system.ps1`

```powershell
# backup-system.ps1
$date = Get-Date -Format "yyyyMMdd"
$backupRoot = "D:\Backups"
$backupDir = "$backupRoot\backup_$date"

# 백업 디렉토리 생성
New-Item -Path $backupDir -ItemType Directory -Force

Write-Host "=== Backup started: $date ==="

# 1. Qdrant 데이터 백업
docker exec qdrant qdrant-backup --output /qdrant/storage/backup-$date
Copy-Item "C:\docker-data\qdrant\backup-$date" "$backupDir\qdrant\" -Recurse
Write-Host "✅ Qdrant backup completed"

# 2. n8n 데이터 백업
Copy-Item "C:\n8n-data" "$backupDir\n8n-data\" -Recurse -Exclude "sessions"
Write-Host "✅ n8n data backup completed"

# 3. Docker Compose 파일 백업
Copy-Item "C:\n8n-project\docker-compose.yml" "$backupDir\"
Write-Host "✅ Docker Compose backup completed"

# 4. 스크립트 백업
Copy-Item "C:\n8n-scripts" "$backupDir\scripts\" -Recurse
Write-Host "✅ Scripts backup completed"

# 압축
Compress-Archive -Path $backupDir -DestinationPath "$backupRoot\backup_$date.zip"
Write-Host "✅ Backup compressed: backup_$date.zip"

Write-Host "=== Backup completed ==="
```

### 9.2 복구 절차

**전체 시스템 복구 (재해 복구)**

```powershell
# PowerShell (관리자 권한)

# 1. 백업 파일 압축 해제
$backupFile = "D:\Backups\backup_20240115.zip"
$restoreDir = "C:\restore_temp"
Expand-Archive -Path $backupFile -DestinationPath $restoreDir

# 2. Docker 컨테이너 중지
cd C:\n8n-project
docker-compose down

# 3. 데이터 복원
Copy-Item "$restoreDir\n8n-data\*" "C:\n8n-data\" -Recurse -Force
Copy-Item "$restoreDir\qdrant\*" "C:\docker-data\qdrant\" -Recurse -Force

# 4. 컨테이너 재시작
docker-compose up -d

# 5. 복원 확인
docker-compose ps
curl http://192.168.1.100:5678

Write-Host "✅ 복구 완료"
```

---

## 🎉 설치 완료!

축하합니다! 사내 RAG 챗봇 시스템 설치가 완료되었습니다.

### 다음 단계:
1. 웹 UI 파일 배포 (`admin.html`, `chat.html`)
2. 실제 문서 업로드 시작
3. 직원들에게 챗봇 URL 공유
4. 일일 모니터링 시작

### 접속 URL:
- **관리자 페이지**: http://192.168.1.100:5678/admin.html
- **사용자 챗봇**: http://192.168.1.100:5678/chat.html
- **n8n 워크플로우**: http://192.168.1.100:5678
- **Qdrant 대시보드**: http://localhost:6333/dashboard

### 지원:
- 프로젝트 문서: `RAG-Chatbot-Project-Plan.md`
- GitHub 저장소: (저장소 URL)
- 문의: admin@company.com

**Happy chatting! 🚀**
