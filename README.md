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

