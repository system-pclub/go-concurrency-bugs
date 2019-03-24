## Data Set for "Understanding Real-World Concurrency Bugs in Go" in ASPLOS'2019

### Abstract
In this paper, we perform the first systematic study on concurrency bugs in real Go programs. We analyzed 171 concurrency bugs in total from six popular Go software, with more than half of them caused by non-traditional, Go-specific problems. Apart from root causes of these bugs, we also studied their fixes, performed experiments to reproduce them, and evaluated them with two publicly-available Go bug detectors.Overall, our study provides a better understanding on Go's concurrency models and can guide future researchers and practitioners in writing better, more reliable Go software and in developing debugging and diagnosis tools for Go.

### Study Methodology

#### Go Applications
We selected six representative, real-world software written in Go, including two container systems (Docker and Kubernetes), one key-value store system (etcd), two databases (CockroachDB and BoltDB), and one RPC library gRPC-go. These applications are open-source projects that have gained wide usages in datacenter environments. The following table is the information of selected applications.

| Application | Stars | Commits | Contributors | LOC | Dev History |
| ------ | ------ | ------ | ------ | ------ | ------ |
| Docker |  48975 |  35149 | 1767   | 786K   | 4.2 Years|  
| Kubernetes |   36581 | 65684 | 1679 | 2297K | 3.9 Years|
| etcd |  18417   |14101  | 436   | 441K  | 4.9 Years|
| CockroachDB |  13461       |  29485     | 197     | 520k		| 4.2 Years|
| gRPC-go |   5594        |  2528      | 148     | 53K   			| 3.3 Years|
| BoltDB |   8530        |  816       | 98      | 9K    			| 4.4 Years|

#### Bug Taxonomy
We propose a new method to categorize Go concurrency bugs according to two orthogonal dimensions. The first dimension is based on the behavior of bugs. If one or more goroutines are unintentionally stuck in their execution and cannot move forward, we call such concurrency issues blocking bugs. If instead all goroutines can finish their tasks but their behaviors are not desired, we call them non-blocking ones. The following table  shows the detailed breakdown of bug categories across each application.

<table >
  <tr>
    <td rowspan="2">Application</td>
    <td colspan="2">Behavior</td>
    <td colspan="2">Root Cause</td>
  </tr>
  <tr>
    <td >Blocking</td>
    <td >Non-Blocking</td>
    <td >Shared Memory</td>
    <td >Message Passing</td>
  </tr>
  <tr>
    <td >Docker</td>
    <td >21</td>
    <td >23</td>
    <td >28</td>
    <td >16</td>
  </tr>
  <tr>
    <td >Kubernetes</td>
    <td >17</td>
    <td >17</td>
    <td >20</td>
    <td >14</td>
  </tr>
  <tr>
    <td >etcd</td>
    <td >21</td>
    <td >16</td>
    <td >18</td>
    <td >19</td>
  </tr>
  <tr>
    <td >CockroachDB</td>
    <td >12</td>
    <td >16</td>
    <td >23</td>
    <td >5</td>
  </tr>
  <tr>
    <td >gRPC-Go</td>
    <td >11</td>
    <td >12</td>
    <td >12</td>
    <td >11</td>
  </tr>
  <tr>
    <td >BoltDB</td>
    <td >3</td>
    <td >2</td>
    <td >4</td>
    <td >1</td>
  </tr>
  <tr>
    <td >Total</td>
    <td >85</td>
    <td >86</td>
    <td >105</td>
    <td >66</td>
  </tr>
</table>

### Blocking Bugs
Overall, we found that there are around 42% blocking bugs caused by errors in protecting shared memory, and 58% are caused by errors in message passing. Considering that shared memory primitives are used more frequently than message passing ones, message passing operations are even more likely to cause blocking bugs.

