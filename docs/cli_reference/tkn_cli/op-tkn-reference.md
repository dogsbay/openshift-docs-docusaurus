---
title: OpenShift Pipelines tkn reference
sidebar_position: 3
---

# OpenShift Pipelines tkn reference {#op-tkn-reference}

<a id="op-tkn-reference"></a>

This section lists the basic `tkn` CLI commands.

## Basic syntax {#_basic_syntax}

`tkn [command or options] [arguments...]`

## Global options {#_global_options}

`--help, -h`

## Utility commands {#op-tkn-utility-commands_op-tkn-reference}

### tkn {#_tkn}

Parent command for `tkn` CLI.

```terminal title="Example: Display all options"
$ tkn
```

### completion [shell] {#_completion_shell}

Print shell completion code which must be evaluated to provide interactive completion. Supported shells are `bash` and `zsh`.

```terminal title="Example: Completion code for bash shell"
$ tkn completion bash
```

### version {#_version}

Print version information of the `tkn` CLI.

```terminal title="Example: Check the tkn version"
$ tkn version
```

## Pipelines management commands {#op-tkn-pipeline-management_op-tkn-reference}

### pipeline {#_pipeline}

Manage pipelines.

```terminal title="Example: Display help"
$ tkn pipeline --help
```

### pipeline delete {#_pipeline_delete}

Delete a pipeline.

```terminal title="Example: Delete the mypipeline pipeline from a namespace"
$ tkn pipeline delete mypipeline -n myspace
```

### pipeline describe {#_pipeline_describe}

Describe a pipeline.

```terminal title="Example: Describe the mypipeline pipeline"
$ tkn pipeline describe mypipeline
```

### pipeline list {#_pipeline_list}

Display a list of pipelines.

```terminal title="Example: Display a list of pipelines"
$ tkn pipeline list
```

### pipeline logs {#_pipeline_logs}

Display the logs for a specific pipeline.

```terminal title="Example: Stream the live logs for the mypipeline pipeline"
$ tkn pipeline logs -f mypipeline
```

### pipeline start {#_pipeline_start}

Start a pipeline.

```terminal title="Example: Start the mypipeline pipeline"
$ tkn pipeline start mypipeline
```

## Pipeline run commands {#op-tkn-pipeline-run_op-tkn-reference}

### pipelinerun {#_pipelinerun}

Manage pipeline runs.

```terminal title="Example: Display help"
$ tkn pipelinerun -h
```

### pipelinerun cancel {#_pipelinerun_cancel}

Cancel a pipeline run.

```terminal title="Example: Cancel the mypipelinerun pipeline run from a namespace"
$ tkn pipelinerun cancel mypipelinerun -n myspace
```

### pipelinerun delete {#_pipelinerun_delete}

Delete a pipeline run.

```terminal title="Example: Delete pipeline runs from a namespace"
$ tkn pipelinerun delete mypipelinerun1 mypipelinerun2 -n myspace
```

```terminal title="Example: Delete all pipeline runs from a namespace, except the five most recently executed pipeline runs"
$ tkn pipelinerun delete -n myspace --keep 5 (1)
```

1. Replace `5` with the number of most recently executed pipeline runs you want to retain.

```terminal title="Example: Delete all pipelines"
$ tkn pipelinerun delete --all
```

:::note

Starting with Red Hat OpenShift Pipelines 1.6, the `tkn pipelinerun delete --all` command does not delete any resources that are in the running state.

:::

### pipelinerun describe {#_pipelinerun_describe}

Describe a pipeline run.

```terminal title="Example: Describe the mypipelinerun pipeline run in a namespace"
$ tkn pipelinerun describe mypipelinerun -n myspace
```

### pipelinerun list {#_pipelinerun_list}

List pipeline runs.

```terminal title="Example: Display a list of pipeline runs in a namespace"
$ tkn pipelinerun list -n myspace
```

### pipelinerun logs {#_pipelinerun_logs}

