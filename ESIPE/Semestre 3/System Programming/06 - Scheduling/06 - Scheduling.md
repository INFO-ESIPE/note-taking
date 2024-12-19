[[System Programming]]
***
**Table of Contents**
```table-of-contents
```

***
## Refresher

Scheduling is about deciding which threads are given access to resources
from moment to moment
	*Often we think about CPU time, but we could also think about access to other kinds of resources (disk access, network BW...)*

Programs usually follow this execution model: Use CPU for some time, then does some I/O operations, then uses CPU again ...
	*So, a program alternates between CPU bursts and I/O bursts*

### Criterias and goals

What we expect from our scheduler:
1. **Minimise Response Time** - elapsed time for an operation (job) shall be small
   *This is what the end-user sees: Time to echo a keystroke in editor, to compile a program... Usually, this requires to **a lot of context switching***
2. **Maximise Throughput** - maximise jobs per second
	*Focus on having an efficient usage of CPU and memory*
3. **Fairness** - sharing CPU usage should be equitable
	*Most difficult to satisfy, as establishing real fairness increases average response time*

Different algorithms are available, each has its pros and cons. None of those algorithms is "a perfect solution" as our goals cannot all be fully satisfied at the same time.


****
## First-Come, First-Served (FCFS)
*Simple, but short jobs are stuck behind long ones*

FCFS Scheduling is like a FIFO: "Run until done"
It was a widely-used type of scheduling in early systems, we just run one program until it's done (I/O operations included...)

If we have:

| Process | Burst Time |
| ------- | ---------- |
| P1      | 24         |
| P2      | 3          |
| P3      | 3          |

We obtain the following executing flow:
```
------------------------------
|        P1        | P2 | P3 |
------------------------------
|                  |    |    |
0                 24   27   30
```
> – Waiting time for P1 = 0; P2 = 24; P3 = 27
> – Average waiting time: (0 + 24 + 27)/3 = 17
> – Average completion time: (24 + 27 + 30)/3 = 27

The problem here is the **Convoy effect** it creates: Short processes are stuck behind long process.
	*If our processes are scheduled in the opposite order (P3 -> P2 -> P1), we obtain both a better avg waiting time ((6 + 0 + 3)/3 = 3), and a better avg completion time ((3 + 6 + 30)/3 = 13)*

*This scheduler's performance depends too much on the submit order...*


***
## Round Robin (RR)
*Fair and suited for short jobs, but can introduce a lot of context switching on long jobs*

Round Robin scheduling assigns a **time quantum** (typically around 10-100 ms of CPU time) to each process. When a process's quantum expires:
1. The process is **preempted** (forcibly paused).
2. It is added to the **end of the ready queue**.
3. The CPU is then allocated to the next process in the queue.

For `n` processes in the ready queue and a time quantum of `q`:
- **Each process gets `1/n` of the CPU time**: All processes share the CPU equally.
- **Processes run in chunks of at most `q` time units**: A process runs until either it finishes or its time quantum expires.
- **No process waits more than `(n-1)q` time units**

So, we can see this scheduling method emphasis on fairness, but how is the performance?
Well, it heavily depends on time quantum `q` we chose:
1. **Large `q`** behaves like FCFS. Each process runs to completion before the next one starts, negating the benefits of preemption...
2. **Moderately sized `q`** results in **interleaved execution**. Processes take turns in an orderly manner, improving responsiveness and interactivity for users.
3. **Small `q`** results in **high overhead (like hyperthreading)**. The CPU spends more time switching between processes (context switching) than actually executing them...

If we have:

| Process | Burst Time |
| ------- | ---------- |
| P1      | 53         |
| P2      | 8          |
| P3      | 68         |
| P4      | 24         |

We obtain the following executing flow for a quantum `q` of 20:
```
---------------------------------------------------
| P1 | P2 | P3 | P4 | P1 | P3 | P4 | P1 | P3 | P3 |
---------------------------------------------------
|    |    |    |    |    |    |    |    |    |    |
0   20   28   48   68   88  108  112  125  145  153
```
> – Waiting time:
> P1=(68-20)+(112-88) = 72
> P2=(20-0) = 20
> P3=(28-0)+(88-48)+(125-108) = 85
> P4=(48-0)+(108-68) = 88
> - Average waiting time:  (72+20+85+88)/4 = 66¼
> - Average completion time:  (125+28+153+112)/4 = 104½


***
## Shortest Job First (SJF)


***
## Shortest Remaining Time First (SRTF)


***
## Multi-Level Feedback Scheduling
