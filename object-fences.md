# Object fences

Document #: TBD <br>
Date: 20-1-2022 <br>
Audience: SG1 <br>
Reply-to: dlustig@nvidia.com, ogiroux@nvidia.com <br>

## Abstract

Object fences are fences that restrict their effects to specific objects, and place evaluations on these objects into _happens before_ but not _strongly happens before_.

## Tony tables

| Before      | After |
| ----------- | ----------- |
| `x = 1;`<br>`atomic_thread_fence(memory_order_release);`<br>`a.store(1, memory_order_relaxed);`      | `x = 1;`<br>`atomic_object_fence(memory_order_release, x);`<br>`a.store(1, memory_order_relaxed);`       |
| `while(a.load(memory_order_relaxed) != 1);`<br>`atomic_thread_fence(memory_order_acquire);`<br>`assert(x == 1);`      | `while(a.load(memory_order_relaxed) != 1);`<br>`atomic_object_fence(memory_order_acquire, x);`<br>`assert(x == 1);`       |
| (slower)   | (faster)        |

## Revisions

R0
* This is the first revision

## Motivation

***FIXME*** Message passing between two threads and no others.

***FIXME*** C++ as the lingua franca of accelerated programming

> ***FIXME*** Discussion: we don't need to put scratchpads into C++, object fences enable you to write portable code that works with non-standard scratchpad declarations

***FIXME*** Example: hardware storage-specific fencing.

***FIXME*** Example: high cost of transitivity over complex fabrics.

## Implementation

***FIXME*** Existing practice.

***FIXME*** Example: OpenMP flush with a non-empty set.

***FIXME*** Example: OpenCL local vs. global fencing.

***FIXME*** Example: non-transitive MPI-like programming systems.

***FIXME*** Conservative implementation, discard object information:

```c++
template<class... Objects>
void atomic_object_fence(memory_order order, Objects const&...) noexcept
{
    atomic_thread_fence(order);
}
```

## How to Teach

***FIXME*** Specifity to objects.

> ***FIXME*** Discussion: for complex objects it may well be that the implementation has no choice but to take the conservative implementation path.

***FIXME*** Exclusion of transitivity.

> ***FIXME*** Discussion: firstly the issue here is that object fences create a happens-before per object, and the memory model would need to become dramatically more complex to capture this. Since the motivation is point-to-point communication

***FIXME*** Exclusion of sequential consistency.

> ***FIXME*** Discussion: it’s obvious that there wouldn’t be a unique total order of object fences naming different objects. For example a FENCE.S in one SM and a FENCE.S in another SM don’t happen in any particular order. So what is “total” here, total per each individual object? So if I have two object fences that name objects O and P, those fences occur in a total order for both O and for P, but possibly different total orders for each O and P? Blecht. Maybe a more direct approach can be taken, that just prohibits relaxed behaviors on all 2-thread SC non-RC litmus tests, and every other non-SC outcome is really explained by the lack of cumulativity anyway.


## Proposed Wording

This wording is relative to N4901.

### ??.?? Fences [atomics.fences]

1. This subclause introduces synchronization primitives called fences. <span style="color: green;">**A fence invoked without object operands is a *thread fence*. A fence invoked with object operands is an *object fence*.**</span>

2. Fences can have acquire semantics, release semantics, or both. A fence with acquire semantics is called an *acquire fence*. A fence with release semantics is called a *release fence*.

3. A *release <span style="color: green;">**thread**</span> fence* A synchronizes with an *acquire <span style="color: green;">**thread**</span> fence* B if there exist atomic operations X and Y , both operating on some atomic object M, such that A is sequenced before X, X modifies M, Y is sequenced before B, and Y reads the value written by X or a value written by any side effect in the hypothetical release sequence X would head if it were a release operation.

4. A *release <span style="color: green;">**thread**</span> fence* A synchronizes with an atomic operation B that performs an acquire operation on an atomic object M if there exists an atomic operation X such that A is sequenced before X, X modifies M, and B reads the value written by X or a value written by any side effect in the hypothetical release sequence X would head if it were a release operation

5. An atomic operation A that is a release operation on an atomic object M synchronizes with an *acquire <span style="color: green;">**thread**</span> fence* B if there exists some atomic operation X on M such that X is sequenced before B and reads the value written by A or a value written by any side effect in the release sequence headed by A.

6. <span style="color: green;">**An evalution A on object O happens before another evaluation B on the same object if there exist object fences X and Y, both invoked with O as an operand, such that A is sequenced before X, Y is sequenced before B, and X would synchronize with Y if they were thread fences.**</span>


```c++
extern "C" void atomic_thread_fence(memory_order order) noexcept;

template<class... Objects>
void atomic_object_fence(memory_order order, Objects const&... objects) noexcept;
```

7. <span style="color: green;">**_Preconditions_: if it is an _object fence_, `order != memory_order::seq_cst`.**</span> <br>
8. _Effects_: Depending on the value of `order`, this operation: <br>
(8.1) — has no effects, if `order == memory_order::relaxed`; <br>
(8.2) — is an acquire fence, if `order == memory_order::acquire` or `order == memory_order::consume`; <br>
(8.3) — is a release fence, if `order == memory_order::release`; <br>
(8.4) — is both an acquire fence and a release fence, if `order == memory_order::acq_rel`; <br>
(8.5) — is a sequentially consistent acquire and release fence, if `order == memory_order::seq_cst`. <br>

## References

[flush] OpenMP, _flush construct_ https://www.openmp.org/spec-html/5.0/openmpsu96.html#x127-4920002.17.8

***FIXME*** OpenCL

***FIXME*** MPI


