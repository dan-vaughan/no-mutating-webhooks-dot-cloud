# No Mutating Webhooks dot cloud

A Polemic by a Tired Platform Engineer

If the road to hell is paved with good intentions, the road to an unusable and unmaintainable Kubernetes platform is paved with mutating webhooks.

This is just my take, your mileage may vary.

## Why not?

### Magic

Mutating webhooks introduce new and unexpected behaviour into the platform. Your users may be surprised when they `kubectl apply` a manifest from a tutorial and get a deployment with an extra initContainer, and extra sidecar and new entrypoint.

### Circular Dependencies

This is arguably the worst sin of all: when you rely on mutating webhooks to keep your extended controlplane functioning. This can lead to bootstrap problems, as well as DR problems. The worst thing is that you may not realise that you have got to this point until you have to recover a cluster.

### Debugging for Platform Teams

If you thought debugging Kubernetes was too easy for you, why not make it harder by adding mutating webhooks with their own logs, metrics and bugs? Even better, you could decide to use a language with a less well-written client library than Go, and introduce your platform team to the fun of debugging in a new language or programming paradigm.

### Debugging for End-Users

A bug in a mutating webhook can produce unreadable errors buried in child resources. Imagine if there's a runtime error in a mutating webhook for pods. A user could apply a `Deployment`, but if there's a problem with the mutating webhook they won't see any `Pods` created, and they'll have to look at the `Replicaset` events to understand what has happened.

### Version Compatibility

You've just introduced a new component which needs to have its Kubernetes client updated and patched, imposing a new overhead on your platform teams?

### Latency

Mutating webhooks can slow down the admission control process. Every time a resource is created or updated, the webhook intercepts the request and potentially modifies it. This additional processing can lead to latency, especially if the webhook service is slow or if there are multiple webhooks configured.

## What can I do instead?

### Provide better docs or training

This is a controversial answer, but many cases of the platform trying to do too much are a symptom of end-user developers who don't understand the platform they run on well enough. Provide the end-user with good, reusable patterns and an understanding of what Kubernetes offers them out of the box and you may not need the magic of mutating webhooks.#

### Library Helm Charts

These have their own drawbacks, but they at least allow you to interrogate the manifests that you apply to the cluster before you do. In a way, they can fulfil the same purpose of allowing you to inject extra functionality.

### Operators & CRDs

Instead of magically injecting data into a pod at start time, why not create a CRD that stores it in a configmap or a CSI Driver which dynamically adds it as a file? Both these methods have superior debuggability and user feedback than a mutating webhook, and won't slow down pod admission.