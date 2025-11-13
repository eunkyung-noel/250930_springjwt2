# 03. reduce, 수집, 그리고 forEach

#java #reduce #collect #수집 #forEach #optional #결합법칙

---

https://github.com/notlikelion/250805_fp/tree/main/src/step1

## reduce

- 정의: 요소를 누적해 단일 결과를 만드는 최종 연산
- 주의: 연산은 결합법칙을 만족해야 병렬/순차에서 일관

예시

```java
int sum = IntStream.rangeClosed(1, 5).reduce(0, Integer::sum); // 15
String concat = Stream.of("a", "b", "c").reduce("", String::concat); // abc
```

리스트 합/곱

```java
List<Integer> l1 = List.of(1, 5, 4, 3, 7);
int s = l1.stream().reduce(0, (cur, acc) -> cur + acc);
int p = l1.stream().reduce(1, (cur, acc) -> cur * acc);
```

문자열 누적

```java
List<String> list = List.of("apple", "asia", "anaconda", "bear", "pear");
String joined = list.stream().reduce("%s,%s"::formatted).orElse("");
```

---

## collect와 Collectors

```java
List<Integer> squares = IntStream.rangeClosed(1, 5).map(n -> n*n).boxed().collect(Collectors.toList());
Map<Integer, List<String>> byLen = list.stream().collect(Collectors.groupingBy(String::length));
```

---

## forEach 주의사항

- forEach는 소비(Consumer)로 사이드 이펙트를 유발할 수 있음
- 체이닝 종결자이므로 이후 파이프라인 연산 불가

```java
list.stream().forEach(System.out::println); // 소모 후 더 이어갈 수 없음
```
