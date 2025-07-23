---
title: "[OS] Ext4 Structure : dentry"
categories:
  - File System
tags:
toc: true
toc_sticky: true
date: 2025-07-23 10:56:00 +0900
---

<strong>[linux kernel sourse tree](https://github.com/torvalds/linux)의 깃허브 코드를 참조해 Ext4의 주요 구조체를 정리한 글입니다.</strong>
{: .notice}


> Ext4 주요 구조체 정리
<br/>
[Superblock](https://ajung7038.github.io/file%20system/OS-Ext4-Structure-Superblock), [inode](https://ajung7038.github.io/file%20system/OS-Ext4-Structure-inode), [dentry](https://ajung7038.github.io/file%20system/OS-Ext4-Structure-dentry), [Block Group](), [extent](), [orphan](), [Journal]()

# 📌 Ext4 inode

## 🫧 dentry
: 디렉터리 내부 항목을 나타내는 객체

- 디렉터리 안 파일 또는 하위 디렉터리에 대한 정보를 담고 있다
- 이름 기반 파일 접근을 빠르게 하기 위한 캐시 구조
- 항목(파일, 디렉터리 등) 과 inode를 연결하는 역할
- 실제 디스크에 존재하지 않는 VFS 레벨의 메모리 객체
- dentry cache(dcache)로 작동하여 디스크 접근을 줄이고자 함

즉, ext4가 dentry를 직접 관리하는 것이 아닌, ext4_lookup()과 같은 함수에서 돌면서 VFS가 dentry를 사용할 수 있도록 연산을 제공한다. 

## 🫧 코드

```c
struct dentry {
	// dentry 상태 여부
	unsigned int d_flags;
	
	// 해시 테이블 내 연결 리스트 노드
	struct hlist_bl_node d_hash;
	
	// dentry 부모 포인터
	struct dentry *d_parent;
	
	// 파일명 or 디렉터리명
	struct qstr d_name;
	
	// 디렉터리가 참조하는 inode
	struct inode *d_inode;
					 
	// 짧은 이름을 위한 내부 버퍼
	// 보통 32바이트 이내인 경우 d_name.name이 d_iname[]을
		// 가리키도록 함. 긴 경우는 kmalloc으로 할당
	unsigned char d_iname[DNAME_INLINE_LEN];
	
	// dentry가 속한 파일 시스템의 super_block
	struct super_block *d_sb;
	
	// 시간
	unsigned long d_time;
	
	// 부모 디렉터리의 자식 목록 내 위치
	struct hlist_node d_sib;
	
	// 현재 dentry의 자식 목록 (트리 구조)
	struct hlist_head d_children;
};
```