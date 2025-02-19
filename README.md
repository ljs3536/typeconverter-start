# 25-02-09
## 프로젝트 생성

# 25-02-10
## 스프링 타입 컨버터 소개

### 분석
HTTP요청 파라미터는 모두 문자로 처리된다. 따라서 요청 파라미터를 자바에서 다른 타입으로 변환해서 사용하고 싶으면 
다음과 같이 숫자 타입으로 변환하는 과정을 거쳐야 한다.
Intger intValue = Intger.valueOf(request.getParameter("data"));

### 실행
앞서 보았듯이 HTTP 쿼리 스트링으로 전달하는 data=10 부분에서 10은 숫자 10이 아니라 문자10이다.
스프링이 제공하는 @RequestParam을 사용하면 이 문자 10을 Integer 타입의 숫자 10으로 편리하게 받을 수 있다.
"이것은 스프링이 중간에서 타입을 변환해주었기 때문이다."

이러한 예는 @ModelAttribute, @PathVariable에서도 확인할 수 있다.

### 스프링의 타입 변환 적용 예
- 스프링 MVC 요청 파라미터
  - @RequestParam, @ModelAttribute, @PathVariable
- @Value 등으로 YML 정보 읽기
- XML에 넣은 스프링 빈 정보를 변환
- 뷰를 렌더링 할 때

### 스프링과 타입 변환
스프링이 중간에 타입 변환기를 사용해 타입을 변환해 주고 있기 때문에 숫자를 문자로 변경하거나, Boolean타입을 숫자로 변경하는 것도 가능하다.

### 컨버터 인터페이스
스프링은 확장 가능한 컨버터 인터페이스를 제공한다.
개발자는 스프링에 추가적인 타입 변환이 필요하면 이 컨버터 인터페이스를 구현해서 등록하면 된다.

### 참고
과거에는 PropertyEditor라는 것으로 타입을 변환했다. 
PropertyEditor는 동시성 문제가 있어서 타입을 변환할 때 마다 객체를 계속 생성해야 하는 단점이 있다.
지금은 Converter의 등장으로 해당 문제들이 해결되었고, 기능 확장이 필요하면 Converter를 사용하면 된다.

# /25-02-11
## 타입 컨버터 - Converter 1

# /25-02-12
## 타입 컨버터 - Converter 2
타입 컨버터 인터페이스는 단순하다.
하지만 타입 컨버터를 하나하나 직접 사용하면, 개발자가 직접 컨버팅 하는 것과 큰 차이가 없다.
타입 컨버터를 등록하고 관리하면서 편리하게 변환 기능을 제공하는 역할을 하는 무언가가 필요하다.

# /25-02-13
## 컨버전 서비스 - ConversionService

### 등록과 분리
컨버터를 등록할 때는 StringToIntegerConverter 같은 타입 컨버터를 명확하게 알아야 한다. 
반면에 컨버터를 사용하는 입장에서는 타입 컨버터를 전혀 몰라도 된다. 타입 컨버터들은 모두 컨버전 서비스 내부에 숨어서 제공된다.
따라서 타입을 변환을 원하는 사용자는 컨버전 서비스 인터페이스에만 의존하면 된다.
물론 컨버전 서비스를 등록하는 부분과 사용하는 부분을 분리하고 의존관계 주입을 사용해야 한다.

### 인터페이스 분리 원칙 - ISP(Interface Segregation Principal)

# /25-02-15
## 스프링에 Converter 적용하기
스프링은 내부에서 ConversionService를 제공한다. 우리는 WebMvcConfigurer가 제공하는 addFormatters()를 사용해서 추가하고 싶은 컨버터를 등록하면 된다.
이렇게 하면 스프링은 내부에서 사용하는 ConversionService에 컨버터를 추가해준다.

