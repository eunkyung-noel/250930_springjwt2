# Exercise 04: 컬렉션 기반 OOP 계산기 설계와 구현

#java #oop #인터페이스 #interface #인터페이스 #추상 #abstract #오버라이딩 #overriding #오버로딩 #overloading #다형성 #polymorphism #폴리모피즘 #컬렉션 #collections #List #Map #배열 #array #예외 #exception #입력 #input #Scanner

---

https://github.com/notlikelion/250805_calculator

## 🎯 학습 목표

- 인터페이스/추상 클래스로 계약·공통 구현을 분리하고 확장성 확보
- 오버로딩/오버라이딩과 다형성으로 타입별 연산 흐름 구성
- 배열/List/Map으로 계산 이력 관리 전략 비교 및 선택
- 예외 처리로 잘못된 입력/불능 연산(0 나누기 등) 방어적 코딩 적용

## 📋 사전 준비사항

1. IDE: [[IntelliJ]]
2. 사전 학습(03_oop)
   - OOP 기초: [[01_클래스, 객체, 메서드]]
   - 상속/인터페이스: [[02_상속, 인터페이스]]
   - 예외 처리: [[03_예외 처리]]

## 📚 자주 쓰는 키워드/개념 빠른 참조

- 계약 고정: `interface` + `implements` — [[02_상속, 인터페이스]]
- 공통 구현: `abstract class` + `extends` — [[02_상속, 인터페이스]]
- 오버로딩/오버라이딩 규칙 — [[01_클래스, 객체, 메서드]] · [[02_상속, 인터페이스]]
- 예외: `throw`/`throws`/`try-catch-finally` — [[03_예외 처리]]
- 자료구조 선택: 배열(고정 길이) vs List(가변) vs Map(캐시/중복 방지)

---

## 🚀 1단계: 인터페이스로 계산기의 “계약” 정의

#인터페이스 #interface #계약

- 실생활 비유: 계산대 업무 매뉴얼(무엇을 받아 무엇을 내놓는가).
- CS 관점: 입력/출력 계약을 타입에 못 박아 교체 가능한 구조 제공.

```java
package calculator;

public interface ICalculator { // [[02_상속, 인터페이스]]
    int calculate(int num1, int num2, String operator) throws Exception;      // [[03_예외 처리]]
    double calculate(double num1, double num2, String operator) throws Exception;
    void showHistory();
}
```

---

## 🚀 2단계: 추상 클래스로 공통 로직 제공(템플릿)

#추상 #abstract #오버로딩 #overloading

- 실생활 비유: 체인 본사의 공통 레시피(세부 재료·포장은 지점 재량).
- CS 관점: 정수→실수 위임, 사칙연산 기본 구현, 공통 예외 메시지 일원화.

```java
package calculator;

public abstract class Calculator implements ICalculator { // [[01_클래스, 객체, 메서드]]
    @Override
    public int calculate(int num1, int num2, String operator) throws Exception {
        return (int) this.calculate((double) num1, (double) num2, operator); // 오버로딩 재사용
    }

    @Override
    public double calculate(double num1, double num2, String operator) throws Exception {
        switch (operator) {
            case "+": return num1 + num2;
            case "-": return num1 - num2;
            case "*": return num1 * num2; // 곱셈(보정)
            case "/": if (num2 == 0) throw new Exception("0으로 나눌 수 없습니다."); return num1 / num2;
            default: throw new Exception("지원하지 않는 연산자입니다!");
        }
    }
}
```

---

## 🚀 3단계: 구현 클래스 — 이력 저장 전략 비교

#컬렉션 #collections #배열 #List #Map

- 배열(ArrayCalculator): 고정 크기, 인덱스 수동 관리
- 리스트(ListCalculator): 가변 크기, 단순 추가
- 맵(MapCalculator): "식"을 키로 결과 캐싱(중복 연산 방지)

```java
package calculator;

public class ArrayCalculator extends Calculator {
    private final String[] historyArr; // 이력 저장
    private int head = 0;              // 다음 기록 위치
    private boolean flag = false;      // 정수/실수 중복 기록 방지

    public ArrayCalculator(){ this.historyArr = new String[20]; }
    public ArrayCalculator(int size){ this.historyArr = new String[size]; }

    @Override
    public double calculate(double num1, double num2, String operator) throws Exception {
        double result = super.calculate(num1, num2, operator);
        if (!flag) {
            if (historyArr.length <= head) throw new Exception("이력 배열 길이 초과");
            historyArr[head++] = "%f %s %f = %f".formatted(num1, operator, num2, result);
        }
        return result;
    }

    @Override
    public int calculate(int num1, int num2, String operator) throws Exception {
        flag = true; // 기록 1회만 남기기
        int result = super.calculate(num1, num2, operator);
        if (historyArr.length <= head) throw new Exception("이력 배열 길이 초과");
        historyArr[head++] = "%d %s %d = %d".formatted(num1, operator, num2, result);
        flag = false;
        return result;
    }

    @Override
    public void showHistory(){
        System.out.println("[🧮 지금까지의 계산 결과]");
        for (String s : historyArr) { if (s == null) return; System.out.println(s); }
    }
}
```

