## `OSU_TESTS_FORCED_GC`

When running tests in a grouped fashion, unobserved exceptions will be thrown at the next garbage collection (as they are handled in the finalizer of related `Task`s). This can make it very hard to track down issues stemming from unobserved exceptions. Setting this variable to `1` will force a garbage collection after each test method, increasing the chances of unobserved exceptions firing against their related tests.

Note that this adds a considerable overhead to test runs, especially on projects with large allocations. For osu! we see an overhead of around 80-100%. For this reason, it is disabled by default.

## `OSU_TESTS_NO_TIMEOUT`

By default, `AddUntilStep` has a timeout to ensure tests don't run forever. Settings this variable to `1` will disable this timeout. This is useful when diagnosing deadlock scenarios, or to allow ample time to attach a debugger and check on the current state of the game in a fail case.

## `OSU_EXECUTION_MODE`

Force an execution mode for test runs. Valid values are `SingleThread` and `MultiThreaded`. Default is `MultiThreaded`.
