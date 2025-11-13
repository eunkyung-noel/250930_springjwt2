---
tags:
  - html
  - github
  - deployment
  - web
---

# 04. 실습: GitHub Pages로 웹사이트 배포하기

지금까지 배운 HTML 지식을 활용하여 간단한 자기소개 페이지를 만들고, **GitHub Pages**를 통해 전 세계 어디서든 접속할 수 있는 실제 웹사이트로 무료 배포하는 과정을 실습합니다.

#학습목표

- `index.html` 파일이 웹 서버에서 어떤 역할을 하는지 이해합니다.
- Git과 GitHub를 사용하여 프로젝트를 원격 저장소에 업로드할 수 있습니다.
- GitHub Pages 설정을 통해 정적 웹사이트를 성공적으로 배포하고, 고유 URL을 확인할 수 있습니다.

---

## 1. 프로젝트 준비: `index.html` 작성

웹 서버는 특별히 파일명을 지정하지 않고 디렉토리 경로로 접속했을 때, 기본적으로 `index.html` 파일을 찾아 보여줍니다. 따라서 우리가 만들 웹 페이지의 메인 파일 이름은 `index.html`로 지정해야 합니다.

#인덱스파일 #index_html #홈페이지

### 실습용 자기소개 페이지 예제

아래 코드를 `index.html`이라는 이름의 파일로 저장하세요. 지금까지 배운 시맨틱 태그, 이미지, 링크, 목록 등을 활용한 간단한 예제입니다.

```html
<!DOCTYPE html>
<html lang="ko">
  <head>
    <meta charset="UTF-8" />
    <meta name="viewport" content="width=device-width, initial-scale=1.0" />
    <title>홍길동의 포트폴리오</title>
    <style>
      body {
        font-family: sans-serif;
        line-height: 1.6;
        margin: 0 auto;
        max-width: 800px;
        padding: 20px;
      }
      header,
      footer {
        text-align: center;
        padding: 20px 0;
        background-color: #f4f4f4;
      }
      img.profile {
        width: 150px;
        height: 150px;
        border-radius: 50%;
      }
      section {
        margin-bottom: 20px;
      }
      h1,
      h2 {
        color: #333;
      }
    </style>
  </head>
  <body>
    <header>
      <img src="https://i.pravatar.cc/150" alt="프로필 사진" class="profile" />
      <h1>홍길동 | 웹 개발자 지망생</h1>
      <p>코드로 세상을 이롭게 만드는 것을 꿈꿉니다.</p>
    </header>

    <main>
      <section>
        <h2>소개</h2>
        <p>
          안녕하세요! 저는 Java와 SQL에 능숙하며, 최근에는 HTML, CSS,
          JavaScript를 집중적으로 학습하며 프론트엔드 개발 역량을 키우고
          있습니다.
        </p>
      </section>

      <section>
        <h2>기술 스택</h2>
        <ul>
          <li><b>언어:</b> Java, SQL, JavaScript</li>
          <li><b>프레임워크:</b> Spring Boot</li>
          <li><b>데이터베이스:</b> MySQL, PostgreSQL</li>
        </ul>
      </section>

      <section>
        <h2>연락처</h2>
        <p>다양한 기회를 기다리고 있습니다. 아래 링크로 연락주세요.</p>
        <ul>
          <li><a href="mailto:gildong@example.com">이메일</a></li>
          <li>
            <a href="https://github.com/your-username" target="_blank"
              >GitHub 프로필</a
            >
          </li>
        </ul>
      </section>
    </main>

    <footer>
      <p>&copy; 2024 홍길동. All rights reserved.</p>
    </footer>
  </body>
</html>
```

_`i.pravatar.cc`는 임의의 프로필 이미지를 제공하는 서비스입니다. 자신의 이미지 파일로 교체해도 좋습니다._
_GitHub 프로필 링크의 `your-username` 부분은 본인의 GitHub 사용자명으로 변경하세요._

---

## 2. GitHub 저장소 생성 및 코드 푸시

#깃허브 #github #저장소 #repository #푸시 #push

1.  **새로운 저장소(Repository) 생성**:

    - GitHub에 로그인한 후, 우측 상단의 `+` 아이콘을 클릭하고 `New repository`를 선택합니다.
    - `Repository name`을 원하는 대로 입력합니다. (예: `my-portfolio`)
    - `Public`으로 설정해야 GitHub Pages를 무료로 사용할 수 있습니다.
    - `Create repository` 버튼을 클릭하여 저장소를 생성합니다.

2.  **로컬 프로젝트와 연결 및 푸시**:

    - 위에서 작성한 `index.html` 파일이 있는 폴더(디렉토리)에서 터미널 또는 Git Bash를 엽니다.
    - 아래 명령어를 순서대로 입력하여 코드를 GitHub 원격 저장소에 업로드합니다.

    ```bash
    # 1. Git 초기화
    git init

    # 2. 원격 저장소 연결 (HTTPS 주소는 생성된 저장소 페이지에서 복사)
    git remote add origin https://github.com/your-username/my-portfolio.git

    # 3. 모든 파일을 Staging 영역에 추가
    git add .

    # 4. 변경사항 커밋
    git commit -m "첫 번째 포트폴리오 페이지 커밋"

    # 5. 기본 브랜치 이름을 main으로 설정
    git branch -M main

    # 6. 원격 저장소로 푸시
    git push -u origin main
    ```

---

## 3. GitHub Pages 활성화 및 배포

#깃허브페이지 #github_pages #배포 #deployment

1.  **Settings 탭으로 이동**: 코드를 푸시한 GitHub 저장소 페이지에서 `Settings` 탭을 클릭합니다.

2.  **Pages 메뉴 선택**: 왼쪽 사이드바에서 `Pages` 메뉴를 선택합니다.

3.  **소스(Source) 설정**:

    - `Build and deployment` 섹션에서 `Source`를 `Deploy from a branch`로 선택합니다.
    - `Branch` 섹션에서 배포할 브랜치를 `main`으로, 폴더는 `/(root)`로 설정하고 `Save` 버튼을 클릭합니다.

4.  **배포 확인**:
    - 잠시 후(1~2분 소요) 페이지가 새로고침되면, 상단에 "Your site is live at `https://your-username.github.io/my-portfolio/`" 라는 메시지와 함께 녹색 배경의 URL이 표시됩니다.
    - 이 URL을 클릭하면 방금 만든 자기소개 웹 페이지가 성공적으로 배포된 것을 확인할 수 있습니다.

이제 여러분은 자신만의 웹사이트를 갖게 되었습니다! `index.html` 파일을 수정하고 다시 `git push`를 하면, GitHub Pages가 자동으로 변경사항을 감지하여 몇 분 내에 사이트를 업데이트해 줍니다.

이것으로 HTML 기초 학습을 마칩니다. 이 지식을 바탕으로 CSS와 JavaScript를 학습하면 더욱 동적이고 아름다운 웹 페이지를 만들 수 있게 될 것입니다.
