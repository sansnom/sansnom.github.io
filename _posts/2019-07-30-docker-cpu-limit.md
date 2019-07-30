---
layout: post
title: "Small story about docker and cpu limit"
date: 2019-07-30
---

I was doing some benchmark with our new solution. Our client gave us some VM so we could check the real performance with their infrastructure.
 
Doing some profiling, I spotted a bug which iccurs large performance. 
After correcting it, I was excepting a large improvement. 
Nothing !
We couldn't handle more than 1 msg/s.

I use `top` to check the load average. Wow ! 
The load average is really important 10+ on 4 CPU machines. 
The CPU is only used at 25%.
OK ! I know the problem: if it's not the CPU then it's the IO !
Obviously, I was wrong but I didn't know that yet.
 
I use `iotop` and `iostat` but even if the IO is used it's seems far form because satured.
 
I install netdata (a nice monitoring graphical tool). 
It confirms the IO is fine.
I scroll a bit and I find the graph about CPU repartition. Only the first CPU is being used !
I'm very perplex, our solution is multi-threaded so the load should be balanced between the CPU.
 
I use `stress` (`stress --cpu 4 --timeout 60s`) to check the CPU balance. Everything is fine.
What if I do the same inside Docker ? Result: only one CPU used ! But why ?

I tried setting the options `--cpus 4`. Nothing. Then I tried `--cpuset-cpus 0-4`, this time I got an error.
I google a bit, found people trying to limit the CPU but not the other way around.

With a bit of luck I stumble on:
```
$ cat /sys/fs/cgroup/cpuset/docker/cpuset.cpus
0
```

cpuset is wrong !

```
$ echo "0-3" > /sys/fs/cgroup/cpuset/docker/cpuset.cpus
```

Everything is working fine now !
 
How could this happen ? After speaking a bit with the guy who made the install. At first, it gave only 1 CPU to the machine then 4.
That's probably where things has gone wrong.

One thing thing I could have done better: I should not have trusted my first impression about the loadaverage and use other commands like `vmstat` or `mpstat` to find the CPU unbalance.

[Nice article from netflix with commands for linux performance](https://medium.com/netflix-techblog/linux-performance-analysis-in-60-000-milliseconds-accc10403c55)