#### Share Memory
For example, Docker\#25384, happens with the use of a shared variable of type WaitGroup, as shown in following figure. The Wait() at line 7 can only be unblocked, when Done() at line 5 is invoked len(pm.plugins) times, since len(pm.plugins) is used as parameter to call Add() at line 2. However, the Wait() is called inside the loop, so that it blocks goroutine creation at line 4 in later iterations and it blocks the invocation of Done() inside each created goroutine. The fix of this bug is to move the invocation of Wait() out from the loop.
```go
1 var group sync.WaitGroup
2 group.Add(len(pm.plugins))
3 for _, p := range pm.plugins {
4   go func(p *plugin) {
5    defer group.Done()
6   }
7 - group.Wait()
8 }
9 + group.Wait()
```

#### Message Passing
The following bug is caused by errors in message passing. The finishReq function creates a child goroutine using an anonymous function at line 4 to handle a request---a common practice in Go server programs. The child goroutine executes fn() and sends result back to the parent goroutine through channel ch at line 6.he child will block at line 6 until the parent pulls result from ch at line 9. Meanwhile, the parent will block at select until either when the child sends result to ch (line 9) or when a timeout happens (line 11). If timeout happens earlier or if Go runtime (non-deterministically) chooses the case at line 11 when both cases are valid, the parent will return from requestReq() at line 12, and no one else can pull result from ch any more, resulting in the child being blocked forever.

```go
1 func finishReq(timeout time.Duration) r ob {
2 -   ch := make(chan ob)
3 +   ch := make(chan ob, 1)
4   go func() {
5     result := fn()
6     ch <- result // block
7   } ()
8   select {
9     case result = <- ch:
10       return result
11     case <- time.After(timeout):
12       return nil
13   }
14 }
```

### Non-Blocking Bugs
We found around 80% of our collected non-blocking bugs are due to un-protected or wrongly protected shared memory accesses and around 20% are caused by errors in message passing.

#### Shared Memory
One example from Docker is shown in  following figure. Local variable i is shared between the parent goroutine and the goroutines it creates at line 2. The developer intends each child goroutine uses a distinct i value to initialize string apiVersion at line 4. However, values of apiVersion are non-deterministic in the buggy program. For example, if the child goroutines begin after the whole loop of the parent goroutine finishes, value of apiVersion are all equal to 'v1.21'. The buggy program only produces desired result when each child goroutine initializes string apiVersion immediately after its creation and before {\texttt{i}} is assigned to a new value.

```go
1  for i := 17; i <= 21; i++ { // write
2 -   go func() { /* Create a new goroutine */
3 +   go func(i int) {
4            apiVersion := fmt.Sprintf("v1.%d", i) // read
5            ...
6 -       }()
7 +       }(i)
8   }
```
#### Message Passing
Docker\#24007 in following figure is caused by the violation of the rule that a channel can only be closed once. When multiple goroutines execute the piece of code, more than one of them can execute the default clause and try to close the channel at line 5, causing a runtime panic in Go.
```go
1 - select {
2 -   case <- c.closed:
3 -   default:
4 +     Once.Do(func() {
5         close(c.closed)
6 +     })
7 - }
```

### Papers
[Understanding Real-World Concurrency Bugs in Go](https://songlh.github.io/paper/go-study.pdf). [Tengfei Tu](https://tengfei1010.github.io/), Xiaoyu Liu, [Linhai Song](https://songlh.github.io/), [Yiying Zhang](https://engineering.purdue.edu/~yiying/). To Appear at the 24th International Conference on Architectural Support for Programming Languages and Operating Systems (ASPLOS '19).

#### Citation
@inproceedings{go-study-asplos,<br />
&nbsp;&nbsp;&nbsp; author = {Tu, Tengfei and Liu, Xiaoyu and Song, Linhai and Zhang, Yiying}, <br />
&nbsp;&nbsp;&nbsp; title = {Understanding Real-World Concurrency Bugs in Go}, <br />
&nbsp;&nbsp;&nbsp; booktitle = {ASPLOS}, <br />
&nbsp;&nbsp;&nbsp; year = {2019}, <br />
}
