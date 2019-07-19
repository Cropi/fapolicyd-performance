# Fapolicyd Performance

# how to use

```
$ sudo dnf install fapolicyd cmake gcc-c++ strace
```

* Put
```
allow all all
```
* at the beginning of the /etc/fapolicyd/fapolicyd.rules

```
$ cmake .
$ make
$ cd src/
$ sudo ./benchmark-single-exec.sh one-binary 10000
$ sudo ./benchmark-multiple-exec.sh multiple-binary 10000
$ sudo ./benchmark-open-in-minute.sh open_test 10000
```

# About Benchmark
## Identified Scenarios

* Single “exec()”
  * Fapolicyd disabled
  * Fapolicyd enabled
* Multiple “exec()”
  * Fapolicyd disabled
  * Fapolicyd enabled
* Multiple “open()”
  * Fapolicyd disabled
  * Fapolicyd enabled
  
## Environment

* RHEL8 - RHEL-8.1.0-20190701.0-22876-2019-07-19-11-51
  * fapolicyd-0.8.9-1.el8.x86_64
  * cmake-3.11.4-3.el8.x86_64
  * gcc-c++-8.3.1-4.4.el8.x86_64
  
### Single "exec()" Scenario

```
$ sudo ./benchmark-single-exec.sh one-binary 10000
```

#### Description

It runs one-binary binary with "strace -T" and it filters out everything but "execve". "one-binary" is runner which is forking itself for x times and executng "/bin/true". This scenario is supposed to be comparing runs with and without fapolicyd. This approach should theoretically be the best optimized usecase in favor of fapolicyd. It is so because fapolicyd has "/bin/true" in cache.

In this case we are trying to measure how much time "execve" takes.

#### Results

* Without fapolicyd disabled
  * 0.0000735291 sec
  * 0.0000699629 sec
  * 0.0000708038 sec
  * avg -> 0.0000714319 sec
  
* With fapolicyd enabled
  * 0.000119749 sec
  * 0.000122209 sec
  * 0.000117983 sec
  * avg -> 0.000119980 sec

* Ratio -> 0.000119980 / 0.0000714319 = 1.6796417286954428
  * ~67% performance hit onn single "execve"
  

### Multi "exec()" Scenario
```
$ sudo ./benchmark-multiple-exec.sh multiple-binary 10000
```

#### Description

It runs multiple-binary binary with "strace -T" and it filters out everything but "execve". "multiple-binary" is runner which is forking itself for x times and executng each time different binary. That is guaranteed by worker script which copies and renames "/bin/true" therefore there are so many different binaries prepared in "/tmp" for this scenario. This scenario is supposed to be comparing runs with and without fapolicyd. This approach should theoretically be the worst optimized usecase. It causes many cache misses

In this case we are trying to measure how much time "execve" takes.

#### Results

* Without fapolicyd disabled
  * 0.0000903935 sec
  * 0.0000889333 sec
  * 0.0000870839 sec
  * avg -> 0.000088803 sec
  
* With fapolicyd enabled
  * 0.000158834 sec
  * 0.000151543 sec
  * 0.000148677 sec
  * avg -> 0.000153018 sec

* Ratio -> 0.000153018 / 0.000088803 = 1.72311746224789
  * ~72% performance hit on single "execve"
  

