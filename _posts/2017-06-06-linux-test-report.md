---
layout: post
title:  "Linux 服务器性能测试报告"
date:   2017-06-06
categories: linux test report
tags: linux test report
---

* content
{:toc}

我们使用linux 工具sysbench 来测试linux服务器性能，目前在Ubuntu上进行操作




## Install sysbench

	apt-get install sysbench

check it

	man sysbench


## CPU Benchmark

测试CPU运行性能

	sysbench --test=cpu --cpu-max-prime=20000 run

测试结果
	
	$ sysbench --test=cpu --cpu-max-prime=20000 run
	sysbench 0.4.12:  multi-threaded system evaluation benchmark
	Running the test with following options:
	Number of threads: 1
	Doing CPU performance benchmark
	Threads started!
	Done.
	Maximum prime number checked in CPU test: 20000
	 
	Test execution summary:
	    total time:                          23.5990s
	    total number of events:              10000
	    total time taken by event execution: 23.5983
	    per-request statistics:
	         min:                                  2.34ms
	         avg:                                  2.36ms
	         max:                                  5.32ms
	         approx.  95 percentile:               2.41ms
	Threads fairness:
	    events (avg/stddev):           10000.0000/0.00
	    execution time (avg/stddev):   23.5983/0.00

记录数据
total time taken by event execution: 23.5983


## Memory Benchmark

测试读性能

	sysbench --test=memory --memory-block-size=1K --memory-scope=global --memory-total-size=100G --memory-oper=read run

测试结果

	$ sysbench --test=memory --memory-block-size=1K --memory-scope=global --memory-total-size=100G --memory-oper=read run
	sysbench 0.4.12: multi-threaded system evaluation benchmark
	 
	Running the test with following options:
	Number of threads: 1
	 
	Doing memory operations speed test
	Memory block size: 1K
	 
	Memory transfer size: 102400M
	 
	Memory operations type: read
	Memory scope type: global
	Threads started!
	Done.
	 
	Operations performed: 104857600 (6045253.78 ops/sec)
	 
	102400.00 MB transferred (5903.57 MB/sec)
	 
	 
	Test execution summary:
	total time: 17.3454s
	total number of events: 104857600
	total time taken by event execution: 12.1786
	per-request statistics:
	min: 0.00ms
	avg: 0.00ms
	max: 0.06ms
	approx. 95 percentile: 0.00ms
	 
	Threads fairness:
	events (avg/stddev): 104857600.0000/0.00
	execution time (avg/stddev): 12.1786/0.00

记录数据
102400.00 MB transferred (5903.57 MB/sec)


测试写性能

	sysbench --test=memory --memory-block-size=1K --memory-scope=global --memory-total-size=100G --memory-oper=write run

测试结果

	$ sysbench --test=memory --memory-block-size=1K --memory-scope=global --memory-total-size=100G --memory-oper=write run
	sysbench 0.4.12: multi-threaded system evaluation benchmark
	 
	Running the test with following options:
	Number of threads: 1
	 
	Doing memory operations speed test
	Memory block size: 1K
	 
	Memory transfer size: 102400M
	 
	Memory operations type: write
	Memory scope type: global
	Threads started!
	Done.
	 
	Operations performed: 104857600 (4056443.11 ops/sec)
	 
	102400.00 MB transferred (3961.37 MB/sec)
	 
	 
	Test execution summary:
	total time: 25.8496s
	total number of events: 104857600
	total time taken by event execution: 20.6986
	per-request statistics:
	min: 0.00ms
	avg: 0.00ms
	max: 0.08ms
	approx. 95 percentile: 0.00ms
	 
	Threads fairness:
	events (avg/stddev): 104857600.0000/0.00
	execution time (avg/stddev): 20.6986/0.00

记录结果

	102400.00 MB transferred (3961.37 MB/sec)


## IO Benchmark

创建文件

	sysbench --test=fileio --file-total-size=1G prepare

	$ sysbench --test=fileio --file-total-size=1G prepare
	sysbench 0.4.12: multi-threaded system evaluation benchmark
	  
	128 files, 8192Kb each, 1024Mb total
	Creating files for the test...			

测试读写性能

	sysbench --test=fileio --file-total-size=1G --file-test-mode=rndrw --init-rng=on --max-time=300 --max-requests=0 run

	$ sysbench --test=fileio --file-total-size=1G --file-test-mode=rndrw --init-rng=on --max-time=300 --max-requests=0 run
	sysbench 0.4.12: multi-threaded system evaluation benchmark
	 
	Running the test with following options:
	Number of threads: 1
	Initializing random number generator from timer.
	 
	 
	Extra file open flags: 0
	128 files, 8Mb each
	1Gb total file size
	Block size 16Kb
	Number of random requests for random IO: 0
	Read/Write ratio for combined random IO test: 1.50
	Periodic FSYNC enabled, calling fsync() each 100 requests.
	Calling fsync() at the end of test, Enabled.
	Using synchronous I/O mode
	Doing random r/w test
	Threads started!
	Time limit exceeded, exiting...
	Done.
	 
	Operations performed: 33000 Read, 22000 Write, 70340 Other = 125340 Total
	Read 515.62Mb Written 343.75Mb Total transferred 859.38Mb (2.8644Mb/sec)
	183.32 Requests/sec executed
	 
	Test execution summary:
	total time: 300.0153s
	total number of events: 55000
	total time taken by event execution: 0.4013
	per-request statistics:
	min: 0.00ms
	avg: 0.01ms
	max: 0.10ms
	approx. 95 percentile: 0.01ms
	 
	Threads fairness:
	events (avg/stddev): 55000.0000/0.00
	execution time (avg/stddev): 0.4013/0.00

记录结果

	Read 515.62Mb Written 343.75Mb Total transferred 859.38Mb (2.8644Mb/sec)

