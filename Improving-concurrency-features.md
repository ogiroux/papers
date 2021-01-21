**Paper:**
	PxxxxR0
**Author:**
	Olivier Giroux (ogiroux@nvidia.com)
**Audience:**
	SG1

# Improving C++20 concurrency features

**Revisions**

This is the initial revision.

**Introduction**

When we applied P1135R6 to C++20, we introduced several new concurrency constructs to the C++ concurrency library:

* In `<atomic>`, the member functions `wait`, `notify_one` and `notify_all` were added to class template `atomic<>` and class `atomic_flag`, and free function versions of the same also.
* In `<semaphore>`, the class template `counting_semaphore<>` and class `binary_semaphore` were introduced.
* In `<barrier>` and `<latch>`, the class template `barrier<>` and class `latch` were introduced.

Though each element included was long coming, and had much implementation experience behind it, fresh user feedback tells us that some improvements could still be made.

**Proposed direction**

The following is a grossly priority-ordered list of requests that users and implementers both have voiced over the last year:

1. Add timed versions of `atomic::wait`.

> The primary purpose of this facility is to make it easier to implement other concurrency facilities, but often these other facilities expose timed waiting facilities themselves. Without timed versions of `wait`, the programmer is left to ad-hoc solutions for timed waiting facilities, and perhaps even all waiting facilities. Anecdotally, at least two implementations of C++20 have added internal timed versions of this facility to implement `<semaphore>`.
>
> Adding timed versions of `atomic::wait` removes hurdles to adoption of this facility for its intended purpose.

2. Return the last observed value from `atomic::wait`.

> After the return from `wait`, it is common for programs to reload the value of the atomic object. By necessity, the implementation of `wait` already loaded this value, to compare it with the operand supplied and return non-spuriously. This is duplicate work which, in principle, could be optimized away by the compiler but conservatively isn't.
>
> Returning the value from `atomic::wait` is a straightforward way to recover performance lost from the duplicate work.

3. Avoid spurious polling in `atomic::wait` with ***at least one of***:

  a. Add an overload of `wait` taking a predicate instead of a value.

> When the program is waiting for a condition different from "not equal to", there is an added re-try loop around the `wait` operation in the program. This loop causes *each* call to `wait` to be performed as if it were the *first* call to `wait`, oblivious to the fact that the program has already been waiting for some time. This leads to re-executing the short-term polling strategy.
>
> Taking a predicate instead of a value allows us to push the program-defined condition inside of `atomic::wait`, delete the outer loop, and allows the implementation to track time spent.

  b. Add a hint operand to `wait` to steer the internal strategy.

> By default, that short-term strategy inside of `wait` is to poll the atomic object's value for some time, so as to avoid limiting the responsiveness of the program to that of the operating system kernel's scheduler. Sometimes, however, it is known that either (a) an event cannot or is not hoped to occur in this short of a window of time, or (b) the program has already supplied its own polling strategy before the call to `wait`, or (c) this call to `wait` is not the first and should be considered a long-term wait.
>
> Taking a hint would let the program indicate whether the short-term strategy of `atomic::wait` should execute or not.

4. Add timed versions of `barrier::wait` and `latch::wait` also.

> Since every waiting facility in the concurrency library has timed wait functions at this point, it makes sense to add timed versions of these as well.
>
> Although this is a very weak reason to do anything, there is also no clear reason why we should not do it.
