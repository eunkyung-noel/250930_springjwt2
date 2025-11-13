# Repository와 CRUD

Repository는 데이터베이스에 접근하여 데이터의 조회, 저장, 수정, 삭제(CRUD) 작업을 처리하는 계층입니다. Spring Data JPA는 `JpaRepository` 인터페이스를 제공하여 이 과정을 매우 간단하게 만들어줍니다.

## 1. Repository 인터페이스 정의

`JpaRepository<T, ID>` 인터페이스를 상속받는 것만으로 기본적인 CRUD 기능을 모두 사용할 수 있습니다.

- `T`: 리포지토리에서 다룰 엔티티 클래스 (예: `Post`)
- `ID`: 해당 엔티티의 기본 키(PK) 타입 (예: `Long`, `UUID`)

```java
// repository/PostRepository.java
import com.example.jpa.domain.Post;
import org.springframework.data.jpa.repository.JpaRepository;
import org.springframework.stereotype.Repository;

@Repository // Spring Bean으로 등록
public interface PostRepository extends JpaRepository<Post, Long> {
    // 이 인터페이스는 비어있지만, JpaRepository로부터 상속받은
    // 수많은 메서드를 이미 포함하고 있습니다.
    // 예: save(), findById(), findAll(), delete(), count() 등
}
```

`@Repository` 어노테이션은 이 인터페이스가 데이터 접근 계층의 컴포넌트임을 명시하며, Spring이 자동으로 빈(Bean)으로 등록해줍니다.

## 2. 기본 CRUD 메서드 활용

`JpaRepository`가 기본으로 제공하는 주요 메서드는 다음과 같습니다.

| 메서드              | 설명                                                                 | SQL 매핑 (유사)                     |
| ------------------- | -------------------------------------------------------------------- | ----------------------------------- |
| `save(S entity)`    | 새로운 엔티티를 저장하거나, 기존 엔티티를 수정(병합)합니다.          | `INSERT` 또는 `UPDATE`              |
| `findById(ID id)`   | 기본 키(PK)로 엔티티 한 건을 조회합니다. `Optional<T>`을 반환합니다. | `SELECT ... WHERE id = ?`           |
| `findAll()`         | 모든 엔티티를 조회합니다. `List<T>`를 반환합니다.                    | `SELECT * FROM ...`                 |
| `deleteById(ID id)` | 기본 키로 엔티티 한 건을 삭제합니다.                                 | `DELETE FROM ... WHERE id = ?`      |
| `delete(T entity)`  | 주어진 엔티티를 삭제합니다.                                          | `DELETE FROM ... WHERE id = ?`      |
| `count()`           | 엔티티의 총 개수를 반환합니다.                                       | `SELECT COUNT(*) FROM ...`          |
| `existsById(ID id)` | 해당 기본 키를 가진 엔티티가 존재하는지 확인합니다.                  | `SELECT CASE WHEN COUNT(*) > 0 ...` |

---

## 3. Service와 Controller에서 Repository 사용하기

일반적으로 애플리케이션은 **Controller - Service - Repository**의 계층 구조를 가집니다.

- **Controller**: HTTP 요청을 받고 응답을 반환합니다. 비즈니스 로직은 Service 계층에 위임합니다.
- **Service**: 핵심 비즈니스 로직을 처리합니다. Repository를 사용하여 데이터베이스와 상호작용하며, 트랜잭션을 관리합니다.
- **Repository**: 데이터베이스 접근을 담당합니다.

### 3.1. DTO (Data Transfer Object)

계층 간 데이터를 전달할 때는 엔티티 클래스를 직접 사용하기보다 **DTO**를 사용하는 것이 좋습니다. DTO는 각 계층이 필요로 하는 데이터만 담는 순수한 데이터 객체입니다.

- **엔티티를 직접 사용하지 않는 이유**:
  - **API 스펙의 유연성**: 엔티티 필드가 변경되어도 API 응답 형식은 DTO를 통해 그대로 유지할 수 있습니다.
  - **보안**: 엔티티의 모든 필드(예: 비밀번호)가 외부에 노출되는 것을 방지합니다.
  - **계층 간 결합도 감소**: View와 데이터베이스 모델을 분리하여 유연한 구조를 만듭니다.

```java
// dto/PostDto.java
import com.example.jpa.domain.Post;
import lombok.Getter;
import lombok.NoArgsConstructor;
import lombok.Setter;
import java.time.LocalDateTime;

public class PostDto {

    // 게시물 등록/수정 요청을 위한 DTO
    @Getter
    @Setter
    @NoArgsConstructor
    public static class SaveRequest {
        private String title;
        private String content;
        private String author;

        // DTO를 Entity로 변환하는 메서드
        public Post toEntity() {
            return Post.builder()
                    .title(title)
                    .content(content)
                    .author(author)
                    .build();
        }
    }

    // 게시물 응답을 위한 DTO
    @Getter
    public static class Response {
        private final Long id;
        private final String title;
        private final String content;
        private final String author;
        private final LocalDateTime createdAt;
        private final LocalDateTime updatedAt;

        // Entity를 DTO로 변환하는 생성자
        public Response(Post post) {
            this.id = post.getId();
            this.title = post.getTitle();
            this.content = post.getContent();
            this.author = post.getAuthor();
            this.createdAt = post.getCreatedAt();
            this.updatedAt = post.getUpdatedAt();
        }
    }
}
```

