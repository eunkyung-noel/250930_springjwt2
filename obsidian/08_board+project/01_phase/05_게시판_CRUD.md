# 1단계: 인증 및 기본 CRUD

## 5. 게시판 CRUD 기능

기본적인 인증 기능이 완성되었으므로, 이제 게시판의 핵심 기능인 게시글 생성(Create), 조회(Read), 수정(Update), 삭제(Delete) 기능을 구현합니다.

### 1. `Board` 엔티티와 DTO

게시글 데이터를 표현하는 `Board` 엔티티와 데이터 전송을 위한 `BoardDto`를 정의합니다.

#### `Board` 엔티티

- `@ManyToOne`: `UserAccount` 엔티티와 다대일(N:1) 관계를 맺습니다. 하나의 사용자는 여러 게시글을 작성할 수 있습니다.
- `@JoinColumn(name = "user_account_id")`: 외래 키(Foreign Key) 컬럼의 이름을 `user_account_id`로 지정합니다.

```java
// src/main/java/com/example/boardpjt/model/entity/Board.java

package com.example.boardpjt.model.entity;

import jakarta.persistence.*;
import lombok.Getter;
import lombok.Setter;

@Entity
@Getter
@Setter
public class Board extends BaseEntity {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    @Column(nullable = false, length = 100)
    private String title;

    @Column(nullable = false, columnDefinition = "TEXT")
    private String content;

    @ManyToOne(fetch = FetchType.LAZY)
    @JoinColumn(name = "user_account_id")
    private UserAccount userAccount;
}
```

#### `BoardDto`

게시글 생성, 수정, 조회 시 사용될 DTO입니다. `record` 타입을 사용하여 간결하게 정의합니다.

```java
// src/main/java/com/example/boardpjt/model/dto/BoardDto.java

package com.example.boardpjt.model.dto;

import com.example.boardpjt.model.entity.Board;

import java.time.format.DateTimeFormatter;

public record BoardDto(
        Long id,
        String title,
        String content,
        String username,
        String createdAt
) {
    public static BoardDto fromEntity(Board board) {
        return new BoardDto(
                board.getId(),
                board.getTitle(),
                board.getContent(),
                board.getUserAccount().getUsername(),
                board.getCreatedAt().format(DateTimeFormatter.ofPattern("yyyy-MM-dd HH:mm"))
        );
    }
}
```

### 2. `BoardRepository`

Spring Data JPA를 사용하여 `Board` 엔티티에 대한 데이터베이스 작업을 처리합니다. `findAllByOrderByIdDesc()`는 ID를 기준으로 내림차순 정렬하여 모든 게시글을 조회하는 쿼리를 자동으로 생성합니다.

```java
// src/main/java/com/example/boardpjt/model/repository/BoardRepository.java

package com.example.boardpjt.model.repository;

import com.example.boardpjt.model.entity.Board;
import org.springframework.data.jpa.repository.JpaRepository;

import java.util.List;

public interface BoardRepository extends JpaRepository<Board, Long> {
    List<Board> findAllByOrderByIdDesc();
}
```

### 3. `BoardService`

게시판 관련 비즈니스 로직을 처리합니다.

- **`createBoard()`**: 게시글을 생성합니다.
  - `userAccountRepository.findByUsername()`: 현재 인증된 사용자(`username`)의 `UserAccount` 엔티티를 조회합니다.
  - 조회된 사용자 정보와 DTO의 내용을 바탕으로 `Board` 엔티티를 생성하고 저장합니다.
- **`getBoards()`**: 모든 게시글을 조회하여 `BoardDto` 리스트로 변환합니다.
- **`getBoard()`**: 특정 ID의 게시글을 조회합니다.
- **`updateBoard()`**: 게시글을 수정합니다.
  - 게시글 작성자와 현재 로그인한 사용자가 동일한지 확인하여 권한을 검사합니다.
- **`deleteBoard()`**: 게시글을 삭제합니다.
  - 수정과 마찬가지로 권한을 검사합니다.

