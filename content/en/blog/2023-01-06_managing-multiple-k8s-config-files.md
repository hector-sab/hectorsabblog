---
title: Managing multiple k8s config files
date: 2023-01-06T08:49:20Z
draft: false
categories:
- kubernetes
- developer-experience
tags:
- k8s
- shell
---

So it finally happened! I got myself a blog that's up and running! And I am inaugurating it with a small blog about how I make my life easier when dealing with multiple kubernetes clusters and having to connect to them.

# What is the problem?
Imagine you are asked to deploy some fancy service in the k8s (kubernetes) cluster `A` and given the kubeconfig for being able to connect to it. Later in the day you are asked to deploy another fancy service in the k8s cluster `B`, for which you are also given its respective kubeconfig. Next day, you are now asked to yet again deploy a different fancy service into cluster `C` and fix and issue on cluster `B`. Ah, from your boss you also heard that cluster `A` has been decomisioned so now you need to clean up your congis so the (now useless) information of cluster `A` doesn't stay around unecesarily.

Hopefully you see my point now. Over time it becomes tedius to deal with this config files, especially if the way you do it is through a single config file (a.k.a. `~/.kube/config`) as you need some way of adding, removing, and even sharing clusters details.

For the sake of completion, let's visualize the problem with some dummy kubeconfigs. The file below represents cluster `A`, and at this point we have already saved it to the default location.
```yaml
# ~/.kube/config
apiVersion: v1
kind: Config

clusters:
- cluster:
    server: https://k8s.example.org/k8s/clusters/example-a
  name: example-a

users:
- name: example-a
  user:
    client-certificate-data: certificate-data-example-a
    client-key-data: key-data-example-a

contexts:
- context:
  name: example-a
```

The file below represents cluster `B`, and it's saved in `$HOME`. Note that in this location `kubectl` is not aware of the file.
```yaml
# ~/example-b.yaml
apiVersion: v1
kind: Config

clusters:
- cluster:
    server: https://k8s.example.org/k8s/clusters/example-b
  name: example-b

users:
- name: example-b
  user:
    client-certificate-data: certificate-data-example-b
    client-key-data: key-data-example-b

contexts:
- context:
  name: example-b
```

# Our options

Now let's talk about what are our options here. So far, I have seen and used myself 3 ways of handling this problem, being the last one listed here the most effective for me.


## Replace default kubeconfig
The first way I dealt with this was as a cave man. I would move whatever kubeconfig I needed to `~/.kube/config`. So, with our example I would move `~/.kube/config` into `~/example-a` and then `~/example-b` back into `~/.kube/config`.

As you can imagine, the problem with this approach is that it is way too manual and prone to errors. A wrong move and say goodby to the one cluster config, or worse, now you have mixed up the clusters configs without being aware of it. It also adds a mental overhead of having to remember where all the files are backed up.

This is a big no.

## Merge files

The second option is to merge both files (and any subsequent one) into `~/.kube/config`. That would look like below. This is much nicer that the first option, now we can use something like [kubectx](https://github.com/ahmetb/kubectx) for switching clusters with ease. However, I still have two problems with it: 1) having to add new clusters info to the file, and 2) having to remove the info of one cluster when I no longer need it.

For the first problem I know there's some bash magic that can merge all new kubeconfig files into one, but I haven't really set that up. And for the second one, I am unaware of how to do the clean up in "one" click.

So, while this is a better option, it's still a no for me.
```yaml
# ~/.kube/config
apiVersion: v1
kind: Config

clusters:
- cluster:
    server: https://k8s.example.org/k8s/clusters/example-a
  name: example-a
- cluster:
    server: https://k8s.example.org/k8s/clusters/example-b
  name: example-b

users:
- name: example-a
  user:
    client-certificate-data: certificate-data-example-a
    client-key-data: key-data-example-a
- name: example-b
  user:
    client-certificate-data: certificate-data-example-b
    client-key-data: key-data-example-b

contexts:
- context:
  name: example-a
- context:
  name: example-b
```

## Edit the KUBECONFIG env var

This third way so far is what I use up to this day. It consists on modifying the env var `KUBECONFIG` and adding all the files that we require that `kubectl` to be aware of. Just this time not as a cave man.

The way that tools like `kubectl`/`kubectx`/`kubens` is that if no `KUBECONFIG` en var is defined they  read `~/.kube/config`. If defined, these tools will interpret the string value as a list of paths separated by a colon (`:`). What we need now to automate the loading of kube config is making an assumption and a few lines of bash in our `~/.bashrc`/`~/.zshrc`.

The assumption we make is that all the config files are going to be saved under the directory `~/.kube/configs/`; all the files in it will be set into `KUBECONFIG`. For setting up the variable I have two receips, one "clear" and other that is one-liner. Both result on having `KUBECONFIG=~/.kube/config:~/.kube/configs/example-a:~/.kube/configs/example-b`, you can choose whatever you prefer.

### Bash magic the clear way


```bash
CONFIGS_DIR=$HOME/.kube/configs
KUBECONFIG="$HOME/.kube/config"
for entry in "$CONFIGS_DIR"/*
do
  if [ -z "$KUBECONFIG" ]
  then
    KUBECONFIG="$entry"
  else
    KUBECONFIG="$KUBECONFIG:$entry"
  fi
done
export KUBECONFIG
```

### Bash magic the one-liner way

```shell
export KUBECONFIG="$HOME/.kube/config:"$(paste -sd ":" <(find ~/.kube/configs -type f))
```
