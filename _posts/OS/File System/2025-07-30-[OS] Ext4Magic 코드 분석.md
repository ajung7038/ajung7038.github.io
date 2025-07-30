---
title: "[OS] Ext4Magic 코드 분석"
categories:
  - File System
tags:
toc: true
toc_sticky: true
date: 2025-07-30 17:32:00 +0900
---

<strong>[Ext4Magic](https://github.com/gktrk/ext4magic)의 깃허브 코드를 참조해 분석한 글입니다.</strong>
{: .notice}

# 📌 Ext4Magic

## 🫧 파일 복구가 불가능하거나 제한적인 경우

1. 일부 멀티미디어 형식, 분할/손상된 파일은 복구 불가능 (동영상, 오디오 등)
<br/> -> 블록 크기가 달라지면 메타데이터 위치와 구조가 달라지기 때문!

2. CD/DVD 이미지, 파일 시스템 컨테이너 등은 거의 불가능 (특히 블록 크기 4KB가 아닐 경우)

3. 매우 큰 ext3 파일은 여러 단계로 나눠서 삭제된 경우 복구 불가
<br/> -> ext4 파일에서도 조각화가 너무 심하면 복구가 어렵다

4. inline data는 복구 불가능
<br/> -> ext4magic에서 지원하지 않음
<br/>

5. 부분적으로 손상된 파일에서의 0 추가 및 삭제
<br/> -> 삭제된 파일 중 매직 스캔으로 찾은 파일은 EOF를 찾을 수 없기 때문에 툴 내에서 0을 추가하거나 남는 길이를 잘라내어 크기를 맞춘다.
<br/> -> 이러한 특성으로 인해 복구된 파일이 정상적으로 열리지 않을 수 있다.

6. inode와 dentry가 할당 해제된 상태인 경우 magic scan 기능을 활용하긴 하지만, 결과가 확실하지는 않다.

## 🫧 파일 복구 과정

![alt text](<../../../assets/image/OS/ext4magic 파일 복구 과정.png>)

## 🫧 옵션

- 해당 내용은 Ext4Magic의 `README 파일`과 `코드`를 참조해 정리했습니다.

### ✨ Information Options  -S -J -H -T 
: 슈퍼블록, 저널, 트랜잭션 및 파일 삭제/변경 사항 확인

| 옵션   | 설명    |
| ---- | -------------------------------------------------------------------- |
| `-S` | **슈퍼블록(Superblock)** 정보 출력    |
| `-J` | **저널(Journal)** 메타데이터 정보 출력 |
| `-H` | **트랜잭션(Transaction)** 리스트 출력 |
| `-T` | 파일 삭제 및 변경을 **시간 순으로 요약(time chart)** |


### ✨ Selections  -I -B  -f
: 복구할 파일, inode, 블록 등을 직접 지정할 수 있는 필터 옵션들

| 옵션           | 설명                          |
| ------------ | --------------------------- |
| `-I <inode>` | 복구 대상 inode 번호를 직접 지정       |
| `-B <block>` | 특정 블록 번호를 기준으로 복구 시도        |
| `-f <파일명>`   | 복구할 파일 이름을 지정 (와일드카드 사용 가능) |


Time Options -a -b -t
------------------------
: 저널 데이터를 검색하는 시간 범위와 inode 데이터의 시간을 결정

| 옵션               | 설명     |
| ---------------- | ------------------------------ |
| `-a <timestamp>` | 복구를 시도할 **시작 시각(after)** |
| `-b <timestamp>` | 복구를 시도할 **종료 시각(before)** |
| `-t`             | **시간 범위를 무시**하고 모든 저널 트랜잭션 대상으로 복구를 시도|



File input and output options  -d -i -j
---------------------------------------
: 출력 디렉토리, 입력 파일 목록 및 외부 저널 파일로 지정

| 옵션           | 설명                                      |
| ------------ | --------------------------------------- |
| `-d <디렉터리>`  | 복구된 파일을 저장할 **출력 디렉터리** 지정              |
| `-i <파일>`    | 복구할 대상 파일 목록을 담은 **입력 파일 경로**           |
| `-j <저널 경로>` | 외부 저널 파일을 명시적으로 지정 (e.g. 외부 로그 장치 사용 시) |



Action Options  -l -L -r -R -m -M 
----------------------------------
: 실질적인 동작을 수행하는 핵심 옵션들

| 옵션   | 설명  |
| ---- | --------- |
| `-l` | 대상 inode나 파일에 대한 정보만 출력 (복구 X)   |
| `-L` | 트랜잭션 로그 출력   |
| `-r` | 일반적인 복구 모드 (디렉터리 복구 X, 파일만 복구)  |
| `-R` | **디렉터리 구조까지 유지하면서 복구** (디렉터리 경로가 손상된 경우 복구 못할 가능성 O)     |
| `-m` | 매직 스캔을 통한 복구       |
| `-M` | **inode + dentry + orphan + journal + magic scan** 을 모두 수행 <br> → -r, -R, -m 기능을 통합하여 복구 |


Expert Options  -s -n -c -D 
-------------------------
: 손상된 파일 시스템 등에 대해 사용

| 옵션          | 설명                                                   |
| ----------- | ---------------------------------------------------- |
| `-s <슈퍼블록>` | **백업 슈퍼블록 번호**를 지정해서 사용할 수 있음 (디스크의 슈퍼블록이 손상된 경우 사용 가능)      |
| `-n`        | `ext4magic`이 디스크를 **읽기 전용으로 열지 않고 쓰기도 가능**하게 함 (기본 모드는 읽기 전용) |
| `-c`        | 손상된 저널 등을 무시하고 강제로 실행  (복구 정확도 ↓)  |
| `-D`        | 디버깅 정보 출력    |

## 🫧 코드

### ✨ ext4magic.c

```c
// start recursiv over the filesystem, used for recover, list and tree-history 
if ((mode & COMMAND_INODE) && (mode & RECOVER_INODE))
       	{
		struct ring_buf *i_list;
		struct ext2_inode* r_inode;
		r_item *item = NULL;

		if (ext2fs_test_inode_bitmap ( current_fs->inode_map, inode_nr )) {
			fprintf(stdout,"Inode %lu is allocated\n",(long unsigned int)inode_nr);
		}

		// 해당 inode의 저널 내 기록 가져오기
		i_list = get_j_inode_list(current_fs->super, inode_nr);
		
		// 시간 범위가 주어져 있으면 삭제되지 않은 inode 찾기 (삭제되었지만 덮어쓰기 되지 않은 상태)
		if (mode & INPUT_TIME)
			item = get_undel_inode(i_list,t_after,t_before);
		else
			item = get_last_undel_inode(i_list);


		if (item) {
			r_inode = (struct ext2_inode*)item->inode;
			// 디렉터리면
			if (LINUX_S_ISDIR(r_inode->i_mode) ) 
			{
				struct dir_list_head_t * dir = NULL;
				
				// 디렉터리의 내용과 메타데이터 읽기
				dir = get_dir3(NULL,0, inode_nr , "",pathname, t_after,t_before, recoverquality );
				if (dir) {
					// 로컬에서 재귀적으로 디렉터리 탐색
					lookup_local(des_dir, dir,t_after,t_before, recoverquality | recovermodus );
					if (recovermodus & HIST_DIR )
						print_coll_list(t_after, t_before, format);
//Magic step 1 + 2 +3
					// 복구 모드(imap)가 활성화 되어 있으면
					if (imap){
						imap_search(des_dir, t_after, t_before, disaster );
						// we use imap as a flag for the disaster mode
						ext2fs_free_inode_bitmap(imap);
						imap = NULL;
						
						// bmap 플래그 체크 후 ext3/ext4 복구 모드 분기
						if (bmap){
							// ext3 => 복구 안 함
							if (!(current_fs->super->s_feature_incompat & EXT3_FEATURE_INCOMPAT_EXTENTS)){ 
								printf("MAGIC function for ext3 not available, use ext4magic 0.2.4 instead\n");
//								magic_block_scan3(des_dir, t_after);
							}

							// ext4 -> 매직 블록 스캔
							else{
								//if (bmap) printf("The MAGIC Function is currently only for ext3 filesystems available\n");
								magic_block_scan4(des_dir,t_after);
							}
						}
					}
					clear_dir_list(dir);
				}
				else
					printf("Inode %lu is a directory but not found after %lu and before %lu\n",
						(long unsigned int)inode_nr, (long unsigned int)t_after, (long unsigned int)t_before);
			}
			// 디렉터리가 아닌 경우 파일 복구 수행
			else {
				if (recovermodus & (RECOV_ALL | RECOV_DEL))
					recover_file(des_dir,"", pathname, r_inode, inode_nr ,(recovermodus & RECOV_ALL )? 0 : 1 );
				else
					printf("Single file can only recovered with option -R or -r\n");
			}
		 }	
		else
			fprintf(stdout,"No undeled inode %u in journal found\n",inode_nr);

#ifdef EXPERT_MODE
		// disaster (magic scan)이 활성화되어있고 imap이 남아 있으면 모든 파일 강제 복구 시도
		if (disaster && imap){
			// Nothing worked, trying to recover all undeleted files under catastrophic conditions
			printf("Force a disaster recovery\n");
			imap_search(des_dir, t_after, t_before, disaster );
			ext2fs_free_inode_bitmap(imap);
			imap = NULL;
		}
#endif
 
		if (i_list) ring_del(i_list);
	}
```


## 🫧 참고 자료
- [ext4magic 복구](https://sourceforge.net/projects/ext4magic/)