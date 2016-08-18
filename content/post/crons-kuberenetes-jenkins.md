+++
author = "John_Behling"
comments = true
date = "2016-08-18T15:27:31-05:00"
draft = false
image = "images/solartech.jpg"
menu = ""
share = true
slug = "crons-kubernetes-jenkins"
tags = ["crons","kubernetes","jenkins"]
title = "Teaching old crons new tricks with Jenkins and Kubernetes"

+++

# Teaching Old Crons New Tricks with Jenkins and Kubernetes
Field Nation is rapidly moving from a monolithic architecture to micro-services. To accomplish this on the infrastructure side we're utilizing [Kuberenetes](http://kubernetes.io). Shifting to the container mindset has been easier for some aspects of our app than others. One place, where some sys-ops Jiu Jitsu was required was on the running of cron jobs.

## Containers vs. Mystery
One thing I love about containers is how they create clean single-function servers. This is the way servers truly should be: a stripped-down engine on which to run an app, no mystery, no drama, just the app. There's nothing worse than turning off a server that you thought noone was using, and unleashing a fury of angry emails. Those stories generally end with some version of "Well there's this cron job on that server that Somebody set up, we don't really know what it does but.. clearly it's important"

Containers provide a possible solution to that mystery. However, we still need cron-like tasks. So how do we do that in this brave-new world of stateless single-function containers?

One possibility is to run the cron as a Kubernetes batch [job](http://kubernetes.io/docs/user-guide/jobs/). In that case the Kubernetes scheduler would create a single ephemeral instance to run the script, afterwards the container would go away. A downside to that approach is it creates a lot of container churn (some of these crons run every few minutes). What I'd much rather do is utilize the existing fleet of containers, and simply choose one to execute the script on. We can accomplish this via `kubectl exec`. Mirroring the semantics of `docker exec`, `kubectl exec` will run a command in a single Kubernetes pod, and return the exit status of that command.


## Jenkins to the rescue
Now that we've got a solution for execution, we need a scheduler. As we've been using Jenkins to trigger our Kubernetes deploys and rolling updates, we already have a template for executing `kubectl` commands against the kubernetes cluster. All we need to do is figure out the guts of the command, and feed Jenkins the interval to run it at.

Recently I've become a big fan of the Jenkins Job DSL plugin. With a little bit of groovy and some data you can generate hundreds of jobs, and forever escape the tedium of manually creating jobs.

So to solve our problem:

* The Jenkins Job DSL would generate a job per cron
* The Job would select a container for the pool of healthy instances, and run the cron reporting failure back to Jenkins
* Jenkins provides an audit trail to trace back every execution, and trigger alerts on failure.
The end result being full visibility into what our cron-like tasks are doing, when they're running, if they're failing etc. Let's look at some code.

## To the code!

First off we need to grab the podname. We use pod labels to group containers by application and environment.

```kubectl get pods -l app=superapp,env=dev -o json```

this gives us the full json representation of the pod including a lot of data we don't need, so I pipe that output to the always-handy [jq](https://stedolan.github.io/jq/) to extract just the name

```kubectl get pods -l app=superapp,env=dev -o json | jq --raw-output '.items[].metadata.name'```

closer, now we have a list of pod names. I could stop there and just take the first value, but it would be most useful to grab one from the list at random. This way the same container isn't always being tasked to execute the cron.

```kubectl get pods -l app=superapp,env=dev -o json | jq --raw-output '.items[].metadata.name | shuf | head -1'```

There we are, a randomly selected container. Now we need to execute the actual cron task. The Jenkins job DSL will handle plugging in the cron command, and setting up the correct frequency.

In the Jenkins DSL job we start with an object to represent all of our crons.
```
crons = [
  [ name: "cron1", freq: "H/15 * * * *", cmd: "/usr/local/bin/php /var/www/scripts/script1.php"],
  [ name: "cron2", freq: "H/15 * * * *", cmd: "/usr/local/bin/php /var/www/scripts/script2.php"]
  ...
  ]
```

Then we simply iterate over the list making a job per cron.

```
freeStyleJob("fieldnation-crons/${app_cron.name}") {
  label('k8s-slave')
  logRotator(2, -1, -1, -1)
  triggers {
    cron(app_cron.freq)
  }
  steps {
      shell('echo "Setting up K8S Workspace"')
      ...
      shell("kubectl get pods -l app=superapp1,environment=${environment} -o json | jq --raw-output '.items[].metadata.name|shuf|head -1' > POD")
      shell("kubectl exec `cat POD` -c appserver ${app_cron.cmd}")
  }
}
```

The end result: our trusty Jenkins provides centralized control over our periodic tasks, and we maintain the high degree of visibility into both the content of our containers and what they're doing.
