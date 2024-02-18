# Cooperative Scheduling
In orders async function to work properly it needs to yield from time to time. We should avoid compute intensive or IO
intensive tasks running in the async functions. In tokio we have **spawn_blocking** tasks. If you know that in scope of the
async function you need to run something heavym we can use it and it basically tells an executor that I am about to block, make sure the other tasks get to run. In this case executor will take some steps to reduce an impact of the problem.

# Select Order
By default select used a pseudo-random algorithm to poll arms, but some libraries f.e. like **futures** have additional macroses like
**select_biased** that respect the order of select branches.

# Fuse
Fuse is the way to say it is safe to poll the future (await a future) even if that future has already completed in the past.

# Async Performance
Async applications usually perfrom better since there is less of context switch and the don't allocate additional memory to spawn thousands threads.
