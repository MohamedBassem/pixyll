# Docker at Trustious

In Trustious, our test suite runs on a single machine in about 12 hours. We used to parallelize the run on three machine. The run takes on average 4 hours. The results still weren't satisfying to us so we started building our in office test cluster.

*The image of the machines shelf being built.*

We added two more machines to our cluster but it didn't make a big difference. The problem is that running a single worker on each machine doesn't fully utilize the machine resources. We need a single machine parallelization. By parallelizing on a single machine you need to take care of database conflicts, elasticsearch conflicts and if you are running multiple spork instances then you'll also face ports conflicts. It's a headache handling all of these conflicts. We need some kind of isolation. We need those single works to act separately as if they are running on different hosts. We need a virtual machine. Virtualization will solve the conflicts problem perfectly but the problem is with the overhead added by the virtual machines on the servers. That's when we found about Docker. `Docker uses resource isolation features of the Linux kernel such as cgroups and kernel namespaces to allow independent "containers" to run within a single Linux instance, avoiding the overhead of starting virtual machines.` *From Wikipedia*. Docker is exactly what we were searching for so we started building our framework using docker. In this blog post we are going to talk about our distributed tests architecture.

## The Docker image
We started our own private docker registry on one of the cluster machines using this docker image (https://github.com/docker/docker-registry). Then we built our own docker image with the dependencies needed for the test run such as ruby, rails, elasticsearch, mongodb and the gems in our Gemfile.

## The Test Run
Commenting on a pull request with a special comment triggers a Jenkins build which starts the full tests run job on the master machine. The master machines spans other worker instances in the cluster based on a build matrix using SSH.

*The image of the build matrix*

The master machine runs a dry run to list all the tests we have and a nodejs server is started to queue the tests and listen for connections from the workers. On the other hand the workers pull the image from the local docker registry, starts mongo, ES and spork and contacts the server asking for tests to run one at a time. The master machines tails the connection output from the workers. When the tests queue is empty, the master machine parses the test results from every worker, merges them and extract failures. If the percentage of failures is under a certain threshold, failed tests are re-queued and the whole process starts again.

## Handling Worker Failures
If a worker failed for any reason, it is restarted by the master process and the test it was running will be discarded. At the end of a test run, if a test without a verdict is found, it is considered as a failure and re-queued for the next tests run.

## Monitoring Progress
The Jenkins build output tails the master process stdout which outputs the progress.

*The image of the server accepting connections*

## Reporting results
Results are reported back to the pull request as a comment from our Trustious bot listing failures - if any -.

## The complete process

*An image describing the complete workflow starting from jenkins till reporting output*

