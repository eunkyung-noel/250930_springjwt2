# Git 설치 및 초기 설정

#git #installation #configuration #install #config

## 1. Git 설치

### Windows

1.  **Git for Windows 다운로드**: [git-scm.com/downloads/win](https://git-scm.com/downloads/win)
2.  다운로드한 `Git for Windows/x64 Setup` 파일 실행.
3.  설치 옵션은 대부분 기본값으로 진행해도 무방합니다.

### macOS

- MacOS에는 Git이 기본으로 설치 되어 있습니다.

## 2. 설치 확인

- 터미널(Windows는 Git Bash)을 열고 다음 명령어를 입력하여 Git 버전을 확인합니다.

```bash
git --version
```

- 예시 출력: `git version 2.45.0.windows.1`

## 3. 최초 사용자 정보 설정

- Git 커밋에 사용될 사용자 이름과 이메일을 설정합니다. 이 정보는 GitHub 계정과 일치시키는 것이 좋습니다.

```bash
# 사용자 이름 설정 (본인 GitHub 이름으로)
git config --global user.name "Your Name"

# 사용자 이메일 설정 (본인 GitHub 이메일로)
git config --global user.email "your.email@example.com"
```

- `--global` 옵션은 해당 시스템의 모든 Git 저장소에 이 설정을 적용합니다.

### 설정 확인 및 삭제

```bash
# 설정 목록 확인
git config --list

# 이름 설정 삭제
git config --global --unset user.name

# 이메일 설정 삭제
git config --global --unset user.email
```
