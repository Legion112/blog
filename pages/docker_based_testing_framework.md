# Docker based Integration testing
## Reason 
I have struggle for many years for developing features when there are multiple services involved (micro or services base arch).
To make sure there are working together I needed to run test which would check that services when interacting with each other do work as expected the following is list of my requirements: 
* I needed flexability to change ENV for each service and restart them then needed.
* I needed ability to see clear logs that my test have done. 
* I needed the ability to be able to use debugger on my tests and ability to add debugger to any developing services

# To achieve those I have come up with following ideas
I have phpunit tests which is running in container have access to `/var/run/docker.sock` and runs under my user PID and UID. This way I could interact with docker daemon and change any services freely withing test. Container is running withing same network which the testing servises are.

# What this give me?
1. I could execute any command in any services if I needed. For example there is command in one service which create admin and there are no way I could create such admin via api. So with docker daemon access I could execute any command for any service withing test. 
2. With access to docker daemon I could modify any service ENV variable restart them and set the desired state. 
3. I have the ability to change network configuration for example add extract services before test execution for example I could add mailhog to capture email which is sending application, or I could add s3 mock implementation and remove after test execution.


# Security consideration
1. Having access to `/var/run/docker.sock` is big security hole which allow to run any command on host potentially compromising code. To mitigate such there should be as low as possible external code to minimize the risk.

# Suggestion on docker socket
To mitigate risk of accessing docker daemon by mulition code withing our test. We could run `docker in docker`. The idea is that we create docker container which all our test container is gonna run so compromising `/var/run/docker.sock` won't effect host machine. 
This give as following dissandvatages.
* Potentially slower. 
* Two level of mapping to host files system if we want to modify code effectifly during development and run test
* Inside docker in docker container we need to run ssh deamon to be able to connnect docker viewing tools effectively. (Do not think it's a problem)
                                                                                                                                                   
# Potentially parallelism
Via docker API we could create multiple instances of our services running on different networks but connecting to different schemas of databases and virtual host. 
This potentially allow as to safely parallel test execution. having N instances of each service and shared virtualized storage we could run test without risk of one breaking due concurrency issues. If integration test start to run more 10 or something minutes having enouth hardware (MEMORY|CPU) would allow to get result withing small threshold, the slowest test would be our limit on getting result.
