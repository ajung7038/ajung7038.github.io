---
title: "[Spring] Tesseract 오픈소스를 활용한 OCR 기능 구현"
categories:
  - Spring
tags:
toc: true
toc_sticky: true
date: 2024-07-11 19:23:00 +0900
---

# ❗Tesseract - Spring 연동❗

## ✨ OCR이란?

광학 문자 인식(OCR) 이란 텍스트 이미지를 기계가 읽을 수 있는 텍스트 포맷으로 변환하는 과정이다. 이미지 파일에서는 텍스트 편집기를 사용하여 단어를 편집, 검색하거나 단어 수를 계산할 수 없지만, OCR을 사용하면 이미지를 텍스트 문서로 변환하여 내용을 텍스트 데이터로 저장할 수 있다.

영수증을 스캔하는 경우, 해당 영수증 내 텍스트를 추출하는 경우 OCR을 사용한다고 할 수 있다.

스프링에서 OCR 기술을 사용하기 위해서는 외부 API과 스프링을 연동 해야 한다. 사용할 수 있는 외부 API는 다음과 같다.

### OCR 기술 구현

1. Amazon Textract
2. 네이버 CLOVA OCR
3. Tesseract
4. EasyOCR
5. 카카오브레인 PORORO
6. Google Vision API

다 좋은데, 비싸다. 대기업(Amazon, 네이버, Google) 셋의 경우 기술은 좋지만, 비싸기 때문에 카카오브레인에서 만든 PORORO 오픈소스를 활용해 OCR을 구현하려고 한다.
<br /> -> 구현하려고 했는데 스프링과 PORORO 연결 자료가 거의 없어 가장 많이 사용되는 Tesseract를 사용하기로 했다.

참고로, PORORO 코드는 깃허브에 올라와 있다. 주소는 아래와 같다. (쓰지는 않겠지만...)

https://github.com/kakaobrain/pororo

## ✨ Spring-Tesseract 연동

### 1. 다운로드

https://github.com/tesseract-ocr/tessdata_best

위 링크에 들어가 kor.traineddata와 kor_vert.traineddate 두 개를 다운받는다.

- kor.traineddata : 가로로 작성된 문자 인식용 학습 모델
- kor_vert.traineddata : 세로로 작성된 문자 인식용 학습 모델

아무 곳에나 저장해도 된다. 나는 scr/main/resources/static/tesseract 아래 파일을 위치시켰다.

### 2. 라이브러리 다운로드

build.gradle 파일에 의존성을 추가해 준다.

```java
	implementation group: 'net.sourceforge.tess4j', name: 'tess4j', version: '5.11.0'
```

### 3. 코드 짜기

로직은 다음과 같다.

1. Controller에서 DTO를 통해 사진을 받아온다.
2. Controller에서 Service를 호출한다. (파라미터는 DTO)
3. Service에서 MultipartFile -> File로 바꾼다.
4. Tesseract를 활용하여 텍스트를 추출한다.

참고로, 아직 로컬 DB를 사용하고 있기 때문에 s3를 연동해 사진을 저장하고 url을 받아오는 로직은 생략했다. 또, Tesseract 메서드 doOCR은 File을 파라미터로 받기 때문에 MultipartFile -> File로 바꾸는 로직이 필요하다.

### 3-1. 코드

#### DTO

```java
// DTO -> OcrRequest
import org.springframework.web.multipart.MultipartFile;

import java.io.File;

public record OcrRequest(MultipartFile file) { }
```

실제로 현재 로직에서는 DTO를 쓰지는 않는다. 추후 확장을 위해 DTO를 임시로 만들어 두었다. (S3 연동을 위해)

#### Service

service는 IOcrService라는 인터페이스로 우선 만든 후, 이를 구현한 OcrService를 사용하였다.

- IocrService

```java
import WELLET.welletServer.ocr.dto.OcrRequest;
import net.sourceforge.tess4j.TesseractException;

import java.io.File;
import java.io.IOException;

public interface IOcrService {
    String model = "{kor.traineddata와 kor_vert.traineddate를 저장해 둔 경로}";
    void getImageToText(OcrRequest dto) throws TesseractException, IOException;
}
```

- OcrService (IocrService 구현)

