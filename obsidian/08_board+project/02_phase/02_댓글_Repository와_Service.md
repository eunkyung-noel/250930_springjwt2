# 2단계: 고급 기능 및 API

## 2. 댓글 기능: Repository와 Service

`Comment` 엔티티를 데이터베이스와 연동하기 위한 `CommentRepository`와 댓글 관련 비즈니스 로직을 처리하는 `CommentService`를 구현합니다.

### 1. `CommentRepository`

Spring Data JPA를 사용하여 `Comment` 엔티티에 대한 데이터베이스 작업을 처리합니다.

- `findAllByBoardId(Long boardId)`: 특정 게시글(`boardId`)에 달린 모든 댓글을 조회하는 쿼리 메서드입니다.

```java
// src/main/java/com/example/boardpjt/model/repository/CommentRepository.java

package com.example.boardpjt.model.repository;

import com.example.boardpjt.model.entity.Comment;
import org.springframework.data.jpa.repository.JpaRepository;

import java.util.List;

public interface CommentRepository extends JpaRepository<Comment, Long> {
    List<Comment> findAllByBoardId(Long boardId);
}
```

### 2. `CommentService`

댓글 생성, 조회, 수정, 삭제 등 비즈니스 로직을 담당합니다.

- **`getComments()`**: 특정 게시글의 모든 댓글을 조회하여 `CommentResponseDto` 리스트로 변환하여 반환합니다.
- **`createComment()`**: 댓글을 생성합니다.
  - `userAccountRepository.findByUsername()`: 현재 인증된 사용자의 `UserAccount` 엔티티를 조회합니다.
  - `boardRepository.findById()`: 댓글이 달릴 `Board` 엔티티를 조회합니다.
  - `CommentCreateRequestDto`의 정보를 바탕으로 `Comment` 엔티티를 생성하고 저장합니다.
- **`updateComment()`**: 댓글을 수정합니다.
  - 댓글 작성자와 현재 로그인한 사용자가 동일한지 확인하여 수정 권한을 검사합니다.
- **`deleteComment()`**: 댓글을 삭제합니다.
  - 수정과 마찬가지로 삭제 권한을 검사합니다.

```java
// src/main/java/com/example/boardpjt/service/CommentService.java

package com.example.boardpjt.service;

import com.example.boardpjt.model.dto.CommentCreateRequestDto;
import com.example.boardpjt.model.dto.CommentResponseDto;
import com.example.boardpjt.model.entity.Board;
import com.example.boardpjt.model.entity.Comment;
import com.example.boardpjt.model.entity.UserAccount;
import com.example.boardpjt.model.repository.BoardRepository;
import com.example.boardpjt.model.repository.CommentRepository;
import com.example.boardpjt.model.repository.UserAccountRepository;
import jakarta.persistence.EntityNotFoundException;
import lombok.RequiredArgsConstructor;
import org.springframework.security.core.userdetails.UsernameNotFoundException;
import org.springframework.stereotype.Service;
import org.springframework.transaction.annotation.Transactional;

import java.util.List;
import java.util.stream.Collectors;

@Service
@RequiredArgsConstructor
@Transactional(readOnly = true)
public class CommentService {

    private final CommentRepository commentRepository;
    private final BoardRepository boardRepository;
    private final UserAccountRepository userAccountRepository;

    public List<CommentResponseDto> getComments(Long boardId) {
        return commentRepository.findAllByBoardId(boardId).stream()
                .map(CommentResponseDto::fromEntity)
                .collect(Collectors.toList());
    }

    @Transactional
    public CommentResponseDto createComment(CommentCreateRequestDto request, String username) {
        UserAccount user = userAccountRepository.findByUsername(username)
                .orElseThrow(() -> new UsernameNotFoundException("사용자를 찾을 수 없습니다."));
        Board board = boardRepository.findById(request.getBoardId())
                .orElseThrow(() -> new EntityNotFoundException("게시글을 찾을 수 없습니다."));

        Comment comment = new Comment();
        comment.setUserAccount(user);
        comment.setBoard(board);
        comment.setContent(request.getContent());
        comment.setParentCommentId(request.getParentCommentId());

        Comment savedComment = commentRepository.save(comment);
        return CommentResponseDto.fromEntity(savedComment);
    }

    @Transactional
    public void updateComment(Long commentId, String content, String username) {
        Comment comment = commentRepository.findById(commentId)
                .orElseThrow(() -> new EntityNotFoundException("댓글을 찾을 수 없습니다."));

        if (!comment.getUserAccount().getUsername().equals(username)) {
            throw new SecurityException("수정 권한이 없습니다.");
        }

        comment.setContent(content);
    }

    @Transactional
    public void deleteComment(Long commentId, String username) {
        Comment comment = commentRepository.findById(commentId)
                .orElseThrow(() -> new EntityNotFoundException("댓글을 찾을 수 없습니다."));

        if (!comment.getUserAccount().getUsername().equals(username)) {
            throw new SecurityException("삭제 권한이 없습니다.");
        }

        commentRepository.delete(comment);
    }
}
```

이제 댓글 데이터를 처리하는 백엔드 로직이 완성되었습니다. 다음 장에서는 이 서비스들을 호출하여 비동기적으로 댓글 기능을 처리하는 REST API 컨트롤러와 프론트엔드 JavaScript 코드를 작성합니다.
