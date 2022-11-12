---
layout:     post
title:      "Understanding Real-World Concurrency Bugs in Go"
subtitle:   "Concurrency Bugs in Go"
date:       2022-11-12 19:00:00
author:     "Reverie"
catalog: false
header-style: text
tags: 
- golang
---

# Concurrency Bugs in Go

## Abstract

Go is a statically-typed programming language that aims to provide a simple, efficient, and safe way to build multithreaded software. Since its creation in 2009, Go has matured and gained significant adoption in production and open-source software. Go advocates for the usage of message passing as the means of inter-thread communication and provides several new concurrency mechanisms and libraries to ease multi-threading programming. It is important to understand the implication of these new proposals and the comparison of message passing and shared memory synchronization in terms of program errors, or bugs. Unfortunately, as far as we know, there has been no study on Go’s concurrency bugs.

In this paper, we perform the first systematic study on concurrency bugs in real Go programs. We studied six popular Go software including Docker, Kubernetes, and gRPC. We analyzed 171 concurrency bugs in total, with more than half of them caused by non-traditional, Go-specific problems. Apart from root causes of these bugs, we also studied their fixes, performed experiments to reproduce them, and evaluated them with two publicly-available Go bug detectors. Overall, our study provides a better understanding on Go’s concurrency models and can guide future researchers and practitioners in writing better, more reliable Go software and in developing debugging and diagnosis tools for Go.

## 1 Introduction
Go is a statically typed language originally developedby Google in 2009. Over the past few years, it has quickly gained attraction and is now adopted by many types of software in real production. These Go applications range from libraries and high-level software to cloud infrastructure software like container systems and key-value databases. 
A major design goal of Go is to improve traditional multithreaded programming languages and make concurrent programming easier and less error-prone. For this purpose, Go centers its multi-threading design around two principles: 1) making threads (called goroutines) lightweight and easy to create and 2) using explicit messaging (called channel) to communicate across threads. With these design principles, Go proposes not only a set of new primitives and new libraries but also new implementation of existing semantics.
It is crucial to understand how Go’s new concurrency primitives and mechanisms impact concurrency bugs, the type of bugs that is the most difficult to debug and the most widely studied in traditional multi-threaded programming languages. Unfortunately, there has been no prior work in studying Go concurrency bugs. As a result, to date, it is still unclear if these concurrency mechanisms actually make Go easier to program and less error-prone to concurrency bugs than traditional languages.
In this paper, we conduct the first empirical study on Go concurrency bugs using six open-source, productiongrade Go applications: Docker and Kubernetes,two datacenter container systems, etcd, a distributed key-value store system, gRPC, an RPC library, and CockroachDB and BoltDB, two database systems.
In total, we have studied 171 concurrency bugs in these applications. We analyzed the root causes of them, performed experiments to reproduce them, and examined their fixing patches. Finally, we tested them with two existing Go concurrency bug detectors (the only publicly available ones).
Our study focuses on a long-standing and fundamental question in concurrent programming: between message passing and shared memory, which of these inter-thread communication mechanisms is less error-prone. Go is a perfect language to study this question, since it provides frameworks for both shared memory and message passing. However, it encourages the use of channels over shared memory with the belief that explicit message passing is less error-prone.
To understand Go concurrency bugs and the comparison between message passing and shared memory, we propose to categorize concurrency bugs along two orthogonal dimensions: the cause of bugs and their behavior. Along the cause dimension, we categorize bugs into those that are caused by misuse of shared memory and those caused by misuse of message passing. Along the second dimension, we separate bugs into those that involve (any number of) goroutines that cannot proceed (we call them blocking bugs) and those that do not involve any blocking (non-blocking bugs).
Surprisingly, our study shows that it is as easy to make concurrency bugs with message passing as with shared memory, sometimes even more. For example, around 58% of blocking bugs are caused by message passing. In addition to the violation of Go’s channel usage rules (e.g., waiting on a channel that no one sends data to or close), many concurrency bugs are caused by the mixed usage of message passing and other new semantics and new libraries in Go, which can easily be overlooked but hard to detect.
To demonstrate errors in message passing, we use a blocking bug from Kubernetes in Figure 1. The finishReq function creates a child goroutine using an anonymous function at line 4 to handle a request—a common practice in Go server programs. The child goroutine executes fn() and sends result back to the parent goroutine through channel ch at line 6. The child will block at line 6 until the parent pulls result from ch at line 9. Meanwhile, the parent will block at select until either when the child sends result to ch (line 9) or when a timeout happens (line 11). If timeout happens earlier or if Go runtime (non-deterministically) chooses the case at line 11 when both cases are valid, the parent will return from requestReq() at line 12, and no one else can pull result from ch any more, resulting in the child being blocked forever. The fix is to change ch from an unbuffered channel to a buffered one, so that the child goroutine can always send the result even when the parent has exit.
This bug demonstrates the complexity of using new features in Go and the difficulty in writing correct Go programs like this. Programmers have to have a clear understanding of goroutine creation with anonymous function, a feature Go proposes to ease the creation of goroutines, the usage of buffered vs. unbuffered channels, the non-determinism of waiting for multiple channel operations using select and the special library time. Although each of these features were designed to ease multi-threaded programming, in reality, it is difficult to write correct Go programs with them.
Overall, our study reveals new practices and new issues of Go concurrent programming, and it sheds light on an answer to the debate of message passing vs. shared memory accesses. Our findings improve the understanding of Go concurrency and can provide valuable guidance for future tool design.
This paper makes the following key contributions.
- We performed the first empirical study of Go concurrency bugs with six real-world, production-grade Go applications.
- We made nine high-level key observations of Go concurrency bug causes, fixes, and detection. They can be useful for Go programmers’ references. We further make eight insights into the implications of our study results to guide future research in the development, testing, and bug detection of Go.
- We proposed new methods to categorize concurrency bugs along two dimensions of bug causes and behaviors. This taxonomy methodology helped us to better compare different concurrency mechanisms and correlations of bug causes and fixes. We believe other bug studies can utilize similar taxonomy methods as well.

All our study results and studied commit logs can be found at https://github.com/system-pclub/go-concurrency-bugs.
```golang
func finishReq(timeout time.Duration) r ob {
	ch := make(chan ob)
	ch := make(chan ob, 1)
 	go func() {
 		result := fn()
 		ch <- result // block
 		} ()
 	select {
 		case result = <- ch:
 			return result
		case <- time.After(timeout):
 			return nil
 	}
}
```

## 2 Background and Applications
Go is a statically-typed programming language that is designed for concurrent programming from day one. Almost all major Go revisions include improvements in its concurrency packages. This section gives a brief background on Go’s concurrency mechanisms, including its thread model, inter-thread communication methods, and thread synchronization mechanisms. We also introduce the six Go applications we chose for this study.