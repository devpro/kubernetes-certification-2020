# Linux Foundation training - Kubernetes for Developers (LFD259) - Lab exercises

## Setup

```bash
wget https://training.linuxfoundation.org/cm/LFD259/LFD259_V2020-06-18_SOLUTIONS.tar.bz2 --user=xxxx --password=xxxx
tar -xvf LFD259_V2020-06-18_SOLUTIONS.tar.bz2
```

## Lab 4.6. Domain Review

- [Documentation > Tasks > Configure Pods and Containers > Assign CPU Resources to Containers and Pods](https://kubernetes.io/docs/tasks/configure-pod-container/assign-cpu-resource/)

```bash
# init
kubectl create namespace cpu-example

# this sample will be ok
kubectl apply -f https://k8s.io/examples/pods/resource/cpu-request-limit.yaml --namespace=cpu-example
kubectl get pod cpu-demo --namespace=cpu-example
kubectl top pod --namespace=cpu-example
kubectl delete pod cpu-demo --namespace=cpu-example

# this won't work as we are exceeding node capacity
kubectl apply -f https://k8s.io/examples/pods/resource/cpu-request-limit-2.yaml --namespace=cpu-example
kubectl describe pod cpu-demo-2 --namespace=cpu-example
kubectl delete pod cpu-demo-2 --namespace=cpu-example

# cleanup
kubectl delete namespace cpu-example
```

- We'll be using [vish/stress](https://hub.docker.com/r/vish/stress/) image.

```txt
Usage of /stress:
  -alsologtostderr
        log to standard error as well as files
  -cpus int
        total number of CPUs to utilize
  -log_backtrace_at value
        when logging hits line file:N, emit a stack trace (default :0)
  -log_dir string
        If non-empty, write log files in this directory
  -logtostderr
        log to standard error instead of files
  -mem-alloc-size string
        amount of memory to be consumed in each allocation (default "4Ki")
  -mem-alloc-sleep duration
        duration to sleep between allocations (default 1ms)
  -mem-total string
        total memory to be consumed. Memory will be consumed via multiple allocations.
  -stderrthreshold value
        logs at or above this threshold go to stderr
  -v value
        log level for V logs
  -vmodule value
        comma-separated list of pattern=N settings for file-filtered logging
```

- Review CPU and memory limits

```bash
kubectl apply -f resources/LFD259/SOLUTIONS/s_04/design-review1.yaml --namespace=cpu-example
kubectl get pods --namespace cpu-example
kubectl describe pod design-pod1 --namespace cpu-example
kubectl logs design-pod1 --namespace cpu-example
kubectl delete -f resources/LFD259/SOLUTIONS/s_04/design-review1.yaml --namespace=cpu-example
```

- Use labels to delete pods

```bash
kubectl apply -f resources/LFD259/SOLUTIONS/s_04/design-review2.yaml
kubectl get pods
kubectl delete pods -l review=tux # pod1 & pod2
kubectl delete pods -l linux=rocks # pod3
kubectl delete pods -l domain=review # pod4
kubectl delete -f resources/LFD259/SOLUTIONS/s_04/design-review2.yaml
```

- Create a new cronjob which runs busybox and the sleep 30 command. Have the cronjob run every three minutes. View the job status to check your work. Change the settings so the pod runs 10 minutes from the current time, every week. For example, if the current time was 2:14PM, I would configure the job to run at 2:24PM, every Monday.

```bash
# use helpers to look at the syntax
kubectl create cronjob -h
sleep --help

# create and update the cronjob
kubectl create cronjob test-job --image=busybox --schedule="*/3 * * * *" -- sleep 30s
kubectl get cronjobs
kubectl get cronjob test-job -o yaml
kubectl create job --from=cronjob/test-job singlejob
kubectl get pods
# careful with timezone, Minikube is GMT so 16:33 GMT -> 18:33 CET
kubectl patch cronjob test-job -p '{"spec":{"schedule": "33 16 * * 1"}}'
kubectl delete cronjob test-job
```

- Extra: metrics
  - [kubernetes/kube-state-metrics](https://github.com/kubernetes/kube-state-metrics) -> kube_cronjob_next_schedule_time
  - [Collecting metrics with built-in Kubernetes monitoring tools](https://www.datadoghq.com/blog/how-to-collect-and-graph-kubernetes-metrics/)

```batch
sudo apt install jq
kubectl get --raw /apis/metrics.k8s.io/v1beta1/nodes/minikube | jq

wget https://github.com/kubernetes/kube-state-metrics/archive/v1.9.7.tar.gz
tar -xvf v1.9.7.tar.gz
kubectl apply -f kube-state-metrics-1.9.7/examples/standard
rm -rf kube-state-metrics-1.9.7
rm v1.9.7.tar.gz
```
