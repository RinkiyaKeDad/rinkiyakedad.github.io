---
title: "Using Kyverno To Enforce EKS Best Practices"
read_time: true
date: 2021-06-21T20:12:05+05:30
tags:
  - kubernetes
  - devops
  - aws
---

Hey folks, in this post we’ll see how you can use [Kyverno](https://kyverno.io/) to enforce some best practices for your [EKS](https://aws.amazon.com/eks/) cluster. For those not familiar, Kyverno is a Kubernetes native policy engine that aims to make your life easy when managing clusters. To know more you can read my [previous post](https://dev.to/rinkiyakedad/kyverno-simplify-managing-k8s-clusters-2kej) on Kyverno where we discuss the project and its internals in detail. With that out of the way, let’s get started!

EKS best practices recommend the use of separate [IAM roles](https://docs.aws.amazon.com/IAM/latest/UserGuide/id_roles.html) for different use cases. For example, for dev and prod environments you should prefer to have separate IAM roles which can configure objects in those environments. Now the problem that arises with this is how do you make sure that the IAM role which has permission for the dev environment doesn’t accidentally create objects in the production environment? If you have the roles configured properly it would obviously not allow this to happen but with Kyverno not only can you fool-proof this but also make sure that if someone does try this, then it gets reported.

We are going to write a policy for Kyverno to check if the object being created in a namespace (dev or prod) has the right IAM role specified in its annotations or not. So let’s begin!

## Installing Kyverno

The simplest way to install Kyverno on your cluster is by running:

```
kubectl create -f https://raw.githubusercontent.com/kyverno/kyverno/main/definitions/release/install.yaml
```

If you visit the [Kyverno documentation](https://kyverno.io/docs/installation/) you’ll see ways to install it using helm and ways to customize your installation of Kyverno. 

## Creating A ConfigMap

Before we create the actual Kyverno policy let us first create a config map that will store the IAM roles for dev and prod environments. This can be done very simply using:

```
kind: ConfigMap
apiVersion: v1
metadata:
  name: ns-roles-dictionary
  namespace: kyverno
data:
  prod: "arn:aws:iam::123456789012:role/prod"
  dev: "arn:aws:iam::123456789012:role/dev"

```

## Creating The Kyverno Policy

Now to the fun part - creating our Kyverno policy. Let us first look at the policy YAML file and then I’ll walk you through it. If this is your first time seeing a Kyverno policy don’t worry at all because one of the most beautiful things about Kyverno is how intuitive it is. It follows a very similar structure to the YAML files for Kubernetes objects.

```
apiVersion: kyverno.io/v1
kind: ClusterPolicy
metadata:
  name: deployment-valid-role
  annotations:
    policies.kyverno.io/category: Security
    policies.kyverno.io/description: Rules to enforce valid roles, based on namespace-role dictionary
spec:
  validationFailureAction: enforce
  rules:
  - name: validate-role-annotation
    context:
      - name: ns-roles-dictionary
        configMap:
          name: ns-roles-dictionary
          namespace: kyverno
    match:
      resources:
        kinds:
        - Deployment
    preconditions:
    - key: "{{ request.object.metadata.namespace }}"
      operator: In
      value: ["prod", "dev"]
    - key: "{{ request.object.spec.template.metadata.annotations.\"iam.amazonaws.com/role\" }}"
      operator: NotEqual
      value: ""
    validate:
      message: "Annotation iam.amazonaws.com/role \"{{ request.object.spec.template.metadata.annotations.\"iam.amazonaws.com/role\" }}\" is not allowed for the \"{{ request.object.metadata.namespace }}\" namespace."
      deny:
        conditions:
        - key: "{{ request.object.spec.template.metadata.annotations.\"iam.amazonaws.com/role\" }}"
          operator: NotIn
          value:  "{{ \"ns-roles-dictionary\".data.\"{{ request.object.metadata.namespace }}\" }}"

```

Not as scary as you expected, right? Let’s go through it line by line now :)

Let’s begin from the spec section since everything prior to that is all the standard boilerplate stuff. The first thing in the spec section is `validationFaliureAction` which we have set to `enforce`. What this does is specify what should happen if this policy is violated by an object. Setting it to `enforce` would make sure that the object which violates this policy does not get created whereas setting it to `audit` will allow the object to get created but will report that violation.

Each Kyverno policy must have at least one rule which is what we define next. After specifying the name of the rule we specify the `configMap` we created earlier. This is done using the `context` key. We do this because it allows us to easily refer to values from this `configMap` in our Kyverno policy, as you will see later.

Next up we specify the Kubernetes objects we want this policy to act on under the `resources` key. Kyverno offers a lot of control over here. For example, you can choose deployments but add an exclude block which would tell Kyverno to not include some specific Deployments. Pretty cool right?

```
match:
  resources:
    kinds:
    - Deployment
exclude:
  clusterroles:
  - cluster-admin
```

This example matches all Deployments excluding those created using the cluster-admin ClusterRole.

After specifying what resources we want the policy to act on, we specify some `preconditions`. Think of them as custom filters giving you more control over when the policy should be applied on the Kubernetes objects you select using the Match and Exclude blocks.
The first precondition simply states that the policy should not be applied if the deployment is being created in a namespace other than `dev` or `prod`. And the second precondition says that if the deployment object simply has no annotation mentioning an IAM role, then too the policy must not be applied. You may want to remove this second precondition based on your particular use case. 

Once we have fine-grain control over what objects this policy would get applied to, we specify what the policy should do. Kyverno policies can [mutate, validate or generate](https://kyverno.io/docs/kyverno-policies/) Kubernetes objects. This policy is going to validate the IAM roles so we specify the `validate` block. In the `message` key we specify what message should be shown in case an object violates the policy. The next lines of the policy simply say that the request should be denied if the `key` is not present in the `value`, that is if the value of the “iam.amazonaws.com/role” annotation in the selected deployment does not correspond to the correct value of the namespace present in our config map. 

And this is it. Yes, it is this simple to create policies in Kyverno! I hope this post was able to show you how powerful yet easy to use Kyverno is. If you’re interested in knowing more, do check out the [Kyverno documentation](https://kyverno.io/docs/). If you have any doubts or feedback feel free to join #kyverno channel on the Kubernetes Slack to talk to the maintainers and other community members!

Thanks for reading :)

> If you’re interested in a similar tutorial but for GKE, you can check out [this](https://cloud.google.com/community/tutorials/restrict-workload-identity-with-kyverno) post. 
