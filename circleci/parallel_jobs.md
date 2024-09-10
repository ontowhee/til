# Parallel jobs

Reference: [https://circleci.com/docs/parallelism-faster-jobs/](https://circleci.com/docs/parallelism-faster-jobs/)

Someone at $dayJob asked me a whether CircleCI runs parallelism using processes or threads. I was confused why they were asking about processes and threads, but I also wasn't sure how CircleCI manages its resources. So, I dug into the question, and this is what I found out.

In general, parallelism would utilize both threads and processes. However, if we're talking about concurrency vs. parallelism, then the answer is, CircleCI supports both!

A project at $dayJob had already been running jobs concurrently in CircleCI: one process running multiple threads, each thread running one job (a series of steps). However, the resources were configured such that we didn't see the jobs being interleaved much.

I looked into another project at $dayJob that had already been updated to run jobs in parallel on CircleCI. I was surprised to see that the CircleCI config file wasn't explicitly specifying the `parallelism` parameter. I'm guessing `-n 4` specified by pytest-xdist is able to tell CircleCI to create the appropriate number of processes/executors.

It's pretty amazing how much granularity can be configured with CircleCI. I'm glad the person asked me this question! It got me curious to dig in and learn about it, at least just enough to have a high level understanding.