
---
date : "2021-12-16"
draft : false
share : true
title : "What software engineer needs to do in 1-2 hours?"
slug : "2-hours-of-engineering"
tags : []
banner : ""
aliases : ['/2-hours-of-engineering/']
---

## Introduction

What can software engineer do in 2 hours? Can you make MVP and deploy it to production in that time? What about a senior software engineer? Recently I applied for a developer position. I did it after a personal suggestion from that company's engineer. I received a fairly simple but not easy task to do and I could do it in any programming language I wanted. I will not mention company's name or any of the people involved. But, my experience could make an interesting read. Also, the task specifics are changed a little bit for anonymity’s sake.

## Rules of the "homework assignment"

Assignment consists of two tasks. First task is about coding. Second task is about packaging the same application in docker container and deploying it on kubernetes. Time constraint to do these tasks is 1-2 hours and a there is explicit mention not to try to make everything ideal. If you haven't encountered time constraints on interview tasks - usually its to prevent candidates from polishing things too much. It leads to wasting candidate's time on non-important things. Now seeing this tight time constraint I knew immediately that I will have to make huge tradeoffs.

## Specifics

First task is to make a Http server with one endpoint. As I mentioned before, I will change endpoint functionality to a similar one. Endpoint task is to count all multiples of 3 or 5 under given `n`. (Taken from first task of Project Euler [1]) Example: localhost:9999/sum_multiples_3_5?n=10. Http endpoint should answer 23, because if we list all the natural numbers below 10 that are multiples of 3 or 5, we get 3, 5, 6 and 9. 3+5+6+9=23.

Now task mentions that Http server should handle a good RPM and prevent malicious attempts which would bring service down. The higher the N is calculated - the more points you get for the task. Tests are suggested, but left optional.

Second task requires the application to be packaged with Dockerfile. Also there needs to be kubernetes YAML file to spin up this image on kube. It also requests an ability to scale up or down, automatic scaling is optional. And it requires to self-heal in case the service goes down.

## Considerations

There are a lot of design decisions to consider and evaluate possible architecture choices:

- Since this endpoint is computationally expensive, it is almost a rule to cache responses for given n
- Computations can be memoized, for example if we count n=10 and later count n=13, do we count again from n=1 or can we use n=10 cache
- Should cache be shareable between kubernetes instances?
- Can memoized calculations be shared between kubernetes instances?
- How long the timeout should be for the user? Does application continue computations so users in the future can request the number even though it timeouted before?
- What algorithm to pick so resources uses are optimal?
- Should http server threads and computational threads run on separate thread pools?
- Should there be a hardcoded max n? What if better server hardware can handle bigger n easily?
- What kind of data types response needs? Does big fn(very_big_n) response fit in Int, BigInt?
- What if users maliciously request expensive computation in parallel? (DOS attack)
- How can algorithm effectively use memory available on the server?
- How do we benchmark the endpoint if we use cache? Can we use static requests?

On top of these design decisions there needs to be validation for query params. Best

would be if application guides the user to use correct params with correct types. For example: localhost:9999/sum_multiples_3_5?y=cat should provide informative message.

Also since I chose to write in Scala, it makes most sense for hiring managers to expect me to write functional Scala.

Of course, all this thinking falls under given time constraint of 1-2 hours.

## Action list

While there are many things to consider, it makes most sense to rank and execute the required non-optional parts of the tasks. Fulfilling requirements under time constraint is what defines success here. Sensible mini-tasks list I chose to do in order:

- Start writing README file to capture some of the considerations and decisions that went into solution
- Code function to calculate sum_multiples_of_3_5
- Rewrite the function to be tail recursive
- Scaffold http server with the endpoint
- Validation for the endpoint
- Connect function and http server to make a naive solution
- Cache with a Ref[Map[Int, BigInt]], where cache is a Map and Ref is there to deal with concurrency
- Apache benchmark with static N to test the throughput
- Wrap sum_of_multiples function in effect, add a timeout of constant 30 seconds. This means if it exceeds that time, user will receive error "Timeout of 30 secs exceeded" with Http code 400
- Benchmark with higher number to test the timeout
- Capture benchmark results into README file, write how big of N I tested application with
- I chose not to write tests as they are optional and there is not enough time for them now. I deemed benchmarks more important as task specifically mentions higher points for better performance

Http server works, in the end I managed to make it around 100 LoC. It is possible to run it and curl to get the result as per task requirements.

Second part is about packaging in docker and spinning it up on k8s. Sensible mini-tasks lists I chose to do:

- Since task mentions self-contained image with Dockerfile, I could not use "industry standard" sbt-native-package, I bootstrapped dockerfile with scala-sbt base image
    - Set SBT_OPTS for memory with ENV in the file
    - Copy whole project into docker image
    - Run sbt assembly
    - Change base image to JDK
    - Expose http server port
    - Run the jar file containing http-server
- Test image locally with docker run -p $PORT and see if it works
- Kubernetes part of deployment and service
    - Deployment of the image with livelinessProbe on httpGet /sum_multiples_3_5?n=10 to meet self-heal requirement
    - Service to forward port 80 to container's exposed port
- Test kubernetes deployment on minikube, benchmark, make pod go down and see if it heals
- Benchmark kubernetes deployment and compare to local sbt deployment, see if results differ

I did not do automatic pod scaling as I never used it before. There was no time to research and test, but since it was optional - I de-prioritised it.

## Feedback from company and conclusion

I get an email from company saying: I did not write tests, did not implement autoscaling (hpa) and timeout requests should not return bad request status code. Thus, I am rejected and I can apply again in X months. No mention of how well my application performed in comparison or if the design choices made were correct. I personally would consider those things as priorities.

I am quite disappointed by this experience. My choice with status code makes sense to me as timeout should not be 200. As for the optional things, I chose specifically to prioritize non-optional stuff first due this insane time constraint. It seems unlikely that all this can be considered and done in 1-2 hours.

Tests you can always find other projects with tests I wrote on Github or just ask me separately to provide some code examples with tests. Or even give me more time on this task beyond initial time constraint. I specifically mentioned in the notes that I chose benchmarking priority over testing due not enough time having for both.

Maybe reviewer's priorities were different. Some developers have weird priorities – spend time making something perfect before application even has users. I personally try to think lean – better ship something before the deadline than go bankrupt. I thought this time constraint was a test of its own to see what would I do. Maybe they expected me to over-extend the initial time constraint and just said 1-2hrs to get me to commit to this assignment. 

Either way, I think main lesson for me is that you need to ignore given time constraints for homework assignments. And over-engineer everything to perfection to show off your skills.

What do you think?