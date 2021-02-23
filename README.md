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
   * Please note that the values we want to override will always be under a block with the exact name of the sub-chart.
   
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

1. Now let's get our k8s dashboard inside our beloved "charts" folder.
1. That's it... from this moment this chart is considered sub-chart and will be part of the installation of our chuckjokes umbrella chart.
but we are not done with it yet. Let's first take care of the dashboard chart dependencies. go to the it's folder and run:
    ```
    helm dep update
    ```
    this will retrieve all the deps that are needed for our k8s dashboard chart as tgz file.
1. now let's add some values we want to override to chuckjokes/values.yaml file
    ```
    kubernetes-dashboard:
      enableInsecureLogin: true
      service:
        externalPort: 9090
      ingress:
        enabled: true
        hosts:
          - dashboard.localhost
        paths:
          - /
    ```
1. Now we want also to add a global value to this file. let's do it:
    ```
    global:
      ingress:
        annotations:
          kubernetes.io/ingress.class: traefik
    ```
    * Now we want to use the global in our k8s dashboard chart. The use of global is done by simply using
    ``` 
    {{ .Values.global.<name of the value> }}
    ``` 
    instead of using 
    ```
    {{ .Values.global.<name of the value> }}
    ```
    but there is more to it as if we just implement it like this, our chart will no longer be a stand-alone, as it
    will fail to run alone because the templating engine will fail on none existing global value when trying to install 
    only the k8s dashboard chart. No worries there are solutions for that... now pause a bit and try think for yourself
    how we can resolve it...

1. Now, we have added a ```ingress.annotations``` value to our global. What we need to do is to find all the uses of ```ingress.annotations``` value in k8s dashboard chart.
and replace it with ```global.ingress.annotations```. but remember that we don't want to harm the stand-alone ability.

    * one possible solution is to use the "if condition", so in our chart the use of the above value is under templates/ingress.yaml.
    let's just replace the lines that use it, so it will look like this:
    ```
    {{- if .Values.global }}
    {{- with .Values.global.ingress.annotations }}
    {{ toYaml . | indent 4 }}
    {{- end }}
    {{- else }}
    {{- with .Values.ingress.annotations }}
    {{ toYaml . | indent 4 }}
    {{- end }}
    {{- end }}
    ```
    you can look at the solved/chuckjokes chart to see how the file should look like.
1. That's it we have now added another sub-chart and used global values. Let's move to the next part...

### Part III - Adding the favorite butler chart

In this part we will add the Jenkins server chart to our master chart.
original chart can be found here: https://github.com/jenkinsci/helm-charts.
In this part you can choose how to add it (dependency or not). 

1. The only step I will tell you here is that you will need to have those values:
    ```
      controller:
        adminSecret: true
        adminUser: admin
        adminPassword: admin
        numExecutors: 1
        installPlugins:
          - kubernetes:1.29.0
          - workflow-job:2.40
          - workflow-aggregator:2.6
          - credentials-binding:1.24
          - git:4.6.0
          - command-launcher:1.5
          - github-branch-source:2.9.6
          - docker-workflow:1.25
          - pipeline-utility-steps:2.6.1
          - configuration-as-code:1.47
        overwritePlugins: true
        ingress:
          enabled: true
          hostName: jenkins.localhost
          annotations:
            kubernetes.io/ingress.class: traefik
    ```

* Pay attention to how you add it and where. You are on your own (but from now you should be fine).

### Part IV - Running our Helm chart and praying that it works...

In this part we will finally run helm install on our cluster and see the fruits of our labor.

1. let's go to the chuckjokes folder and run the following command:
    ```
    helm install --dry-run --debug chuck .
    ```
   This should do as-if install and will alert us on any errors that might be.
2. if everything is ok, then let's run the same command without the --dry-run flag. this will actually install our chart.
3. If all goes well, we can run the following command to see what is the state of the pods that we just deployed:
    ```
    kubectl get pods
    ```
4. Let's go and check our services. Usually the first ones that will be ready are the traefik and dashboard. Let's go and see them:
    * traefik - http://traefik.localhost
    * k8s dashboard - http://dashboard.localhost
        * for login to the k8s dashboard you will need it's token. this could be retrieved by:
        ```
        kubectl get secrets
        kubectl describe <the secret that ends with dashboard-token>
        ```
        copy the token and paste it in the login screen.
    * In traefik you can also see if there is a problem with one of the services (it will be red).  
    * Jenkins can take a few minutes as it installs plugins and stuff. when up it will be available in http://jenkins.localhost
        * login with the user and password that is set in it's values.
    * Our Chuck Norris web app should be available at: http://chuck-jokes.localhost/chuck-yanko
        * Try to refresh to get new jokes :)
    
Hope you enjoyed this lab.
Any comments are welcome... unless they are bad :)