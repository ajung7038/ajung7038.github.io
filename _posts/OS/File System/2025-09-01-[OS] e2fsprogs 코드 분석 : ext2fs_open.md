---
title: "[OS] e2fsprogs 코드 분석 : ext2fs_open"
categories:
  - File System
tags:
toc: true
toc_sticky: true
date: 2025-09-01 14:55:00 +0900
---

<strong>[e2fsprogs](https://github.com/tytso/e2fsprogs/tree/master)의 깃허브 코드를 참조해 분석한 글입니다.</strong>
{: .notice}

# 📌 e2fsprogs 코드 분석 : ext2fs_open

## 🫧 ext2fs_open()
- 슈퍼블록을 읽어올 때 사용


## 🫧 과정

1. 매직 넘버 체크
2. 구조체 초기화 및 io 장치 오픈
3. 슈퍼블록 읽기
4. 블록 사이즈 검사
	- inode table이 차지하는 블록 수 계산
5. 그룹 디스크립터 읽기

![alt text](../../../assets/image/OS/ext2fs_open.png)

## 🫧 특징

- `inode 테이블이 차지하는 블록 수` 계산 (각 그룹 당 inode 수 * inode 사이즈 (inode table) + (블록크기-1)/블록크기), `각 블록 그룹의 블록 수 계산` 등의 작업을 진행함

## 🫧 사용

ext4magic에서는 다음과 같이 사용한다.

```c

// 호출
open_filesystem(argv[optind], open_flags,superblock, blocksize, magicscan, data_filename);

// open_filesystem 정의
static void open_filesystem(char *device, int open_flags, blk_t superblock,
                            blk_t blocksize, int magicscan,
                            char *data_filename)

// 내부적으로 e2fsprogs 호출
// !! 핵심 함수 !! (분석하고자 하는 함수)
retval = ext2fs_open(device, open_flags, superblock, blocksize, unix_io_manager, &current_fs);
```

- `device` : 파일시스템이 존재하는 경로 (ex, /dev/loop13 또는 디스크 덤프 이미지)
- `open_flags` : -r과 같은 인자에 따라 case를 나누면서 플래그를 설정해줌.

```c
case 'z':   //experimental not activ at default
    data_filename = optarg;
    break;

case 'y':   //experimental not activ at default
    open_flags |= EXT2_FLAG_IMAGE_FILE;
    break;
```

- `blk_t superblock` : 0으로 초기화
- `blk_t blocksize` : 0으로 초기화
- `unix_io_manager` : 뭔지 모르겠음. 안 보여요..
- `ext2_filsys current_fs` : 초기화된 (NULL) 포인터


## 🫧 코드

### ✨ ext2fs_open()

- libs/ext2fs/openfs.c, $88

```c
errcode_t ext2fs_open(const char *name, int flags, int superblock,
		      unsigned int block_size, io_manager manager,
		      ext2_filsys *ret_fs)
{
	return ext2fs_open2(name, 0, flags, superblock, block_size,
			    manager, ret_fs);
}
```




### ✨ ext2fs_open()

- libs/ext2fs/openfs.c, $88

```c
blk64_t ext2fs_descriptor_block_loc2(ext2_filsys fs, blk64_t group_block,
				     dgrp_t i)
{
	int	bg;
	int	has_super = 0, group_zero_adjust = 0;
	blk64_t	ret_blk;

	/*
	 * On a bigalloc FS with 1K blocks, block 0 is reserved for non-ext4
	 * stuff, so adjust for that if we're being asked for group 0.
	 */
	if (i == 0 && fs->blocksize == 1024 && EXT2FS_CLUSTER_RATIO(fs) > 1)
		group_zero_adjust = 1;

	if (!ext2fs_has_feature_meta_bg(fs->super) ||
	    (i < fs->super->s_first_meta_bg))
		return group_block + i + 1 + group_zero_adjust;

	bg = EXT2_DESC_PER_BLOCK(fs->super) * i;
	if (ext2fs_bg_has_super(fs, bg))
		has_super = 1;
	ret_blk = ext2fs_group_first_block2(fs, bg);
	/*
	 * If group_block is not the normal value, we're trying to use
	 * the backup group descriptors and superblock --- so use the
	 * alternate location of the second block group in the
	 * metablock group.  Ideally we should be testing each bg
	 * descriptor block individually for correctness, but we don't
	 * have the infrastructure in place to do that.
	 */
	if (group_block != fs->super->s_first_data_block &&
	    ((ret_blk + has_super + fs->super->s_blocks_per_group) <
	     ext2fs_blocks_count(fs->super))) {
		ret_blk += fs->super->s_blocks_per_group;

		/*
		 * If we're going to jump forward a block group, make sure
		 * that we adjust has_super to account for the next group's
		 * backup superblock (or lack thereof).
		 */
		if (ext2fs_bg_has_super(fs, bg + 1))
			has_super = 1;
		else
			has_super = 0;
	}
	return ret_blk + has_super + group_zero_adjust;
}

blk_t ext2fs_descriptor_block_loc(ext2_filsys fs, blk_t group_block, dgrp_t i)
{
	return ext2fs_descriptor_block_loc2(fs, group_block, i);
}

errcode_t ext2fs_open(const char *name, int flags, int superblock,
		      unsigned int block_size, io_manager manager,
		      ext2_filsys *ret_fs)
{
	return ext2fs_open2(name, 0, flags, superblock, block_size,
			    manager, ret_fs);
}

static void block_sha_map_free_entry(void *data)
{
	free(data);
	return;
}

/*
 *  Note: if superblock is non-zero, block-size must also be non-zero.
 * 	Superblock and block_size can be zero to use the default size.
 *
 * Valid flags for ext2fs_open()
 *
 * 	EXT2_FLAG_RW	- Open the filesystem for read/write.
 * 	EXT2_FLAG_FORCE - Open the filesystem even if some of the
 *				features aren't supported.
 *	EXT2_FLAG_JOURNAL_DEV_OK - Open an ext3 journal device
 *	EXT2_FLAG_SKIP_MMP - Open without multi-mount protection check.
 *	EXT2_FLAG_64BITS - Allow 64-bit bitfields (needed for large
 *				filesystems)
 */
/*
* 참고: 슈퍼블록이 0이 아닌 경우 블록 크기도 0이 아닌 값이어야 합니다.
* 기본 크기를 사용하려면 Superblock과 block_size를 0으로 설정할 수 있습니다.
*
* ext2fs_open()에 유효한 플래그
*
* EXT2_FLAG_RW - 읽기/쓰기를 위해 파일 시스템을 엽니다.
* EXT2_FLAG_FORCE - 파일 시스템 중 일부라도 열기 (기능이 지원되지 않습니다)
* EXT2_FLAG_Journal_DEV_OK - ext3 저널 장치 열기
* EXT2_FLAG_SKIP_MMP - 멀티 마운트 보호 검사 없이 열립니다.
* EXT2_FLAG_64BITS - 64비트 비트 필드 허용(대형에 필요)
* 파일 시스템)
*/
// io_options는 0으로 들어옴
errcode_t ext2fs_open2(const char *name, const char *io_options,
		       int flags, int superblock,
		       unsigned int block_size, io_manager manager,
		       ext2_filsys *ret_fs)
{
	ext2_filsys	fs;
	errcode_t	retval;
	unsigned long	i, first_meta_bg;
	__u32		features;
	unsigned int	blocks_per_group, io_flags;
	blk64_t		group_block, blk;
	char		*dest, *cp;
	int		group_zero_adjust = 0;
	unsigned int	inode_size;
	__u64		groups_cnt;
#ifdef WORDS_BIGENDIAN
	unsigned int	groups_per_block;
	struct ext2_group_desc *gdp;
	int		j;
#endif
	char		*time_env;
	int		csum_retries = 0;

	// 1. 매직 넘버 체크
	EXT2_CHECK_MAGIC(manager, EXT2_ET_MAGIC_IO_MANAGER);

	// 2. 구조체 초기화? 정확히 뭘 하는 건지 모르겠음 (오픈 불가)
	retval = ext2fs_get_mem(sizeof(struct struct_ext2_filsys), &fs);
	if (retval)
		return retval;

	memset(fs, 0, sizeof(struct struct_ext2_filsys));
	fs->magic = EXT2_ET_MAGIC_EXT2FS_FILSYS;
	fs->flags = flags;
	/* don't overwrite sb backups unless flag is explicitly cleared */
	fs->flags |= EXT2_FLAG_MASTER_SB_ONLY;
	fs->umask = 022;

	time_env = ext2fs_safe_getenv("SOURCE_DATE_EPOCH");
	if (time_env) {
		fs->now = strtoul(time_env, NULL, 0);
		fs->flags2 |= EXT2_FLAG2_USE_FAKE_TIME;
	} else {
		time_env = ext2fs_safe_getenv("E2FSPROGS_FAKE_TIME");
		if (time_env)
			fs->now = strtoul(time_env, NULL, 0);
	}

	retval = ext2fs_get_mem(strlen(name)+1, &fs->device_name);
	if (retval)
		goto cleanup;
	strcpy(fs->device_name, name);
	cp = strchr(fs->device_name, '?');
	if (!io_options && cp) {
		*cp++ = 0;
		io_options = cp;
	}

	io_flags = 0;
	if (flags & EXT2_FLAG_RW)
		io_flags |= IO_FLAG_RW;
	if (flags & EXT2_FLAG_EXCLUSIVE)
		io_flags |= IO_FLAG_EXCLUSIVE;
	if (flags & EXT2_FLAG_DIRECT_IO)
		io_flags |= IO_FLAG_DIRECT_IO;
	if (flags & EXT2_FLAG_THREADS)
		io_flags |= IO_FLAG_THREADS;

	// 2. i/o 장치 오픈
	retval = manager->open(fs->device_name, io_flags, &fs->io);
	if (retval)
		goto cleanup;
	if (io_options &&
	    (retval = io_channel_set_options(fs->io, io_options)))
		goto cleanup;
	fs->image_io = fs->io;
	fs->io->app_data = fs;
	retval = io_channel_alloc_buf(fs->io, -SUPERBLOCK_SIZE, &fs->super);
	if (retval)
		goto cleanup;
	if (flags & EXT2_FLAG_IMAGE_FILE) {
		retval = ext2fs_get_mem(sizeof(struct ext2_image_hdr),
					&fs->image_header);
		if (retval)
			goto cleanup;
		retval = io_channel_read_blk(fs->io, 0,
					     -(int)sizeof(struct ext2_image_hdr),
					     fs->image_header);
		if (retval)
			goto cleanup;
		if (ext2fs_le32_to_cpu(fs->image_header->magic_number) != EXT2_ET_MAGIC_E2IMAGE)
			return EXT2_ET_MAGIC_E2IMAGE;
		superblock = 1;
		block_size = ext2fs_le32_to_cpu(fs->image_header->fs_blocksize);
	}

	/*
	 * If the user specifies a specific block # for the
	 * superblock, then he/she must also specify the block size!
	 * Otherwise, read the master superblock located at offset
	 * SUPERBLOCK_OFFSET from the start of the partition.
	 *
	 * Note: we only save a backup copy of the superblock if we
	 * are reading the superblock from the primary superblock location.
	 */

	// 3. 슈퍼블록 읽기
	// 장치가 오픈된 경우
	if (superblock) {
		if (!block_size) {
			retval = EXT2_ET_INVALID_ARGUMENT;
			goto cleanup;
		}
		io_channel_set_blksize(fs->io, block_size);
		group_block = superblock;
		fs->orig_super = 0;
	} else {
		io_channel_set_blksize(fs->io, SUPERBLOCK_OFFSET);

		// 없는 경우 슈퍼블록을 1로 만들어 다음 실행
		superblock = 1;
		group_block = 0;
		retval = ext2fs_get_mem(SUPERBLOCK_SIZE, &fs->orig_super);
		if (retval)
			goto cleanup;
	}
retry:
	retval = io_channel_read_blk(fs->io, superblock, -SUPERBLOCK_SIZE,
				     fs->super);
	if (retval)
		goto cleanup;
	if (fs->orig_super)
		memcpy(fs->orig_super, fs->super, SUPERBLOCK_SIZE);

	if (!(fs->flags & EXT2_FLAG_IGNORE_CSUM_ERRORS)) {
		retval = 0;
		if (!ext2fs_verify_csum_type(fs, fs->super))
			retval = EXT2_ET_UNKNOWN_CSUM;
		if (!ext2fs_superblock_csum_verify(fs, fs->super)) {
			if (csum_retries++ < 3)
				goto retry;
			retval = EXT2_ET_SB_CSUM_INVALID;
		}
	}

#ifdef WORDS_BIGENDIAN
	fs->flags |= EXT2_FLAG_SWAP_BYTES;
	ext2fs_swap_super(fs->super);
#else
	if (fs->flags & EXT2_FLAG_SWAP_BYTES) {
		retval = EXT2_ET_UNIMPLEMENTED;
		goto cleanup;
	}
#endif

	if (fs->super->s_magic != EXT2_SUPER_MAGIC)
		retval = EXT2_ET_BAD_MAGIC;
	if (retval)
		goto cleanup;

	if (fs->super->s_rev_level > EXT2_LIB_CURRENT_REV) {
		retval = EXT2_ET_REV_TOO_HIGH;
		goto cleanup;
	}

	/*
	 * Check for feature set incompatibility
	 */
	if (!(flags & EXT2_FLAG_FORCE)) {
		features = fs->super->s_feature_incompat;
#ifdef EXT2_LIB_SOFTSUPP_INCOMPAT
		if (flags & EXT2_FLAG_SOFTSUPP_FEATURES)
			features &= ~EXT2_LIB_SOFTSUPP_INCOMPAT;
#endif
		if (features & ~EXT2_LIB_FEATURE_INCOMPAT_SUPP) {
			retval = EXT2_ET_UNSUPP_FEATURE;
			goto cleanup;
		}

		features = fs->super->s_feature_ro_compat;
#ifdef EXT2_LIB_SOFTSUPP_RO_COMPAT
		if (flags & EXT2_FLAG_SOFTSUPP_FEATURES)
			features &= ~EXT2_LIB_SOFTSUPP_RO_COMPAT;
#endif
		if ((flags & EXT2_FLAG_RW) &&
		    (features & ~EXT2_LIB_FEATURE_RO_COMPAT_SUPP)) {
			retval = EXT2_ET_RO_UNSUPP_FEATURE;
			goto cleanup;
		}

		if (!(flags & EXT2_FLAG_JOURNAL_DEV_OK) &&
		    ext2fs_has_feature_journal_dev(fs->super)) {
			retval = EXT2_ET_UNSUPP_FEATURE;
			goto cleanup;
		}
	}

	// 4. 블록 사이즈 검사
	if ((fs->super->s_log_block_size >
	     (unsigned) (EXT2_MAX_BLOCK_LOG_SIZE - EXT2_MIN_BLOCK_LOG_SIZE)) ||
	    (fs->super->s_log_cluster_size >
	     (unsigned) (EXT2_MAX_CLUSTER_LOG_SIZE - EXT2_MIN_CLUSTER_LOG_SIZE)) ||
	    (fs->super->s_log_block_size > fs->super->s_log_cluster_size) ||
	    (fs->super->s_log_groups_per_flex > 31)) {
		retval = EXT2_ET_CORRUPT_SUPERBLOCK;
		goto cleanup;
	}

	/*
	 * bigalloc requires cluster-aware bitfield operations, which at the
	 * moment means we need EXT2_FLAG_64BITS.
	 */
	if (ext2fs_has_feature_bigalloc(fs->super) &&
	    !(flags & EXT2_FLAG_64BITS)) {
		retval = EXT2_ET_CANT_USE_LEGACY_BITMAPS;
		goto cleanup;
	}

	if (!ext2fs_has_feature_bigalloc(fs->super) &&
	    (fs->super->s_log_block_size != fs->super->s_log_cluster_size)) {
		retval = EXT2_ET_CORRUPT_SUPERBLOCK;
		goto cleanup;
	}
	fs->fragsize = fs->blocksize = EXT2_BLOCK_SIZE(fs->super);
	inode_size = EXT2_INODE_SIZE(fs->super);
	if ((inode_size < EXT2_GOOD_OLD_INODE_SIZE) ||
	    (inode_size > fs->blocksize) ||
	    (inode_size & (inode_size - 1))) {
		retval = EXT2_ET_CORRUPT_SUPERBLOCK;
		goto cleanup;
	}

	/* Enforce the block group descriptor size */
	// 슈퍼블록에서 그룹 디스크립터 사이즈 가져오기
	if (ext2fs_has_feature_64bit(fs->super)) {
		unsigned desc_size = fs->super->s_desc_size;

		if (desc_size == 0 ||
		    (!(flags & EXT2_FLAG_IGNORE_SB_ERRORS) &&
		     ((desc_size > EXT2_MAX_DESC_SIZE) ||
		      (desc_size < EXT2_MIN_DESC_SIZE_64BIT) ||
		      (desc_size & (desc_size - 1)) != 0))) {
			retval = EXT2_ET_BAD_DESC_SIZE;
			goto cleanup;
		}
	}

	fs->cluster_ratio_bits = fs->super->s_log_cluster_size -
		fs->super->s_log_block_size;
	if (EXT2_BLOCKS_PER_GROUP(fs->super) !=
	    EXT2_CLUSTERS_PER_GROUP(fs->super) << fs->cluster_ratio_bits) {
		retval = EXT2_ET_CORRUPT_SUPERBLOCK;
		goto cleanup;
	}
	// 한 블록에 inode table이 차지하는 블록 수 계산
	// 각 그룹 당 inode 수 * inode 사이즈 (inode table) + (블록크기-1)/블록크기
	fs->inode_blocks_per_group = ((EXT2_INODES_PER_GROUP(fs->super) *
				       EXT2_INODE_SIZE(fs->super) +
				       EXT2_BLOCK_SIZE(fs->super) - 1) /
				      EXT2_BLOCK_SIZE(fs->super));
	if (block_size) {
		if (block_size != fs->blocksize) {
			retval = EXT2_ET_UNEXPECTED_BLOCK_SIZE;
			goto cleanup;
		}
	}
	/*
	 * Set the blocksize to the filesystem's blocksize.
	 */
	io_channel_set_blksize(fs->io, fs->blocksize);

	/*
	 * If this is an external journal device, don't try to read
	 * the group descriptors, because they're not there.
	 */
	 // 외부 저널 장치인 경우
	if (ext2fs_has_feature_journal_dev(fs->super)) {
		fs->group_desc_count = 0;
		*ret_fs = fs;
		return 0;
	}

	if (EXT2_INODES_PER_GROUP(fs->super) == 0) {
		retval = EXT2_ET_CORRUPT_SUPERBLOCK;
		goto cleanup;
	}
	/* Precompute the FS UUID to seed other checksums */
	ext2fs_init_csum_seed(fs);

	/*
	 * Read group descriptors
	 */

	// 5. 그룹 디스크립터 읽기
	blocks_per_group = EXT2_BLOCKS_PER_GROUP(fs->super);
	
	// 오류 나지 않기 위한 조건
		// 블록그룹 당 최소 블록 수는 8을 넘어야 함
		// 최대 블록 그룹 수보다 작아야 함
		// 그룹당 최대 inode 수보다 작아야 함
		// 한 블록 그룹 안 desc 개수가 0이면 안 됨
	if (blocks_per_group < 8 ||
	    blocks_per_group > EXT2_MAX_BLOCKS_PER_GROUP(fs->super) ||
	    fs->inode_blocks_per_group > EXT2_MAX_INODES_PER_GROUP(fs->super) ||
           EXT2_DESC_PER_BLOCK(fs->super) == 0 ||
           fs->super->s_first_data_block >= ext2fs_blocks_count(fs->super)) {
		retval = EXT2_ET_CORRUPT_SUPERBLOCK;
		goto cleanup;
	}
	groups_cnt = ext2fs_div64_ceil(ext2fs_blocks_count(fs->super) -
				       fs->super->s_first_data_block,
				       blocks_per_group);
	if (groups_cnt >> 32) {
		retval = EXT2_ET_CORRUPT_SUPERBLOCK;
		goto cleanup;
	}
	fs->group_desc_count = 	groups_cnt;
	if (!(flags & EXT2_FLAG_IGNORE_SB_ERRORS) &&
	    (__u64)fs->group_desc_count * EXT2_INODES_PER_GROUP(fs->super) !=
	    fs->super->s_inodes_count) {
		retval = EXT2_ET_CORRUPT_SUPERBLOCK;
		goto cleanup;
	}
	fs->desc_blocks = ext2fs_div_ceil(fs->group_desc_count,
					  EXT2_DESC_PER_BLOCK(fs->super));
	if (ext2fs_has_feature_meta_bg(fs->super) &&
	    (fs->super->s_first_meta_bg > fs->desc_blocks) &&
	    !(flags & EXT2_FLAG_IGNORE_SB_ERRORS)) {
		retval = EXT2_ET_CORRUPT_SUPERBLOCK;
		goto cleanup;
	}
	if (flags & EXT2_FLAG_SUPER_ONLY)
		goto skip_read_bg;
	retval = ext2fs_get_array(fs->desc_blocks, fs->blocksize,
				&fs->group_desc);
	if (retval)
		goto cleanup;
	if (!group_block)
		group_block = fs->super->s_first_data_block;
	/*
	 * On a FS with a 1K blocksize, block 0 is reserved for bootloaders
	 * so we must increment block numbers to any group 0 items.
	 *
	 * However, we cannot touch group_block directly because in the meta_bg
	 * case, the ext2fs_descriptor_block_loc2() function will interpret
	 * group_block != s_first_data_block to mean that we want to access the
	 * backup group descriptors.  This is not what we want if the caller
	 * set superblock == 0 (i.e. auto-detect the superblock), which is
	 * what's going on here.
	 */
	if (group_block == 0 && fs->blocksize == 1024)
		group_zero_adjust = 1;
	dest = (char *) fs->group_desc;
#ifdef WORDS_BIGENDIAN
	groups_per_block = EXT2_DESC_PER_BLOCK(fs->super);
#endif
	if (ext2fs_has_feature_meta_bg(fs->super) &&
	    !(flags & EXT2_FLAG_IMAGE_FILE)) {
		first_meta_bg = fs->super->s_first_meta_bg;
		if (first_meta_bg > fs->desc_blocks)
			first_meta_bg = fs->desc_blocks;
	} else
		first_meta_bg = fs->desc_blocks;
	if (first_meta_bg) {
		retval = io_channel_read_blk(fs->io, group_block +
					     group_zero_adjust + 1,
					     first_meta_bg, dest);
		if (retval)
			goto cleanup;
#ifdef WORDS_BIGENDIAN
		gdp = (struct ext2_group_desc *) dest;
		for (j=0; j < groups_per_block*first_meta_bg; j++) {
			gdp = ext2fs_group_desc(fs, fs->group_desc, j);
			if (gdp)
				ext2fs_swap_group_desc2(fs, gdp);
		}
#endif
		dest += fs->blocksize*first_meta_bg;
	}

	for (i = first_meta_bg ; i < fs->desc_blocks; i++) {
		blk = ext2fs_descriptor_block_loc2(fs, group_block, i);
		io_channel_cache_readahead(fs->io, blk, 1);
	}

	for (i=first_meta_bg ; i < fs->desc_blocks; i++) {
		blk = ext2fs_descriptor_block_loc2(fs, group_block, i);
		retval = io_channel_read_blk64(fs->io, blk, 1, dest);
		if (retval)
			goto cleanup;
#ifdef WORDS_BIGENDIAN
		for (j=0; j < groups_per_block; j++) {
			gdp = ext2fs_group_desc(fs, fs->group_desc,
						i * groups_per_block + j);
			if (gdp)
				ext2fs_swap_group_desc2(fs, gdp);
		}
#endif
		dest += fs->blocksize;
	}

	fs->stride = fs->super->s_raid_stride;

	/*
	 * If recovery is from backup superblock, Clear _UNININT flags &
	 * reset bg_itable_unused to zero
	 */
	if (superblock > 1 && ext2fs_has_group_desc_csum(fs)) {
		dgrp_t group;

		for (group = 0; group < fs->group_desc_count; group++) {
			ext2fs_bg_flags_clear(fs, group, EXT2_BG_BLOCK_UNINIT);
			ext2fs_bg_flags_clear(fs, group, EXT2_BG_INODE_UNINIT);
			ext2fs_bg_itable_unused_set(fs, group, 0);
			/* The checksum will be reset later, but fix it here
			 * anyway to avoid printing a lot of spurious errors. */
			ext2fs_group_desc_csum_set(fs, group);
		}
		if (fs->flags & EXT2_FLAG_RW)
			ext2fs_mark_super_dirty(fs);
	}
skip_read_bg:
	if (ext2fs_has_feature_mmp(fs->super) &&
	    !(flags & EXT2_FLAG_SKIP_MMP) &&
	    (flags & (EXT2_FLAG_RW | EXT2_FLAG_EXCLUSIVE))) {
		retval = ext2fs_mmp_start(fs);
		if (retval) {
			fs->flags |= EXT2_FLAG_SKIP_MMP; /* just do cleanup */
			ext2fs_mmp_stop(fs);
			goto cleanup;
		}
	}

	if (fs->flags & EXT2_FLAG_SHARE_DUP) {
		fs->block_sha_map = ext2fs_hashmap_create(ext2fs_djb2_hash,
					block_sha_map_free_entry, 4096);
		if (!fs->block_sha_map) {
			retval = EXT2_ET_NO_MEMORY;
			goto cleanup;
		}
		ext2fs_set_feature_shared_blocks(fs->super);
	}

	if (ext2fs_has_feature_casefold(fs->super))
		fs->encoding = ext2fs_load_nls_table(fs->super->s_encoding);

	fs->flags &= ~EXT2_FLAG_NOFREE_ON_ERROR;
	*ret_fs = fs;

	return 0;
cleanup:
	if (!(flags & EXT2_FLAG_NOFREE_ON_ERROR)) {
		ext2fs_free(fs);
		fs = NULL;
	}
	*ret_fs = fs;
	return retval;
}
```