---
title: "[Error] 컨트롤러 테스트 코드에서 @BeforeEach 실행 오류"
categories:
  - Spring
  - Troubleshooting
tags:
toc: true
toc_sticky: true
date: 2024-07-11 19:49:00 +0900
---

# ❗트러블슈팅 - 컨트롤러 테스트 코드에서 @BeforeEach 실행이 되지 않는 오류❗

## ✨ 오류 상황

프로젝트를 하면서 단위 테스트 코드를 작성하는 중 생긴 오류이다. 상황은 다음과 같다.

1. 컨트롤러 테스트 코드를 작성
2. MockMvc, objectMapper을 사용하면서 중복되는 코드 발생
3. 중복 코드를 없애고자 ControllerTest라는 이름의 abtract class 작성
4. 테스트 실행 시 @BeforeEach 실행이 되지 않는 문제 발생

## ✨ 문제의 코드

### 오류 코드

```
java.lang.NullPointerException: Cannot invoke "org.springframework.test.web.servlet.MockMvc.perform(org.springframework.test.web.servlet.RequestBuilder)" because "this.mockMvc" is null

	at outfoot.outfootserver.checkpage.controller.CheckPageControllerTest.saveCheckPage(CheckPageControllerTest.java:80)
	at java.base/java.lang.reflect.Method.invoke(Method.java:578)
	at java.base/java.util.ArrayList.forEach(ArrayList.java:1511)
	at java.base/java.util.ArrayList.forEach(ArrayList.java:1511)

```

### ControllerTest.java

```java
import com.fasterxml.jackson.databind.ObjectMapper;
import org.junit.jupiter.api.BeforeEach;
import org.junit.jupiter.api.extension.ExtendWith;
import org.springframework.boot.test.autoconfigure.web.servlet.AutoConfigureMockMvc;
import org.springframework.boot.test.context.SpringBootTest;
import org.springframework.test.context.junit.jupiter.SpringExtension;
import org.springframework.test.web.servlet.MockMvc;
import org.springframework.test.web.servlet.setup.MockMvcBuilders;
import org.springframework.web.filter.CharacterEncodingFilter;
import outfoot.outfootserver.common.GlobalExceptionHandler;

@SpringBootTest
public abstract class ControllerTest {
    protected MockMvc mockMvc;
    protected ObjectMapper objectMapper = new ObjectMapper();

    @BeforeEach
    void setUp() {
        this.mockMvc = MockMvcBuilders.standaloneSetup(injectController())
                .setControllerAdvice(GlobalExceptionHandler.class)
                .addFilter(new CharacterEncodingFilter("UTF-8", true))
                .build();

    }

    protected abstract Object injectController();
}

```

setUp 부분이 컨트롤러 테스트 클래스마다 중복되어, ControllerTest로 따로 빼 주고 컨트롤러 테스트 코드를 작성할 때 상속 받도록 설정해 두었다.

### CheckPageControllerTest.java

```java
import com.fasterxml.jackson.databind.ObjectMapper;
import org.junit.jupiter.api.BeforeEach;
import org.junit.jupiter.api.DisplayName;
import org.junit.jupiter.api.Test;
import org.junit.jupiter.api.extension.ExtendWith;
import org.mockito.Mock;
import org.mockito.junit.jupiter.MockitoExtension;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.test.autoconfigure.web.servlet.AutoConfigureMockMvc;
import org.springframework.boot.test.context.SpringBootTest;
import org.springframework.http.MediaType;
import org.springframework.test.web.servlet.MockMvc;
import org.springframework.test.web.servlet.ResultActions;
import org.springframework.test.web.servlet.setup.MockMvcBuilders;
import org.springframework.web.filter.CharacterEncodingFilter;
import outfoot.outfootserver.ControllerTest;
import outfoot.outfootserver.checkpage.dto.CheckPageRequest;
import outfoot.outfootserver.checkpage.dto.CheckPageResponse;
import outfoot.outfootserver.checkpage.service.CheckPageService;
import outfoot.outfootserver.common.GlobalExceptionHandler;

import java.util.Locale;

import static org.mockito.ArgumentMatchers.any;
import static org.mockito.Mockito.when;
import static org.springframework.test.web.servlet.request.MockMvcRequestBuilders.post;
import static org.springframework.test.web.servlet.result.MockMvcResultHandlers.print;
import static org.springframework.test.web.servlet.result.MockMvcResultMatchers.*;


@ExtendWith(MockitoExtension.class)
class CheckPageControllerTest extends ControllerTest {

    @Mock
    private CheckPageService checkPageService;

    private static CheckPageRequest checkPageRequest;
    private static CheckPageResponse checkPage;

    @BeforeEach
    public void setUp() {
        checkPageRequest = CheckPageRequest.builder()
                .title("목표")
                .intro("한 줄 소개")
                .animalId("1")
                .build();

        checkPage = CheckPageResponse.builder()
                .title("목표")
                .intro("한 줄 소개")
                .createdAt("2024-07-07")
                .animalPosition(1)
                .animal("고양이")
                .build();
    }

    @Test
    @DisplayName("[성공] 도장판 생성")
    public void saveCheckPage() throws Exception {
        // when
        when(checkPageService.saveCheckPage(any())).thenReturn(checkPage);

        // then
        String body = objectMapper.writeValueAsString(checkPageRequest);

        ResultActions perform = mockMvc.perform(post("/checkpages")
                .contentType(MediaType.APPLICATION_JSON)
                .locale(Locale.KOREA)
                .content(body));


        // ControllerTest
        perform.andDo(print())
                .andExpect(status().isOk())
                .andExpect(jsonPath("$.success").value(true))
                .andExpect(jsonPath("$.response.title").value(checkPage.title()))
                .andExpect(jsonPath("$.response.intro").value(checkPage.intro()))
                .andExpect(jsonPath("$.response.createdAt").value(checkPage.createdAt()))
                .andExpect(jsonPath("$.response.animalPosition").value(checkPage.animalPosition()))
                .andExpect(jsonPath("$.response.animal").value(checkPage.animal()));
    }

    @Test
    @DisplayName("[예외] 도장판 유효성 검사 실패")
    public void validException() throws Exception {
        // given
        checkPageRequest = CheckPageRequest.builder()
                .title("  ")
                .intro("한 줄 소개")
                .animalId("1")
                .build();

        // then
        String body = objectMapper.writeValueAsString(checkPageRequest);

        ResultActions perform = mockMvc.perform(post("/checkpages")
                .contentType(MediaType.APPLICATION_JSON)
                .locale(Locale.KOREA)
                .content(body));

        perform.andDo(print())
                .andExpect(status().isBadRequest())
                .andExpect(jsonPath("$.success").value(false))
                .andExpect(jsonPath("$.response.errorCode").value("BAD_INPUT"))
                .andExpect(jsonPath("$.response.errorMessage").value("입력이 올바르지 않습니다."))
                .andExpect(jsonPath("$.response.errors.title").value("공백일 수 없습니다"));
    }

    @Override
    protected Object injectController() {
        return new CheckPageController(checkPageService);
    }
}
```

