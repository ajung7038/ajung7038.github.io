---
title: "[Spring] CORS 처리"
categories:
  - Spring
tags:
toc: true
toc_sticky: true
date: 2024-05-15 21:16:00 +0900
---

# ❗CORS 처리❗

## ✨ CORS란?

: Cross-Origin Resource Sharing의 줄임말로, 출처가 다른 스크립트가 실행되지 않도록 브라우저에서 사전에 방지하는 것이다.

서버 문제도, 프론트 문제도 아닌 웹 브라우저에서 해커의 악의적 탈취를 막기 위한 수단으로 CORS를 사용하는 것이다.

그렇다면 여기서 말하는 출처란 뭘까?

출처(Origin)는 Protocol, Host, Port가 동일한지 여부에 대해 판단한다.

https://ajung7038.github.io/

예를 들면, 다음 url에 대해 출처를 나누어 보자면 다음과 같다.

- Protocol : https://
- HOST : www.ajung7038.github.io
- PORT : 숨겨져 있음

## ✨ HttpSecurity

- HttpSecurity 객체 사용 시 리소스 접근 권한 설정, 필터, csrf, 강제 thhps 호출 등 설정이 가능하다.

### 설정

Security 설정했으면 SecurityConfig.java 파일에 빈으로 등록해주면 된다.

```java
@Configuration
public class SecurityConfig {
  @Bean
  SecurityFilterChain filterChain(HttpSecurity http) throws Exception {
      http.cors(cors -> cors.configurationSource(corsConfigurationSource()));
  }

  @Bean
  public CorsConfigurationSource corsConfigurationSource() {
      CorsConfiguration configuration = new CorsConfiguration();
      configuration.setAllowedOrigins(Arrays.asList("http://localhost:3000")); // 프론트
      configuration.setAllowCredentials(true);
      configuration.setAllowedHeaders(Arrays.asList("*"));
      configuration.setAllowedMethods(Arrays.asList("GET", "POST", "PUT", "DELETE", "OPTIONS", "PATCH"));
      configuration.setMaxAge(7200L);
      UrlBasedCorsConfigurationSource source = new UrlBasedCorsConfigurationSource();
      source.registerCorsConfiguration("/**", configuration);
      return source;
  }
}
```

## ✨ 참고 자료

- https://velog.io/@woojjam/Spring-Security-HttpSecurity-%EC%84%A4%EC%A0%95
- https://inpa.tistory.com/entry/WEB-%F0%9F%93%9A-CORS-%F0%9F%92%AF-%EC%A0%95%EB%A6%AC-%ED%95%B4%EA%B2%B0-%EB%B0%A9%EB%B2%95-%F0%9F%91%8F