Display the logs of a pipeline run.

```terminal title="Example: Display the logs of the mypipelinerun pipeline run with all tasks and steps in a namespace"
$ tkn pipelinerun logs mypipelinerun -a -n myspace
```

## Task management commands {#op-tkn-task-management_op-tkn-reference}

### task {#_task}

Manage tasks.

```terminal title="Example: Display help"
$ tkn task -h
```

### task delete {#_task_delete}

Delete a task.

```terminal title="Example: Delete mytask1 and mytask2 tasks from a namespace"
$ tkn task delete mytask1 mytask2 -n myspace
```

### task describe {#_task_describe}

Describe a task.

```terminal title="Example: Describe the mytask task in a namespace"
$ tkn task describe mytask -n myspace
```

### task list {#_task_list}

List tasks.

```terminal title="Example: List all the tasks in a namespace"
$ tkn task list -n myspace
```

### task logs {#_task_logs}

Display task logs.

```terminal title="Example: Display logs for the mytaskrun task run of the mytask task"
$ tkn task logs mytask mytaskrun -n myspace
```

### task start {#_task_start}

Start a task.

```terminal title="Example: Start the mytask task in a namespace"
$ tkn task start mytask -s <ServiceAccountName> -n myspace
```

## Task run commands {#op-tkn-task-run_op-tkn-reference}

### taskrun {#_taskrun}

Manage task runs.

```terminal title="Example: Display help"
$ tkn taskrun -h
```

### taskrun cancel {#_taskrun_cancel}

Cancel a task run.

```terminal title="Example: Cancel the mytaskrun task run from a namespace"
$ tkn taskrun cancel mytaskrun -n myspace
```

### taskrun delete {#_taskrun_delete}

Delete a TaskRun.

```terminal title="Example: Delete the mytaskrun1 and mytaskrun2 task runs from a namespace"
$ tkn taskrun delete mytaskrun1 mytaskrun2 -n myspace
```

```terminal title="Example: Delete all but the five most recently executed task runs from a namespace"
$ tkn taskrun delete -n myspace --keep 5 (1)
```

1. Replace `5` with the number of most recently executed task runs you want to retain.

### taskrun describe {#_taskrun_describe}

Describe a task run.

```terminal title="Example: Describe the mytaskrun task run in a namespace"
$ tkn taskrun describe mytaskrun -n myspace
```

### taskrun list {#_taskrun_list}

List task runs.

```terminal title="Example: List all the task runs in a namespace"
$ tkn taskrun list -n myspace
```

### taskrun logs {#_taskrun_logs}

Display task run logs.

```terminal title="Example: Display live logs for the mytaskrun task run in a namespace"
$ tkn taskrun logs -f mytaskrun -n myspace
```

## Condition management commands {#op-tkn-condition-management_op-tkn-reference}

### condition {#_condition}

Manage Conditions.

```terminal title="Example: Display help"
$ tkn condition --help
```

### condition delete {#_condition_delete}

Delete a Condition.

```terminal title="Example: Delete the mycondition1 Condition from a namespace"
$ tkn condition delete mycondition1 -n myspace
```

### condition describe {#_condition_describe}

Describe a Condition.

```terminal title="Example: Describe the mycondition1 Condition in a namespace"
$ tkn condition describe mycondition1 -n myspace
```

### condition list {#_condition_list}

List Conditions.

```terminal title="Example: List Conditions in a namespace"
$ tkn condition list -n myspace
```

## Pipeline Resource management commands {#op-tkn-pipeline-resource-management_op-tkn-reference}

### resource {#_resource}

Manage Pipeline Resources.

```terminal title="Example: Display help"
$ tkn resource -h
```

### resource create {#_resource_create}

Create a Pipeline Resource.

```terminal title="Example: Create a Pipeline Resource in a namespace"
$ tkn resource create -n myspace
```

