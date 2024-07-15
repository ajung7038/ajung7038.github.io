---
title: "[Troubleshooting] 하나의 service에서 여러 개의 service 의존 처리하는 법"
categories:
  - Spring
  - Troubleshooting
tags:
toc: true
toc_sticky: true
date: 2024-07-16 03:40:00 +0900
---

# ❗트러블슈팅 - 하나의 service에서 여러 개의 service 의존 처리하는 법❗

## ✨ 문제 상황

프로젝트에서 like 기능을 구현하던 중, likeService에서 member, like, checkPage, Confirm 네 개의 repository를 활용 해야 하는 상황이 발생했다.

## ✨ 시도

### 📌 시도 (1차)

생각 없이 repository를 사용하려던 차에, 전에 강의에서 본 "단일 책임 원칙" 에 어긋난다는 생각이 들어 다시 나의 코드를 확인했다.

#### 문제의 코드

```java
import lombok.RequiredArgsConstructor;
import org.springframework.stereotype.Service;
import org.springframework.transaction.annotation.Transactional;
import outfoot.outfootserver.checkpage.domain.CheckPage;
import outfoot.outfootserver.checkpage.repository.CheckPageRepository;
import outfoot.outfootserver.confirm.repository.ConfirmRepository;
import outfoot.outfootserver.member.repository.MemberRepository;
import outfoot.outfootserver.confirm.entity.Confirm;
import outfoot.outfootserver.like.repository.LikeRepository;
import outfoot.outfootserver.member.domain.Member;

@Service
@RequiredArgsConstructor
@Transactional(readOnly = true)
public class LikeService {
    private final LikeRepository likeRepository;
    private final CheckPageRepository checkPageRepository;
    private final ConfirmRepository confirmRepository;
    private final MemberRepository memberRepository;

    @Transactional
    public void addLike() {
    }
}
```

아직 코드를 구현하기 전이라 텅텅 비어 있지만 딱 봐도 Repository가 복잡하게 얽혀 있는 것을 확인할 수 있다. 단일 책임 원칙이 깨지고 있는 것이다.

### 🛠️ 해결 방안 (1차)

그래서 생각한 방안은 기존 프로젝트처럼 컨트롤러에서 service를 통해 그 객체를 찾아내는 것이다. (repository의 findById를 통해 service에서 메서드 구현) 이러한 객체들을 찾아 likeService를 호출할 때 파라미터로 받아 넘기면 service에서 굳이 다른 service를 참조하지 않아도 된다.

### 📌 시도 (최종)

그러나 이것 또한 좋지 않은 방법이라 생각했다. 기존 프로젝트에서는 엮여야 하는 repository가 단 하나였고, 이것 또한 memberRepository에서 loadUser로, 로그인한 유저 정보 로직을 가져오는 것이었다.

그러나 이번에는 아직 로그인 기능을 구현하기 전이고, 또 그렇게 하기에는 controller에서 처리해야 할 로직이 너무 많아 또 단일 책임 원칙을 깨는 것 같았다.

+) 찾아보니 여러 서비스를 하나의 트랜잭션으로 묶을 수 없다는 단점이 있다고 한다. 특히, 여러 개를 조회만 하는 것이 아닌 "저장"하는 로직을 구현 해야 하는 상황에서는 문제가 연달아 터질 위험이 있는 것이었다.

이를 어떻게 해결하면 좋을까, 생각하며 다양한 블로그 글을 읽었다. DDD, pacade pattern 등 다양한 방법이 있었고, 나는 service에 한정해 퍼시드 패턴을 도입하는 것이 최선이라 생각하여 퍼시드 패턴으로 이러한 문제를 해결하기로 했다.

## ✨ Pacade Pattern