ControllerTest 클래스에서 구현해 둔 injectController을 통해 service를 주입하기로 했다.

## ✨ 시도

처음에는 애노테이션이 잘못되었나 싶어 애노테이션을 바꿔가며 이것저것 시도를 했었다. 그러나 계속해서 똑같은 오류가 나는 것을 보고, 무작정 구글링 해 가며 시도를 하면 나중에 똑같은 문제가 닥쳤을 때 해결할 수 없을 것 같다는 생각이 들어 문제의 원인을 제대로 파악해 보기로 했다.

"this.mockMvc" is null

이 부분에서 힌트를 얻어, mockMvc가 초기화되고 있지 않다는 문제를 발견했다. 분명 ControllerTest에서 초기화했을 텐데 안 되는 것이 이상해, 열심히 디버그를 해 보았더니 아예 @BeforeEach 부분이 실행되고 있지 않은 것을 깨달았다.

기존에 짰던 테스트 코드를 그대로 복붙해 와도 실행되지 않는 것을 보고, 당연히 ControllerTest 클래스는 제대로 짰을 거라 생각한 것이 잘못되었던 것이다.

혹여 인텔리제이의 문제인가 싶어 캐시도 지우고 클래스를 다시 만들어 다시 치기도 했지만 문제는 달라지지 않았다. 여전히 NullPointerException이 떴다.

다시 디버그로 찍어보니, 아예 실행이 안 되는 것이 아닌, @BeforeEach 부분만 실행되지 않았으며, HealthCheck 메서드를 임시로 만든 부분도 제대로 실행되는 것을 확인했다.

또한, ControllerTest를 상속 받은 CheckPageControllerTest에서 mockMvc를 초기화 시켰더니 제대로 동작하였다. 따라, @BeforeEach 메서드 부분에서 어떠한 문제가 생긴 것이라 확신을 내렸다.

## ✨ 해결

정답은 정말 간단했다...

ControllerTest에서의 setUp과 CheckPageControllerTest에서의 setUp의 이름이 겹쳤기에 상속된 CheckPageControllerTest의 setUp만 실행한 것이었다.

이후, ControllerTest에서의 setUp을 setUpAbstract로 바꿔주니 정상 작동했다.

https://www.inflearn.com/questions/933072/beforeeach-%EB%A5%BC-%ED%95%98%EC%9C%84-%ED%81%B4%EB%9E%98%EC%8A%A4%EC%97%90%EC%84%9C-%EB%8B%A4%EC%8B%9C-%EC%82%AC%EC%9A%A9%ED%95%98%EB%8A%94%EA%B2%83%EC%97%90-%EB%8C%80%ED%95%B4%EC%84%9C-%EC%A7%88%EB%AC%B8-%EB%93%9C%EB%A6%BD%EB%8B%88%EB%8B%A4

참고한 자료는 위와 같다.

다음부터는 꼼꼼히 확인하도록 해야겠다.

## ✨ 느낀 점

이번 문제를 통해, 무작정 구글링하는 것보다는 왜 이러한 오류가 떴고 이를 해결하기 위해 어떤 부분을 중점적으로 보아야 하는지 깨달았던 것 같다. 구글링하는 것도 중요하지만, 정말 중요한 건 이론을 알고 이러한 문제가 발생한 원인을 제대로 파악하는 것이라고 생각했다.

자바의 상속과 관련된 문제를 간과하고 있던 것이 이번 트러블슈팅의 원인이었다. 아예 문제를 안 만드는 것이 최선이겠지만, 프로젝트를 하다 보면 하나씩 빠뜨리는 부분이 있을 수 있으므로 문제가 발생했을 때 오류가 나는 원인을 제대로 파악하는 것이 무엇보다 중요하다는 것을 깨달았다.

다음 번에 비슷한 문제가 생겼을 시 자바의 기본 개념과 구현에 대해 다시 한 번 확인할 수 있을 것 같다.
