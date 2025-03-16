---
title: "[FS] testdisk 사용법"
categories:
  - File Forensic
tags:
toc: true
toc_sticky: true
date: 2025-03-16 12:33:00 +0900
---

## 🫧 testdisk 사용법

### ✨ 기본 환경 세팅

![image](https://github.com/user-attachments/assets/cb141e89-c38d-4298-97ca-57ec7a7fa83d)


| ~/test.txt, test1.yml 파일 생성 후 rm -rf 명령어로 테스트 파일을 삭제한 상황.


### ✨ testdisk로 파일 복구 방법

1. 쉘에 testdisk 명령어 작성 (다음과 같은 창으로 자동 접속)

![image](https://github.com/user-attachments/assets/fb827bbd-43a1-4127-bcde-5f497702feb1)


- Create : 새 로그 파일 생성
- Append : 로그 파일에 정보 추가
- No Log : 아무것도 기록 X

2. 위 세 선택지 중 하나 선택 시 뜨는 창

<br/> ⇒ Proceed 누르기
    
![image](https://github.com/user-attachments/assets/c0b9db22-e3b6-45b4-9b5e-9b129fdb1ddb)


3. EFI GPT 클릭

![image](https://github.com/user-attachments/assets/5bbab571-bb51-4e44-8c77-f000fca2fe14)


4. 엔터 (맞는 파티션이 자동으로 지정됨)

![image](https://github.com/user-attachments/assets/2323d58a-b2c2-4d39-a934-f5aec641cf0b)


5. Quick Search 클릭

![image](https://github.com/user-attachments/assets/086e183e-81bb-46a5-bd54-e14300edfc56)


6. 이런 과정이 나오며 파일 탐색, 이후 Continue 눌러주기

![image](https://github.com/user-attachments/assets/6110f90b-4429-452b-bbfb-fce809f5f736)


![image](https://github.com/user-attachments/assets/a2eb7080-154f-4f95-b942-38bb0e452af6)


7. 엔터

![image](https://github.com/user-attachments/assets/8c26c457-3479-4037-829d-1e764c7d5378)


8. 빨간색으로 표시된 파일 (지워진 파일) 복구 ⇒ c 누르기
- 원래는 이렇게 떠야 하는데, 제가 뭔가 잘못 건드렸는지 뜨지 않아… 다른 분의 블로그 이미지로 대체합니다.

![image](https://github.com/user-attachments/assets/b79b3ac4-f715-4f39-9e63-335385dce007)