![image](https://github.com/user-attachments/assets/7dba2cb6-29d7-421c-8a66-1aedcf538c88)

간단하게 설명하자면 퍼시드 패턴이란 클라이언트는 복잡하게 얽혀 있는 서브 시스템은 모른 채 Facade 객체에만 의존하는 것이다.

추가로, 퍼시드 패턴에 관한 내용은 다른 글에서 다룰 예정이다.

### 🛠️ 해결 방안 (최종)

퍼시드 패턴을 위 프로젝트에서 적용하게 되면 여러 Service를 다루는 하나의 큰 Service 객체를 만들 수 있다.

따라서, 서비스는 Facade 객체에만 의존하게 되고, Facade 객체는 사용할 Service를 주입 받는 형태로 구성된다.

이렇게 문제를 해결한다면 여러 Service를 하나의 Service에서 관리함으로써 순환 참조 및 코드 중복을 모두 해결할 수 있게 된다.

물론 작은 규모의 프로젝트지만, 언제나 확장 가능성을 열어두는 것이 좋은 설계인 것 같았고, 앞으로 할 큰 프로젝트에서 또 같은 문제를 겪었을 때 그나마 최선의 방법인 컨트롤러에서 repository나 다른 service를 호출하고 싶지는 않았다. 그렇게 된다면 확장에 열려있지 않기 때문이다.

작은 프로젝트이지만 퍼시드 패턴에 대해 알아보고, 직접 사용하고 싶어 이러한 방법을 선택하게 되었다. 작은 프로젝트이기 때문에 다양한 시도를 해 볼 수 있었다고 생각한다.

## ✨ 최종 수정 코드

### LikeManageService (with Pacade Pattern)

```java
package outfoot.outfootserver.like.service;

import lombok.RequiredArgsConstructor;
import org.springframework.stereotype.Service;
import outfoot.outfootserver.checkpage.domain.CheckPage;
import outfoot.outfootserver.checkpage.service.CheckPageService;
import outfoot.outfootserver.confirm.domain.Confirm;
import outfoot.outfootserver.confirm.service.ConfirmService;
import outfoot.outfootserver.member.domain.Member;
import outfoot.outfootserver.member.service.MemberService;

@Service
@RequiredArgsConstructor
public class LikeManageService {

    private final CheckPageService checkPageService;
    private final ConfirmService confirmService;
    private final MemberService memberService;

    public Member loadMember (Long member_id) {
        return memberService.loadMember(member_id);
    }

    public CheckPage loadCheckPage(Long checkPageId) {
        return checkPageService.findById(checkPageId);
    }

    public Confirm loadConfirm(Long confirmId) {
        return confirmService.findById(confirmId);
    }
}

```

LikeController에서 너무 많은 Service의 사용을 방지하기 위해, 퍼사드 패턴을 이용하여 MemberService, CheckPageService, ConfirmService를 LikeManageService라는 하나의 Service에서 다루도록 했다.

memberService의 메서드 이름만 다른 건 로그인 기능 연동 시 확장 가능성을 열어둔 것이다.

또한, CheckPage, Confirm의 findById 메서드는 각각 서비스에서 객체 하나를 찾을 때 재사용되기도 한다. (하나의 객체를 찾는 경우, dto로 변환해서 사용)

### LikeController

```java
package outfoot.outfootserver.like.controller;

import jakarta.validation.Valid;
import lombok.RequiredArgsConstructor;
import org.springframework.web.bind.annotation.*;
import outfoot.outfootserver.checkpage.domain.CheckPage;
import outfoot.outfootserver.common.response.BasicResponse;
import outfoot.outfootserver.common.response.ResponseUtil;
import outfoot.outfootserver.confirm.domain.Confirm;
import outfoot.outfootserver.like.dto.LikeRequest;
import outfoot.outfootserver.like.service.LikeManageService;
import outfoot.outfootserver.like.service.LikeService;
import outfoot.outfootserver.member.domain.Member;

@RestController
@RequiredArgsConstructor
@RequestMapping("/confirm")
public class LikeController {
    private final LikeService likeService;
    private final LikeManageService likeManageService;

    @PostMapping("/like")
    public BasicResponse<String> addLike(@Valid @RequestBody LikeRequest dto) {
        Member member = likeManageService.loadMember(dto.memberId());
        CheckPage checkPage = likeManageService.loadCheckPage(dto.checkPageId());
        Confirm confirm = likeManageService.loadConfirm(dto.confirmId());
        likeService.addLike(member, checkPage, confirm);
        // 전체 개수 추가 필요
        return ResponseUtil.success("좋아요 누르기에 성공했습니다.");
    }
}
```

LikeManageService를 통해 각각 member, checkPage, confirm 객체를 찾아 LikeService의 addLike 메서드를 호출해 저장하는 로직을 구현하였다.

## ✨ 느낀 점

솔직히 이것이 정답이라 생각하지 않는다. 파서드 패턴을 활용해 구현했지만, service 하나에서 여러 service를 참조하는 것이 좋은지 안 좋은지 여전히 모르겠다.

이런 문제를 해결하기 위해 인터페이스를 활용하는 등의 여러 방법을 찾았지만, 모두 내가 처해진 상황에서는 오히려 서비스 간 의존성만 강화할 것 같아 내 기준 가장 괜찮은 방법으로 구현했다.

작은 프로젝트라 오히려 service를 가져다 쓰는 것이 더 나을 수도 있겠지만, 어쨌든 공부를 위한 프로젝트이므로 구현을 위해 설계를 고민하는 것으로 충분한 가치가 있다고 판단했다.

그리고 아무리 작은 프로젝트라도 구현하기 편하도록 만드는 건 내 적성에 맞지 않기도 했고...

추후에 수정을 거듭하며 이러한 문제를 보완하고 싶다. 클린 코드, 여러 강의들을 섭렵하며 디자인 패턴 및 클린 코드에 대해 더 공부해야 겠다는 생각이 들었다.

오늘도 트러블슈팅 처리 완료... (인가...?)

## ✨ 참고 자료

- https://kchung1995.github.io/posts/%ED%95%98%EB%82%98%EC%9D%98-Service%EA%B0%80-%EC%97%AC%EB%9F%AC-Repository%EC%97%90-%EC%9D%98%EC%A1%B4%ED%95%A0-%EB%95%8C/
- https://velog.io/@lej7122/Spring-%ED%95%9C-Service%EA%B0%80-%EB%8B%A4%EB%A5%B8-Service-%EB%98%90%EB%8A%94-Repository%EB%A5%BC-%EC%9D%98%EC%A1%B4%ED%95%98%EB%8A%94-%EA%B2%BD%EC%9A%B0
