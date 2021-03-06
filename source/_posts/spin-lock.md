---
title: Spin Lock
date: 2020-04-19 12:02:23
tags:
- linux
- lock
---

spin lock是一种锁，使用它的线程在循环中简单地等待（“spin”），同时重复检查锁是否可用

<!-- more -->
理论上，当一个线程访问一个已经被被锁定(mutex)的临界区资源时，它将进入睡眠状态。因为这种操作比较费时，完成这些操作时，其它线程可能已经释放了锁。

在这些情况下，多处理系统使用了spin lock。当一个线程发现临界区资源被另外的线程锁着时，它将不断“旋转”，直到资源释放，在这期间，该线程始终处于运行态(但是， 一旦超过了当前线程的CPU时间片，操作系统将强制切换到另一个线程。)

在多核/多CPU系统中，如果大量的锁只保留很短的时间，那么不断的让线程进入睡眠状态并唤醒它们所浪费的时间可能会显著降低运行性能。当使用自旋锁时，线程总是在很短的时间内阻塞，但随后立即继续工作，从而获得更高的处理吞吐量。



在单处理环境下，spin lock 是无效的，因为轮询 spin lock将阻塞了唯一可用的cup内核，就没有其他线程可以运行，也就没有机会释放这个自旋锁。在这些系统中使用spin lock 只会浪费CPU时间，不会有任何好处。



hybrid mutex: 在多核系统中，混合互斥锁的行为起初类似于自旋锁。如果线程无法锁定mutex，它不会立即进入睡眠状态，因为互斥锁可能很快就会被解锁，因此互斥锁首先的行为将与自旋锁完全相同。只有在经过一定时间仍未获得锁时，线程才会真正进入睡眠状态。

Reference:
- 深入理解Linux内核 （31p）
- https://mail.python.org/pipermail/python-ideas/2010-July/007645.html
- https://stackoverflow.com/questions/5869825/when-should-one-use-a-spinlock-instead-of-mutex