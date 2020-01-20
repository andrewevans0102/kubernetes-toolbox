# minikube

This is walkthrough of using minikube for local development. This is a modified version of the [official K8s minikube docs here](https://kubernetes.io/docs/tasks/tools/install-minikube/).

Please note that in the bottom sections I have sections covering `registry` and `local` image deployments. The reason this is important is because it determines how you want to use minikube. There are reasons to try both. Here are the differences

- `registry image` = minikube will pull the image from a registry and then run it on a local clust on your machine
- `local image` = minikube will use the docker daemon running inside the VM to build and deploy images you've built locally

## minikube setup

The steps below are specifically for a Mac, but for other platforms please consult the [official K8s minikube docs here](https://kubernetes.io/docs/tasks/tools/install-minikube/).

First, you'll need to verify your machine supports virtualization (minikube operats inside a VM). Please run the following:

```bash
sysctl -a | grep -E --color 'machdep.cpu.features|VMX'
```

You should see something like the following as output:

```bash
machdep.cpu.features: FPU VME DE PSE TSC MSR PAE MCE CX8 APIC SEP MTRR PGE MCA CMOV PAT PSE36 CLFSH DS ACPI MMX FXSR SSE SSE2 SS HTT TM PBE SSE3 PCLMULQDQ DTES64 MON DSCPL VMX SMX EST TM2 SSSE3 FMA CX16 TPR PDCM SSE4.1 SSE4.2 x2APIC MOVBE POPCNT AES PCID XSAVE OSXSAVE SEGLIM64 TSCTMR AVX1.0 RDRAND F16C
```

If you see `VMX` listed then you should be good. Otherwise, please consult the above K8s docs links in the first section for ways to fix this.

Next, install minikube with `homebrew` with the following:

```bash
brew install minikube
```

When that completes, you can start up your minikube with:

```bash
minikube start
```

At anytime you can verify your minikube running with:

```bash
minikube status
```

When it is running succcessfully `minikube status` should return the following:

```bash
    ➜  ~ minikube status
    host: Running
    kubelet: Running
    apiserver: Running
    kubeconfig: Configured
```

Now you can run containerized applications in deployments on your minikube.

## deploy a registry image

In all of these steps, I've created npm scripts. This simplifies all of the terminal commands, and you just have to run `npm run <command_name>`.

To deploy an image to your local minikube that is pulled from a docker registry please do the following:

1. Run the following command (note the `--image` value is a registry image):

```bash
minikube kubectl create deployment hello-minikube --image=k8s.gcr.io/echoserver:1.1
```

2. Run the following command to expose the deployment as a service:

```bash
kubectl expose deployment hello-minikube --type=NodePort --port=8080
```

3. Run `kubectl get pod` and you should see something like the following:

```bash

➜  ~ kubectl get pod
NAME                              READY   STATUS    RESTARTS   AGE
hello-minikube-797f975945-8v6vd   1/1     Running   0          88s
```

4. Run `minikube service hello-minikube --url` to get the url of the service that is running on your local cluster.
5. Do a `curl` request against the URL you got from step 4. This should result in the following:

````bash

➜  ~ curl http://192.168.64.2:31182


Hostname: hello-minikube-797f975945-8v6vd

Pod Information:
        -no pod information available-

Server values:
        server_version=nginx: 1.13.3 - lua: 10008

Request Information:
        client_address=172.17.0.1
        method=GET
        real path=/
        query=
        request_version=1.1
        request_scheme=http
        request_uri=http://192.168.64.2:8080/

Request Headers:
        accept=*/*
        host=192.168.64.2:31182
        user-agent=curl/7.64.1

Request Body:
        -no body in request-
        ```
````

## deploy a locally built image

Locally built images provide you with the ability to quickly test things. Consult the official [minikube README for more info on this here](https://github.com/kubernetes/minikube/blob/0c616a6b42b28a1aab8397f5a9061f8ebbd9f3d9/README.md#reusing-the-docker-daemon).

To do this, please do the following:

1. star minikube with the following:

```bash
minikube start
```

2. Next, set the daemon in your minikube instance to match the docker daemon running on your system. Using the command here, enables you to build images with the deamon and local registry inside minikube. Then you can use those images to deploy to your single node clusters. To do this just run the following:

```bash
eval $(minikube docker-env)
```

3. Build your image with the following:

```bash
docker build -t helloworld-minikube .
```

4. Creat a deployment with the image you built locally:

```bash
kubectl run helloworld-minikube --image=helloworld-minikube:latest --image-pull-policy=Never
```

> Notice here that we are setting the `image-pull-policy` to `Never` so that the deamon will not attempt to pull the image from a registry.

5. Do a `kubectl get pods` to verify that the container is running

6. Expose the deployment as a service with the following:

```bash
kubectl expose deployment helloworld-minikube --type=NodePort --port=8080
```

7. Get the URL of the service you've deployed with the following

```bash
minikube service helloworld-minikube --url
```

8. Do a curl request to the URL that was returned with step 7 and you should see something like the following:

```bash
➜  minikube git:(master) ✗ minikube service helloworld-minikube --url
http://192.168.64.6:32100
➜  minikube git:(master) ✗ curl http://192.168.64.6:32100
Hello World from your local minikube!%
```

## Cleanup Deployment

To delete a deployment and "cleanup" your resources to do the following:

1. First delete the service with:

```bash
kubectl delete services helloworld-minikube
```

2. Next delete the deployment with:

```bash
kubectl delete deployment helloworld-minikube
```

3. Finally shutdown minikube with:

```bash
minikube delete
```

## Final Thoughts

If you see any issues here please open a PR. Otherwise, feel free to reach out to me on Twitter at [@AndrewEvans0102](https://www.twitter.com/andrewevans0102).