### 3.2. Service 계층 구현

`PostService`는 `PostRepository`를 주입받아 CRUD 비즈니스 로직을 수행합니다.

- `@Transactional`: 해당 메서드가 하나의 트랜잭션으로 실행되도록 보장합니다. 쓰기 작업(CUD) 중 오류가 발생하면 모든 변경사항이 롤백됩니다.
- `@Transactional(readOnly = true)`: 조회 전용 트랜잭션으로 설정하여 성능을 최적화합니다. (JPA가 변경 감지를 위한 스냅샷을 만들지 않음)

```java
// service/PostService.java
import com.example.jpa.domain.Post;
import com.example.jpa.dto.PostDto;
import com.example.jpa.repository.PostRepository;
import lombok.RequiredArgsConstructor;
import org.springframework.stereotype.Service;
import org.springframework.transaction.annotation.Transactional;
import java.util.List;
import java.util.stream.Collectors;

@Service
@RequiredArgsConstructor // final 필드에 대한 생성자 주입
public class PostService {

    private final PostRepository postRepository;

    @Transactional
    public Long save(PostDto.SaveRequest dto) {
        Post post = dto.toEntity();
        return postRepository.save(post).getId();
    }

    @Transactional(readOnly = true)
    public List<PostDto.Response> findAll() {
        return postRepository.findAll().stream()
                .map(PostDto.Response::new) // .map(post -> new PostDto.Response(post))
                .collect(Collectors.toList());
    }

    @Transactional(readOnly = true)
    public PostDto.Response findById(Long id) {
        Post post = postRepository.findById(id)
                .orElseThrow(() -> new IllegalArgumentException("해당 게시글이 없습니다. id=" + id));
        return new PostDto.Response(post);
    }

    @Transactional
    public Long update(Long id, PostDto.SaveRequest dto) {
        Post post = postRepository.findById(id)
                .orElseThrow(() -> new IllegalArgumentException("해당 게시글이 없습니다. id=" + id));

        // Dirty Checking (변경 감지)
        // 트랜잭션이 끝날 때 JPA가 변경된 필드를 감지하여 자동으로 UPDATE 쿼리를 실행합니다.
        post.update(dto.getTitle(), dto.getContent());

        return id;
    }

    @Transactional
    public void delete(Long id) {
        Post post = postRepository.findById(id)
                .orElseThrow(() -> new IllegalArgumentException("해당 게시글이 없습니다. id=" + id));
        postRepository.delete(post);
    }
}
```

### 3.3. Controller 계층 구현

`PostController`는 `PostService`를 사용하여 클라이언트의 CRUD 요청을 처리합니다.

```java
// controller/PostController.java
import com.example.jpa.dto.PostDto;
import com.example.jpa.service.PostService;
import lombok.RequiredArgsConstructor;
import org.springframework.stereotype.Controller;
import org.springframework.ui.Model;
import org.springframework.web.bind.annotation.*;

@Controller
@RequiredArgsConstructor
@RequestMapping("/posts")
public class PostController {

    private final PostService postService;

    // 게시물 목록 조회
    @GetMapping
    public String list(Model model) {
        model.addAttribute("posts", postService.findAll());
        return "post/list"; // View 이름
    }

    // 게시물 상세 조회
    @GetMapping("/{id}")
    public String detail(@PathVariable Long id, Model model) {
        model.addAttribute("post", postService.findById(id));
        return "post/detail";
    }

    // 게시물 등록 폼
    @GetMapping("/new")
    public String createForm(Model model) {
        model.addAttribute("post", new PostDto.SaveRequest());
        return "post/form";
    }

    // 게시물 등록 처리
    @PostMapping("/new")
    public String create(@ModelAttribute("post") PostDto.SaveRequest dto) {
        postService.save(dto);
        return "redirect:/posts"; // 등록 후 목록으로 리다이렉트
    }

    // 게시물 수정 처리
    @PostMapping("/{id}/edit")
    public String update(@PathVariable Long id, @ModelAttribute("post") PostDto.SaveRequest dto) {
        postService.update(id, dto);
        return "redirect:/posts/" + id; // 수정 후 상세 페이지로 리다이렉트
    }

    // 게시물 삭제 처리
    @PostMapping("/{id}/delete")
    public String delete(@PathVariable Long id) {
        postService.delete(id);
        return "redirect:/posts"; // 삭제 후 목록으로 리다이렉트
    }
}
```
