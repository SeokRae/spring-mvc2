# TypeConverter

## Intro

- 문자를 숫자로 변환하거나, 반대로 숫자를 문자로 변환해야 하는 것 처럼 애플리케이션을 개발하다 보면 타입을 변환해야 하는 경우가 상당히 많다.

## 문자 타입을 숫자 타입으로 변경

- HTTP 요청 파라미터는 모두 문자로 처리된다. 따라서 요청 파라미터를 자바에서 다른 타입으로 변환해서 사용하고 싶으면 다음과 같이 숫자 타입으로 변환하는 과정을 거쳐야 한다.

### @RequestParam

- HTTP 쿼리 스트링으로 전달하는 data=10 부분에서 10은 숫자 10이 아니라 문자 10이다.
- 스프링이 제공하는 @RequestParam 을 사용하면 이 문자 10을 Integer 타입의 숫자 10으로 편리하게 받을 수 있다.


- 스프링의 타입 변환 적용 예시
	- 스프링 MVC 요청 파라미터
		- @RequestParam , @ModelAttribute , @PathVariable
	- @Value 등으로 YML 정보 읽기
	- XML에 넣은 스프링 빈 정보를 변환
	- 뷰를 렌더링 할 때

- 스프링과 타입 변환
	- 스프링은 확장 가능한 컨버터 인터페이스를 제공
	- 스프링에서 제공하는 타입 컨버터 `org.springframework.core.convert.converter.Converter`

## Spring에서 제공하는 Type Converter

- Converter
	- 기본 타입 컨버터
- ConverterFactory
	- 전체 클래스 계층 구조가 필요할 때
- GenericConverter
	- 정교한 구현, 대상 필드의 애노테이션 정보 사용 가능
- ConditionalGenericConverter
	- 특정 조건이 참인 경우에만 실행

- 스프링은 문자, 숫자, 불린, Enum등 일반적인 타입에 대한 대부분의 컨버터를 기본으로 제공한다.

## 컨버전 서비스 - ConversionService

- 개별 컨버터를 모아두고 그것들을 묶어서 편리하게 사용할 수 있는 기능을 제공

```java
public interface ConversionService {
    boolean canConvert(@Nullable Class<?> sourceType, Class<?> targetType);
    boolean canConvert(@Nullable TypeDescriptor sourceType, TypeDescriptor targetType);

    <T> T convert(@Nullable Object source, Class<T> targetType);
    Object convert(@Nullable Object source, @Nullable TypeDescriptor sourceType, TypeDescriptor targetType);
}
```
