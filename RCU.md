---
layout: page
title: RCU
---

[Source:  
Exploiting Deferred Destruction:
An Analysis of Read-Copy-Update Techniques
in Operating System Kernels.  
Paul E. McKenney][analysis]


## Types of overhead
1. **Instruction execution**
2. **Pipeline stall**  
   branch misprediction, memory barriers
3. **Memory latency**  
   data moves among memory and the CPUs caches
4. **Cotention**  
   spinning, context swithing

In 199X only one problem -"'4. Contention":
- overhead due to memory latency was quite manageable
- CPU pipelines were quite short, pipeline-stall overheads were quite small.

## Read lock

| Lock\Type of overhead          | 1. | 2. | 3. | 4. | Info |
|--------------------------------|----|----|----|----|------|
| Spin lock                      | ?  | ?  | X  | X  | "code locking" |
| RW lock                        | ?  | ?  | X  |    ||
| RW lock, partitioned           | ?  | ?  | x  |    | partitioning of "data locking" |
| Asymmetric lock, (br lock)     | ?  | X  |    |    | br = big reader, lock on each CPU|

## Instruction costs
Instruction/Pipeline Costs on a
4-CPU 700MHz i386 P-III

| Operation                             | Nanoseconds |
|---------------------------------------|-------------|
| Instruction                           | 0.7         |
| Clock Cycle                           | 1.4         |
| L2 Cache Hit                          | 12.9        |
| Atomic Increment                      | 58.2        |
| Cmpxchg Atomic Increment              | 107.3       |
| Atomic Incr. Cache Transfer           | 113.2       |
| Main Memory                           | 162.4       |
| CPU-Local Lock                        | 163.7       |
| Cmpxchg Blind Cache Transfer          | 170.4       |
| Cmpxchg Cache Transfer and Invalidate | 360.9       |

[analysis]: https://duckduckgo.com/?q=Exploiting+Deferred+Destruction%3A+An+Analysis+of+Read-Copy-Update+Techniques+in+Operating+System+Kernels&ia=web
