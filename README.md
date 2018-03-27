# Kubernetes on IBM Cloud: Building a Cloud Native Application

In this Code Pattern, we will build a dummy digital bank composed of a set of microservices that communicate with each other. We'll be using Node.js, Express, MongoDB, and the IBM Cloud Container Service.

Development of [cloud native apps](https://www.cncf.io/blog/2017/05/15/developing-cloud-native-applications/) that are broken down into a set of [microservices](http://microservices.io/) has been praised and commended as best-practice in software development methodologies. Software stacks like [Kubernetes](https://kubernetes.io/), which enable cloud native computing, have therefore picked up quite a bit of popularity.

It’s a little _(a lot)_ more fun, however, to try build a so-called cloud native app, than to talk about one.

So here's our attempt:

We’ll take a use-case that has a bit of real-world familiarity to it — A digital bank. Naturally inspired by [Monzo](http://monzo.com/). Let’s call it Innovate.

[A live version deployed on a kubernetes cluster in IBM Cloud is available for you try here](http://ibm.biz/digibank).
To test it out, sign up for an account. A process runs periodically to dump randomized transactions and bills for user accounts, so give it a couple of minutes and refresh to see your populated profile. Otherwise, you can insert, modify & delete your own transactions, bills or accounts by directly invoking the APIs described in the [docs](#docs).

![Screens](docs/screens-1.png)

![Screens](docs/screens-2.png)

When you've completed this Code Pattern, you will understand how to:

* Break an application down to a set of microservices
* Create and manage a Kubernetes cluster on IBM Cloud
* Deploy to a Kubernetes cluster on IBM Cloud
* Deploy to IBM Cloud Private

## Flow

When thinking of business capabilities, our imaginary bank will need the following set of microservices:

1. *Portal:* Loads the UI and takes care of user sessions. Relies on all other microservices for core functionality.
2. *Authentication:* Handles user profile creation, as well as login & logout.
3. *Accounts:* Handles creation, management, and retrieval of a user’s banking accounts.
4. *Transactions:* Handles creation and retrieval of transactions made against users' bank accounts.
5. *Bills:* Handles creation, payment, and retrieval of bills.
6. *Support:* Handles communication with Watson Conversation to enable a support chat feature.

![Demo architecture](docs/flow.png)

## Included components

* [IBM Cloud Container Service](https://console.bluemix.net/docs/containers/container_index.html): IBM Bluemix Container Service manages highly available apps inside Docker containers and Kubernetes clusters on the IBM Cloud.
* [Kubernetes Cluster](https://console.bluemix.net/docs/containers/container_index.html): Create and manage your own cloud infrastructure and use Kubernetes as your container orchestration engine.
* [Microservice Builder](https://developer.ibm.com/microservice-builder): Learn, build, run, and manage applications in a microservices framework.
* [Watson Conversation](https://www.ibm.com/watson/developercloud/conversation.html): Create a chatbot with a program that conducts a conversation via auditory or textual methods.

## Featured technologies

* [Microservices](https://www.ibm.com/developerworks/community/blogs/5things/entry/5_things_to_know_about_microservices?lang=en): Collection of fine-grained, loosely coupled services using a lightweight protocol to provide building blocks in modern application composition in the cloud.
* [Node.js](https://nodejs.org/): An open-source JavaScript run-time environment for executing server-side JavaScript code.
* [Containers](https://www.ibm.com/cloud-computing/bluemix/containers): Virtual software objects that include all the elements that an app needs to run.
* [Databases](https://en.wikipedia.org/wiki/IBM_Information_Management_System#.22Full_Function.22_databases): Repository for storing and managing collections of data.
* [Hybrid Integration](https://www.ibm.com/cloud-computing/bluemix/hybrid-architecture): Enabling customers to draw on the capabilities of public cloud service providers while using private cloud deployment for sensitive applications and data.

# Watch the Video

If you want a quick walkthrough of the end result, a video is available [here](https://ibm.box.com/s/fgpqiacn9ewaorgp8l97bnrk760s8rx3)

# Deploy to IBM Cloud
> NOTE: This guide requires a paid/upgraded account on IBM Cloud. You **cannot** complete the steps with a free or lite account

1. Get the tools
2. Clone the repo
3. Login to IBM Cloud
4. Create a cluster
5. Create an instance of MongoDB
6. Configure your deploy target
7. Configure your environment variables
8. Configure kubectl
9. Initialize helm
10. Deploy

### 1. Get the tools
You'll need each of the following pre-requisits:
* [IBM Cloud Developer Tools CLI](https://console.bluemix.net/docs/cli/reference/bluemix_cli/get_started.html#getting-started)

* the [Kubernetes CLI](https://kubernetes.io/docs/user-guide/prereqs/)

* the container service plugin

once you've downloaded the IBM Cloud Developer Tools CLI, in a terminal, run:

```
$ bx plugin install container-service -r Bluemix
```

* the container registry plugin

in a terminal, run:

```
$ bx plugin install container-registry -r Bluemix
```
### 1. Clone the repo
Clone the `innovate-digital-bank` repository locally. In a terminal, run:

```
$ git clone https://github.com/amalamine/innovate-digital-bank.git
```
### 3. Login to IBM Cloud
Both through the [console](https://console.bluemix.net/) and your terminal
> NOTE: If you need to specify the region you want to deploy in, you can do so by adding the -a flag followed by the region url.

```
$ bx login
```

### 4. Create a cluster

From the catalog, find **Containers in Kubernetes Clusters** and click create. Choose a region and a cluster type, and create your cluster. Allow it some time to deploy.

![kubectl config](docs/9.png)

### 5. Create an instance of MongoDB

This demo heavily depends on mongo as a session & data store.

From the [catalog](https://console.bluemix.net/catalog/), find **Compose for MongoDB** and click create. Give it a name, choose a region, pick the standard pricing plan and click create.

**Get your mongo connection string. Almost all your microservices need it; keep it safe!**

![kubectl config](docs/11.png)


## Configuring your Application

### Setting your deploy target
Each of the 8 docker images needs to be pushed to your docker image registry on IBM Cloud. You need to set the correct _**deploy target**_.
Your url will be in the following format

```
registry.<REGION_ABBREVIATION>.bluemix.net/<YOUR_NAMESPACE>/<YOUR_IMAGE_NAME>
```

For example, to deploy the accounts microservice to my docker image registry in the US-South region, my deploy_target will be:

```
registry.ng.bluemix.net/amalamine/innovate-accounts
```
#### to get your namespace, run:

```
bx cr namespace-list
```

You can also add a namespace by running:

```
bx cr namespace-add <NAME>
```

From the directory of each microservice, replace the deploy target in ***cli-config.yml*** & in ***/chart/innovate-<MICROSERVICE_NAME>/values.yaml*** with the correct one

For example, from within the /innovate folder, navigate into the accounts folder

```
cd accounts
```

Next, edit line 58 of [cli-config.yaml](https://github.com/aamine0/innovate-digital-bank/blob/master/accounts/cli-config.yml) file. Replace the ***deploy-image-target*** with the correct value.

```
deploy-image-target: "registry.ng.bluemix.net/amalamine/innovate-accounts"
```

![kubectl config](docs/12.png)

Edit line 6 of the [values.yaml](https://github.com/aamine0/innovate-digital-bank/blob/master/accounts/chart/innovate-accounts/values.yaml) file. Replace the ***repository*** with the correct value.

```
repository: registry.ng.bluemix.net/amalamine/innovate-accounts
```

![kubectl config](docs/13.png)

#### Repeat these steps for all microservices.

### Setting your environment variables
Each of the 8 microservices must have a _**.env**_ file.

An example is already provided within each folder. From the directory of each microservice, copy the example file, rename it to _**.env**_, and fill it with the appropriate values.

For example, from within the /innovate folder, navigate into the accounts folder

```
cd accounts
```

Next, copy and rename the _**.env.example**_ folder

```
cp .env.example .env
```

Finally, edit your .env folder and add your Mongodb connection string

#### Repeat these steps for all microservices. In addition to your mongo url, most will need the public IP address of your kubernetes cluster, _You can find that under the overview of your cluster on IBM Cloud_.

## Deploying all Components
#### 1. Login to IBM Cloud
Specify the region you've deployed your cluster in
```
bx login -a https://api.<REGION_ABBREVIATION>.bluemix.net
```

#### 2. Configure kubectl
Run the following command:

```
bx cs cluster-config <YOUR_CLUSTER_NAME>
```

Then copy the output and paste it in your terminal

#### 3. Initialize helm

```
helm init
```

#### 4. Deploy
Finally, navigate to each microservice, and run the following command
```
bx dev deploy
```

# Guide: Deploying on IBM Cloud Private
> NOTE: These steps are only needed when deploying to IBM Cloud Private instead of IBM Cloud Platform.

## Creating an instance of MongoDB
This demo heavily depends on mongo as a session & data store.

#### 1. Create a persistent volume
Give it a name and a capacity, choose storage type _**Hostpath**_, and add a _**path parameter**_

![Persistent Volume](docs/1.png)

#### 2. Create a persistent volume claim
Give it a name and a storage request value

![Persistent Volume Claim](docs/2.jpg)

#### 3. Create and configure mongo
From the catalog, choose MongoDb. Give it a **_name_**, specify the **_existing volume claim name_**, and give it a *_password_*

![Mongo](docs/3.jpg)

![Mongo](docs/4.jpg)

#### 4. Get your mongo connection string
Your mongo connection string will be in the following format:
```
mongodb://<USERNAME>:<PASSWORD>@<HOST>:<PORT>/<DATABASE_NAME>
```

Almost all your microservices need it; keep it safe!

## Configuring your Environment Variables
Each of the 8 microservices must have a _**.env**_ file.

An example is already provided within each folder. From the directory of each microservice, copy the example file, rename it to _**.env**_, and fill it with the appropriate values.

For example, from within the /innovate folder, navigate into the accounts folder

```
cd accounts
```

Next, copy and rename the _**.env.example**_ folder

```
cp .env.example .env
```

Finally, edit your .env folder and add your Mongodb connection string

#### Repeat those steps for all microservices. In addition to your mongo url, the portal microservice will need the address of your ICP.

## Deploying all Components
#### 1. Add your ICP's address to your hosts file
Add an entry to your /etc/hosts file as follows

```
<YOUR_ICP_IP_ADDRESS> mycluster.icp
```

#### 2. Login to docker

```
docker login mycluster.icp:8500
```

#### 3. Configure kubectl
From your ICP's dashboard, copy the kubectl commands under admin > configure client

![kubectl config](docs/5.png)

#### 4. Deploy
Finally, navigate to each microservice, and run the following command
```
bx dev deploy
```
_If you don't have the IBM Cloud Developer Tools CLI installed, get it [here](https://console.bluemix.net/docs/cli/reference/bluemix_cli/download_cli.html) first_

## (Optional) Adding Support with Watson Conversation
The support microservice connects to an instance of Watson Conversation on IBM Cloud to simulate a chat with a virtual support agent.

#### 1. Create an instance of Watson Conversation
From the [IBM Cloud Catalog](bluemix.net), choose Watson Conversation, and click create.

![Watson Conversation](docs/6.png)

#### 2. Import the support workspace
Import the [support workspace](/support/conversation-workspace.json) into your newly created Watson Conversation instance

![Watson Conversation](docs/7.png)

#### 3. Get your credentials
Navigate to the deploy tab and copy your username, password, and workspace ID

![Watson Conversation](docs/8.png)

#### 4. Edit your .env file
From within the support folder, edit your .env to include your newly acquired credentials

#### 5. Deploy
Redeploy the microservice, the support feature should now be accessible through the portal.

```
bx dev deploy
```

# Docs

## Microservices

### Portal [3000:30060]

Loads the UI and takes care of user sessions. Communicates with all other microservices.

### Authentication [3200:30100]

Handles user profile creation, as well as login & logout.

#### Endpoints:

##### /api/user/create

Description: Creates a new user account

Method: POST

Example input:

```
{
  uuid: String,
  name: String,
  email: String,
  phone: String,
  gender: String,
  dob: String,
  eid: String,
  password: String
}
```

##### /api/user/authenticate

Description: Authenticates a user

Method: POST

Example input:

```
{
  email: String,
  password: String
}
```

##### /api/user/get

Description: Returns a list of all users

Method: GET

### Accounts [3400:30120]

Handles creation, management, and retrieval of a user's banking accounts.

#### Endpoints:

##### /api/accounts/create

Description: Creates a new user account

Method: POST

Example input:

```
{
  uuid: String,
  type: String,
  currency: String,
}
```
Notes:

The parameter uuid links the account to a user's unique identifier. Type has to be one of the following: current, savings, credit, prepaid

##### /api/accounts/get

Description: Retrieves a user's accounts

Method: POST

Example input:

```
{
  uuid: String
}
```

##### /api/accounts/deposit

Description: Deposits an amount to a user's account

Method: POST

Example input:

```
{
  number: String,
  amount: Number
}
```

Notes:

The parameter number references an account

##### /api/accounts/withdraw

Description: Withdraws an amount from a user's account

Method: POST

Example input:

```
{
  number: String,
  amount: Number
}
```

##### /api/accounts/drop

Description: Drops the accounts collection

Method: GET

### Transactions [3600:30140]

Handles creation and retrieval of transactions

#### Endpoints:

##### /api/transactions/create

Description: Creates a new transaction

Method: POST

Example input:

```
{
  uuid: String,
  amount: String,
  currency: String,
  description: String,
  date: String,
  category: String
}
```

##### /api/transactions/get

Description: Retrieves a user's transactions

Method: POST

Example input:

```
{
  uuid: String
}
```

##### /api/transactions/drop

Description: Drops the transactions collection

Method: GET

### Bills [3800:30160]

Handles creation, payment, and retrieval of bills

#### Endpoints:

##### /api/bills/create

Description: Creates a new bill

Method: POST

Example input:

```
{
  uuid: String,
  category: String,
  entity: String,
  account_no: String,
  amount: String,
  date: String
}
```

##### /api/bills/get

Description: Retrieves a user's bills

Method: POST

Example input:

```
{
  uuid: String
}
```

##### /api/bills/drop

Description: Drops the bills collection

Method: GET

### Support [4000:30180]

Handles communication with Watson Conversation on IBM Cloud to enable a dummy support chat feature.

### Userbase [3600:30140]

Simulates a fake userbase for the app. Periodically loops through all user accounts and adds randomized bills and transactions for them.
