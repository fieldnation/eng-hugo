+++
author = "Tim_Mader-Brown"
comments = true
date = "2020-08-12"
draft = false
image = "images/adapt.jpeg"
menu = ""
share = true
slug = "dynamic-code-coverage"
tags = ["Field Nation", "test automation", "unit testing", "code coverage", "jest"]
title = "Keeping Up with Unit Testing"

+++
**Finally a topic deemed worthy of being part of the Field Nation Engineering blog!**
Unit and Integration tests are an integral part of maintaining quality in software engineering.  So, it's not surprising that many software companies try their best to enforce some version of the software testing pyramid.  As you can see below, Unit and Integration tests are the base for all forms of testing.

![testing pyrimid](/images/automation-pyramid-1.png)

**So, what's the problem?**

*“I just wrote the most amazing feature ever and it works perfectly, it doesn’t need unit tests”*

The problem is that its human nature to forget, be lazy, be over- confident, be focused on completing the story, or miss stuff in code-reviews.  Maybe the developer is tired of all the work it took to develop a feature, and now they're too exhausted to figure out how to write unit tests.

We are solving this problem programmatically.  Rather than relying on code reviews to make sure there is good unit test coverage, we are enforcing code coverage using jest’s `--coverage` feature.

The feature is very simple to use, just run the jest tests with the `--coverage` parameter passed.  You should see a table with something like this on top:
![jest coverage](/images/coverage.png)

Take those values and add them to your jest.config.js:
![jest config](/images/jestConfig.png)

That takes care of code coverage at this single point in time, but what about when we add new features?  We found a solution that is a combination of 3 different tools:

* Husky
  * Husky is a very popular node_module that can make things happen automatically at commit or at push

* jest-it-up
  * Jest-it-up is a less popular node_module that will analyze the difference between the current jest threshold and the new threshold

*Shell script
  * (We all know what a shell script is)[https://lmgtfy.com/?q=shell+script]

*What did we do?*

1. Add husky and jest-it-up to the devDependancies in the package.json
1. Add a husky configuration that looks like this
![husky file](/images/husky.png)
1. Add a ‘posttest’ script in package.json that looks like this 
        
        "posttest": "jest-it-up",
1. Create a shell script called commit_threshold_updates.sh that looks like this
```
#!/usr/bin/env bash
git update-index -q --refresh
CHANGED=$(git diff jest.config.js)
if [ -n "$CHANGED" ]; then
 git add jest.config.js
 git commit -m "updating threshold" --no-verify
fi
```

With all of this in place, every time a dev goes to push their code it will trigger the unit tests to run.  If the threshold isn’t met, then the push will fail.  If the threshold is higher than jest-it-up, it will update the config and the shell script will make a new commit with that update.

Eventually, this should naturally bring the code coverage values up. Once we get to an agreed-upon value (perhaps 90% or 95%) of our top threshold, we can remove the dynamic section and just use jest’s threshold value.  
