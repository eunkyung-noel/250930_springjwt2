# CompletableFuture: 논블로킹 체이닝

#completablefuture #completionstage #논블로킹 #nonblocking

---

## 개념

- Future + CompletionStage 결합. `thenApply/thenAccept/thenCompose/exceptionally`로 단계 연결

## 코드

```java
List<Future<Void>> futures = new ArrayList<>();
for (int i = 0; i < 5; i++) {
    Future<Void> f = CompletableFuture.supplyAsync(() -> {
        try { Thread.sleep(1000); } catch (InterruptedException e) {}
        return 1;
    })
    .thenApply(x -> x * 100) // map
    .thenAccept(System.out::println);
    futures.add(f);
}
for (Future<Void> f : futures) f.get(); // 모두 완료 대기
```

## 팁

- 기본 풀은 ForkJoin commonPool. 필요 시 사용자 지정 풀 주입
- `thenApplyAsync` 등 Async 메서드로 단계별 스케줄링 조절

## 시각화

```mermaid
flowchart LR
  S[입력] --> A[Async 공급]
  A --> B[thenApply]
  B --> C[thenAccept]
  C --> O[출력]
```
