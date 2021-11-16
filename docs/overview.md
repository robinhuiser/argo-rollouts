# Overview

[Back to README](../README.md)

* A Kubernetes controller and set of CRDs which provide advanced deployment capabilities such as
  * blue-green, canary, 
  * canary analysis, 
  * experimentation, 
  * progressive delivery
* Integrates with ingress controllers and service meshes
* See [demo](https://youtu.be/hIL0E2gLkf8)

## What problems are solved?

  * Few controls over the speed of the rollout
  * Inability to control traffic flow to the new version
  * Readiness probes are unsuitable for deeper, stress, or one-time checks
  * No ability to query external metrics to verify an update
  * Can halt the progression, but unable to automatically abort and rollback the update

> In large scale high-volume production environments, a rolling update is often considered too risky of an update procedure since it provides no control over the blast radius, may rollout too aggressively, and provides no automated rollback upon failures.

## Controller features

* Blue-Green update strategy
* Canary update strategy
* Fine-grained, weighted traffic shifting
* Automated rollbacks and promotions
* Manual judgement
* Customizable metric queries and analysis of business KPIs
* Ingress controller integration: NGINX, ALB
* Service Mesh integration: Istio, Linkerd, SMI
* Simultaneous usage of multiple providers: SMI + NGINX, Istio + ALB, etc.
* Metric provider integration: Prometheus, Wavefront, Kayenta, Web, Kubernetes Jobs, Datadog, New Relic, Graphite

## How does it work?

* Manages the creation, scaling, and deletion of ReplicaSets
* These ReplicaSets are defined by the `spec.template` field inside the Rollout resource, which uses the same pod template as the deployment object
* When the `spec.template` is changed, that signals to the Argo Rollouts controller that a new ReplicaSet will be introduced
* The controller will use the strategy set within the `spec.strategy` field in order to determine how the rollout will progress from the old ReplicaSet to the new ReplicaSet
* Once that new ReplicaSet is scaled up (and optionally passes an [Analysis](https://argoproj.github.io/argo-rollouts/features/analysis/)), the controller will mark it as "stable"

> If another change occurs in the `spec.template` during a transition from a stable ReplicaSet to a new ReplicaSet (i.e. you change the application version in the middle of a rollout), then the previously new ReplicaSet will be scaled down, and the controller will try to progress the ReplicasSet that reflects the updated `spec.template` field.

## Use cases

* **A user wants to run last-minute functional tests on the new version before it starts to serve production traffic**. 
  * With the BlueGreen strategy, Argo Rollouts allows users to specify a preview service and an active service. The Rollout will configure the preview service to send traffic to the new version while the active service continues to receive production traffic. Once a user is satisfied, they can promote the preview service to be the new active service. ([example](https://github.com/argoproj/argo-rollouts/blob/master/examples/rollout-bluegreen.yaml))
* **Before a new version starts receiving live traffic, a generic set of steps need to be executed beforehand**. 
  * With the BlueGreen Strategy, the user can bring up the new version without it receiving traffic from the active service. Once those steps finish executing, the rollout can cut over traffic to the new version
* **A user wants to give a small percentage of the production traffic to a new version of their application for a couple of hours.** 
  * Afterward, they want to scale down the new version and look at some metrics to determine if the new version is performant compared to the old version. Then they will decide if they want to roll out the new version for all of the production traffic or stick with the current version. With the canary strategy, the rollout can scale up a ReplicaSet with the new version to receive a specified percentage of traffic, wait for a specified amount of time, set the percentage back to 0, and then wait to rollout out to service all of the traffic once the user is satisfied. ([example](https://github.com/argoproj/argo-rollouts/blob/master/examples/rollout-analysis-step.yaml))
* **A user wants to slowly give the new version more production traffic**. 
  * They start by giving it a small percentage of the live traffic and wait a while before giving the new version more traffic. Eventually, the new version will receive all the production traffic. With the canary strategy, the user specifies the percentages they want the new version to receive and the amount of time to wait between percentages. ([example](https://github.com/argoproj/argo-rollouts/blob/master/examples/rollout-canary.yaml))
* **A user wants to use the normal Rolling Update strategy from the deployment**. 
  * If a user uses the canary strategy with no steps, the rollout will use the max surge and max unavailable values to roll to the new version. ([example](https://github.com/argoproj/argo-rollouts/blob/master/examples/rollout-rolling-update.yaml))

