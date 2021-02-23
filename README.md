# An Umbrella chart lab

This repo contains an hands-on lab about Umbrella chart with helm3.
This lab is based on the great helm tutorial by David Campos that is published here:
https://hands-on-tech.github.io/2020/03/15/k8s-jenkins-example.html

At the end of this lab you will have a full umbrella chart that will deploy a full deployment
consist of Traefik, k8s dashboard, jenkins server and a small Chuck Norris web page with random joke.

This lab will not touch security measures, cluster management or CI/CD creation.
This means that the chrome will probably shout at you when you will go to the urls of the 
deployed services and that's ok because it is not in the scope of this lab.

In the lab-charts folder you will find all the charts that we are going to work on.
In the solved folder you can find the end result we want to achieve.

## Requirements

1. single node k8s cluster (minikube or similar is good enough).
1. kubectl
1. helm3
1. This repo cloned to your machine :)

## Let's start

After getting all the needed stuff, let's start creating our chart of charts.

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

1. We want that our chuckjokes chart will be our umbrella chart. So first let's create a folder called "charts" under "chuckjokes" folder and copy "traefik" chart folder to it. 
1. When creating sub-charts as dependency you can choose between putting the dependency chart under the "charts" folder or we can supply a repository url to get it.
for our example we placed it under "charts" and we are going to create a reference to it. So open the chuckjokes/Chart.yaml and add the following lines:
    ```
    dependencies:
      - name: traefik
        version: "9.14.3"
        repository: "file://charts/traefik"
    ```
1. While in Chart.yaml file let's take care of bumping the chart major version to 1.0.0 as we are making breaking changes to the chart.
1. Now we want to use the umbrella chart awasome power of overriding values of it's sub-charts. Let's open the chuckjokes/values.yaml file and at the bottom of it will add:
    ```
    traefik:
      dashboard:
        enabled: true
        domain: traefik.localhost
        auth:
          basic:
            admin: $2y$05$kpCJY2gJWlgG5CUs5tdPx.2xGJ4xyqhWtjiiM/NKfHmj3pfUPsap2
      ssl:
        enabled: true
        enforced: true
        permanentRedirect: true
        insecureSkipVerify: true
        generateTLS: true
        defaultCN: "*.localhost"
    ```
   those lines will make traefik expose it's dashboard at the domain we want.
5. that's it. now traefik is a dependency chart of chuckjokes. now if we will install our umbrella it will deploy both our app and traefik.
but we are not done yet...

### Part II - K8s Dashboard for fun and monitoring

In this part we want to make the k8s dashboard chart as sub-chart.
original chart can be found here: https://github.com/kubernetes/dashboard/tree/master/aio/deploy/helm-chart/kubernetes-dashboard.

It is mostly the same as dependecy, only difference is that the deployment will not fail if
the chart is missing.
In this part we will also make a simple use of global values an implement a simple solution
for the problem of using global and harming the stand-alone chart.

Pay attention - kubernetes-dashboard has it's own dependencies !!!



### Part III - Adding the favorite butler chart


In this part we will add the Jenkins server chart to our master chart.
original chart can be found here: https://github.com/jenkinsci/helm-charts.
In this part you can choose how to add it (dependency or not).


### Part IV - Running our Helm chart and praying that it works...

In this part we will finally run helm install on our cluster and see the fruits of our labor.
