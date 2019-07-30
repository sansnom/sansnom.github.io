---
layout: post
title: "Alpine Docker fork or not to fork"
date: 2019-08-01
---

Very short story about alpine and docker.

As you probably know (if you create Dockerfile), it's not recommanded to use `shell -c command` (see https://docs.docker.com/engine/reference/builder/#entrypoint) because if the process is forked it won't receive signals. For exemple, the signal sent by docker with `docker stop`.

So when I stumble on Dockerfile like this, I said "Hum, I should correct this".
But I was a bit surprised to see that the `docker stop` seemns to work anyway.

These Dockerfile were using the alpine image, I even saw blog where it show the `shell -c` forking a process.
So why was it working here ?

After doing a test, we were using the latest version (3.8) of the alpine image. And this version did contain a change about `shell -c`. It doesn't fork anymore. So it's safe to use it.

I found the change looking at busybox: "[SHELL] Optimize dash -c "command" to avoid a fork".

I wonder how many people have enjoy this update whitout even knowning it.