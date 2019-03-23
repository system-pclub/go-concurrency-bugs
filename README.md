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

<table>
  <tr>
    <td rowspan="2">Application</td>
    <td colspan="2">Behavior</td>
    <td colspan="2">Cause</td>
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

```go


```

### Non-Blocking Bugs

```go

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
