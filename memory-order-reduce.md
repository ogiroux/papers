# A sub-relaxed memory order for reductions

Document #: TBD <br>
Date: 20-1-2022 <br>
Audience: SG1 <br>
Reply-to: dlustig@nvidia.com, ogiroux@nvidia.com <br>

## Abstract

Atomic operations that are not intended to synchronize should be allowed on unsequenced execution agents.

As an added benefit, moving reduction operations away from `memory_order_relaxed` reduces the potential harm that could come from strengthening `memory_order_relaxed` to solve the OOTA problem.

## Tony tables

| Before      | After |
| ----------- | ----------- |
| `a.fetch_add(1, memory_order_relaxed);`      | `a.fetch_add(1, memory_order_reduced);`       |
| (potentially synchronizing, invalid under `par_unseq`)   | (not synchronizing, valid under `par_unseq`)        |

## Revisions

R0
* This is the first revision

## Motivation

***FIXME*** Allowing lock-free non-synchronizing atomic operations to occur in unsequenced contexts

***FIXME*** Improving readability and performance by documenting lesser guarantees

***FIXME*** Reducing conflation of different use-cases for `memory_order_relaxed`

***FIXME*** Reifying `memory_order_relaxed` as having load-store ordering to solve OOTA

## Implementation

***FIXME*** Existing practice.

***FIXME*** Example: OpenMP reduction

***FIXME*** Example: atomic operations on execution agents with low progress guarantees

***FIXME*** Conservative implementation, simplify to memory_order_relaxed:

```c++
enum memory_order
{
    memory_order_seq_cst,
    memory_order_acq_rel,
    memory_order_acquire,
    memory_order_release,
    memory_order_relaxed,
    memory_order_reduced = memory_order_relaxed
};
```

## How to Teach

***FIXME*** An atomic operation that does not synchronize

***FIXME*** An atomic operation that is allowed in unsequenced context

## Proposed Wording

This wording is relative to N4901.

### ??.??.?? Sequential execution [intro.execution]

10. Except where noted, evaluations of operands of individual operators and of subexpressions of individual expressions are unsequenced.<br>
[_Note 5_: In an expression that is evaluated more than once during the execution of a program, unsequenced and indeterminately sequenced evaluations of its subexpressions need not be performed consistently in different evaluations. — _end note_]<br>
The value computations of the operands of an operator are sequenced before the value computation of the result of the operator. If a side effect on a memory location (6.7.1) is unsequenced relative to either another side effect on the same memory location or a value computation using the value of any object in the same memory location, and they are not potentially concurrent (6.9.2) <span style="color: green;">**or lock-free reduction atomic operations**</span>, the behavior is undefined.

### ??.?? Order and consistency [atomics.order]

```c++
namespace std {
  enum class memory_order : unspecified {
    reduced, //...
  };
  inline constexpr memory_order memory_order_reduced = memory_order::reduced;
}
```

1. The enumeration `memory_order` specifies the detailed regular (non-atomic) memory synchronization order as defined in 6.9.2 and may provide for operation ordering. Its enumerated values and their meanings are as follows:<br>
(1.1) — <span style="color: green;">**memory_order::reduced,**</span> `memory_order::relaxed`: no operation orders memory.<br>


2. <span style="color: green;">**An atomic operation performed with `memory_order_reduced` is a _reducing atomic operation_. Any other atomic operation is a _synchronizing atomic operations_.**</span>

### ??.?? Fences [atomics.fences]

1. This subclause introduces synchronization primitives called fences. Fences can have acquire semantics, release semantics, or both. A fence with acquire semantics is called an *acquire fence*. A fence with release semantics is called a *release fence*.

2. A *release fence* A synchronizes with an *acquire fence* B if there exist <span style="color: green;">**synchronizing**</span> atomic operations X and Y, both operating on some atomic object M, such that A is sequenced before X, X modifies M, Y is sequenced before B, and Y reads the value written by X or a value written by any side effect in the hypothetical release sequence X would head if it were a release operation.

3. A *release fence* A synchronizes with an atomic operation B that performs an acquire operation on an atomic object M if there exists a <span style="color: green;">**synchronizing**</span> atomic operation X such that A is sequenced before X, X modifies M, and B reads the value written by X or a value written by any side effect in the hypothetical release sequence X would head if it were a release operation

4. An atomic operation A that is a release operation on an atomic object M synchronizes with an *acquire fence* B if there exists some <span style="color: green;">**synchronizing**</span> atomic operation X on M such that X is sequenced before B and reads the value written by A or a value written by any side effect in the release sequence headed by A.

## References

[reduction] OpenMP, _reduction clauses_ https://www.openmp.org/spec-html/5.0/openmpsu107.html#x140-5800002.19.5

