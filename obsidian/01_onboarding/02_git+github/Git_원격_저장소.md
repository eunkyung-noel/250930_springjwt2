# 원격 저장소(GitHub) 연동

#git #github #remote #push #pull #clone #authentication

## 1. GitHub에서 원격 저장소 생성

1.  [github.com/new](https://github.com/new) 로 이동합니다.
2.  **Repository name**을 입력합니다.
3.  **Public** (공개) 또는 **Private** (비공개)를 선택합니다.
4.  **Create repository** 버튼을 클릭합니다.
5.  생성된 저장소의 URL (예: `https://github.com/username/repo-name.git`)을 복사합니다.

## 2. 로컬 저장소와 원격 저장소 연결

### 원격 저장소 추가

- 로컬 저장소에 원격 저장소를 `origin`이라는 이름으로 등록합니다. `origin`은 관례적으로 사용하는 이름입니다.

```bash
git remote add origin <복사한_URL>
```

### 원격 저장소 확인 및 관리

```bash
# 등록된 원격 저장소 목록 확인
git remote -v

# 원격 저장소 URL 변경
git remote set-url origin <새로운_URL>

# 원격 저장소 이름 변경
git remote rename origin new-origin

# 원격 저장소 삭제
git remote remove origin
```

## 3. 로컬 변경사항을 원격 저장소로 보내기 (Push)

- 로컬 저장소의 커밋을 원격 저장소로 업로드합니다.

```bash
# 현재 브랜치를 origin의 main 브랜치로 푸시
git push origin main

# -u 옵션으로 업스트림 브랜치 설정 (최초 1회)
# 이후에는 'git push'만으로 해당 브랜치에 푸시 가능
git push -u origin main
```

### 강제 푸시 (Force Push)

- **주의**: 협업 시 데이터 유실의 위험이 있으므로 신중하게 사용해야 합니다.
- 원격 저장소의 이력을 무시하고 로컬 이력으로 덮어씁니다.

```bash
# 일반 강제 푸시 (위험)
git push -f origin main
# 또는
git push --force origin main

# 안전한 강제 푸시 (원격에 내가 모르는 변경사항이 없으면 푸시)
git push --force-with-lease origin main
```

## 4. 원격 저장소의 변경사항 가져오기

### Clone

- 원격 저장소를 통째로 로컬에 복제합니다.

```bash
git clone <원격_저장소_URL> [폴더명]
```

### Pull

- 원격 저장소의 최신 변경사항을 가져와 현재 브랜치와 병합(merge)합니다.

```bash
git pull origin main
```

## 5. 인증 정보 문제 해결

- `Permission denied` 오류가 발생하면 다른 GitHub 계정으로 인증 정보가 저장되어 있을 수 있습니다.

### Windows

1.  **제어판** > **자격 증명 관리자** > **Windows 자격 증명**으로 이동합니다.
2.  `git:https://github.com` 항목을 찾아 삭제합니다.

### macOS

- `gh` (GitHub CLI)를 사용하면 편리하게 관리할 수 있습니다.

```bash
# gh 설치 (Homebrew 필요)
brew install gh

# gh 버전 확인
gh --version

# 로그아웃
gh auth logout -h github.com

# 다시 로그인
gh auth login
```

- 로그인 시 `HTTPS` 프로토콜을 선택하고 웹 브라우저를 통해 인증을 진행합니다.