```java
// src/main/java/com/example/boardpjt/service/BoardService.java

@Service
@RequiredArgsConstructor
@Transactional(readOnly = true)
public class BoardService {

    private final BoardRepository boardRepository;
    private final UserAccountRepository userAccountRepository;

    @Transactional
    public BoardDto createBoard(BoardDto dto, String username) {
        UserAccount user = userAccountRepository.findByUsername(username)
                .orElseThrow(() -> new UsernameNotFoundException("사용자를 찾을 수 없습니다."));

        Board board = new Board();
        board.setTitle(dto.title());
        board.setContent(dto.content());
        board.setUserAccount(user);

        return BoardDto.fromEntity(boardRepository.save(board));
    }

    public List<BoardDto> getBoards() {
        return boardRepository.findAllByOrderByIdDesc().stream()
                .map(BoardDto::fromEntity)
                .toList();
    }

    public BoardDto getBoard(Long boardId) {
        return boardRepository.findById(boardId)
                .map(BoardDto::fromEntity)
                .orElseThrow(() -> new EntityNotFoundException("게시글을 찾을 수 없습니다."));
    }

    @Transactional
    public void updateBoard(Long boardId, BoardDto dto, String username) {
        Board board = boardRepository.findById(boardId)
                .orElseThrow(() -> new EntityNotFoundException("게시글을 찾을 수 없습니다."));

        if (!board.getUserAccount().getUsername().equals(username)) {
            throw new SecurityException("수정 권한이 없습니다.");
        }

        board.setTitle(dto.title());
        board.setContent(dto.content());
    }

    @Transactional
    public void deleteBoard(Long boardId, String username) {
        Board board = boardRepository.findById(boardId)
                .orElseThrow(() -> new EntityNotFoundException("게시글을 찾을 수 없습니다."));

        if (!board.getUserAccount().getUsername().equals(username)) {
            throw new SecurityException("삭제 권한이 없습니다.");
        }

        boardRepository.delete(board);
    }
}
```

### 4. `BoardController`

게시판 CRUD 요청을 처리하는 컨트롤러입니다.

- `@AuthenticationPrincipal UserDetails userDetails`: 현재 인증된 사용자의 `UserDetails` 객체를 주입받아 사용자명을 얻습니다.
- **`boardList()`**: `GET /posts` 요청을 받아 게시글 목록 페이지를 보여줍니다.
- **`boardForm()`**: `GET /posts/new` 요청을 받아 게시글 작성 폼을 보여줍니다.
- **`createBoard()`**: `POST /posts` 요청을 받아 게시글을 생성하고 목록 페이지로 리다이렉트합니다.
- **`boardDetail()`**: `GET /posts/{id}` 요청을 받아 게시글 상세 페이지를 보여줍니다.
- **`updateForm()`**: `GET /posts/{id}/edit` 요청을 받아 게시글 수정 폼을 보여줍니다.
- **`updateBoard()`**: `POST /posts/{id}/edit` 요청을 받아 게시글을 수정하고 상세 페이지로 리다이렉트합니다.
- **`deleteBoard()`**: `POST /posts/{id}/delete` 요청을 받아 게시글을 삭제하고 목록 페이지로 리다이렉트합니다.

```java
// src/main/java/com/example/boardpjt/controller/BoardController.java

@Controller
@RequiredArgsConstructor
@RequestMapping("/posts")
public class BoardController {

    private final BoardService boardService;

    @GetMapping
    public String boardList(Model model) {
        model.addAttribute("boards", boardService.getBoards());
        return "board-list";
    }

    @GetMapping("/new")
    public String boardForm() {
        return "board-form";
    }

    @PostMapping
    public String createBoard(@RequestParam String title, @RequestParam String content, @AuthenticationPrincipal UserDetails userDetails) {
        boardService.createBoard(new BoardDto(null, title, content, null, null), userDetails.getUsername());
        return "redirect:/posts";
    }

    @GetMapping("/{id}")
    public String boardDetail(@PathVariable Long id, Model model) {
        model.addAttribute("board", boardService.getBoard(id));
        return "board-detail";
    }

    @GetMapping("/{id}/edit")
    public String updateForm(@PathVariable Long id, Model model) {
        model.addAttribute("board", boardService.getBoard(id));
        return "board-edit-form";
    }

    @PostMapping("/{id}/edit")
    public String updateBoard(@PathVariable Long id, @RequestParam String title, @RequestParam String content, @AuthenticationPrincipal UserDetails userDetails) {
        boardService.updateBoard(id, new BoardDto(null, title, content, null, null), userDetails.getUsername());
        return "redirect:/posts/" + id;
    }

    @PostMapping("/{id}/delete")
    public String deleteBoard(@PathVariable Long id, @AuthenticationPrincipal UserDetails userDetails) {
        boardService.deleteBoard(id, userDetails.getUsername());
        return "redirect:/posts";
    }
}
```

### 5. 뷰(View) 파일

게시판 CRUD를 위한 Thymeleaf 뷰 파일들입니다.

- **`board-list.html`**: 게시글 목록
- **`board-form.html`**: 게시글 작성 폼
- **`board-detail.html`**: 게시글 상세 보기
- **`board-edit-form.html`**: 게시글 수정 폼

이 파일들은 Bootstrap과 같은 CSS 프레임워크를 사용하여 기본적인 스타일을 적용할 수 있습니다. 각 뷰는 `th:each`, `th:object`, `th:field`, `th:if` 등의 Thymeleaf 속성을 사용하여 동적으로 데이터를 표시하고, 로그인 상태나 작성자 여부에 따라 특정 버튼(수정/삭제)을 보여주거나 숨깁니다.

이것으로 1단계의 모든 기능 구현이 완료되었습니다. 사용자는 회원가입/로그인 후 게시글을 작성, 조회, 수정, 삭제할 수 있습니다.
