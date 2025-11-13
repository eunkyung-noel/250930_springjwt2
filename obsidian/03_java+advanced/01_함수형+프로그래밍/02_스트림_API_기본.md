# 02. 스트림 API 기본

#java #stream #스트림 #pipeline #파이프라인 #lazy #지연평가 #internal-iteration #내부반복 #collectors #컬렉터

---

https://github.com/notlikelion/250805_fp/tree/main/src/step1

## 스트림 개요

- 정의: 컬렉션/배열을 선언형 파이프라인으로 처리하는 추상 계층
- 특징
  - 내부 반복 + 지연 평가(lazy)
  - 원본 불변(가급적)과 사이드 이펙트 최소화
  - 생성 → 중간(0+) → 최종(1) 단계

생성 예시

```java
List<Integer> nums = List.of(1, 2, 3);
Stream<Integer> s1 = nums.stream();
IntStream s2 = IntStream.rangeClosed(1, 5);
```

---

## map / filter

- map(Function<? super T, ? extends R>): 일대일 변환

```java
List<String> words = List.of("java", "stream");
List<Integer> lengths = words.stream().map(String::length).toList();
```

- filter(Predicate<? super T>): 조건 통과 요소만 유지

```java
List<String> ids = List.of("A1", "B12", "C3");
ids.stream().filter(s -> s.startsWith("B")).forEach(System.out::println); // B12
```

---

## sorted / distinct / limit / skip

```java
int[] arr = {1,5,7,9,2,4};
int[] sorted = Arrays.stream(arr).sorted().toArray();
List<Integer> desc = Arrays.stream(arr).boxed().sorted((a,b)->b-a).toList();
```

---

## Collectors와 partitioningBy

```java
Map<Boolean, List<Integer>> partition = IntStream.rangeClosed(1, 10)
    .boxed()
    .filter(n -> n > 2)
    .map(n -> n * n)
    .collect(Collectors.partitioningBy(n -> n % 2 == 0));
System.out.println(partition);
// {false=[9, 25, 49, 81], true=[16, 36, 64, 100]}
```

---

## 박싱/언박싱과 프리미티브 스트림

- 빈번한 숫자 연산은 박싱 비용이 크므로 IntStream/LongStream/DoubleStream 사용 권장

```java
IntStream.range(0, 3).map(x -> x * 2).sum();
```
