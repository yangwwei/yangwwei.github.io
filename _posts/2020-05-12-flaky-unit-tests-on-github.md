---
title: Annoying Flaky Tests?
tags:
  - GoLang
  - Github
  - Travis CI
---
Golang tests are running good on local but becoming flaky on CI/CD pipeline?
This post introduces a few possibilities could cause this and help you improve
your unit tests stability.

<!--more-->

When we started to build CI/CD pipeline by leveraging the free sources on github,
first with [Github Actions](https://github.com/features/actions), and then [Travis CI](https://travis-ci.org/). I found our tests become to be flaky. There are some intermittent failures cannot be reproduced in our local env.

![Ocean Search](/assets/ocean-search.jpg "Ocean Search")

## What is happening?

The unit tests we have in the [yunikorn-core repo](https://github.com/apache/incubator-yunikorn-core) have 2 main types:
* functional unit tests
* smoke tests (e2e)

where I found all flaky tests are in the smoke test bucket. These tests usually
start all services, trigger a certain number of scheduling cycles and then verify
the results. The execution of such tests are very unstable once we move them
onto github. Well, I understand there must be some environment issue, based on the facts
that these tests are running on docker container or small VMs. But how can we find
out these issues?

## Try to Reproduce

The first thing you want to do is to reproduce these issues, at least some part of the issues.
DO NOT counting on finding out the problems by READING code. As we already said,
this should be related to environment, the box runs these tests. How can we simulate the env?
It's all about hardware and runtime dependences. Since these tests are all written in go,
when running on linux box, the go library already handled the cross-platform compatibilities.
What left? Yes, the hardware. This leads me to guess these failures might be caused by
competing resources, the CPU.

There are few tricks to do this with go-test.

### 1. Tests Cache
First, do not cache tests. By default, go automatically cache tests and when you run the tests again, it skips the successful ones. This can save time, but sometimes this hides some issues. When the failures are actually caused by some succeed ones. To ensure we always get all tests running, we can clean up cache all the time.

```bash
// clean up cache and cache for tests
go clean -cache -testcache
go test ./...
```

### 2. Number of CPU
Second, play with different [test flags](https://golang.org/cmd/go/#hdr-Testing_flags).
Pay attention to those flags that related to environment. Such as

```bash
-cpu 1,2,4
```

this option controls how many CPUs to use while running tests. This is the problem for me.
By default, go will try to leverage as many as CPU to run the tests.
When I set `-cpu 1`, I was able to see some test failures, the one I am expecting to see!
This setting is impacting the execution of the program heavily, as different go routines
are competing CPU resources intensively. This makes those tests that highly depend on a
certain amount of time waiting for some condition to happen become flaky.

### 3. Parallel Tests
Third, parallel execution. By default, go test will try to parallel the tests.
If your tests are having good designed, this is usually not a problem. But if there
are race conditions, this could be causing problems. To execute tests in sequence, try
run the tests with `-p` option `go test -p 1 ./...`.

## Make you tests more stable

There are few takeaways that can help to build more stable tests

### 1. Avoid go-routine leaks

We know go-routine is light-weighted, but that is not the excuse to let leaking happen.
The tests are starting/stopping the services (with many go-routines) again and again,
to avoid interfering each other, ensure the STOP function actual works!

To stop a set of go-routines in a go service, the simplest approach is to leverage
`sync.waitGroup` and standard channel. Like:

```GoLang
// start
defer m.waitGroup.Done()
for {
	select {
	case ev := <-m.pendingEvents:
		switch v := ev.(type) {
		case *someEvent:
		  // do stuff
    default:
      // do stuff
		}
	case <-m.stopChan:
		return
	}
}


// stop
close(m.stopChan)
```

Note, do not use `context.Cancel` for canceling services. It is not recommended to
store `context` in structs. See more: https://golang.org/pkg/context/.

### 2. Do NOT count on time.sleep

It's pretty common to desing test case like:

```GoLang
func TestBlabla() {
  // do something

  time.sleep("sometime")

  // verify conditions
}
```

do not count on sleeping some time waiting for some condition to happen.
Because you never know what is the proper time for the `sleep`. 1s, 3s, 10s?
Instead, we can do waitUntilCondition,

```GoLang
func WaitForCondition(eval func() bool, interval time.Duration, timeout time.Duration) error {
	deadline := time.Now().Add(timeout)
	for {
		if eval() {
			return nil
		}

		if time.Now().After(deadline) {
			return fmt.Errorf("timeout waiting for condition")
		}

		time.Sleep(interval)
	}
}

func TestBlabla() {
  // do something

  // wait for condition
  WaitForCondition(func ()  {
    // check condition
  }, "interval", "timeout")
}
```

### 3. Do NOT run for-loop in main thread

A `for` loop is consuming the CPU resource heavily. What happened to us was, we
launch a for loop in the testing code, and then rest of go routines had a hard time
to get enough CPU time to execute. Basically, you can't assume they will be running in parallel.
The behavior becomes pretty unpredicable when CPU resource is very limited. We
simply fixed this by:

```GoLang
for {
  select {
    // do something
    // give up the CPU for a small amount of time
    time.sleep(100*time.MilliSeconds)
  }
}
```

## At the End

When I was working on Apache Hadoop, I've been having problems with flaky tests already.
But for GoLang project, things are a bit different. More info can be found in this PR: https://github.com/apache/incubator-yunikorn-core/pull/140. Hope this is helpful, Good luck!!
