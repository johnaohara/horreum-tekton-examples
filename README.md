## Example Horreum tekton tasks

### Pre-Requisits
1. Running k8's cluster
2. (Optional) tkn cli installed

## Config

1. You need to define the remote Horreum credentials as a secret in the k8's cluster

2. Create 3 files `username`, `password`, and `horreumUrl`, containing a valid username, password and horreum url in each respective file

3. Create the screts in k8's

```shell
$ kubectl create secret generic horreum-credentials  --from-file=./username --from-file=./password --from-file=./horreumUrl
```

4. Create PVC for workspaces

```shell
$ kubectl apply  -f ./perf-test-pvc.yaml
```

5. Optional: Install tekton dashboard

```shell
$ kubectl apply --filename https://storage.googleapis.com/tekton-releases/dashboard/latest/release.yaml
$ kubectl port-forward -n tekton-pipelines service/tekton-dashboard 9097:9097
```

## Setup tekton tasks tasks

Define tekton task;

```shell
$ kubectl apply --filename run-list.yaml
task.tekton.dev/horreum-run-list configured
$ kubectl apply --filename run-payload.yaml
task.tekton.dev/horreum-run-payload configured
$ kubectl apply --filename test-describe.yaml
task.tekton.dev/horreum-test-describe configured
$ kubectl apply --filename test-list.yaml
task.tekton.dev/horreum-test-list configured
```

## Execute Tekton tasks

Now we have the tekton tasks configured, we can execute the tasks directly, or incorporate them into a tekton pipeline;

```shell
$ tkn task list
NAME                    DESCRIPTION   AGE
horreum-run-list                      44 minutes ago
horreum-run-payload                   43 minutes ago
horreum-test-describe                 1 hour ago
horreum-test-list                     1 hour ago

$ tkn task start horreum-test-list
? Value for param `credentialsSecret` of type `string`? (Default is `horreum-credentials`) horreum-credentials
TaskRun started: horreum-test-list-run-h52jn

In order to track the TaskRun progress run:
tkn taskrun logs horreum-test-list-run-h52jn -f -n default

$ tkn taskrun logs horreum-test-list-run-h52jn -f -n default
[upload] Starting the Java application using /opt/jboss/container/java/run/run-java.sh test list...
[upload] INFO exec  java -Dquarkus.http.host=0.0.0.0 -Djava.util.logging.manager=org.jboss.logmanager.LogManager -XX:MaxRAMPercentage=50.0 -XX:+UseParallelGC -XX:MinHeapFreeRatio=10 -XX:MaxHeapFreeRatio=20 -XX:GCTimeRatio=4 -XX:AdaptiveSizePolicyWeight=90 -XX:+ExitOnOutOfMemoryError -cp "." -jar /deployments/quarkus-run.jar test list
[upload] Found Tests: 
[upload] 114 - amq-broker-message-processing
...
[upload] 14 - Quarkus - config-quickstart - JVM
...

$ tkn task start horreum-run-list 
? Value for param `credentialsSecret` of type `string`? (Default is `horreum-credentials`) horreum-credentials
? Value for param `testid` of type `string`? 14
TaskRun started: horreum-run-list-run-p79h2

In order to track the TaskRun progress run:
tkn taskrun logs horreum-run-list-run-p79h2 -f -n default

$ tkn taskrun logs horreum-run-list-run-p79h2 -f -n default
[upload] Starting the Java application using /opt/jboss/container/java/run/run-java.sh run list -i 14...
[upload] INFO exec  java -Dquarkus.http.host=0.0.0.0 -Djava.util.logging.manager=org.jboss.logmanager.LogManager -XX:MaxRAMPercentage=50.0 -XX:+UseParallelGC -XX:MinHeapFreeRatio=10 -XX:MaxHeapFreeRatio=20 -XX:GCTimeRatio=4 -XX:AdaptiveSizePolicyWeight=90 -XX:+ExitOnOutOfMemoryError -cp "." -jar /deployments/quarkus-run.jar run list -i 14
[upload] 7930: 1652360062000
[upload] 14782: 1677091644000
[upload] 7950: 1652412281000
...

$ tkn task start horreum-run-payload
? Value for param `credentialsSecret` of type `string`? (Default is `horreum-credentials`) horreum-credentials
? Value for param `runid` of type `string`? 7910
TaskRun started: horreum-run-payload-run-9kp4h

In order to track the TaskRun progress run:
tkn taskrun logs horreum-run-payload-run-9kp4h -f -n default

$ tkn taskrun logs horreum-run-payload-run-9kp4h -f -n default
[upload] Starting the Java application using /opt/jboss/container/java/run/run-java.sh run data -i 7910...
[upload] INFO exec  java -Dquarkus.http.host=0.0.0.0 -Djava.util.logging.manager=org.jboss.logmanager.LogManager -XX:MaxRAMPercentage=50.0 -XX:+UseParallelGC -XX:MinHeapFreeRatio=10 -XX:MaxHeapFreeRatio=20 -XX:GCTimeRatio=4 -XX:AdaptiveSizePolicyWeight=90 -XX:+ExitOnOutOfMemoryError -cp "." -jar /deployments/quarkus-run.jar run data -i 7910
[upload] {
[upload]   "job" : "quarkus-startup",
[upload]   "data" : {
[upload]     "1" : {
[upload]       "warm" : [ 148084, 148440, 148612, 148624, 148632, 148724, 148724, 148724, 148880 ],
[upload]       "started" : [ 114540, 115412, 109372, 113492, 115568 ],
[upload]       "latencyAvg" : 9.94,
[upload]       "latencyMax" : 235.37,
[upload]       "throughput" : 55215.64,
[upload]       "startedVmRss" : [ 113116, 113660, 107732, 112044, 114052 ],
...

```


## Example Pipeline

Define tekton pipeline and tasks;

```shell
$ kubectl apply -f ./performance-test-run.yaml 
$ kubectl apply -f ./performance-test-show-results.yaml 
$ kubectl apply -f ./performance-test-horreum-upload.yaml 
$ kubectl apply -f ./performance-test-pipeline.yaml 
```

Start a predefined pipeline run;

```shell
$ kubectl apply -f ./performance-test-pipeline-run.yaml 
```

Start an interactive pipeline run;

```shell
$  tkn p start performance-test-pipeline
? Value for param `credentialsSecret` of type `string`? horreum-credentials
? Value for param `qdup-url` of type `string`? https://raw.githubusercontent.com/johnaohara/horreum-tekton-examples/main/examples/demo-pipeline.qdup.yaml
? Value for param `resultFile` of type `string`? LOCAL/output.json
? Value for param `testName` of type `string`? tekton-poc
? Value for param `testOwner` of type `string`? (Default is `perf-team`) perf-team
Please give specifications for the workspace: results 
? Name for the workspace : results
? Value of the Sub Path :  
? Type of the Workspace : pvc
? Value of Claim Name : perf-tekton-workspace
PipelineRun started: performance-test-pipeline-run-z5t4z
```