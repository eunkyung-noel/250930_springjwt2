# 2단계: 고급 기능 및 API

1단계에서 구현한 기본 인증 및 CRUD 기능 위에 댓글, 팔로우 등 소셜 기능을 추가하고, 비동기 통신을 위한 REST API를 설계합니다.

## 1. 댓글 기능: 엔티티와 DTO

게시글에 대한 댓글 기능을 구현하기 위해 `Comment` 엔티티와 관련 DTO를 설계합니다.

### 1. `Comment` 엔티티

댓글 데이터를 저장하는 `Comment` 엔티티입니다.

- **`userAccount` 관계 (작성자)**: 댓글을 작성한 사용자를 나타냅니다. `UserAccount`와 다대일(N:1) 관계입니다.
- **`board` 관계 (게시글)**: 댓글이 달린 게시글을 나타냅니다. `Board`와 다대일(N:1) 관계입니다.
- **`parentCommentId` (대댓글)**: 대댓글 기능을 위해 부모 댓글의 ID를 저장하는 필드입니다. 부모 댓글이 없는 경우(최상위 댓글) `null` 값을 가집니다. 이 필드를 통해 계층적인 댓글 구조를 표현할 수 있습니다.

```java
// src/main/java/com/example/boardpjt/model/entity/Comment.java

package com.example.boardpjt.model.entity;

import jakarta.persistence.*;
import lombok.Getter;
import lombok.Setter;

@Entity
@Getter
@Setter
public class Comment extends BaseEntity {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    @ManyToOne(fetch = FetchType.LAZY)
    @JoinColumn(name = "user_account_id")
    private UserAccount userAccount;

    @ManyToOne(fetch = FetchType.LAZY)
    @JoinColumn(name = "board_id")
    private Board board;

    @Column(nullable = false, columnDefinition = "TEXT")
    private String content;

    // 대댓글 기능을 위한 필드
    private Long parentCommentId;
}
```

### 2. `CommentDto`

댓글 데이터 전송을 위한 DTO입니다. 댓글 생성 요청과 응답에 사용됩니다.

#### `CommentCreateRequestDto`

- 댓글 생성 시 클라이언트에서 서버로 필요한 최소한의 데이터를 전달하기 위한 DTO입니다.
- `boardId`: 어느 게시글에 달리는 댓글인지 식별합니다.
- `content`: 댓글 내용입니다.
- `parentCommentId`: 대댓글인 경우 부모 댓글의 ID를, 최상위 댓글인 경우 `null`을 전달합니다.

```java
// src/main/java/com/example/boardpjt/model/dto/CommentDto.java

package com.example.boardpjt.model.dto;

import lombok.Getter;
import lombok.Setter;

// ... (다른 DTO들)

@Getter
@Setter
public class CommentCreateRequestDto {
    private Long boardId;
    private String content;
    private Long parentCommentId;
}
```

#### `CommentResponseDto`

- 서버가 클라이언트로 댓글 정보를 응답할 때 사용하는 DTO입니다.
- `username`: 댓글 작성자의 이름을 포함하여 화면에 표시할 수 있도록 합니다.
- `fromEntity()`: `Comment` 엔티티를 `CommentResponseDto`로 변환하는 정적 팩토리 메서드입니다.

```java
// src/main/java/com/example/boardpjt/model/dto/CommentDto.java

package com.example.boardpjt.model.dto;

import com.example.boardpjt.model.entity.Comment;
import lombok.AllArgsConstructor;
import lombok.Getter;

import java.time.format.DateTimeFormatter;

// ... (다른 DTO들)

@Getter
@AllArgsConstructor
public class CommentResponseDto {
    private Long id;
    private String content;
    private String username;
    private String createdAt;

    public static CommentResponseDto fromEntity(Comment comment) {
        return new CommentResponseDto(
                comment.getId(),
                comment.getContent(),
                comment.getUserAccount().getUsername(),
                comment.getCreatedAt().format(DateTimeFormatter.ofPattern("yyyy-MM-dd HH:mm"))
        );
    }
}
```

이제 댓글 데이터를 관리하기 위한 모델링이 완료되었습니다. 다음 장에서는 이 엔티티와 DTO를 사용하여 댓글 CRUD 로직을 처리하는 `Repository`와 `Service`를 구현합니다.
