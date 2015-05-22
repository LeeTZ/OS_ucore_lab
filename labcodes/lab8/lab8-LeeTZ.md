操作系统Lab8实验报告
===================
计14班 李铁峥 2011011268

-------------------

1. 练习1：中给出设计实现”UNIX的PIPE机制“的概要设方案，鼓励给出详细设计方案

其详细设计如下：
```
if ((blkoff = offset % SFS_BLKSIZE) != 0) {
        size = (nblks != 0) ? (SFS_BLKSIZE - blkoff) : (endpos - offset);
        if ((ret = sfs_bmap_load_nolock(sfs, sin, blkno, &ino)) != 0)
            goto out;
        if ((ret = sfs_buf_op(sfs, buf, size, ino, blkoff)) != 0)
            goto out;
        alen += size;
        if (nblks == 0)
            goto out;
        buf += size;
        ++blkno;
        --nblks;
    }
    while (nblks > 0) {
        if ((ret = sfs_bmap_load_nolock(sfs, sin, blkno, &ino)) != 0)
            goto out;
        if ((ret = sfs_block_op(sfs, buf, ino, 1)) != 0)
            goto out;
        buf += SFS_BLKSIZE;
        ++blkno;
        --nblks;
        alen += SFS_BLKSIZE;
    }
    if ((size = endpos % SFS_BLKSIZE) != 0) {
        if ((ret = sfs_bmap_load_nolock(sfs, sin, blkno, &ino)) != 0)
            goto out;
        if ((ret = sfs_buf_op(sfs, buf, size, ino, 0)) != 0)
            goto out;
        alen += size;
    }     
```

建立管道的时候，给发送和接收的进程绑定同一个描述符，并在OS内核中记录这个描述符。新建与这个描述符对应的信号量。
每次发送的进程发送信息到管道，OS内核会在内存中缓冲这部分内容
每次接收的进程从管道接收信息，OS内核会从缓冲中给予内容，否则进程进入阻塞

2. 练习2：中给出设计实现基于”UNIX的硬链接和软链接机制“的概要设方案

硬链接：在文件的描述中加入一个指针位和一个指针。当一个文件是硬链接时，指针位置位1，指针指向该硬链接链接的文件
软链接：拷贝文件对应的inode，复制一份相同的文件即可。

## 与参考答案的对比

本次实验代码量较大，虽然按照注释完成，但与参考答案实现有所出入。

## 实验中的知识点

和之前的几次试验相比，实验八增加了文件系统，并因此实现了通过文件系统来加载可执行文件到内存中运行的功能，是本次实验的重点。

## 原理课中的知识点

SFS文件系统层的处理流程已经实现好了。 VFS提供的统一接口的使用没有实现。



