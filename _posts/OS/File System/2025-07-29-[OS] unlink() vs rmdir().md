---
title: "[OS] unlink() vs rmdir()"
categories:
  - File System
tags:
toc: true
toc_sticky: true
date: 2025-07-29 18:19:00 +0900
---

<strong>[linux kernel sourse tree](https://github.com/torvalds/linux)의 깃허브 코드를 참조해 시스템 콜 호출 시 변화 과정을 분석한 글입니다.</strong>
{: .notice}

# 📌 unlink() vs rmdir()

- 파일 또는 디렉터리 삭제 시 어떤 변화가 일어나는지 시스템 콜을 통해 관찰
- inode, dentry의 변화 과정을 주로 관찰하고자 함

## 🫧 unlink() 시스템 콜

### ✨ 정의

```c
#include <unistd.h>

int unlink(const char *pathname);

```

- unlink()는 파일 삭제 시 호출되는 함수로, hard link의 이름을 삭제하고, hard link가 참조하는 count를 1 감소시킨다.

만약 이 <strong>하드 링크</strong>의 참조 count가 0이 되면, 실제 파일의 내용이 저장되어 있는 disk space를 free 상태로 만들어 OS가 사용 가능한 공간으로 인식될 수 있도록 한다.

### ✨ orphan file

그러나 open()으로 파일이 열려진 상태에서 close()를 하지 않은 상태로 unlink()를 호출하게 되어 하드 링크 참조 카운트가 0이 될 때, 실제로 `열린 파일`로 인식되어 다음과 같은 상황이 발생한다.

1. dentry 해제 (inode 참조 해제)
2. disk space가 free 상태가 되지 않음.

원래 정리될 데이터들은 다음과 같다.

- dentry 삭제 (inode가 0으로 초기화)
- inode bitmap이 1->0으로 변화
- inode table의 해당 inode 블록은 사용 가능한 상태로 변화, 그러나 0으로 밀리지 않음
- block bitmap 1->0으로 초기화

그러나 해당 상황이 발생하면, 다음과 같이 데이터가 정리된다.

- dentry 정리 (dentry -> inode가 0으로 초기화)

즉, dentry의 연결된 inode만 0으로 변하며, 해당 파일의 inode는 실제 사용 중으로 계속 남아있다.

이와 같은 상황이 발생해 제대로 정리가 되지 않은 파일을 `orphan file`이라고 지칭하며, 이는 슈퍼 블록의 orphan dentry에 의해 관리된다. [자세한 정보](https://ajung7038.github.io/file%20system/OS-Ext4-Structure-Superblock/)

> 즉, OS는 하드링크 참조 카운트가 0이고, file open 참조 카운트가 0일 때 파일의 내용이 저장된 disk space를 해제하게 된다.

### ✨ 처리 흐름 (inode, dentry 변화 기준)

#### 1. unlink() 시스템 콜 호출
#### 2. 파일명으로 dentry 찾기
#### 3. dentry가 존재하던 디스크 블록을 0으로 지우거나 엔트리 앞뒤를 병합해 지움

- fs/ext4/inode.c

### ✨ ext4 directory entry 구조
```c
struct ext4_dir_entry_2 {
    uint32_t inode;        // 연결된 inode 번호
    uint16_t rec_len;      // 디렉토리 엔트리 전체 길이
    uint8_t  name_len;     // 이름 길이
    uint8_t  file_type;    // 파일 타입
    char     name[];       // 파일 이름
};
```

파일명이 유동적이기 때문에 각 엔트리의 크기가 다르다.

해당 위치의 디스크 블록을 0으로 지우거나 엔트리 앞뒤의 쓰이지 않는 블록 길이를 파악해 함께 0으로 밀어버리는 것도, 위와 같은 이유다.

고정 크기의 블록에서는 0으로 덮지 않아도 문제가 생기지 않지만, 가변 할당되는 `directory entry`의 경우 충돌 문제가 발생할 수 있기 때문에 충돌을 미연에 방지하고자 해당 디렉터리 엔트리 블록을 0으로 초기화한다.

더 나아가 쓰이지 않는 디엔트리 공간도 미리 확인 후 확보함으로써 충돌을 방지한다.

> 참고로, 디렉터리 엔트리 리스트의 첫 번째 요소인 경우 inode=0으로 초기화하고 rec_len 이후부터 밀린다.
> <br/>파일 시스템에서 디렉터리를 스캔하기 위해서는 첫 번째 길이가 남아 있어야 하기 때문이다.

#### 5. 저널 기록 후 트랜잭션 종료

### ✨ drop_nlink()
- 일반 파일이면 링크 수 1개일 때 지워졌다고 판단
- 링크 수가 0이 되면 ext4_orphan_add()로 orphan 리스트에 추가
- close()가 호출되기 전까지 유지 후, 정상적으로 파일이 닫혔다면 orphan list에서 제거

### ✨ unlink() 요약

- `링크카운트==0` && `inode 참조 카운트==0`이면

| 항목               | 처리 방식                                        |
| ---------------- | -------------------------------------------- |
| **inode table**  | 비어있는 inode로만 표시, 실제로 0으로 밀리지 않음   |
| **inode bitmap** | 해당 inode 위치에 있는 비트가 1->0으로 변경                |
| **block bitmap** | inode bitmap과 같이, 해당 블록 위치에 있는 비트가 1->0으로 변경 |
| **data block**    | inode table과 같이, 비어있는 블록으로 표시만 한 후 0으로 밀리지는 않음  |

- `링크카운트==0`만 만족하면

| 항목               | 처리 방식                                        |
| ---------------- | -------------------------------------------- |
| **inode table**  | inode가 가용 상태 X, 여전히 살아있음   |
| **inode bitmap** | 여전히 비트맵 1로 유지                |
| **block bitmap** | 관련 데이터 블록이 1로 유지 |
| **data block**    | 살아있고, 사용 중인 블록으로 표시 (1로 유지)  |

dentry의 경우 링크 카운트가 0이 되면 위치에 따라 다음과 같은 역할을 한다. (덮어쓰기의 경우)

- dentry를 지우려고 할 때 해당 dentry 앞에 다른 dentry가 존재하는 경우
<br/> -> `앞의 dentry가 삭제된 상태?` => 둘을 합쳐서 rec_len으로 연결 후 전체를 0으로 밀어버림
<br/> -> `앞의 dentry가 살아있으면?` (inode != 0) => 현재 dentry만 삭제

## 🫧 rmdir()

- 디렉터리를 대상으로 함수 호출
- 내부적으로 unlink()와 똑같이 `ext4_delete_entry()` 함수 호출

## 🫧 unlink() vs rmdir()

| 항목          | `unlink()`                                    | `rmdir()`                            |
| ----------- | --------------------------------------------- | ------------------------------------ |
| 대상          | 일반 파일                                         | 디렉터리                                 |
| 호출되는 함수    | `ext4_delete_entry()`| `ext4_delete_entry()` |
| i_nlink | 해당 파일 inode만 감소 (1 감소) | 부모 디렉터리, 현재 디렉터리의 링크수 감소 (2 감소) |

- 내부적으로 파일을 삭제하기 위해 inode bitmap과 block bitmap의 비트를 0으로 만드는 것 등의 작업을 제외하고는 unlink(), rmdir() 모두 똑같은 함수를 호출해 dentry를 해제한다.


## 🫧 참고 자료
- [unlink 설명](https://www.it-note.kr/177)