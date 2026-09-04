---
title: Running jobs with quota limits
sidebar_position: 16
---

# Running jobs with quota limits {#running-kueue-jobs}

<a id="running-kueue-jobs"></a>

You can run Kubernetes jobs with Red Hat build of Kueue enabled to manage resource allocation within defined quota limits. Running jobs with quota limits provides predictable resource availability, cluster stability, and optimized performance.

## Identifying available local queues {#identifying-local-queues_running-kueue-jobs}

Before you can submit a job to a queue, you must find the name of the local queue.

**Prerequisites**

- A cluster administrator has installed and configured Red Hat build of Kueue on your OpenShift Container Platform cluster.
- A cluster administrator has assigned you the `kueue-batch-user-role` cluster role.
- You have installed the OpenShift CLI (`oc`).

**Procedure**

- Run the following command to list available local queues in your namespace:
  ```terminal
  $ oc -n <namespace> get localqueues
  ```

  ```terminal title="Example output"
  NAME         CLUSTERQUEUE    PENDING WORKLOADS
  user-queue   cluster-queue   3
  ```

## Defining a job to run with Red Hat build of Kueue {#defining-running-jobs_running-kueue-jobs}

When you are defining a job to run with Red Hat build of Kueue, ensure that it meets the required criteria.

The job must:

- Specify the local queue to submit the job to, by using the `kueue.x-k8s.io/queue-name` label.
- Include the resource requests for each job pod.

Red Hat build of Kueue suspends the job, and then starts it when resources are available. Red Hat build of Kueue creates a corresponding workload, represented as a `Workload` object with a name that matches the job.

**Prerequisites**

- A cluster administrator has installed and configured Red Hat build of Kueue on your OpenShift Container Platform cluster.
- A cluster administrator has assigned you the `kueue-batch-user-role` cluster role.
- You have installed the OpenShift CLI (`oc`).
- You have identified the name of the local queue that you want to submit jobs to.

**Procedure**

1. Create a `Job` object.
   ```yaml title="Example job"
   apiVersion: batch/v1
   kind: Job
   metadata:
     generateName: sample-job-
     namespace: my-namespace
     labels:
       kueue.x-k8s.io/queue-name: user-queue
   spec:
     parallelism: 3
     completions: 3
     template:
       spec:
         containers:
         - name: dummy-job
           image: registry.k8s.io/e2e-test-images/agnhost:2.53
           args: ["entrypoint-tester", "hello", "world"]
           resources:
             requests:
               cpu: 1
               memory: "200Mi"
         restartPolicy: Never
   ```

   where:

   <dl>
   <dt><code>kind</code></dt>
   <dd>Defines the resource type as a <code>Job</code> object, which represents a batch computation task.</dd>
   <dt><code>metadata.generateName</code></dt>
   <dd>Provides a prefix for generating a unique name for the job.</dd>
   <dt><code>metadata.labels.kueue.x-k8s.io/queue-name</code></dt>
   <dd>Identifies the queue to send the job to.</dd>
   <dt><code>spec.template.spec.containers[].resources</code></dt>
   <dd>Defines the resource requests for each pod.</dd>
   </dl>
2. Run the job by running the following command:
   ```terminal
   $ oc create -f <filename>.yaml
   ```

**Verification**

- Verify that pods are running for the job you have created, by running the following command and observing the output:
  ```terminal
  $ oc get job <job-name>
  ```

  ```terminal title="Example output"
  NAME               STATUS      COMPLETIONS   DURATION   AGE
  sample-job-sk42x   Suspended   0/1                      2m12s
  ```
- Verify that a workload has been created in your namespace for the job, by running the following command and observing the output:
  ```terminal
  $ oc -n <namespace> get workloads
  ```

  ```terminal title="Example output"
  NAME                         QUEUE          RESERVED IN   ADMITTED   FINISHED   AGE
  job-sample-job-sk42x-77c03   user-queue                                         3m8s
  ```
