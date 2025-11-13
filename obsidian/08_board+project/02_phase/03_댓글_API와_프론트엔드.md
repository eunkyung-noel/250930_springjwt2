# 2단계: 고급 기능 및 API

## 3. 댓글 기능: REST API와 프론트엔드

댓글 기능은 페이지 전체를 새로고침하지 않고 동적으로 처리하는 것이 사용자 경험에 좋습니다. 이를 위해 REST API를 만들고, 프론트엔드에서는 JavaScript의 `fetch` API를 사용하여 비동기적으로 서버와 통신합니다.

### 1. `CommentApiController` - 댓글 REST API

`@RestController` 어노테이션을 사용하여 이 컨트롤러의 모든 메서드가 뷰(View) 대신 JSON과 같은 데이터를 직접 반환하도록 합니다.

- `@RequestMapping("/api/v1/comments")`: API 버전 관리를 위해 URL에 버전을 명시합니다.
- `@AuthenticationPrincipal UserDetails userDetails`: 현재 인증된 사용자의 정보를 가져옵니다.
- **`getComments()`**: `GET /api/v1/comments/{boardId}`
  - 특정 게시글의 댓글 목록을 조회하여 `ResponseEntity<List<CommentResponseDto>>` 형태로 반환합니다.
- **`createComment()`**: `POST /api/v1/comments`
  - `@RequestBody CommentCreateRequestDto request`: 요청의 본문(body)에 담긴 JSON 데이터를 `CommentCreateRequestDto` 객체로 변환하여 받습니다.
  - 댓글을 생성하고, 생성된 댓글 정보를 `ResponseEntity<CommentResponseDto>` 형태로 반환합니다.
- **`updateComment()`**: `PUT /api/v1/comments/{commentId}`
  - 댓글을 수정하고, 성공 시 `ResponseEntity.ok().build()` (HTTP 200 OK)를 반환합니다.
- **`deleteComment()`**: `DELETE /api/v1/comments/{commentId}`
  - 댓글을 삭제하고, 성공 시 `ResponseEntity.ok().build()` (HTTP 200 OK)를 반환합니다.
- **예외 처리**: `try-catch` 블록을 사용하여 `EntityNotFoundException`이나 `SecurityException` 발생 시 적절한 HTTP 상태 코드(404 Not Found, 403 Forbidden)를 반환합니다.

```java
// src/main/java/com/example/boardpjt/controller/CommentApiController.java

package com.example.boardpjt.controller;

import com.example.boardpjt.model.dto.CommentCreateRequestDto;
import com.example.boardpjt.model.dto.CommentResponseDto;
import com.example.boardpjt.service.CommentService;
import jakarta.persistence.EntityNotFoundException;
import lombok.RequiredArgsConstructor;
import org.springframework.http.HttpStatus;
import org.springframework.http.ResponseEntity;
import org.springframework.security.core.annotation.AuthenticationPrincipal;
import org.springframework.security.core.userdetails.UserDetails;
import org.springframework.web.bind.annotation.*;

import java.util.List;

@RestController
@RequiredArgsConstructor
@RequestMapping("/api/v1/comments")
public class CommentApiController {

    private final CommentService commentService;

    @GetMapping("/{boardId}")
    public ResponseEntity<List<CommentResponseDto>> getComments(@PathVariable Long boardId) {
        List<CommentResponseDto> comments = commentService.getComments(boardId);
        return ResponseEntity.ok(comments);
    }

    @PostMapping
    public ResponseEntity<CommentResponseDto> createComment(@RequestBody CommentCreateRequestDto request, @AuthenticationPrincipal UserDetails userDetails) {
        CommentResponseDto createdComment = commentService.createComment(request, userDetails.getUsername());
        return ResponseEntity.status(HttpStatus.CREATED).body(createdComment);
    }

    @PutMapping("/{commentId}")
    public ResponseEntity<Void> updateComment(@PathVariable Long commentId, @RequestBody String content, @AuthenticationPrincipal UserDetails userDetails) {
        try {
            commentService.updateComment(commentId, content, userDetails.getUsername());
            return ResponseEntity.ok().build();
        } catch (SecurityException e) {
            return ResponseEntity.status(HttpStatus.FORBIDDEN).build();
        } catch (EntityNotFoundException e) {
            return ResponseEntity.status(HttpStatus.NOT_FOUND).build();
        }
    }

    @DeleteMapping("/{commentId}")
    public ResponseEntity<Void> deleteComment(@PathVariable Long commentId, @AuthenticationPrincipal UserDetails userDetails) {
        try {
            commentService.deleteComment(commentId, userDetails.getUsername());
            return ResponseEntity.ok().build();
        } catch (SecurityException e) {
            return ResponseEntity.status(HttpStatus.FORBIDDEN).build();
        } catch (EntityNotFoundException e) {
            return ResponseEntity.status(HttpStatus.NOT_FOUND).build();
        }
    }
}
```

### 2. 프론트엔드 JavaScript

게시글 상세 페이지(`board-detail.html`)에 댓글 UI와 JavaScript 코드를 추가합니다.

#### 댓글 UI (`board-detail.html`)

