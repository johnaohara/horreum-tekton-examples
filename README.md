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

View Pipeline run log;

```shell
$ tkn pr logs performance-test-pipeline-run-z5t4z
[performance-test-run : run-perf-test] 16:55:50.878 running with detached console
[performance-test-run : run-perf-test] 16:55:50.884 Running qDup version 0.7.1 @ 1fc7ef9
[performance-test-run : run-perf-test] 16:55:50.884 output path = /workspace/results
[performance-test-run : run-perf-test] 16:55:50.884 shell exit code checks enabled
[performance-test-run : run-perf-test] 16:55:51.002 json server listening at performance-test-pipeline-run-z5t4z-performance-test-run-pod:31337
[performance-test-run : run-perf-test] 16:55:51.158 starting 1 scripts
[performance-test-run : run-perf-test] 16:55:51.161 create-some-data:38@LOCAL: script-cmd: create-some-data
[performance-test-run : run-perf-test] 16:55:51.162 create-some-data:38@LOCAL: create-some-data
[performance-test-run : run-perf-test] 16:55:51.570 create-some-data:38@LOCAL: date +%s%N | cut -b1-13
[performance-test-run : run-perf-test] 1687452951165
[performance-test-run : run-perf-test] 16:55:51.573 create-some-data:38@LOCAL: set-state: RUN.output.start 1687452951165
[performance-test-run : run-perf-test] 16:55:54.979 create-some-data:38@LOCAL: sleep 3
[performance-test-run : run-perf-test] 16:55:55.385 create-some-data:38@LOCAL: date +%s%N | cut -b1-13
[performance-test-run : run-perf-test] 1687452954981
[performance-test-run : run-perf-test] 16:55:55.387 create-some-data:38@LOCAL: set-state: RUN.output.stop 1687452954981
[performance-test-run : run-perf-test] 16:55:55.389 create-some-data:38@LOCAL: set-state: RUN.output.value 9
[performance-test-run : run-perf-test] 16:55:55.390 create-some-data:38@LOCAL: set-state: RUN.output.build 101
[performance-test-run : run-perf-test] 16:55:55.395 create-some-data:16@LOCAL. reader thread is stopping
[performance-test-run : run-perf-test] 16:55:55.500 starting 1 scripts
[performance-test-run : run-perf-test] 16:55:55.501 inlab-cleanup:50@LOCAL: inlab-cleanup
[performance-test-run : run-perf-test] 16:55:55.502 inlab-cleanup:50@LOCAL: script-cmd: download-result
[performance-test-run : run-perf-test] 16:55:55.502 inlab-cleanup:50@LOCAL: download-result
[performance-test-run : run-perf-test] 16:55:55.913 inlab-cleanup:50@LOCAL: echo '{"start":1687452951165,"stop":1687452954981,"value":9,"build":101}' > /tmp/output.json
[performance-test-run : run-perf-test] 16:55:55.915 inlab-cleanup:50@LOCAL: queue-download: /tmp/output.json=/tmp/output.json /workspace/results/LOCAL/
[performance-test-run : run-perf-test] 16:55:55.918 inlab-cleanup@LOCAL. reader thread is stopping
[performance-test-run : run-perf-test] 16:55:55.918 run-1687452950040 downloading queued downloads
[performance-test-run : run-perf-test] 16:55:55.920 Local.download(LOCAL:/tmp/output.json,/workspace/results/LOCAL/)
[performance-test-run : run-perf-test] Finished in 04.897 at /workspace/results

[performance-test-show-results : show-results] total 16
[performance-test-show-results : show-results] drwxrwxrwx. 1 root root    40 Jun 22 11:25 .
[performance-test-show-results : show-results] drwxrwxrwx. 1 root root    14 Jun 22 16:56 ..
[performance-test-show-results : show-results] drwxr-xr-x. 1 1000 1000    22 Jun 22 11:25 LOCAL
[performance-test-show-results : show-results] -rw-r--r--. 1 1000 1000 11091 Jun 22 16:55 run.json
[performance-test-show-results : show-results] -rw-r--r--. 1 1000 1000  1324 Jun 22 16:55 run.log
[performance-test-show-results : show-results] {"start":1687452951165,"stop":1687452954981,"value":9,"build":101}

[performance-test-horreum-upload : upload-to-horreum] Starting the Java application using /opt/jboss/container/java/run/run-java.sh run new --test tekton-poc --owner perf-team --access PRIVATE --start $.start --stop $.stop --file /workspace/results/LOCAL/output.json...
[performance-test-horreum-upload : upload-to-horreum] INFO exec  java -Dquarkus.http.host=0.0.0.0 -Djava.util.logging.manager=org.jboss.logmanager.LogManager -XX:MaxRAMPercentage=50.0 -XX:+UseParallelGC -XX:MinHeapFreeRatio=10 -XX:MaxHeapFreeRatio=20 -XX:GCTimeRatio=4 -XX:AdaptiveSizePolicyWeight=90 -XX:+ExitOnOutOfMemoryError -cp "." -jar /deployments/quarkus-run.jar run new --test tekton-poc --owner perf-team --access PRIVATE --start $.start --stop $.stop --file /workspace/results/LOCAL/output.json
[performance-test-horreum-upload : upload-to-horreum] new run uploaded: https://example.com/run/17986#run
```