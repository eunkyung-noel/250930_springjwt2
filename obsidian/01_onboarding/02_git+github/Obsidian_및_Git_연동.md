# Obsidian과 Git 연동

#obsidian #git #sync #backup #workflow

## 1. Obsidian 보관함(Vault) 생성

1.  **Obsidian**을 실행하고 `보관함 폴더 열기`를 선택합니다.
2.  로컬 컴퓨터에 보관함으로 사용할 새 폴더를 생성하고 선택합니다. (예: `문서/MyObsidianVault`)

## 2. 보관함을 Git 저장소로 초기화

1.  생성한 보관함 폴더에서 터미널(또는 Git Bash)을 엽니다.
    - **Windows**: 폴더에서 마우스 우클릭 > `Git Bash Here`
    - **macOS**: 폴더에서 마우스 우클릭 > `새로운 터미널 열기`
2.  다음 명령어를 실행하여 Git 저장소로 만듭니다.

```bash
git init
```

## 3. GitHub에 백업용 원격 저장소 생성

1.  [github.com/new](https://github.com/new)에서 새로운 **비공개(Private)** 저장소를 생성합니다. 개인 노트이므로 비공개로 설정하는 것을 권장합니다.
2.  생성된 저장소의 URL을 복사합니다.

## 4. 로컬 보관함과 원격 저장소 연결 및 푸시

1.  터미널에서 다음 명령어들을 순서대로 실행합니다.

```bash
# 원격 저장소 연결
git remote add origin <복사한_GitHub_URL>

# 기본 브랜치 이름을 'main'으로 설정
git branch -M main

# 모든 파일을 Staging
git add .

# 첫 커밋 생성
git commit -m "Initial commit: Setup Obsidian vault"

# 원격 저장소로 푸시
git push -u origin main
```

## 5. 동기화 워크플로우

- 이제 Obsidian에서 노트를 작성하고 변경사항을 Git과 GitHub를 통해 동기화할 수 있습니다.

### 변경사항 보내기 (Push)

- 로컬에서 노트 수정 후, 터미널에서 다음 명령어를 실행하여 변경사항을 GitHub에 업로드합니다.

```bash
# 1. 변경된 파일 추가
git add .

# 2. 커밋 생성
git commit -m "docs: Update my notes"

# 3. 원격으로 푸시
git push
```

- 위 세 명령어를 합쳐서 사용할 수도 있습니다.

```bash
git add . && git commit -m "Update notes" && git push
```

### 변경사항 받기 (Pull)

- 다른 컴퓨터에서 작업하거나, 원격 저장소의 변경사항을 로컬로 가져올 때 사용합니다.

```bash
git pull
```

- **충돌(Conflict) 발생 시**: 만약 로컬과 원격에서 같은 파일의 같은 부분을 수정했다면 충돌이 발생할 수 있습니다. 이 경우, 충돌이 발생한 파일을 직접 열어 수정하고 다시 커밋해야 합니다.
- **강제 덮어쓰기 (주의!)**: 만약 로컬 변경사항을 무시하고 원격 저장소의 내용으로 완전히 덮어쓰고 싶다면 다음 명령어를 사용할 수 있습니다. **로컬 변경사항이 모두 사라지므로 매우 주의해야 합니다.**

```bash
git reset --hard origin/main
```