### 처리 과정
@RequestParam은 @RequestParam을 처리하는 ArgumentResolver인 RequestParamMethodArgumentResolver에서 ConversionService를 사용해서 타입을 변환한다.
부모 클래스와 다양한 외부 클래스를 호출하는 등 복잡한 내부 과정을 거치기 때문에 대략 이렇게 처리되는 것으로 이해해도 충분하다.
만약 더 깊이있게 확인하고 싶으면 IpPortConverter에 디버그 브레이크 포인트를 걸어서 확인해보자.

# /25-02-16
## 뷰 템플릿에 컨버터 적용하기
타임리프는 렌더링 시에 컨버터를 적용해서 렌더링 하는 방법을 편리하게 지원한다.
이전까지는 문자를 객체로 변환했다면, 이번에는 그 반대로 객체를 문자로 변환하는 작업을 확인할 수 있다.

타임리프는 ${{...}}를 사용하면 자동으로 컨버전 서비스를 사용해서 변환된 결과를 출력해준다.
물론 스프링과 통합되어서 스프링이 제공하는 컨버전 서비스를 사용하므로, 우리가 등록한 컨버터들을 사용할 수 있다.
변수표현식 : ${...}
컨버전 서비스 적용 : ${{...}}

타임리프의 th:field는 앞서 설명했듯이 id, name을 출력하는 등 다양한 기능이 있는데, 여기에 컨버전 서비스도 함께 적용이 된다.

# /25-02-17
## 포멧터 - Formatter
Converter는 입력과 출력 타입에 제한이 없는 범용 타입 변환 기능을 제공한다.

### 웹 애플리케이션에서 객체를 문자로, 문자를 객체로 변환하는 예
- 화면에 숫자를 출력해야 하는데 Integer -> String 출력 시점에 숫자 1000 -> 문자 "1000" 이렇게 1000단위에 쉼표를 넣어서 출력하거나, 또는 "1,000" 라는 문자를 1000 이라는 숫자로 변경해야 한다.
- 날짜 객체를 문자인 "2025-02-17 10:26:00"와 같이 출력하거나 또는 그 반대의 상황

### Locale
여기에 추가로 날짜 숫자의 표현 방법은 Locale 현지화 정보가 사용될 수 있다.
이렇게 객체를 특정한 포멧에 맞추어 문자로 출력하거나 또는 그 반대의 역할을 하는 것에 트고하된 기능이 바로 포멧터(Formatter)이다.
포맷터는 컨버터의 특별한 버전으로 이해하면 된다.

### Converter vs Formatter
- Converter는 범용 (객체 -> 객체)
- Formatter는 문자에 특화 (객체 -> 문자, 문자 -> 객체) + 현지화 (Locale)
  - Converter의 특별한 버전
 
### 포맷터 - Formatter 만들기
포맷터는 객체를 문자로 변경하고, 문자를 객체로 변경하는 두 가지 기능을 모두 수행한다.

# /25-02-18
## 포맷터를 지원하는 컨버전 서비스
컨버전 서비스에는 컨버터만 등록할 수 있고, 포맷터를 등록할 수는 없다.
그런데 생각해보면 포맷터는 객체 -> 문자, 문자 -> 객체로 변환하는 특별한 컨버터일 뿐이다.
포맷터를 지원하는 컨버전 서비스를 사용하면 컨버전 서비스에 포맷터를 추가할 수 있다.
내부에서 어댑터 패턴을 사용해서 Formatter가 Converter처럼 동작하도록 지원한다.

### DefaultFormattingConversionService 상속 관계
FormattingConversionService는 ConversionService 관련 기능을 상속받기 때문에 결과적으로 컨버터도 포맷터도 모두 등록할 수 있다.
그리고 사용할 때는 ConversionService가 제공하는 convert를 사용하면 된다.
추가로 스프링 부트는 DefaultFormattingConversionService를 상속받은 WebConversionService를 내부에서 사용한다.

# /25-02-19
## 포맷터 적용하기

