# kubectl-daemons

[![Lint](https://github.com/jaymzh/kubectl-daemons/actions/workflows/lint.yml/badge.svg?branch=main&event=push)](https://github.com/jaymzh/kubectl-daemons/actions/workflows/lint.yml)
[![Build](https://github.com/jaymzh/kubectl-daemons/actions/workflows/build.yml/badge.svg?branch=main&event=push)](https://github.com/jaymzh/kubectl-daemons/actions/workflows/build.yml)

## Intro

kubectl-daemons is a kubectl plugin to help with daemonset work. When running a
kubernetes infrastructure, it is common to do lots of work with deamonsets, but
the tools are optimized for other workflows where node names aren't
particularly important.. With daemonsets, you often know a node and a DS you
care about, and this plugin lets you focus on that.

Previously you'd do things like:

```shell
kubectl get pods -o wide | grep <daemonset> | grep <node>
kubectl describe pod <pod_from_above>
kubectl delete pod <pod_from_above>
kubectl get pods -o wide | grep <daemonset> | grep <node>
...
```

Or maybe:

```shell
kubectl get pods --field-selector spec.nodeName=<node>
kubectl describe pod <pod_from above>
kubectl delete pod <pod_from above>
kubectl get pods ---field-selector spec.nodeName=<node>
...
```

No more using a long `get` command to find the pod name,
and having to keep track of it! Just specify the DS and
the node:

```shell
kubectl d describe <daemonset> -N <node>
kubectl d delete <daemonset> -N <node>
kubectl d logs <daemonset> -N <node>
```

## General Usage

Get pods from a daemonset:

```bash
kubectl d get <daemonset>
```

Or get the pod from a daemonset on a specific node

```bash
kubectl d get <daemonset> -N <node>
```

Or get all daemonset-related pods on a node.  I think of this as the equivalent
of asking systemd to list all services on a node. :)

```bash
kubectl d get -N <nodename>
```

Or you can delete the pod from a daemonset on a specific node

```bash
kubectl d delete <daemonset> -N <nodename>
```

You can do logs as well:

```bash
kubectl d logs <daemonset> -N <node>
```

NOTE: -N is required here.

You can describe pods:

```bash
kubectl d describe <daemonset> [-N <node>] # node optional
```

You can even exec:

```bash
kubectl d exec <daemonset> -N <node> -- echo "Hello world"
```

Or interactively:

```bash
kubectl d exec <daemonset> -N <node> -it -- /bin/bash
```

And you can list all daemonsets on a node:

```bash
kubectl d list <node>
```

## Installing

The easiest way to install, right now, is to grab the right build from our
[releases](https://github.com/jaymzh/kubectl-daemons/releases) page, and drop
the binary (`kubectl-d`) in your PATH.

If you get an error like "macOS cannot verify this app..." see [this page](
https://zaiste.net/os/macos/howtos/resolve-macos-cannot-be-opened-because-the-developer-cannot-be-verified-error/).

We've submitted this plugin to Kraw, and will update these installation
instructions once that's merged.

## Building from source

The quick-n-easy way to build is:

```shell
go build -o kubectl-d
```

Then stick `kubectl-d` somewhere in your path.

## Thanks

A huge thanks to Benjamin Muschko's [Writing your first kubectl
plugin](https://bmuschko.com/blog/writing-your-first-kubectl-plugin/) blog post
and associated [GH repo](https://github.com/bmuschko/kubectl-server-version).
This was invaluable in getting me up and running.

## FAQ

**Why not just write a simple shell wrapper?**

For many reasons. First, I had that, and it's quite slow. You end up doing more
queries than you need, and if your API servers are on the other side of
privatelinks, it can get quite slow.

Second, I wanted a good excuse to learn some golang and get better at Kube
internals.

**Aren't there already other plugins that do this?**

Not that I could find!

**Can I contribute?**

Sure, send a pull request!

**Why did you end up re-implementing so much kubectl formatting code?**

Trust me, I didn't want to. Unfortunately kubectl plugins aren't really
"plugins". They're standalone binaries that `kubectl` executes for you. So you
can use whatever is in the k8s libraries, but various things kubectl does like
formatting `describe` output, or various calculated fields, are note exposed as
functions other code can call.

Ideally, plugins would be a library that could hook into various stages within
kubectl and allow you to not have to implement so much yourself, but it turns
out that's not how kubectl plugins work.

**Will you be submitting this to Krew?**

Yes! That'll happen very soon.
