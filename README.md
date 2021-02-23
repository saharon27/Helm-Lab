# An Umbrella chart lab

This repo contains an hands-on lab about Umbrella chart with helm3.
This lab is based on the great helm tutorial by David Campos that is published here:
https://hands-on-tech.github.io/2020/03/15/k8s-jenkins-example.html

At the end of this lab you will have a full umbrella chart that will deploy a full deployment
consist of Traefik, k8s dashboard, jenkins server and a small Chuck Norris web page with random joke.

In the lab-charts folder you will find all the charts that we are going to work on.
In the solved folder you can find the end result we want to achieve.

## Requirements

1. single node k8s cluster (minikube or similar is good enough).
1. kubectl
1. helm3

## Let's start

After installing all the needed stuff, let's start creating our chart of charts.

### Part I - Taking Care of Traefik Chart

In this part we make the Traefik as dependency to our deployment chart.
Under the lab-charts folder you will find a forked Traefik chart. You can also find the chart
in the original repo: https://github.com/traefik/traefik-helm-chart

It is mostly the same as sub-chart, only difference is that this chart is a most have to our chart.
Dependencies should be updated for first time use by running the command:
```
helm dependency update 
```
or by adding 'repository' URL to our chart/dependencies field.


### Part II - K8s Dashboard for fun and monitoring

In this part we want to make the k8s dashboard chart as sub-chart.
original chart can be found here: https://github.com/kubernetes/dashboard/tree/master/aio/deploy/helm-chart/kubernetes-dashboard.

It is mostly the same as dependecy, only difference is that the deployment will not fail if
the chart is missing.



### Part III - Adding the favorite butler chart


In this part we will add the Jenkins server chart to our master chart.
original chart can be found here: https://github.com/jenkinsci/helm-charts.
In this part you can choose how to add it (dependency or not).
But now will make use og global values and how to make the chart still stand-alone.


### Part IV - Running our Helm chart and praying that it works...

In this part we will finally run helm install on our cluster and see the fruits of our labor.
