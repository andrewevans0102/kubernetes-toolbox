# google-cloud

This is modified version of the [Google Cloud Quickstart Document here](https://cloud.google.com/kubernetes-engine/docs/quickstarts/deploying-a-language-specific-app).

This project deploys a Node "Hello World" app on Google Cloud Kubernetes Engine.

# Google Cloud Console Setup

You'll need to create a project in Google Cloud and [enable billing](https://cloud.google.com/billing/docs/how-to/modify-project). This project will not use many resources and [should be able to be free](https://cloud.google.com/free/) if you create a new Google Cloud Account.

When you finish, you should have:

- a Google Cloud project called `kubernetes-toolbox` created
- [billing enabled](https://cloud.google.com/billing/docs/how-to/modify-project)
- the [Cloud Build API](https://cloud.google.com/cloud-build/docs/api/reference/rest/) enabled (this should be on by default)
- the [Kubernetes Engine API](https://cloud.google.com/kubernetes-engine/docs/reference/rest/) enabled

# Local Setup

In order to work with this project I'm assuming you already have the following installed

- [NodeJS](https://nodejs.org/en/)
- [kubectl](https://kubernetes.io/docs/tasks/tools/install-kubectl/)
- [Docker](https://docs.docker.com/v17.09/get-started/)
- [Install and Initialize the Google Cloud SDK](https://cloud.google.com/sdk/docs/)

# Deploy to Google Cloud Kubernetes Engine

In order to make things easier, I've created npm scripts for all the steps required. You can just call them directly once you've navigated to this directory in your terminal.

To deploy on Google Cloud Kubernetes Engine, do the following:

1. Run `npm run build-image`. This builds the image in Google Cloud's docker registry.
2. Run `npm run build-cluster`. This builds the cluster with the image you just built.
3. Run `npm run cluster-nodes`. This will use kubectl to get the status of the nodes in the cluster you just built.
4. Run `npm run apply-deployment`. This will create a deployment within your provisioned cluster.
5. Run `npm run get-deployments`. This will retrieve information about the deployment that was created.
6. Run `npm run apply-service`. This will expose the application in your deployment so that you can hit it directly with HTTP requests.
7. Run `npm run cluster-service`. This will get the information about the service you created, and an IP address you can hit to verify its working. The output from when you run this should look like the following:

```bash
> kubectl get services

NAME         TYPE           CLUSTER-IP      EXTERNAL-IP   PORT(S)        AGE
hello        LoadBalancer   10.59.248.141   <pending>     80:31836/TCP   40s
kubernetes   ClusterIP      10.59.240.1     <none>        443/TCP        2m40s
```

Notice the `EXTERNAL-IP` value next to `hello`. This value is marked as `<pending>`. This is becuase Google Cloud is provisioning this IP address. Wait a few seconds and try again with `npm run cluster-service`.

When you have an `EXTERNAL-IP` for `hello` go ahead and hit it with a `curl` request and the output. The whole thing should look like this:

```bash
> kubectl get services

NAME         TYPE           CLUSTER-IP      EXTERNAL-IP   PORT(S)        AGE
hello        LoadBalancer   10.59.250.200   34.68.88.11   80:31954/TCP   82s
kubernetes   ClusterIP      10.59.240.1     <none>        443/TCP        6m10s
➜  google-cloud git:(master) ✗ curl 34.68.88.11
Hello World from Google Cloud Kubernetes Engine!%
```

# Cleanup Deployment

Once you've finished working, you'll want to delete the cluster and services you've created. This is so you won't get billed for usage (unless you want to).

1. Run `npm run cluster-delete` to delete the cluster you've created.
2. Run `npm run container-delete` to delete the docker image you've created.

You can also do a lot more in the [Google Cloud Platform Console](https://console.cloud.google.com/).

# Final Thoughts

If you see any issues here please open a PR. Otherwise, feel free to reach out to me on Twitter at [@AndrewEvans0102](https://www.twitter.com/andrewevans0102).
