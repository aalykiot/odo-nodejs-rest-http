# Node.js REST HTTP using ODO

This repo shows how you can deploy the rest-http example from nodeshift starters https://github.com/nodeshift-starters/nodejs-rest-http to openshift using ODO [https://github.com/openshift/odo].

## Login to Openshift

```sh
$ odo login -u developer -p developer
```
For all of the above steps you have to be logged in to openshift.

## Create a new openshift project

```sh
$ odo project create nodeshift-starters
```
A project is a workspace where we will deploy all of our components and services.

## Enable experimental mode

To be able to specify the deployment environment we have to enable odo's experimental mode.

```sh
$ odo preference set experimental true 
```

## Create a new component

```sh
$ odo create nodejs nodejs-rest-http
```
Because we have enabled the **experimental** option of odo the above command will generate a generic nodejs **devfile.yaml** file.

## Customize devfile.yaml

In order for the nodejs-rest-http example to run we have to replace the generated devfile with a custom one.

```
apiVersion: 1.0.0
metadata:
  generateName: nodejs-
projects:
  - name: nodejs-rest-http
    source:
      type: git
      location: "https://github.com/alexalikiotis/odo-nodejs-rest-http.git"
components:
  - type: dockerimage
    alias: runtime
    image: registry.access.redhat.com/ubi8/nodejs-12
    memoryLimit: 256Mi
    mountSources: true
    endpoints:
      - name: "nodejs"
        port: 8080
commands:
  - name: devBuild
    actions:
      - type: exec
        component: runtime
        command: npm install
        workdir: ${CHE_PROJECTS_ROOT}/nodejs-rest-http
  - name: devRun
    actions:
      - type: exec
        component: runtime
        command: npm start
        workdir: ${CHE_PROJECTS_ROOT}/nodejs-rest-http

```

## Create a new route

In order to access the example from the browser we need to expose the deployment to the world using an openshift route.

```sh
$ odo url create nodejs-rest-http --port 8080 --host apps-crc.testing
```

## Deploy to Openshift

The last step is to deploy the example to our openshift cluster.

```sh
$ odo push
```

And we're done 👍