- 댓글 목록을 표시할 `<div id="comment-list"></div>` 영역을 만듭니다.
- 댓글 작성을 위한 `<textarea>`와 `<button>`을 포함하는 폼을 만듭니다.

```html
<!-- src/main/resources/templates/board-detail.html -->

<!-- ... (게시글 내용) ... -->

<hr />
<h3>댓글</h3>
<!-- 댓글 작성 폼 -->
<div>
  <textarea
    id="comment-content"
    rows="3"
    placeholder="댓글을 입력하세요"
  ></textarea>
  <button id="comment-submit-btn">등록</button>
</div>

<!-- 댓글 목록 -->
<div id="comment-list">
  <!-- 댓글이 동적으로 추가될 영역 -->
</div>

<!-- ... (스크립트 추가) ... -->
```

#### JavaScript 로직 (`board-detail.html` 내 `<script>` 태그)

- **`loadComments()`**: 페이지 로드 시 `fetch`를 사용하여 `GET /api/v1/comments/{boardId}`를 호출하고, 받아온 댓글 목록을 동적으로 화면에 렌더링합니다.
- **댓글 작성**: '등록' 버튼 클릭 시 `fetch`를 사용하여 `POST /api/v1/comments`를 호출합니다. 요청 본문에 `boardId`, `content`를 JSON 형태로 담아 전송합니다. 성공 시 댓글 목록을 다시 로드하여 방금 작성한 댓글을 포함한 최신 목록을 보여줍니다.
- **이벤트 위임(Event Delegation)**: 댓글 목록(`comment-list`)에 이벤트 리스너를 하나만 등록하여, 동적으로 추가되는 '수정', '삭제' 버튼의 클릭 이벤트를 효율적으로 처리합니다.
  - **댓글 삭제**: '삭제' 버튼 클릭 시 `fetch`를 사용하여 `DELETE /api/v1/comments/{commentId}`를 호출하고, 성공 시 해당 댓글 요소를 DOM에서 제거합니다.
  - **댓글 수정**: '수정' 버튼 클릭 시 댓글 내용을 `input`으로 바꾸고, '저장' 버튼을 누르면 `fetch`를 사용하여 `PUT /api/v1/comments/{commentId}`를 호출합니다.

```javascript
// src/main/resources/templates/board-detail.html 내 <script>

const boardId = [[${board.id}]];
const commentList = document.getElementById('comment-list');
const commentContent = document.getElementById('comment-content');
const commentSubmitBtn = document.getElementById('comment-submit-btn');

// 댓글 목록 로드 함수
async function loadComments() {
    const response = await fetch(`/api/v1/comments/${boardId}`);
    const comments = await response.json();
    commentList.innerHTML = ''; // 기존 목록 초기화

    comments.forEach(comment => {
        const commentEl = document.createElement('div');
        commentEl.dataset.commentId = comment.id;
        commentEl.innerHTML = `
            <strong>${comment.username}</strong> (${comment.createdAt})
            <p>${comment.content}</p>
            <button class="edit-btn">수정</button>
            <button class="delete-btn">삭제</button>
        `;
        commentList.appendChild(commentEl);
    });
}

// 댓글 작성
commentSubmitBtn.addEventListener('click', async () => {
    const content = commentContent.value;
    if (!content) {
        alert('댓글 내용을 입력하세요.');
        return;
    }

    const response = await fetch('/api/v1/comments', {
        method: 'POST',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify({ boardId, content, parentCommentId: null })
    });

    if (response.ok) {
        commentContent.value = '';
        loadComments(); // 목록 새로고침
    } else {
        alert('댓글 작성에 실패했습니다.');
    }
});

// 댓글 수정 및 삭제 (이벤트 위임)
commentList.addEventListener('click', async (e) => {
    const commentEl = e.target.closest('[data-comment-id]');
    if (!commentEl) return;
    const commentId = commentEl.dataset.commentId;

    // 삭제 처리
    if (e.target.classList.contains('delete-btn')) {
        if (!confirm('정말로 삭제하시겠습니까?')) return;

        const response = await fetch(`/api/v1/comments/${commentId}`, { method: 'DELETE' });
        if (response.ok) {
            commentEl.remove();
        } else {
            alert('삭제 권한이 없거나 오류가 발생했습니다.');
        }
    }

    // 수정 처리 (간략화된 예시)
    if (e.target.classList.contains('edit-btn')) {
        const newContent = prompt('수정할 내용을 입력하세요:', commentEl.querySelector('p').textContent);
        if (!newContent) return;

        const response = await fetch(`/api/v1/comments/${commentId}`, {
            method: 'PUT',
            headers: { 'Content-Type': 'application/json' },
            body: JSON.stringify(newContent)
        });

        if (response.ok) {
            loadComments(); // 목록 새로고침
        } else {
            alert('수정 권한이 없거나 오류가 발생했습니다.');
        }
    }
});


// 페이지 로드 시 댓글 목록 로드
window.addEventListener('DOMContentLoaded', loadComments);
```

이제 사용자는 페이지를 새로고침하지 않고도 댓글을 작성, 수정, 삭제할 수 있습니다. 다음 장에서는 팔로우/언팔로우 기능을 구현합니다.
