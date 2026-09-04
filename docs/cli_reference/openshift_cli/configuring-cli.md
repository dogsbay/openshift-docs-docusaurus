---
title: Configuring the OpenShift CLI
sidebar_position: 2
---

# Configuring the OpenShift CLI {#cli-configuring-cli}

<a id="cli-configuring-cli"></a>

You can customize your command-line environment after installing the OpenShift CLI (`oc`).

## Enabling tab completion {#cli-enabling-tab-completion}

You can enable tab completion for the Bash or Zsh shells.

### Enabling tab completion for Bash {#cli-enabling-tab-completion_cli-configuring-cli}

After you install the OpenShift CLI (`oc`), you can enable tab completion to automatically complete `oc` commands or suggest options when you press Tab. The following procedure enables tab completion for the Bash shell.

**Prerequisites**

- You must have the OpenShift CLI (`oc`) installed.
- You must have the package `bash-completion` installed.

**Procedure**

1. Save the Bash completion code to a file:
   ```terminal
   $ oc completion bash > oc_bash_completion
   ```
2. Copy the file to `/etc/bash_completion.d/`:
   ```terminal
   $ sudo cp oc_bash_completion /etc/bash_completion.d/
   ```

   You can also save the file to a local directory and source it from your `.bashrc` file instead. Tab completion is enabled when you open a new terminal.

### Enabling tab completion for Zsh {#cli-enabling-tab-completion-zsh_cli-configuring-cli}

After you install the OpenShift CLI (`oc`), you can enable tab completion to automatically complete `oc` commands or suggest options when you press Tab. The following procedure enables tab completion for the Zsh shell.

**Prerequisites**

- You must have the OpenShift CLI (`oc`) installed.

**Procedure**

- To add tab completion for `oc` to your `.zshrc` file, run the following command:
  ```terminal
  $ cat >>~/.zshrc<<EOF
  autoload -Uz compinit
  compinit
  if [ $commands[oc] ]; then
    source <(oc completion zsh)
    compdef _oc oc
  fi
  EOF
  ```

  Tab completion is enabled when you open a new terminal.

## Accessing kubeconfig by using the oc CLI {#cli-accessing-kubeconfig-using-cli_cli-configuring-cli}

You can use the `oc` CLI to log in to your OpenShift cluster and retrieve a kubeconfig file for accessing the cluster from the command line.

:::warning

If you plan to reuse the exported `kubeconfig` file across sessions or machines, store it securely and avoid committing it to source control.

:::

**Prerequisites**

- You have access to the OpenShift Container Platform web console or API server endpoint.

**Procedure**

1. Log in to your OpenShift cluster by running the following command:
   ```terminal
   $ oc login <api_server_url> -u <username> -p <password>
   ```

   where:

   <dl>
   <dt><code>&lt;api_server_url&gt;</code></dt>
   <dd>Specifies the full API server URL; for example, <code>https://api.my-cluster.example.com:6443</code>.</dd>
   <dt><code>&lt;username&gt;</code></dt>
   <dd>Specifies a valid username; for example, <code>kubeadmin</code>.</dd>
   <dt><code>&lt;password&gt;</code></dt>
   <dd>Specifies the password for the specified user; for example, the <code>kubeadmin</code> password generated during cluster installation.</dd>
   </dl>
2. Save the cluster configuration to a local file by running the following command:
   ```terminal
   $ oc config view --raw > kubeconfig
   ```
3. Set the `KUBECONFIG` environment variable to point to the exported file by running the following command:
   ```terminal
   $ export KUBECONFIG=./kubeconfig
   ```
4. Use `oc` to interact with your OpenShift cluster by running the following command:
   ```terminal
   $ oc get nodes
   ```