```java
package calculator;

import java.util.ArrayList;
import java.util.List;

public class ListCalculator extends Calculator {
    private final List<String> historyList = new ArrayList<>();
    private boolean flag = false;

    @Override
    public double calculate(double num1, double num2, String operator) throws Exception {
        double result = super.calculate(num1, num2, operator);
        if (!flag) historyList.add("%f %s %f = %f".formatted(num1, operator, num2, result));
        return result;
    }

    @Override
    public int calculate(int num1, int num2, String operator) throws Exception {
        flag = true;
        int result = super.calculate(num1, num2, operator);
        historyList.add("%d %s %d = %d".formatted(num1, operator, num2, result));
        flag = false;
        return result;
    }

    @Override
    public void showHistory(){
        System.out.println("[🧮 지금까지의 계산 결과]");
        for (String s : historyList) System.out.println(s);
    }
}
```

```java
package calculator;

import java.util.HashMap;
import java.util.Map;

public class MapCalculator extends Calculator {
    private final Map<String, String> historyMap = new HashMap<>(); // key: 식, value: 결과
    private boolean flag = false;

    @Override
    public double calculate(double num1, double num2, String operator) throws Exception {
        String key = "%f %s %f".formatted(num1, operator, num2);
        if (historyMap.containsKey(key)) { System.out.println("이미 계산한 결과가 있습니다"); return Double.parseDouble(historyMap.get(key)); }
        double result = super.calculate(num1, num2, operator);
        if (!flag) historyMap.put(key, "%f".formatted(result));
        return result;
    }

    @Override
    public int calculate(int num1, int num2, String operator) throws Exception {
        String key = "%d %s %d".formatted(num1, operator, num2);
        if (historyMap.containsKey(key)) { System.out.println("이미 계산한 결과가 있습니다"); return Integer.parseInt(historyMap.get(key)); }
        flag = true;
        int result = super.calculate(num1, num2, operator);
        historyMap.put(key, "%d".formatted(result));
        flag = false;
        return result;
    }

    @Override
    public void showHistory(){
        System.out.println("[🧮 지금까지의 계산 결과]");
        for (Map.Entry entry : historyMap.entrySet()) { // raw 타입 예시
            System.out.println(entry.getKey() + " = " + entry.getValue());
        }
    }
}
```

---

## 🚀 4단계: 입력 파서와 실행 프로그램

#Scanner #입력 #예외 #exception

- 입력 파서(InputHandler): 공백 기준 3항목 검증 — [[03_예외 처리]]
- 실행(Application): 입력 루프, 정수/실수 분기, 다형성으로 구현체 교체 — [[02_상속, 인터페이스]]

```java
package util;

public class InputHandler {
    public static String[] handleInput(String input) throws Exception {
        String[] arr = input.split(" ");
        if (arr.length != 3) throw new Exception("잘못된 입력");
        return arr;
    }
}
```

```java
import calculator.*;
import util.InputHandler;
import java.util.Scanner;

public class Application { // [[01_클래스, 객체, 메서드]]
    public static void main(String[] args) {
        System.out.println("계산기가 실행되었습니다");
        Calculator cal = new MapCalculator(); // Array/List/Map 자유 교체 — 다형성
        Scanner sc = new Scanner(System.in);
        while (true) {
            System.out.print("계산할 식을 입력해주세요 ex) 1 + 1 : ");
            String input = sc.nextLine();
            if (input.equals("종료")) { System.out.println("계산기를 종료합니다"); break; }
            try {
                String[] a = InputHandler.handleInput(input);
                String n1s = a[0], op = a[1], n2s = a[2];
                if (n1s.contains(".") || n2s.contains(".")) {
                    double n1 = Double.parseDouble(n1s);
                    double n2 = Double.parseDouble(n2s);
                    System.out.println(cal.calculate(n1, n2, op));
                } else {
                    int n1 = Integer.parseInt(n1s);
                    int n2 = Integer.parseInt(n2s);
                    System.out.println(cal.calculate(n1, n2, op));
                }
                cal.showHistory();
            } catch (Exception e) {
                System.err.println(e.getMessage()); // [[03_예외 처리]]
            }
        }
    }
}
```

---

## 🔧 실행 및 테스트

1. 실행: `Application.main` 실행 → 식 입력 → 결과/이력 확인 → "종료" 종료
2. 구현체 교체: `new ArrayCalculator()`/`new ListCalculator()`/`new MapCalculator()` 비교
3. 예외 테스트: `1 / 0`, 잘못된 연산자, 입력 형식 오류 — [[03_예외 처리]]

---

## 🎯 학습 포인트 요약

- 계약(Interface) · 공통(추상 클래스) · 확장(구현 클래스) 분리 → 교체/테스트 용이 — [[02_상속, 인터페이스]]
- 오버로딩/오버라이딩 규칙 준수, 접근/정적 요소는 [[01_클래스, 객체, 메서드]] 참고
- 예외는 복구 가능한 지점에서 처리, 공통 메시지 일관성 유지 — [[03_예외 처리]]
