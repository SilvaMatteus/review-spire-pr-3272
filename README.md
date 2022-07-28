# SPIRE Azure MSI Plugin Node Resolver into Attestor review

## 1. About this review

This review was done by the [Laboratório de Sistemas Distribuídos](https://www.lsd.ufcg.edu.br/#/) for the purpose of testing Andrew Harding's Merge Request for the [spiffe/spire repository](https://github.com/spiffe/spire/pull/3272).


This review looks at three different testing scenarios:

- SPIRE Server with Node Attestor and Node Resolver configured as normal.
- SPIRE Server with Node Attestor with configuration using MSI token.
- SPIRE Server with Node Attestor with configuration using appid+accesstoken as well as the Node Resolver.


## 2. Dependecies

* A virtual machine with Ubuntu operating system;
    * [An Azure CLI](https://docs.microsoft.com/en-us/cli/azure/install-azure-cli);
    * A valid [Golang programing language](https://go.dev/doc/install) installation;
    * [git version control](https://git-scm.com/);
    * [sqlite3](https://linuxhint.com/install-sqlite-ubuntu-linux-mint/)
    * [gcc compiler](https://linuxize.com/post/how-to-install-gcc-compiler-on-ubuntu-18-04/)
* [An Azure Account](https://azure.microsoft.com/pt-br/);


## 3. Azure MSI and SPIRE configurations

In your Azure VM, follow the commands below to create a managed identity and build the spire server/agent.

### 3.1. Enable system-assigned managed identity

To enable system-assigned managed identity on a VM that already exists, your account needs the [Virtual Machine Contributor](https://docs.microsoft.com/en-us/azure/role-based-access-control/built-in-roles#virtual-machine-contributor) role assignment.

Install Azure CLI and sign in.

```bash
sudo apt install azure-cli

az login
```

Use identity assign command to enable the system-assigned identity to an existing VM.

```bash
# Where -g myResourceGroup is the name of the resource group and -n myVm is the name of the VM
az vm identity assign -g myResourceGroup -n myVm
```

### 3.2. Build Spire Server and Spire Agent from Andrew repository.


In `nodeattestor/` directory, clone the Andrew repository and switch to the **merge-azure-resolver-into-attestor** branch made by Andrew.

```bash
git clone https://github.com/azdagron/spire
cd spire/
git checkout merge-azure-resolver-into-attestor
```

Inside the `spire/` directory run the commands below to build the Spire Server and Spire Agent.

```bash
go build ./cmd/spire-server
go build ./cmd/spire-agent
sudo cp -r ./spire-server /usr/bin/
sudo cp -r ./spire-agent /usr/bin/
```

## 4. Scenarios for Azure MSI plugin review step by step

In `nodeattestor/` directory run the commands below to follow the three scenarios.

### 4.1. SPIRE Server with Node Attestor and Node Resolver configured as normal.

<br/>

Check list:

- SPIRE Server logs contain deprecation warnings for the Node Resolver
- The attested node has the selectors produced by the Node Resolver.

<br/>

Start the Spire Server with NodeAttestor and NodeResolver.

```bash
spire-server run -config ./conf/server/server1.conf
```

Create an entry register.

```bash
spire-server entry create -node -spiffeID spiffe://example.org/agent_node -selector azure_msi:vm-name:RESOURCE_GROUP:MACHINE_NAME
```

*In order to delivery a custom SVID to the agent node it's necessary to create an entry register, this can be done using one or more selectors, [see more](https://github.com/spiffe/spire/blob/main/doc/plugin_server_noderesolver_azure_msi.md).*

Start the Spire Agent.

```bash
spire-agent run -config ./conf/agent/agent.conf
```

[Logs received](./logs/server1.log) from spire server.

[Logs received](/logs/agent.log) from spire agent.

Delete `.data` directory.

```bash
rm -rf ./.data
```

### 4.2. SPIRE Server with Node Attestor with configuration using MSI token.

<br/>

Check list:

- SPIRE Server logs contains no deprecation warnings.
- The attested node has the selectors produced by the Node Attestor.

<br/>

Start the Spire Server with NodeAttestor. Remember to put an **App Registration** credentials into `server2.conf` file in NodeAttestor.

```bash
spire-server run -config ./conf/server/server2.conf
```

Create an entry register.

```bash
spire-server entry create -node -spiffeID spiffe://example.org/agent_node -selector azure_msi:vm-name:RESOURCE_GROUP:MACHINE_NAME
```

Start the Spire Agent.

```bash
spire-agent run -config ./conf/agent/agent.conf
```

[Logs received](./logs/server2.log) from Spire Server.

[Logs received](./logs/agent2.log) from Spire Agent.


Delete `.data` directory.

```bash
rm -rf ./.data
```

### 4.3. SPIRE Server with Node Attestor with configuration using appid+accesstoken as well as the Node Resolver.

<br/>

Check list:

- SPIRE Server logs contain deprecation warnings for the Node Resolver
- The attested node has the selectors produced by the Node Attestor and Node Resolver with no duplicates.

<br/>


Start the Spire Server with NodeAttestor. Remember to put an **App Registration** credentials into `conf/server/server3.conf` file in NodeAttestor.

```bash
spire-server run -config ./conf/server/server3.conf
```

Create an entry register.

```bash
spire-server entry create -node -spiffeID spiffe://example.org/agent_node -selector azure_msi:vm-name:RESOURCE_GROUP:MACHINE_NAME
```

Start the Spire Agent.

```bash
spire-agent run -config ./conf/agent/agent.conf
```

[Logs received](./logs/server3.log) from Spire Server.

[Logs received](./logs/agent3.log) from Spire Agent.

Delete `.data` directory.

```bash
rm -rf ./.data
```