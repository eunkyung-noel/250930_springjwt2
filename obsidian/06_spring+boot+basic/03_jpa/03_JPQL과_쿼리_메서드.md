# JPQL과 쿼리 메서드

`JpaRepository`가 제공하는 기본 CRUD 메서드만으로는 복잡한 비즈니스 요구사항을 모두 처리하기 어렵습니다. Spring Data JPA는 이러한 경우를 위해 **쿼리 메서드(Query Method)**와 **JPQL(@Query)**이라는 두 가지 강력한 조회 기능을 제공합니다.

## 1. 쿼리 메서드 (Query Method)

쿼리 메서드는 **메서드 이름을 분석하여 JPQL 쿼리를 자동으로 생성**하는 기능입니다. 정해진 명명 규칙에 따라 리포지토리 인터페이스에 메서드를 선언하기만 하면, Spring Data JPA가 런타임에 해당 구현을 제공합니다.

### 1.1. 명명 규칙

`find...By...`, `read...By...`, `get...By...`, `count...By...`, `exists...By...` 등의 접두사로 시작하며, `By` 뒤에는 엔티티의 필드 이름을 조합하여 조회 조건을 명시합니다.

- **기본 조회**:

  - `List<Post> findByAuthor(String author);`
    - `SELECT p FROM Post p WHERE p.author = ?1`
  - `Optional<Post> findByTitle(String title);`
    - `SELECT p FROM Post p WHERE p.title = ?1`

- **조건 조합 (And, Or)**:

  - `List<Post> findByAuthorAndTitle(String author, String title);`
    - `... WHERE p.author = ?1 AND p.title = ?2`
  - `List<Post> findByTitleOrContent(String title, String content);`
    - `... WHERE p.title = ?1 OR p.content = ?2`

- **다양한 조건 키워드**:
  - `Is`, `Equals`: 동일성 비교 (생략 가능)
    - `findByTitleIs(String title);`
  - `Between`: 특정 범위 사이
    - `List<Post> findByCreatedAtBetween(LocalDateTime start, LocalDateTime end);`
  - `LessThan`, `GreaterThan`: 대소 비교
    - `List<Post> findByIdLessThan(Long id);`
  - `IsNull`, `IsNotNull`: `NULL` 여부
    - `List<Post> findByUpdatedAtIsNull();`
  - `Like`, `Containing`: 문자열 포함 여부 (`%keyword%`)
    - `List<Post> findByTitleContaining(String keyword);`
  - `StartingWith`, `EndingWith`: 특정 문자열로 시작/끝
    - `List<Post> findByTitleStartingWith(String prefix);`
  - `In`: 여러 값 중 하나에 포함
    - `List<Post> findByAuthorIn(List<String> authors);`

### 1.2. 정렬 및 페이징

- **정렬 (OrderBy)**:

  - `List<Post> findByAuthorOrderByCreatedAtDesc(String author);`
    - `... WHERE p.author = ?1 ORDER BY p.createdAt DESC`

- **페이징 (`Pageable`)**:
  - `Page<Post> findByAuthor(String author, Pageable pageable);`
  - `Pageable` 객체는 `PageRequest.of(페이지번호, 페이지크기, 정렬)`로 생성할 수 있습니다.
  - 반환 타입인 `Page` 객체는 전체 데이터 수, 전체 페이지 수, 현재 페이지 번호 등 페이징 처리에 유용한 정보를 담고 있습니다.

```java
// Service 계층에서 페이징 사용 예시
import org.springframework.data.domain.Page;
import org.springframework.data.domain.PageRequest;
import org.springframework.data.domain.Pageable;
import org.springframework.data.domain.Sort;

// ...

public Page<PostDto.Response> findPostsByPage(int page, int size) {
    // 0페이지부터 시작, 10개씩, 생성일 기준 내림차순 정렬
    Pageable pageable = PageRequest.of(page, size, Sort.by("createdAt").descending());
    Page<Post> postPage = postRepository.findAll(pageable); // JpaRepository 기본 메서드
    return postPage.map(PostDto.Response::new); // Page<Post> -> Page<PostDto.Response>
}
```

> **쿼리 메서드의 장단점**
>
> - **장점**: 간단한 쿼리를 매우 직관적이고 빠르게 작성할 수 있습니다.
> - **단점**: 메서드 이름이 너무 길어지고 복잡해질 수 있으며, 동적 쿼리나 조인(Join) 같은 복잡한 로직을 구현하기 어렵습니다.

---

## 2. JPQL과 `@Query`

