# Fantastic Pipelines and where to run them

## What is a pipeline

You pass some input, it does some magic and you get some output.

### What are fantastic pipelines

Fantastic pipelines are pipelines that don't deploy anything. An example of a good pipeline is some code that goes into a pipeline, and out goes an artifact (e.g. Docker image).

!!! quote "If you deploy with your pipeline, come see me afterwards"

## Where can you run them

Jenkins, Gitlab CI, GitHub Actions (they come in really slow, because that's what they are), BitBucket Pipelines, Azure DevOps (which is GitHub Actions before Microsoft bought it).

### So whats the problem

Basically he worked in an enterprise that used Jenkins and Bitbucket and Gitlab, the Gitlab was somewhere but no one really knew what was in there. Then they saw the light and moved to GitHub.
_And then all the pipelines broke._ So they migrated their actions and everything was fine. Sike, here comes the security officer. Long story short, they need to migrate away from GitHub Actions
because nobody knows where the code runs.

### The 2 major problems

- Developing pipelines can be hard
- Being at the mercy of people who don't understand IT is hard

## So lets fix it

What do we need: Git and Pipelines

Product selection: The company had already used ArgoCD. So the discussion boiled down to Argo Workflows or Tekton. 

!!! quote "Argo is really really awesome, so we went with Tekton"

Conclusion: Pipelines is just magic in the middle and magic in the middle can run anywhere. Everything runs in a container. We just need git separated from our pipeline logic.

_Here comes the rant_

Bla bla Mesosphere, every cool feature was an enterprise feature. On the other hand, there's Kubernetes and everything is free.

!!! quote "Kubernetes is 'free' to run, it just costs experience"

For the orchestration, we go to the OSS. For Git, we go straight to the enterprise monolith (e.g. GitHub and Gitlab).

_back to the pipelines_

Pipeline is just a combination of magic, called functions. Functions are isolated and repeatable. But where do you run the functions? Everywhere, as long as its on Kubernetes. Argo Worklfows
is really strong, and Tekton is also really strong.

One downside of putting the pipeline with the code is that you need to make a 100 "testing pipeline" commits to get it to work. Upside of running pipelines in Kubernetes is that you can test
locally and then push them to Git later. **Do not underestimate local development for pipelines.**

Now for the downsides of running pipelines yourself, it's yet another standard. It can be complex, but let's be honest, so was GitHub Actions the first time.

## Conclusion

When the security officer tells you to move your pipelines, make a sane decision and pick something that you can run anywhere. _Kubernetes can run anywhere btw_