This is an interactive command that asks for input on the name of the Resource, type of the Resource, and the values based on the type of the Resource.

### resource delete {#_resource_delete}

Delete a Pipeline Resource.

```terminal title="Example: Delete the myresource Pipeline Resource from a namespace"
$ tkn resource delete myresource -n myspace
```

### resource describe {#_resource_describe}

Describe a Pipeline Resource.

```terminal title="Example: Describe the myresource Pipeline Resource"
$ tkn resource describe myresource -n myspace
```

### resource list {#_resource_list}

List Pipeline Resources.

```terminal title="Example: List all Pipeline Resources in a namespace"
$ tkn resource list -n myspace
```

## ClusterTask management commands {#op-tkn-clustertask-management-commands_op-tkn-reference}

:::warning

In Red Hat OpenShift Pipelines 1.10, ClusterTask functionality of the `tkn` command-line utility is deprecated and is planned to be removed in a future release.

:::

### clustertask {#_clustertask}

Manage ClusterTasks.

```terminal title="Example: Display help"
$ tkn clustertask --help
```

### clustertask delete {#_clustertask_delete}

Delete a ClusterTask resource in a cluster.

```terminal title="Example: Delete mytask1 and mytask2 ClusterTasks"
$ tkn clustertask delete mytask1 mytask2
```

### clustertask describe {#_clustertask_describe}

Describe a ClusterTask.

```terminal title="Example: Describe the mytask ClusterTask"
$ tkn clustertask describe mytask1
```

### clustertask list {#_clustertask_list}

List ClusterTasks.

```terminal title="Example: List ClusterTasks"
$ tkn clustertask list
```

### clustertask start {#_clustertask_start}

Start ClusterTasks.

```terminal title="Example: Start the mytask ClusterTask"
$ tkn clustertask start mytask
```

## Trigger management commands {#op-tkn-trigger-management_op-tkn-reference}

### eventlistener {#_eventlistener}

Manage EventListeners.

```terminal title="Example: Display help"
$ tkn eventlistener -h
```

### eventlistener delete {#_eventlistener_delete}

Delete an EventListener.

```terminal title="Example: Delete mylistener1 and mylistener2 EventListeners in a namespace"
$ tkn eventlistener delete mylistener1 mylistener2 -n myspace
```

### eventlistener describe {#_eventlistener_describe}

Describe an EventListener.

```terminal title="Example: Describe the mylistener EventListener in a namespace"
$ tkn eventlistener describe mylistener -n myspace
```

### eventlistener list {#_eventlistener_list}

List EventListeners.

```terminal title="Example: List all the EventListeners in a namespace"
$ tkn eventlistener list -n myspace
```

### eventlistener logs {#_eventlistener_logs}

Display logs of an EventListener.

```terminal title="Example: Display the logs of the mylistener EventListener in a namespace"
$ tkn eventlistener logs mylistener -n myspace
```

### triggerbinding {#_triggerbinding}

Manage TriggerBindings.

```terminal title="Example: Display TriggerBindings help"
$ tkn triggerbinding -h
```

### triggerbinding delete {#_triggerbinding_delete}

Delete a TriggerBinding.

```terminal title="Example: Delete mybinding1 and mybinding2 TriggerBindings in a namespace"
$ tkn triggerbinding delete mybinding1 mybinding2 -n myspace
```

### triggerbinding describe {#_triggerbinding_describe}

Describe a TriggerBinding.

```terminal title="Example: Describe the mybinding TriggerBinding in a namespace"
$ tkn triggerbinding describe mybinding -n myspace
```

### triggerbinding list {#_triggerbinding_list}

List TriggerBindings.

```terminal title="Example: List all the TriggerBindings in a namespace"
$ tkn triggerbinding list -n myspace
```

### triggertemplate {#_triggertemplate}

Manage TriggerTemplates.

```terminal title="Example: Display TriggerTemplate help"
$ tkn triggertemplate -h
```