```java
import WELLET.welletServer.ocr.dto.OcrRequest;
import lombok.extern.slf4j.Slf4j;
import net.sourceforge.tess4j.ITesseract;
import net.sourceforge.tess4j.Tesseract;
import net.sourceforge.tess4j.TesseractException;
import org.springframework.stereotype.Service;
import org.springframework.transaction.annotation.Transactional;

import java.io.File;
import java.io.IOException;

@Service
@Slf4j
@Transactional
public class OcrService implements IOcrService {

    private static final String filePath =  "{테스트할 파일 경로}";
    @Override
    @Transactional
    public void getImageToText(OcrRequest dto) throws TesseractException, IOException {

        // 서버에 파일 저장 -> url 받아오는 로직 필요

        File file = new File(filePath);

        ITesseract instance = new Tesseract();
        instance.setDatapath(IOcrService.model);
        instance.setLanguage("kor");
        String ImageToText = instance.doOCR(file);

        System.out.println(ImageToText);
    }
}
```

filePath에서 DTO에서 받아온 파일을 그대로 쓰는 것이 아닌 테스트할 파일 경로를 넣어준 것은 추후에 S3에 파일을 업로드하고 URL을 받아오는 로직으로 바꾸기 위함이다.

Multipartfile -> File로 바꾸는 로직은 따로 구현하지 않았다.

new File의 파라미터로 경로가 필요했기 때문에 저렇게 filePath를 넣어준 것이다. MultipartFile을 넣으려고 시도는 했지만, 애초부터 파일 자체의 파라미터를 받지는 않는다.

추후에 S3에 파일을 저장 후 URL을 받아, filePath에 넣어주는 로직을 구현할 예정이다.

#### Controller

```java
import WELLET.welletServer.common.response.BasicResponse;
import WELLET.welletServer.common.response.ResponseUtil;
import WELLET.welletServer.ocr.dto.OcrRequest;
import WELLET.welletServer.ocr.service.OcrService;
import lombok.RequiredArgsConstructor;
import net.sourceforge.tess4j.TesseractException;
import org.springframework.web.bind.annotation.ModelAttribute;
import org.springframework.web.bind.annotation.PostMapping;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RestController;

import java.io.IOException;

@RestController
@RequestMapping("/ocr")
@RequiredArgsConstructor
public class OcrController {
    private final OcrService ocrService;

    @PostMapping
    public BasicResponse<String> ocr (@ModelAttribute OcrRequest dto) throws TesseractException, IOException {
        ocrService.getImageToText(dto);
        return ResponseUtil.success("텍스트 추출에 성공하였습니다.");
    }
}
```

여기서 BasicResponse와 ResponseUtil은 공통된 응답을 반환하기 위해 따로 만든 클래스이다.

## ✨ 결과

- 포스트맨 테스트 결과

![image](https://github.com/APPS-sookmyung/2024-WELLET-Server/assets/80907516/ad165589-aa02-4efc-a063-9b73dc4f7c08)

- 테스트 사진

![testFile](https://github.com/APPS-sookmyung/2024-WELLET-Server/assets/80907516/7956e45a-6e4b-41cf-9386-f5a2a716b2a9)

- 추출 결과

![image](https://github.com/APPS-sookmyung/2024-WELLET-Server/assets/80907516/76114a96-cf28-4102-bff5-7f73e3067c46)

## ✨ 정리

오픈소스 OCR을 구현한 건 이번이 처음이라 정확도가 다른 것에 비해 좋다, 안 좋다를 판단할 수는 없지만, 나름 괜찮은 것 같았다. 한글만 인식하도록 설정해뒀기 때문에 영어를 인식하지 못하는 것 빼고는 좋은 것 같다!

근데 영어와 한글 두 가지를 모두 인식하지 못한다는 큰 문제가 있어서... 아마 다른 방법을 찾게 될 것 같다.

추가로, 명함 OCR을 만들기 위해서는 이름과 직장 등을 뽑아야 할 텐데, 단순히 스프링으로만 구현하기에는 한계를 느꼈기에 파이썬을 활용해 전처리를 한 후 두 개의 서버로 통신해 OCR을 구현할 지 고민 중이다.

## ✨ 참고 자료

- https://velog.io/@junsoo1230/Tesseract%EB%A1%9C-%EC%9E%90%EC%97%B0%EC%96%B4-%EC%B2%98%EB%A6%AC%ED%95%98%EA%B8%B0-ch03%EC%8B%A4%EC%8A%B5
- https://aws.amazon.com/ko/what-is/ocr/
- https://velog.io/@sionshin/OCR-%EC%B0%BE%EC%95%84-%EC%82%BC%EB%A7%8C%EB%A6%AC
- https://kakaobrain.github.io/pororo/miscs/ocr.html