JPQL(Java Persistence Query Language)은 **엔티티 객체를 대상**으로 하는 객체지향 쿼리 언어입니다. 데이터베이스 테이블이 아닌 엔티티 클래스와 필드 이름을 사용하여 쿼리를 작성합니다.

`@Query` 어노테이션을 사용하면 리포지토리 메서드에 직접 JPQL을 작성할 수 있어, 쿼리 메서드로 표현하기 어려운 복잡한 조회를 처리할 수 있습니다.

### 2.1. 기본 사용법

- 파라미터는 이름 기반(`:name`) 또는 위치 기반(`?1`)으로 바인딩할 수 있습니다.

```java
import org.springframework.data.jpa.repository.Query;
import org.springframework.data.repository.query.Param;

public interface PostRepository extends JpaRepository<Post, Long> {

    // 이름 기반 파라미터 바인딩
    @Query("SELECT p FROM Post p WHERE p.author = :author AND p.title LIKE %:keyword%")
    List<Post> findByAuthorAndTitleKeyword(@Param("author") String author, @Param("keyword") String keyword);

    // 위치 기반 파라미터 바인딩
    @Query("SELECT p FROM Post p ORDER BY p.createdAt DESC")
    List<Post> findAllDesc();
}
```

### 2.2. DTO로 직접 조회

JPQL의 `new` 키워드를 사용하면 조회 결과를 엔티티가 아닌 DTO 객체로 직접 매핑할 수 있습니다. 이는 필요한 필드만 선택적으로 조회하여 성능을 최적화하는 **프로젝션(Projection)** 기법입니다.

- **주의**: `new` 키워드 뒤에는 DTO의 **패키지 경로를 포함한 전체 클래스 이름**을 명시해야 하며, DTO에는 JPQL 조회 결과의 순서와 타입이 일치하는 생성자가 있어야 합니다.

```java
// dto/PostSummaryDto.java
@Getter
public class PostSummaryDto {
    private final Long id;
    private final String title;
    private final String author;

    public PostSummaryDto(Long id, String title, String author) {
        this.id = id;
        this.title = title;
        this.author = author;
    }
}

// PostRepository.java
public interface PostRepository extends JpaRepository<Post, Long> {

    @Query("SELECT new com.example.jpa.dto.PostSummaryDto(p.id, p.title, p.author) FROM Post p ORDER BY p.createdAt DESC")
    List<PostSummaryDto> findPostSummaries();
}
```

### 2.3. 네이티브 쿼리 (Native Query)

JPQL로 표현하기 어려운 복잡한 쿼리나 특정 데이터베이스에 종속적인 기능을 사용해야 할 경우, `@Query`의 `nativeQuery = true` 속성을 사용하여 순수 SQL을 실행할 수 있습니다.

```java
@Query(
    value = "SELECT * FROM post WHERE title LIKE %?1% ORDER BY created_at DESC LIMIT 10",
    nativeQuery = true
)
List<Post> findTop10ByTitleWithNativeQuery(String keyword);
```

> **네이티브 쿼리 사용 시 주의점**
>
> - 데이터베이스에 종속적이 되므로, DB 변경 시 쿼리 수정이 필요할 수 있습니다.
> - 페이징, 정렬 등 Spring Data JPA의 부가 기능을 활용하기 어려울 수 있습니다.
> - 가급적 JPQL을 우선적으로 고려하고, 최후의 수단으로 사용하는 것이 좋습니다.

### 2.4. 수정/삭제 쿼리 (`@Modifying`)

`@Query`는 기본적으로 조회(SELECT)용이지만, `@Modifying` 어노테이션을 함께 사용하면 `UPDATE`나 `DELETE` 같은 데이터 변경 쿼리도 실행할 수 있습니다.

- **주의**: 벌크(Bulk) 연산은 영속성 컨텍스트를 무시하고 데이터베이스에 직접 실행됩니다. 따라서 영속성 컨텍스트의 상태와 데이터베이스의 상태가 달라질 수 있으므로, `clearAutomatically = true` 옵션을 통해 연산 직후 영속성 컨텍스트를 초기화하는 것이 안전합니다.

```java
import org.springframework.data.jpa.repository.Modifying;

public interface PostRepository extends JpaRepository<Post, Long> {

    @Modifying(clearAutomatically = true)
    @Query("UPDATE Post p SET p.author = :newAuthor WHERE p.author = :oldAuthor")
    int updateAuthorName(@Param("oldAuthor") String oldAuthor, @Param("newAuthor") String newAuthor);
}
```