### triggertemplate delete {#_triggertemplate_delete}

Delete a TriggerTemplate.

```terminal title="Example: Delete mytemplate1 and mytemplate2 TriggerTemplates in a namespace"
$ tkn triggertemplate delete mytemplate1 mytemplate2 -n `myspace`
```

### triggertemplate describe {#_triggertemplate_describe}

Describe a TriggerTemplate.

```terminal title="Example: Describe the mytemplate TriggerTemplate in a namespace"
$ tkn triggertemplate describe mytemplate -n `myspace`
```

### triggertemplate list {#_triggertemplate_list}

List TriggerTemplates.

```terminal title="Example: List all the TriggerTemplates in a namespace"
$ tkn triggertemplate list -n myspace
```

### clustertriggerbinding {#_clustertriggerbinding}

Manage ClusterTriggerBindings.

```terminal title="Example: Display ClusterTriggerBindings help"
$ tkn clustertriggerbinding -h
```

### clustertriggerbinding delete {#_clustertriggerbinding_delete}

Delete a ClusterTriggerBinding.

```terminal title="Example: Delete myclusterbinding1 and myclusterbinding2 ClusterTriggerBindings"
$ tkn clustertriggerbinding delete myclusterbinding1 myclusterbinding2
```

### clustertriggerbinding describe {#_clustertriggerbinding_describe}

Describe a ClusterTriggerBinding.

```terminal title="Example: Describe the myclusterbinding ClusterTriggerBinding"
$ tkn clustertriggerbinding describe myclusterbinding
```

### clustertriggerbinding list {#_clustertriggerbinding_list}

List ClusterTriggerBindings.

```terminal title="Example: List all ClusterTriggerBindings"
$ tkn clustertriggerbinding list
```

## Hub interaction commands {#op-tkn-hub-interaction_op-tkn-reference}

Interact with Tekton Hub for resources such as tasks and pipelines.

### hub {#_hub}

Interact with hub.

```terminal title="Example: Display help"
$ tkn hub -h
```

```terminal title="Example: Interact with a hub API server"
$ tkn hub --api-server https://api.hub.tekton.dev
```

:::note

For each example, to get the corresponding sub-commands and flags, run `tkn hub <command> --help`.

:::

### hub downgrade {#_hub_downgrade}

Downgrade an installed resource.

```terminal title="Example: Downgrade the mytask task in the mynamespace namespace to its older version"
$ tkn hub downgrade task mytask --to version -n mynamespace
```

### hub get {#_hub_get}

Get a resource manifest by its name, kind, catalog, and version.

```terminal title="Example: Get the manifest for a specific version of the myresource pipeline or task from the tekton catalog"
$ tkn hub get [pipeline | task] myresource --from tekton --version version
```

### hub info {#_hub_info}

Display information about a resource by its name, kind, catalog, and version.

```terminal title="Example: Display information about a specific version of the mytask task from the tekton catalog"
$ tkn hub info task mytask --from tekton --version version
```

### hub install {#_hub_install}

Install a resource from a catalog by its kind, name, and version.

```terminal title="Example: Install a specific version of the mytask task from the tekton catalog in the mynamespace namespace"
$ tkn hub install task mytask --from tekton --version version -n mynamespace
```

### hub reinstall {#_hub_reinstall}

Reinstall a resource by its kind and name.

```terminal title="Example: Reinstall a specific version of the mytask task from the tekton catalog in the mynamespace namespace"
$ tkn hub reinstall task mytask --from tekton --version version -n mynamespace
```

### hub search {#_hub_search}

Search a resource by a combination of name, kind, and tags.

```terminal title="Example: Search a resource with a tag cli"
$ tkn hub search --tags cli
```

### hub upgrade {#_hub_upgrade}

Upgrade an installed resource.

```terminal title="Example: Upgrade the installed mytask task in the mynamespace namespace to a new version"
$ tkn hub upgrade task mytask --to version -n mynamespace
```
