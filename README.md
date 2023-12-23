# How to run a highly scalable Mastodon instance for about $200 /month on GCP

## Introduction

### Google Cloud Platform (GCP)

Google Cloud Platform (GCP) is a suite of cloud computing services provided by Google. It offers a wide range of services for computing, storage, data analytics, machine learning, networking, and other functionalities that enable businesses and developers to build, deploy, and scale applications on Google's infrastructure.

Key components and services of Google Cloud Platform include:

- **Compute Engine:** Provides virtual machines (VMs) for running applications. Users can choose from various machine types and configurations.

- **Cloud Storage:** Offers scalable and secure object storage for data, including archival, online, and backup storage.

- **Cloud SQL:** A fully managed relational database service that supports MySQL, PostgreSQL, and SQL Server.

- **Memorystore:** A fully managed in-memory data store service for building and deploying applications that require low-latency data access.

- **Cloud Networking:** Provides a range of networking services, including Virtual Private Cloud (VPC), Load Balancing, and CDN (Content Delivery Network).

- **Identity and Access Management (IAM):** A service for managing user and application permissions, allowing organizations to control access to resources securely.

GCP competes with other major cloud providers like Amazon Web Services (AWS) and Microsoft Azure, offering a comprehensive set of tools and services to meet the needs of businesses and developers in the cloud computing space.

### Docker

Docker is an open-source platform that automates the process of deploying, managing, and running applications in lightweight, portable containers. Containers are a form of virtualization that encapsulates an application and its dependencies, allowing it to run consistently across various computing environments.

Key features of Docker include:

- **Containerization:** Docker uses container technology to package an application and its dependencies into a standardized unit known as a container. This containerization allows the application to run consistently and reliably on any system that supports Docker.

- **Efficiency:** Containers share the host operating system's kernel, which makes them more resource-efficient compared to traditional virtual machines. They start quickly and consume fewer resources.

- **Docker Hub:** Docker Hub is a cloud-based registry service provided by Docker that allows users to share and distribute containerized applications. It also serves as a repository for pre-built Docker images.

- **Docker Compose:** Docker Compose is a tool that enables the definition and configuration of multi-container Docker applications. It uses a YAML file to specify the services, networks, and volumes required for an application to run.

Docker is widely used in software development and IT operations to streamline the process of building, packaging, and deploying applications. It has become a standard tool in the DevOps workflow, enabling faster development cycles, improved consistency, and easier scalability of applications.

### Postgres

PostgreSQL, often referred to as "Postgres," is an open-source relational database management system (RDBMS). It is known for its reliability, extensibility, and adherence to SQL standards. PostgreSQL supports a wide range of features that make it suitable for various types of applications, from small projects to large-scale enterprise systems.

Key features and characteristics of PostgreSQL include:

- **Open Source Relational Database Management System (RDBMS):** PostgreSQL is a robust, open-source relational database management system known for its reliability, adherence to SQL standards, and extensibility.

- **Advanced Features:** It supports advanced features such as ACID compliance, extensibility through custom data types and functions, concurrency control using MVCC, and advanced data types like arrays, JSON, and XML.

PostgreSQL is widely used in various industries and is considered a reliable choice for applications that require a powerful and extensible relational database. Many organizations appreciate its features, performance, and the fact that it is open source, allowing for easy customization and integration into different environments.

### Redis

Redis is an open-source, in-memory data structure store that is used as a caching mechanism, message broker, and general-purpose data store. It is known for its high performance, simplicity, and versatility. Redis stores data in memory, which allows for fast read and write operations, making it well-suited for use cases that require low-latency access to data.

Key features and characteristics of Redis include:

- **In-Memory Data Store:** Redis primarily stores data in RAM (Random Access Memory), enabling fast data access and retrieval. This makes it suitable for use cases where low-latency responses are critical.

- **Data Structures:** Redis supports various data structures, including strings, hashes, lists, sets, sorted sets with range queries, bitmaps, hyperloglogs, and geospatial indexes. This flexibility allows developers to model data in ways that fit specific application needs.

- **Persistence:** While Redis is an in-memory store, it provides options for persistence, allowing data to be saved to disk for durability. This feature is useful for scenarios where data needs to be retained even if the system is restarted.

- **Caching:** Redis is commonly used as a caching solution to improve the performance of applications by storing frequently accessed data in memory. Its fast read and write operations make it well-suited for caching purposes.

- **Pub/Sub Messaging:** Redis supports Publish/Subscribe messaging patterns, allowing different parts of an application or different applications to communicate asynchronously by publishing and subscribing to channels.

- **Atomic Operations:** Redis supports atomic operations on these data structures, making it suitable for building complex, multi-step operations that need to be performed without interference from other operations.

Due to its speed and versatility, Redis is commonly used in web development, real-time analytics, caching, session storage, and other scenarios where quick and efficient data access is crucial.

## Preliminary Housekeeping

Before doing anything Mastodon related, we need to first set up the infastructure that we will be using for this project. That includes setting up a new project in GCP. Creating a project involves a few simple steps. Here's a general guide:

1. **Sign In to Google Cloud Console:**

   - Open a web browser and go to the [Google Cloud Console](https://console.cloud.google.com/).
   - Sign in with your Google account. If you don't have an account, you'll need to create one.

2. **Navigate to the Cloud Console Dashboard:**

   - Once signed in, you'll be taken to the Cloud Console dashboard.

3. **Open the Project Selector:**

   - In the top bar of the Cloud Console, click on the project name. If you're just starting, it might say "Select a project."

4. **Click on "New Project":**

   - In the project selector, click on the "New Project" button.

5. **Enter Project Details:**

   - In the "New Project" window, enter a name for your project. Here we will use `Mastodon-Tutorial`
   - Optionally, you can edit the project ID, or let Google Cloud generate one for you.
   - Choose an organization if applicable.
   - Select the billing account for the project.

6. **Click "Create":**

   - Once you've entered the project details, click the "Create" button.

7. **Wait for Project Creation:**

   - Google Cloud will create your project, which may take a few moments. You'll see a notification when the project is ready.

8. **Access Your New Project:**

   - After the project is created, you can access it by clicking on the project name in the top bar and selecting the newly created project from the project selector.

9. **Create a Service Account:**
   - In the left navigation menu, go to "IAM & Admin" and then click on "Service accounts."
   - Click the "Create Service Account" button.
   - For this service account, set both the Name and Service account ID to be `mastodon-tutorial`.
   - Click "Continue" and then click "Done" to create the service account.

You've now created a new project in Google Cloud Platform with all the appropriate scaffolding. This will serve as the home for all of our cloud functionality. For the sake of consistency, the naming conventions in this tutorial will all be based on the names of the Mastodon instance we are builing, and the GCP project that it belongs to. In this case those are `mastodongcp.social` and `mastodon-tutorial` respectively.

Addtionally you will need to purchase a domain name. This can be done by using most domain service providers like GoDaddy, Porkbun, Namecheap, etc. For the sake of this tutorial we used Porkbun. In this tutorial the domain used was `mastodongcp.social`. Once you have your domain name make sure to go to respective API key management page and generate a new key. MAKE SURE TO SAVE THE KEY AND SECRET SOMEWHERE SAFE. We will be using these later.

### Reserving IP Addresses

In Google Cloud Platform (GCP), reserving IP addresses is a straightforward process. For this project we will need to reserve one external and two internal IP addresses. The first one will be for our nginx VM (responsible for communication with outside clients trying to access out Mastodon instance), while the latter two will be used for our web and streaming VMs (responsible for handling communication between our Mastodon services).

To reserve an external IP address, navigate to the GCP Console, go to the "VPC network" section, and then select "IP addresses" from the sidebar. Once there click on the blue text that reads "RESERVE EXTERNAL STATIC IP ADDRESS" near the top of the page. Here, you can reserve a new static external IP address, ensuring a persistent and unchanging address for your resources, such as load balancers or instances (which will come in handy later). Firstly name this address `mastodon-nginx`. Keep most of the other selections as is but make sure the IP version is specified as IPv4 and that the Region is `us-central1` (you can use a differnt region if it is more convenient for you but make sure that you always use that region from here on out). Once done hit the blue "RESERVE" button.

For the internal IP addresses, within the same "IP addresses" section, click on the blue text that reads "RESERVE INTERNAL STATIC IP ADDRESS" near the top of the page. Here, you can reserve an internal IP address, which is used for communication within a Virtual Private Cloud (VPC). After reserving one address, hit the blue "RESERVE" button to do the same for the other. Here is what the specifications for the two addresses should look like:

```
INTERNAL ADDRESS 1

Name: mastodon-web
Description: <LEAVE THIS EMPTY>
IP version: IPv4
Network: default
Subnetwork: us-central1
Static IP address: Let me choose
Custom IP address: 10.128.15.200
Purpose: Non-shared
```

```
INTERNAL ADDRESS 2

Name: mastodon-streaming
Description: <LEAVE THIS EMPTY>
IP version: IPv4
Network: default
Subnetwork: us-central1
Static IP address: Let me choose
Custom IP address: 10.128.15.201
Purpose: Non-shared
```

While we are in the VPC network portal, there are a couple more things to do that will make things more convenient for us down the road. The first is to create a new subnet. To do this hover over the VPC network icon and click on "VPC networks". From the list of VPC networks click on "default", then click on the "SUBNETS" tab. Near the top of the page there should be text in blue that reads "ADD SUBNET"; click that then fill out the modal with the follwing attributes.

```
Name: mastodongcp-social-proxy-only-subnet
Region: us-central1
Purpose: Regional Managed Proxy
Role: Active
IPv4 range: 10.122.0.0/23
```

Ok now ee need to set up the two new rules. To do this hover over the VPC network icon and click on "Firewall". Click on "CREATE FIREWALL RULE". After creating one rule, hit the blue "CREATE" button to do the same for the other. Here is what the specifications for the two rules should look like:

```
VPC FIREWALL RULE 1

Name: default-allow-health-check
Targets: All instances in the network
Source IPv4 ranges: 130.211.0.0/22, 35.191.0.0/16
Protocols and ports: Specified protocols and ports
TCP: 22, 3000, 4000
```

```
VPC FIREWALL RULE 2

Name: default-allow-load-balancer
Targets: All instances in the network
Source IPv4 ranges: 10.122.0.0/23
Protocols and ports: Specified protocols and ports
TCP: 3000, 4000, 80, 8080
```

### Setting Up DNS Settings

Using your domain service provider, go to the domain name that you just purchased and within its DNS settings, add two new "A" type records. For these new records, specify the host as an empty string for the first and "www" for the second record. Set the Value/Destination/Answer to the nginx external IP address that was auto generated by GCP for both then hit save/confirm.

### Cloud Storage

Now we need to create a Google Cloud Storage bucket that will be used to hold all of the media (images, videos, etc.) that users upload on our Mastodon instance. We will also need to create a bucket to maintain our automated SSL certificate renewal workflow. In this section we will be doing both, with the buckets being named `mastodongcp-social-storage` and `mastodongcp-social-config` in this tutorial. Below are the steps that we will be following:

1. **Navigate to Cloud Storage:**

   - In the Cloud Console, locate and click on the "Navigation menu" (three horizontal lines) in the upper left corner.
   - Click on the "Cloud Storage" option.

2. **Click "Create Bucket":**

   - On the Cloud Storage page, click the blue text that reads "CREATE" near the top of the page.

3. **Enter Bucket Details:**

   - In the "Create a bucket" page, enter the following details for the two buckets. After creating one bucking, hit the blue "CREATE" button to do the same for the other.

     ```
     BUCKET 1

     Name: mastodongcp-social-storage
     Location type: Region
     Region: us-central1
     Select "Standard" under "Set a default class"
     Keep "Enforce public access prevention on this bucket" selected
     Access contol: Fine-grained
     Protection tools: None
     ```

     ```
     BUCKET 2

     Name: mastodongcp-social-config
     Location type: Region
     Region: us-central1
     Select "Standard" under "Set a default class"
     Keep "Enforce public access prevention on this bucket" selected
     Access contol: Uniform
     Protection tools: None
     ```

Now that we are done creating the buckets, we will have a place to store media and also our SSL keys securely. Later on in this torial we will see these buckets play important roles.

Before closing out this section we need to create a Cloud Storage-specific service account key. To do that follow these steps:

1. **Navigate to Cloud Storage Settings Interoperability:**

   - Click the Cloud Storage icon from the sidebar to expand it. Once done, click on "Settings"
   - Once in Settings, click on the "INTEROPERABILITY" tab.

2. **Create a new Access Key:**

   - Scroll down to the section of the page with the subheader "Access keys for service accounts".
   - Click the button that reads "CREATE A KEY FOR A SERVICE ACCOUNT" and select the service account that we created earlier in the tutorial.
   - Click "CREATE KEY".

3. **Record Credentials:**

   - You should see a modal pop up titled "New service account HMAC key" that contains the the Access key and Secret for the service account. RECORD THESE SOMEWHERE SAFE. Close the modal once these values have been properly recorded.

We are _now_ officially done with this section.

### PostgreSQL Database

Setting up a PostgreSQL database in GCP involves several steps, including creating a PostgreSQL instance, configuring access, and managing the database. Below is a step-by-step guide using Google Cloud Console:

1. **Navigate to Cloud SQL:**

   - In the Cloud Console, locate and click on the "Navigation menu" (three horizontal lines) in the upper left corner.
   - Select "SQL".

2. **Create a PostgreSQL Instance:**

   - Click the "Create Instance" button.
   - Choose "PostgreSQL" as the database engine.
   - For the Instance ID use `mastodon-db`.
   - For the password use a unique string of characters that you will remember later on.
   - For the Database version use PostgreSQL 15.
   - Select "Enterprise" for the Cloud SQL edition.
   - For the Region use `us-central1`.
   - For the Zonal availability select "Single zone" and set the Primary zone to be `us-central1-b`, secondary can be kept as "Any".
   - For Machine shapes under Machine configuration select `1 vCPU, 3.75 GB`.
   - For Storage capacity under Storage select `100 GB`.
   - For Instance IP assignment under Connections select both "Private IP" and "Public IP", with Network set to `default` for the Private IP.
   - For Automated backups and point-in-time recovery under Data Protection disselect "Enable point-in-time recovery".
   - Under Query insights enable "Enable Query insights".
   - Keep everything else as is.

3. **Create the Instance:**
   - Click the blue "CREATE INSTANCE" button to create the PostgreSQL instance. This process may take a few minutes. Make sure to remember the IP address of this new instance.

Once all of the above has been completed
the last thing we need to do in this section is to create another user named `mastodon`. To do this hover over the SQL icon in the sidebar, then click on "Users". Here click the blue text that reads "ADD USER ACCOUNT". Leave mostly everything as is but set the User name to be `mastodon` and the password to be the one from before, or if you forgot it generate a new one, properly record it, then click the blue "ADD" button. Now we are done here.

### Redis Memorystore

Here is how we will go about creating a new Redis Memorystore instance in GCP:

1. **Navigate to Memorystore:**

   - In the Cloud Console search bar, type in "Memorystore" and click on the first suggested result.

2. **Create a Redis Instance:**

   - Click the blue text that reads "CREATE INSTANCE" near the top of the page.
   - For the Instance ID use `mastodongcp-social-redis`
   - For the Capacity set it to 1.
   - For Tier Selection choose Basic
   - For the Region use `us-central1`.
   - For Configure read replicas under Read Replicas, select `No read replicas (high availability only)`.
   - For Network under Set up connection, use `default`.
   - Under Connections, select "Private service access".
   - For Version under Configuration select 7.0 and for Snapshots select "Schedule Redis Database (RDB) snapshot".

3. **Create the Instance:**
   - Click the blue "CREATE INSTANCE" button to create the Redis instance. This process may take a few minutes. Make sure to remember the IP address of this new instance.

Remember to configure access control and firewall rules based on your security needs. Additionally, consider monitoring and optimizing your Redis instance for performance and reliability.

### SMTP Server

Setting up an SMTP server with Mailgun involves creating a Mailgun account, adding a domain, and configuring the SMTP settings. Here's a step-by-step guide:

#### 1. Create a Mailgun Account:

- Go to the [Mailgun website](https://www.mailgun.com/).
- Click on the "Sign Up" button and complete the registration process to create a Mailgun account.

#### 2. Add a Domain:

- After signing in, go to the Mailgun dashboard.
- Click on the "Domains" tab and then click the "Add New Domain" button.
- Enter the domain you want to use for sending emails and follow the instructions to verify ownership. This typically involves adding DNS records provided by Mailgun to your domain's DNS configuration.

#### 3. Get API Key and SMTP Credentials:

- In the Mailgun dashboard, navigate to the "Domains" tab and click on the domain you just added.
- Under the "Domain Information" section, you'll find your API Key. Save this key as you'll need it later.
- Click on the "SMTP" tab to access your SMTP credentials. Here, you'll find the SMTP server, port, username, and password.

#### 4. Configure SMTP Settings in Your Application:

- Use the SMTP server, port, username, and password provided by Mailgun to configure your application or email client.
- Here's a general example of SMTP settings:

  - **SMTP Server:** smtp.mailgun.org
  - **Port:** 2525
  - **Username:** Your Mailgun SMTP username (typically in the form 'postmaster@yourdomain.com')
  - **Password:** Your Mailgun SMTP password

#### 5. Test Sending Emails:

- Use your application or email client to send a test email using the configured Mailgun SMTP settings.
- Check the Mailgun dashboard for any error messages or logs related to the email sending process.

#### Note:

- Ensure that your DNS records are correctly configured to avoid issues with domain verification.
- Mailgun provides detailed documentation, so refer to it for any specific details or updates.

By following these steps, you should be able to set up an SMTP server with Mailgun for sending emails from your application or email client.

## Mastodon Staging VM

Now that we have done most of the stage setting here is where the fun begins! We will be creating our staging VM which will be responsible for creating the template by which our Mastodon platform works off of.

### Creating the VM

Creating our VM involves several steps. Below is how we will get this done:

1. **Navigate to Compute Engine:**

   - In the Cloud Console, locate and click on the "Navigation menu" (three horizontal lines) in the upper left corner.
   - Select "Compute Engine".

2. **Create a New VM Instance:**

   - Click the "CREATE INSTANCE" button to create a new VM instance.
   - Enter the following details in the "Create an instance" page:
     - **Name:** `mastodongcp-social-staging`
     - **Region and Zone:** `us-central1` and `us-central1-a`.
     - **Machine configuration:** Select T2D and `t2d-standard-1 (1 vCPU, 4 GB memory)`
     - **Availability policies:** Select "Standard" for the VM provisioning model.
     - **Container:** Click the "DEPLOY CONTAINER" button, when the modal pops up:
       - Set the Container image to `docker.io/tootsuite/mastodon:v4.2.3`.
       - Set Restart policy to "Never".
       - Select "Run as privileged".
     - **Boot Disk:** Click "CHANGE", then set the Operating system to `Container Optimized OS` and the Version to the latest LTS release of `Container-Optimized OS`. Once done click the blue "SELECT" button.
     - **Identity and API access:** For Access scopes select "Allow default access". Keep all the other settings as is.

3. **Create the Instance:**
   - Click the blue "CREATE" button to create the staging VM. This process may take a few minutes.

Now that we have our machine, we will use it to run the setup wizard for our Mastodon instance.

### Running the Mastodon Setup Wizard

To run the new VM we can SSH into it from the GCP Cloud Compute Console. From the list of VM instances, click on the "SSH" on the right hand side of the row for the new `mastodongcp-social-staging` VM.

Once you've been SSH'd in run the following commands in the following order to start the Mastodon setup wizard.

1. `docker run -it tootsuite/mastodon:v4.2.3 /bin/bash`
2. `cd /opt/mastodon`
3. `RAILS_ENV=production bundle exec rake mastodon:setup`

Once you do this you should see that the Mastodon setup wizard is asking you to enter the domain name for your new instance. This means that we're on the right track! For the wizard here is how you should respond to all of the prompts. Again make sure that you set things up with your respective variables for all the times '< >' are used below and not literally those strings!

```
Domain name: <Your domain name, like mastodongcp.social>
Do you want to enable single user mode?: N
Are you using Docker to run Mastodon?: Y
PostgreSQL host: <Your mastodon-db SQL instance Private IP address>
PostgreSQL port: 5432
Name of PostgreSQL database: mastodon
Name of PostgreSQL user: mastodon
Password of PostgreSQL user: <Your password for the mastodon user within the mastodon-db SQL instance>
Redis host: <Your mastodongcp-social-redis Redis instance Primary Endpoint>
Redis port: 6379
Redis password: <LEAVE THIS EMPTY>
Do you want to store uploaded files on the cloud?: Y
Provider: Google Cloud Storage
GCS bucket name: <mastodongcp-social-storage>
GCS region: us-central1
GCS access key: <Your generated Google Cloud Storage HMAC Access Key from earlier>
GCS secret key: <Your generated Google Cloud Storage HMAC Secret from earlier>
Do you want to access the uploaded files from your own domain?: N
Do you want to send e-mails from localhost?: N
SMTP server: smtp.mailgun.org
SMTP port: 2525
SMTP username: <Your Mailgun SMTP username from before, like of the email format @mastodongcp.social>
SMTP password: <Your Mailgun SMTP password>
SMTP authentication: plain
SMTP OpenSSL verify mode: none
Enable STARTTLS: auto
E-mail address to send e-mails "from": <Just hit enter here unless you want to change the default sender email>
Send a test e-mail with this configuration right now?: Y
Do you want Mastodon to periodically check for important updates and notify you? (Recommended): Y
Save configuration? Y
```

At this point the terminal will output the Mastodon instance variables. THESE ARE INCREDIBLY IMPORTANT SO SAVE THEM SOMEWHERE SAFE. The wizard is still running though and will ask you if you want to prepare the database now. Type `Y` and hit enter to say yes, so the the necessary data will be propagated over. We are officially done with the staging process and you can close the SSH terminal now. To save fees you can also stop this instance so that it does not incur further charges from Google.

### Using Outputs

The environment variables that we got from the setup wizard will be used across the instances that we will make later in this tutorial so keep them around.

## The One-Off VMs

Before we continuing we will need to set up a few one-off VMs that will foster scalability for finalized Mastodon instance. These VMs will be for our PgBouncer, Elasticsearch, and Certbot functionalities.

### PgBouncer VM

PgBouncer is a lightweight connection pooler for PostgreSQL databases, designed to improve database performance and scalability by efficiently managing database connections. It acts as an intermediary layer between client applications and PostgreSQL, allowing a limited number of connections to be shared among a larger number of clients. PgBouncer helps prevent connection overflow, optimizes resource utilization, and enhances the overall responsiveness of PostgreSQL database systems.

Below is how we will create our PgBouncer VM:

1. **Navigate to Compute Engine:**

   - In the Cloud Console, locate and click on the "Navigation menu" (three horizontal lines) in the upper left corner.
   - Select "Compute Engine".

2. **Create a New VM Instance:**

   - Click the "CREATE INSTANCE" button to create a new VM instance.
   - Enter the following details in the "Create an instance" page:
     - **Name:** `mastodongcp-social-pgbouncer`
     - **Region and Zone:** `us-central1` and `us-central1-a`.
     - **Machine configuration:** Select E2 and `e2-micro (2 vCPU, 1 core, 1 GB memory)`
     - **Availability policies:** Select "Standard" for the VM provisioning model.
     - **Container:** Click the "DEPLOY CONTAINER" button, when the modal pops up:
       - Set the Container image to `docker.io/bitnami/pgbouncer:latest`.
       - Set Restart policy to "Never".
       - Select "Run as privileged".
       - Environment variables
         - POSTGRESQL_HOST: `<Your mastodon-db SQL instance Private IP address>`
         - POSTGRESQL_USERNAME: `mastodon`
         - POSTGRESQL_DATABASE: `mastodon`
         - POSTGRESQL_PASSWORD: `<Your password for the mastodon user within the mastodon-db SQL instance>`
         - PGBOUNCER_POOL_MODE: `transaction`
         - PGBOUNCER_MAX_CLIENT_CONN: `150`
         - PGBOUNCER_DEFAULT_POOL_SIZE: `40`
         - PGBOUNCER_DATABASE: `mastodon`
     - **Boot Disk:** Click "CHANGE", then set the Operating system to `Container Optimized OS` and the Version to the latest LTS release of `Container-Optimized OS`. Once done click the blue "SELECT" button.
     - **Identity and API access:** For Access scopes select "Allow default access". Keep all the other settings as is.

3. **Create the Instance:**
   - Click the blue "CREATE" button to create the staging VM. This process may take a few minutes.

Now that we have our machine, it should automatically run based on the provisioned PgBouncer container and the preset environment variables.

### Elasticsearch VM

Elasticsearch is an open-source, distributed search and analytics engine designed for horizontal scalability and real-time data exploration. It is built on top of the Apache Lucene search library and provides a RESTful interface for indexing, searching, and analyzing large volumes of structured and unstructured data. Elasticsearch is commonly used for log and event data analysis, full-text search, and business intelligence applications.

Below is how we will create our Elasticsearch VM:

1. **Navigate to Compute Engine:**

   - We should already be here based on the previous work done towards creating the Elasticsearch VM.

2. **Create a New VM Instance:**

   - Click the "CREATE INSTANCE" button to create a new VM instance.
   - Enter the following details in the "Create an instance" page:

     - **Name:** `mastodongcp-social-es`
     - **Region and Zone:** `us-central1` and `us-central1-a`.
     - **Machine configuration:** Select E2 and `e2-small (2 vCPU, 1 core, 1 GB memory)`
     - **Availability policies:** Select "Standard" for the VM provisioning model.
     - **Container:** Click the "DEPLOY CONTAINER" button, when the modal pops up:
       - Set the Container image to `docker.io/bitnami/elasticsearch:latest`.
       - Set Restart policy to "Never".
       - Select "Run as privileged".
       - Environment variables
         - ELASTICSEARCH_CLUSTER_NAME: `mastodongcp-social`
         - ELASTICSEARCH_HEAP_SIZE: `768m`
     - **Boot Disk:** Click "CHANGE", then set the Operating system to `Container Optimized OS` and the Version to the latest LTS release of `Container-Optimized OS`. Once done click the blue "SELECT" button.
     - **Identity and API access:** For Access scopes select "Allow default access". Keep all the other settings as is.
     - **Advanced options:**

       - Management:

         - Under Metadata click on the "ADD ITEM" button.
         - Set the key to `user-data` and set the Value 1 to the following:

           ```
           #cloud-config

           bootcmd:
           - fsck.ext4 -tvy /dev/sdb
           - mkdir -p /mnt/disks/elasticsearch-data-persistence
           - mount -o discard,defaults /dev/sdb /mnt/disks/elasticsearch-data-persistence
           - chown 1001:1001 /mnt/disks/elasticsearch-data-persistence
           - sysctl -w vm.max_map_count=262144
           ```

3. **Create the Instance:**
   - Click the blue "CREATE" button to create the staging VM. This process may take a few minutes.

Now that we have our machine, it should automatically run based on the provisioned Elasticsearch container and the preset environment variables.

### Certbot VM

Certbot is a free, open-source tool for automatically managing SSL/TLS certificates, primarily used to enable secure HTTPS connections for web servers. Developed by the Electronic Frontier Foundation (EFF), Certbot simplifies the process of obtaining, renewing, and configuring SSL certificates from the Let's Encrypt certificate authority. It supports various web servers and provides a command-line interface for straightforward certificate management.

Below is how we will create our Certbot VM:

1. **Navigate to Compute Engine:**

   - We should already be here based on the previous work done towards creating the Certbot VM.

2. **Create a New VM Instance:**

   - Click the "CREATE INSTANCE" button to create a new VM instance.
   - Enter the following details in the "Create an instance" page:

     - **Name:** `mastodongcp-social-certbot`
     - **Region and Zone:** `us-central1` and `us-central1-a`.
     - **Machine configuration:** Select E2 and `e2-micro (2 vCPU, 1 core, 1 GB memory)`
     - **Availability policies:** Select "Standard" for the VM provisioning model.
     - **Boot Disk:** Click "CHANGE", then set the Operating system to `Container Optimized OS` and the Version to the latest LTS release of `Container-Optimized OS`. Once done click the blue "SELECT" button.
     - **Identity and API access:** For Access scopes select "Allow default access". Keep all the other settings as is.
     - **Advanced options:**

       - Management:

         - Under Metadata click on the "ADD ITEM" button.
         - Set the key to `user-data` and set the Value 1 to the following. KEEP IN MIND THAT THESE ARE THE COMMANDS FOR IF YOUR DNS USED . If you did not use Porkbun, you can find the respective certbot letsencrypt commands for your domain service provider at [this](https://eff-certbot.readthedocs.io/en/stable/using.html#third-party-plugins) link. Otherwise use the following:

           ```
           #cloud-config

           runcmd:
           - /usr/bin/docker pull gcr.io/google.com/cloudsdktool/google-cloud-cli:alpine
           - /usr/bin/docker pull certbot/certbot
           - /usr/bin/docker pull infinityofspace/certbot_dns_porkbun
           - docker run --rm --volume /var/lib/letsencrypt:/etc/letsencrypt --volume /var/log/letsencrypt:/var/log/letsencrypt infinityofspace/certbot_dns_porkbun:latest certonly --non-interactive --agree-tos --register-unsafely-without-email --preferred-challenges dns --authenticator dns-porkbun --dns-porkbun-key <YOUR DNS API KEY> --dns-porkbun-secret <YOUR DNS API SECRET> --dns-porkbun-propagation-seconds 60 -d "mastodongcp.social" --max-log-backups 0
           - /usr/bin/docker run --rm --volume /var/lib/letsencrypt/:/var/lib/letsencrypt/ gcr.io/google.com/cloudsdktool/google-cloud-cli:alpine gcloud storage cp --recursive /var/lib/letsencrypt/live/* gs://mastodongcp-social-config/letsencrypt
           ```

3. **Create the Instance:**

   - Click the blue "CREATE" button to create the staging VM. This process may take a few minutes.

4. **Create an Instance Schedule for the new VM:**

   - Since we want to recertify our SSL certificate periodically, we will need to create an automated on/off switch. To do this we will make a minor detour to create an Instance Schedule which will automate the process of turning on and off its respective VM. To do this click on the "INSTANCE SCHEDULES" tab.
   - Once you are on the "Instance schedules" page, click on the ttext in blue that reads "CREATE SCHEDULE" hit the blue "SUBMIT" button after filling out the popup with the following attributes:

     ```
     Name: mastodongcp-social-cron
     Region: us-central1
     Start time: 3:00 AM
     Stop time: 3:30 AM
     Time zone: <Your preferred time zone>
     Frequency: Repeat daily
     ```

   - With the instance scheduler now created, click on it and then click on the "ADD INSTANCES TO SCHEDULE" button. Click on the certbot instance to add it.

Now that we have our machine, it should automatically run based on the instance scheduler and the preset environment variables.

## Nginx VM

Nginx is an open-source web server known for its high performance and efficient handling of concurrent connections. It serves static content with low resource utilization and excels as a reverse proxy and load balancer. With its event-driven architecture, Nginx is widely used to improve web server performance, scalability, and reliability in various hosting environments. For the sake of this project it will serve as the central nexus of our Mastodon instance.

### Creating an Instance Template

The process of creating instance templates is very similar to that of creating regular instances. The only difference is that these templates serve as blueprints for the manual or automated creation of new instances with preset values. Below is how we will get this done:

1. **Navigate to Instance Templates:**

   - Since we are already in the Compute Engine portal, we jest need to hover over the Compute Engine icon.
   - From the sidebar click on "Instance templates".

2. **Create a New VM Instance:**

   - Click the "CREATE INSTANCE TEMPLATE" button to create a new VM instance.
   - Enter the following details in the "Create an instance" page:

     - **Name:** `mastodongcp-social-nginx-spot`
     - **Location:** Select "Regional" and then set the Region to `us-central1`.
     - **Machine configuration:** Select E2 and `e2-micro (2 vCPU, 1 core, 1 GB memory)`
     - **Availability policies:** Select "Spot" for the VM provisioning model.
     - **Boot Disk:** Click "CHANGE", then set the Operating system to `Container Optimized OS` and the Version to the latest LTS release of `Container-Optimized OS`. Once done click the blue "SELECT" button.
     - **Identity and API access:** For Access scopes select "Allow default access". Keep all the other settings as is.
     - **Firewall:** Select "Allow HTTP traffic" andf "Allow HTTPS traffic"
     - **Advanced options:**

       - Management:

         - Under Metadata click on the "ADD ITEM" button.
         - Set the key to `user-data` and set the Value 1 to the following (MAKE SURE TO REPLACE ALL INSTANCES OF `mastodongcp-social` WITH YOUR ACTUAL PLATFORM NAME BEFORE PASTING):

           ```
           #cloud-config

           write_files:
           - path: /var/lib/certbot/certbot.sh
               permissions: "0744"
               owner: root
               content: |
               #!/bin/bash

               while :
               do
                   sleep 1m
                   /usr/bin/docker run --rm --volume /var/lib/letsencrypt/live:/var/lib/letsencrypt/live gcr.io/google.com/cloudsdktool/google-cloud-cli:alpine gcloud storage cp --recursive gs://mastodongcp-social-config/letsencrypt/* /var/lib/letsencrypt/live
                   sleep 12h
               done
           - path: /etc/systemd/system/nginx.service
               permissions: "0644"
               owner: root
               content: |
               [Unit]
               Description=nginx service

               [Service]
               ExecStart=/usr/bin/docker run --rm --name mastodongcp-social-nginx --volume /etc/nginx/conf.d/mastodon.conf:/etc/nginx/conf.d/default.conf:ro --volume /home/mastodon/live/public:/home/mastodon/live/public:ro --volume /var/lib/letsencrypt:/etc/letsencrypt/:ro --publish 80:80 --publish 443:443 nginx
               ExecStop=/usr/bin/docker stop --time 5 mastodongcp-social-nginx
               ExecReload=/usr/bin/docker exec mastodongcp-social-nginx nginx -s reload
           - path: /etc/systemd/system/certbot.service
               permissions: "0644"
               owner: root
               content: |
               [Unit]
               Description=certbot service

               [Service]
               ExecStart=/bin/sh /var/lib/certbot/certbot.sh
           - path: /etc/nginx/conf.d/mastodon.conf
               permissions: "0644"
               owner: root
               content: |
               map $http_upgrade $connection_upgrade {
                   default upgrade;
                   ''      close;
               }

               upstream backend {
                   server 10.128.15.200:3000 fail_timeout=0;
               }

               upstream streaming {
                   # Instruct nginx to send connections to the server with the least number of connections
                   # to ensure load is distributed evenly.
                   least_conn;

                   server 10.128.15.201:4000 fail_timeout=0;
                   # Uncomment these lines for load-balancing multiple instances of streaming for scaling,
                   # this assumes your running the streaming server on ports 4000, 4001, and 4002:
                   # server 127.0.0.1:4001 fail_timeout=0;
                   # server 127.0.0.1:4002 fail_timeout=0;
               }

               proxy_cache_path /var/cache/nginx levels=1:2 keys_zone=CACHE:10m inactive=7d max_size=1g;

               server {
                   listen 80;
                   listen [::]:80;
                   server_name mastodongcp.social;
                   root /home/mastodon/live/public;
                   location /.well-known/acme-challenge/ { allow all; }
                   location / { return 301 https://$host$request_uri; }
               }

               server {
                   listen 443 ssl http2;
                   listen [::]:443 ssl http2;
                   server_name mastodongcp.social;

                   ssl_protocols TLSv1.2 TLSv1.3;

                   # You can use https://ssl-config.mozilla.org/ to generate your cipher set.
                   # We recommend their "Intermediate" level.
                   ssl_ciphers ECDHE-ECDSA-AES128-GCM-SHA256:ECDHE-RSA-AES128-GCM-SHA256:ECDHE-ECDSA-AES256-GCM-SHA384:ECDHE-RSA-AES256-GCM-SHA384:ECDHE-ECDSA-CHACHA20-POLY1305:ECDHE-RSA-CHACHA20-POLY1305:DHE-RSA-AES128-GCM-SHA256:DHE-RSA-AES256-GCM-SHA384:DHE-RSA-CHACHA20-POLY1305;

                   ssl_prefer_server_ciphers on;
                   ssl_session_cache shared:SSL:10m;
                   ssl_session_tickets off;

                   # Uncomment these lines once you acquire a certificate:
                   ssl_certificate     /etc/letsencrypt/live/mastodongcp.social/fullchain.pem;
                   ssl_certificate_key /etc/letsencrypt/live/mastodongcp.social/privkey.pem;

                   keepalive_timeout    70;
                   sendfile             on;
                   client_max_body_size 99m;

                   # root /home/mastodon/live/public;

                   gzip on;
                   gzip_disable "msie6";
                   gzip_vary on;
                   gzip_proxied any;
                   gzip_comp_level 6;
                   gzip_buffers 16 8k;
                   gzip_http_version 1.1;
                   gzip_types text/plain text/css application/json application/javascript text/xml application/xml application/xml+rss text/javascript image/svg+xml image/x-icon;

                   location / {
                   try_files $uri @proxy;
                   }

                   # If Docker is used for deployment and Rails serves static files,
                   # then needed must replace line `try_files $uri =404;` with `try_files $uri @proxy;`.
                   location = /sw.js {
                   add_header Cache-Control "public, max-age=604800, must-revalidate";
                   add_header Strict-Transport-Security "max-age=63072000; includeSubDomains";
                   try_files $uri @proxy;
                   }

                   location ~ ^/assets/ {
                   add_header Cache-Control "public, max-age=2419200, must-revalidate";
                   add_header Strict-Transport-Security "max-age=63072000; includeSubDomains";
                   try_files $uri @proxy;
                   }

                   location ~ ^/avatars/ {
                   add_header Cache-Control "public, max-age=2419200, must-revalidate";
                   add_header Strict-Transport-Security "max-age=63072000; includeSubDomains";
                   try_files $uri @proxy;
                   }

                   location ~ ^/emoji/ {
                   add_header Cache-Control "public, max-age=2419200, must-revalidate";
                   add_header Strict-Transport-Security "max-age=63072000; includeSubDomains";
                   try_files $uri @proxy;
                   }

                   location ~ ^/headers/ {
                   add_header Cache-Control "public, max-age=2419200, must-revalidate";
                   add_header Strict-Transport-Security "max-age=63072000; includeSubDomains";
                   try_files $uri @proxy;
                   }

                   location ~ ^/packs/ {
                   add_header Cache-Control "public, max-age=2419200, must-revalidate";
                   add_header Strict-Transport-Security "max-age=63072000; includeSubDomains";
                   try_files $uri @proxy;
                   }

                   location ~ ^/shortcuts/ {
                   add_header Cache-Control "public, max-age=2419200, must-revalidate";
                   add_header Strict-Transport-Security "max-age=63072000; includeSubDomains";
                   try_files $uri @proxy;
                   }

                   location ~ ^/sounds/ {
                   add_header Cache-Control "public, max-age=2419200, must-revalidate";
                   add_header Strict-Transport-Security "max-age=63072000; includeSubDomains";
                   try_files $uri @proxy;
                   }

                   location ~ ^/system/ {
                   add_header Cache-Control "public, max-age=2419200, immutable";
                   add_header Strict-Transport-Security "max-age=63072000; includeSubDomains";
                   add_header X-Content-Type-Options nosniff;
                   add_header Content-Security-Policy "default-src 'none'; form-action 'none'";
                   try_files $uri @proxy;
                   }

                   location ^~ /api/v1/streaming {
                   proxy_set_header Host $host;
                   proxy_set_header X-Real-IP $remote_addr;
                   proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
                   proxy_set_header X-Forwarded-Proto $scheme;
                   proxy_set_header Proxy "";

                   proxy_pass http://streaming;
                   proxy_buffering off;
                   proxy_redirect off;
                   proxy_http_version 1.1;
                   proxy_set_header Upgrade $http_upgrade;
                   proxy_set_header Connection $connection_upgrade;

                   add_header Strict-Transport-Security "max-age=63072000; includeSubDomains";

                   tcp_nodelay on;
                   }

                   location @proxy {
                   proxy_set_header Host $host;
                   proxy_set_header X-Real-IP $remote_addr;
                   proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
                   proxy_set_header X-Forwarded-Proto $scheme;
                   proxy_set_header Proxy "";
                   proxy_pass_header Server;

                   proxy_pass http://backend;
                   proxy_buffering on;
                   proxy_redirect off;
                   proxy_http_version 1.1;
                   proxy_set_header Upgrade $http_upgrade;
                   proxy_set_header Connection $connection_upgrade;

                   proxy_cache CACHE;
                   proxy_cache_valid 200 7d;
                   proxy_cache_valid 410 24h;
                   proxy_cache_use_stale error timeout updating http_500 http_502 http_503 http_504;
                   add_header X-Cached $upstream_cache_status;

                   tcp_nodelay on;
                   }

                   error_page 404 500 501 502 503 504 /500.html;
               }

           runcmd:
           - /usr/bin/docker pull nginx
           - /usr/bin/docker pull gcr.io/google.com/cloudsdktool/google-cloud-cli:alpine
           - /usr/bin/docker run --rm --volume /var/lib/letsencrypt/live:/var/lib/letsencrypt/live gcr.io/google.com/cloudsdktool/google-cloud-cli:alpine gcloud storage cp --recursive gs://mastodongcp-social-config/letsencrypt/* /var/lib/letsencrypt/live
           - systemctl daemon-reload
           - systemctl start nginx.service
           - systemctl start certbot.service
           ```

3. **Create the Instance:**
   - Click the blue "CREATE" button to create the Nginx VM instance template. This process may take a few minutes.

Now that we have our instance template, we will use it to create an instance group that with automate the process of creating the actual VMs periodically.

### Creating the Instance Group

An Instance Group is a managed and scalable set of VM instances that are created from a common instance template. Instance groups are designed to make it easier to manage and scale applications by providing automated and efficient ways to distribute traffic, balance workloads, and handle instance failures. Below is how we will create one for our Nginx instance template:

1. **Navigate to Instance Groups:**

   - Since we are already in the Compute Engine portal, we jest need to hover over the Compute Engine icon.
   - From the sidebar click on "Instance groups".

2. **Create a New Instance Group:**

   - Click the "CREATE INSTANCE TEMPLATE" button to create a new VM instance.
   - Enter the following details in the "Create Instance Group" page, then hit the blue "CREATE" button:
     ```
     Name: mastodongcp-social-nginx-mig
     Instance template: mastodongcp-social-nginx-spot
     Location: Multiple zones
     Region: us-central1
     Zones: us-central1-c, us-central1-a, us-central1-f, us-central1-b
     Maximum number of instances: 3
     Health check -> CREATE A HEALTH CHECK
         Health Check
         Name: mastodongcp-social-nginx-health-check
         Scope: Regional
         Region: us-central1
         Protocol: TCP
         Port: 80
         Hit SAVE
     ```

## Setting up the Web, Streaming, Sidekiq, and Sidekiq Scheduler VMs

Now that we're familiar with the workflow of creating instance groups to automate the generation of VMs using instance templates, we will do a similar thing for our web, streaming, sidekiq, and sidekiq scheduler VMs.

### Creating the Instance Templates

This process is almost identical what we dis for our Nginx VM, but there will be slight differences for each of the instance templates we are making here. These are all of the respective differences for the web, streaming, sidekiq, and sidekiq scheduler VMs vis a vis the nginx VM:

#### _web_

- **Name:** `mastodongcp-social-web-spot`
- **Location:** Select "Regional" and then set the Region to `us-central1`.
- **Machine configuration:** Select T2D and `t2d-standard-1 (1 vCPU, 4 GB memory)`
- **Availability policies:** Select "Spot" for the VM provisioning model.
- **Container:** Click the "DEPLOY CONTAINER" button, when the modal pops up:
- Set the Container image to `docker.io/tootsuite/mastodon:v4.2.3`.
- Set Restart policy to "Never".
- Select "Run as privileged".

- Arguments

  - Argument 1 = `bundle`
  - Argument 2 = `exec`
  - Argument 3 = `puma`
  - Argument 4 = `-C`
  - Argument 5 = `config/puma.rb`

- Environment variables

  - LOCAL_DOMAIN = `mastodongcp.social`
  - SINGLE_USER_MODE = `false`
  - SECRET_KEY_BASE = `<Your setup wizard SECRET_KEY_BASE>`
  - OTP_SECRET = `<Your setup wizard OTP_SECRET>`
  - VAPID_PRIVATE_KEY = `<Your setup wizard VAPID_PRIVATE_KEY>`
  - VAPID_PUBLIC_KEY = `<Your setup wizard VAPID_PUBLIC_KEY>`
  - DB_HOST = `mastodongcp-social-pgbouncer.us-central1-a.c.mastodon-tutorial.internal` (Use this format but replace the VM name, project name, and region as appropriate)
  - DB_PORT = `6432`
  - DB_NAME = `mastodon`
  - DB_USER = `mastodon`
  - DB_PASS = `<Your password for the mastodon user within the mastodon-db SQL instance>`
  - REDIS_HOST = `<Your mastodongcp-social-redis Redis instance Primary Endpoint>`
  - REDIS_PORT = `6379`
  - REDIS_PASSWORD = `<LEAVE THIS EMPTY>`
  - S3_ENABLED = `true`
  - S3_PROTOCOL = `https`
  - S3_HOSTNAME = `storage.googleapis.com`
  - S3_ENDPOINT = `https://storage.googleapis.com`
  - S3_MULTIPART_THRESHOLD = `52428800`
  - S3_BUCKET = `mastodongcp-social-storage`
  - S3_REGION = `us-central1`
  - AWS_ACCESS_KEY_ID = `<Your generated Google Cloud Storage HMAC Access Key from earlier>`
  - AWS_SECRET_ACCESS_KEY = `<Your generated Google Cloud Storage HMAC Secret from earlier>`
  - SMTP_SERVER = `smtp.mailgun.org`
  - SMTP_PORT = `2525`
  - SMTP_LOGIN = `<Your Mailgun SMTP username from before, like of the email format @mastodongcp.social>`
  - SMTP_PASSWORD = `<Your Mailgun SMTP password>`
  - SMTP_AUTH_METHOD = `plain`
  - SMTP_OPENSSL_VERIFY_MODE = `none`
  - SMTP_ENABLE_STARTTLS = `auto`
  - SMTP_FROM_ADDRESS = `Mastodon <notifications@mastodongcp.social>`
  - RAILS_LOG_LEVEL = `warn`
  - PREPARED_STATEMENTS = `false`
  - ES_ENABLED = `true`
  - ES_HOST = `mastodongcp-social-es.us-central1-a.c.mastodon-tutorial.internal` (Use this format but replace the VM name, project name, and region as appropriate)
  - ES_PORT = `9200`
  - PORT = `3000`
  - LD_PRELOAD = `libjemalloc.so.2`
  - WEB_CONCURRENCY = `0`
  - MAX_THREADS = `40`

- **Boot Disk:** Click "CHANGE", then set the Operating system to `Container Optimized OS` and the Version to the latest LTS release of `Container-Optimized OS`. Once done click the blue "SELECT" button.
- **Identity and API access:** For Access scopes select "Allow default access". Keep all the other settings as is.
- **Advanced options:**

- Management:

- Under Metadata click on the "ADD ITEM" button.
- Set the key to `google-logging-enabled` and set the Value 1 to `true`.

#### _streaming_

- **Name:** `mastodongcp-social-streaming-spot`
- **Location:** Select "Regional" and then set the Region to `us-central1`.
- **Machine configuration:** Select E2 and `e2-micro (2 vCPU, 1 core, 1 GB memory)`
- **Availability policies:** Select "Spot" for the VM provisioning model.
- **Container:** Click the "DEPLOY CONTAINER" button, when the modal pops up:
- Set the Container image to `docker.io/tootsuite/mastodon:v4.2.3`.
- Set Restart policy to "Never".
- Select "Run as privileged".

- Arguments

  - Argument 1 = `/usr/local/bin/node`
  - Argument 2 = `./streaming`

- Environment variables

  - LOCAL_DOMAIN = `mastodongcp.social`
  - SINGLE_USER_MODE = `false`
  - SECRET_KEY_BASE = `<Your setup wizard SECRET_KEY_BASE>`
  - OTP_SECRET = `<Your setup wizard OTP_SECRET>`
  - VAPID_PRIVATE_KEY = `<Your setup wizard VAPID_PRIVATE_KEY>`
  - VAPID_PUBLIC_KEY = `<Your setup wizard VAPID_PUBLIC_KEY>`
  - DB_HOST = `mastodongcp-social-pgbouncer.us-central1-a.c.mastodon-tutorial.internal` (Use this format but replace the VM name, project name, and region as appropriate)
  - DB_PORT = `6432`
  - DB_NAME = `mastodon`
  - DB_USER = `mastodon`
  - DB_PASS = `<Your password for the mastodon user within the mastodon-db SQL instance>`
  - REDIS_HOST = `<Your mastodongcp-social-redis Redis instance Primary Endpoint>`
  - REDIS_PORT = `6379`
  - REDIS_PASSWORD = `<LEAVE THIS EMPTY>`
  - S3_ENABLED = `true`
  - S3_PROTOCOL = `https`
  - S3_HOSTNAME = `storage.googleapis.com`
  - S3_ENDPOINT = `https://storage.googleapis.com`
  - S3_MULTIPART_THRESHOLD = `52428800`
  - S3_BUCKET = `mastodongcp-social-storage`
  - S3_REGION = `us-central1`
  - AWS_ACCESS_KEY_ID = `<Your generated Google Cloud Storage HMAC Access Key from earlier>`
  - AWS_SECRET_ACCESS_KEY = `<Your generated Google Cloud Storage HMAC Secret from earlier>`
  - SMTP_SERVER = `smtp.mailgun.org`
  - SMTP_PORT = `2525`
  - SMTP_LOGIN = `<Your Mailgun SMTP username from before, like of the email format @mastodongcp.social>`
  - SMTP_PASSWORD = `<Your Mailgun SMTP password>`
  - SMTP_AUTH_METHOD = `plain`
  - SMTP_OPENSSL_VERIFY_MODE = `none`
  - SMTP_ENABLE_STARTTLS = `auto`
  - SMTP_FROM_ADDRESS = `Mastodon <notifications@mastodongcp.social>`
  - RAILS_LOG_LEVEL = `warn`
  - PREPARED_STATEMENTS = `false`
  - ES_ENABLED = `true`
  - ES_HOST = `mastodongcp-social-es.us-central1-a.c.mastodon-tutorial.internal` (Use this format but replace the VM name, project name, and region as appropriate)
  - ES_PORT = `9200`
  - LD_PRELOAD = `libjemalloc.so.2`
  - PORT = `4000`

- **Boot Disk:** Click "CHANGE", then set the Operating system to `Container Optimized OS` and the Version to the latest LTS release of `Container-Optimized OS`. Once done click the blue "SELECT" button.
- **Identity and API access:** For Access scopes select "Allow default access". Keep all the other settings as is.
- **Advanced options:**

- Management:

- Under Metadata click on the "ADD ITEM" button.
- Set the key to `google-logging-enabled` and set the Value 1 to `true`.

#### _sidekiq_

- **Name:** `mastodongcp-social-sidekiq-spot`
- **Location:** Select "Regional" and then set the Region to `us-central1`.
- **Machine configuration:** Select T2D and `t2d-standard-1 (1 vCPU, 4 GB memory)`
- **Availability policies:** Select "Spot" for the VM provisioning model.
- **Container:** Click the "DEPLOY CONTAINER" button, when the modal pops up:
- Set the Container image to `docker.io/tootsuite/mastodon:v4.2.3`.
- Set Restart policy to "Never".
- Select "Run as privileged".

- Arguments

  - Argument 1 = `bundle`
  - Argument 2 = `exec`
  - Argument 3 = `sidekiq`
  - Argument 4 = `-c`
  - Argument 5 = `20`
  - Argument 6 = `-q`
  - Argument 7 = `default,8`
  - Argument 8 = `-q`
  - Argument 9 = `push,6`
  - Argument 10 = `-q`
  - Argument 11 = `ingress,4`
  - Argument 12 = `-q`
  - Argument 13 = `pull,1`

- Environment variables

  - LOCAL_DOMAIN = `mastodongcp.social`
  - SINGLE_USER_MODE = `false`
  - SECRET_KEY_BASE = `<Your setup wizard SECRET_KEY_BASE>`
  - OTP_SECRET = `<Your setup wizard OTP_SECRET>`
  - VAPID_PRIVATE_KEY = `<Your setup wizard VAPID_PRIVATE_KEY>`
  - VAPID_PUBLIC_KEY = `<Your setup wizard VAPID_PUBLIC_KEY>`
  - DB_HOST = `mastodongcp-social-pgbouncer.us-central1-a.c.mastodon-tutorial.internal` (Use this format but replace the VM name, project name, and region as appropriate)
  - DB_PORT = `6432`
  - DB_NAME = `mastodon`
  - DB_USER = `mastodon`
  - DB_PASS = `<Your password for the mastodon user within the mastodon-db SQL instance>`
  - REDIS_HOST = `<Your mastodongcp-social-redis Redis instance Primary Endpoint>`
  - REDIS_PORT = `6379`
  - REDIS_PASSWORD = `<LEAVE THIS EMPTY>`
  - S3_ENABLED = `true`
  - S3_PROTOCOL = `https`
  - S3_HOSTNAME = `storage.googleapis.com`
  - S3_ENDPOINT = `https://storage.googleapis.com`
  - S3_MULTIPART_THRESHOLD = `52428800`
  - S3_BUCKET = `mastodongcp-social-storage`
  - S3_REGION = `us-central1`
  - AWS_ACCESS_KEY_ID = `<Your generated Google Cloud Storage HMAC Access Key from earlier>`
  - AWS_SECRET_ACCESS_KEY = `<Your generated Google Cloud Storage HMAC Secret from earlier>`
  - SMTP_SERVER = `smtp.mailgun.org`
  - SMTP_PORT = `2525`
  - SMTP_LOGIN = `<Your Mailgun SMTP username from before, like of the email format @mastodongcp.social>`
  - SMTP_PASSWORD = `<Your Mailgun SMTP password>`
  - SMTP_AUTH_METHOD = `plain`
  - SMTP_OPENSSL_VERIFY_MODE = `none`
  - SMTP_ENABLE_STARTTLS = `auto`
  - SMTP_FROM_ADDRESS = `Mastodon <notifications@mastodongcp.social>`
  - RAILS_LOG_LEVEL = `warn`
  - PREPARED_STATEMENTS = `false`
  - ES_ENABLED = `true`
  - ES_HOST = `mastodongcp-social-es.us-central1-a.c.mastodon-tutorial.internal` (Use this format but replace the VM name, project name, and region as appropriate)
  - ES_PORT = `9200`
  - DB_POOL = `20`
  - MALLOC_ARENA_MAX = `2`
  - LD_PRELOAD = `libjemalloc.so.2`

- **Boot Disk:** Click "CHANGE", then set the Operating system to `Container Optimized OS` and the Version to the latest LTS release of `Container-Optimized OS`. Once done click the blue "SELECT" button.
- **Identity and API access:** For Access scopes select "Allow default access". Keep all the other settings as is.
- **Advanced options:**

- Management:

- Under Metadata click on the "ADD ITEM" button.
- Set the key to `google-logging-enabled` and set the Value 1 to `true`.

#### _sidekiq scheduler_

- **Name:** `mastodongcp-social-sidekiq-scheduler-spot`
- **Location:** Select "Regional" and then set the Region to `us-central1`.
- **Machine configuration:** Select E2 and `e2-micro (2 vCPU, 1 core, 1 GB memory)`
- **Availability policies:** Select "Spot" for the VM provisioning model.
- **Container:** Click the "DEPLOY CONTAINER" button, when the modal pops up:
- Set the Container image to `docker.io/tootsuite/mastodon:v4.2.3`.
- Set Restart policy to "Never".
- Select "Run as privileged".

- Arguments

  - Argument 1 = `bundle`
  - Argument 2 = `exec`
  - Argument 3 = `sidekiq`
  - Argument 4 = `-c`
  - Argument 5 = `2`
  - Argument 6 = `-q`
  - Argument 7 = `mailers`
  - Argument 8 = `-q`
  - Argument 9 = `scheduler`

- Environment variables

  - LOCAL_DOMAIN = `mastodongcp.social`
  - SINGLE_USER_MODE = `false`
  - SECRET_KEY_BASE = `<Your setup wizard SECRET_KEY_BASE>`
  - OTP_SECRET = `<Your setup wizard OTP_SECRET>`
  - VAPID_PRIVATE_KEY = `<Your setup wizard VAPID_PRIVATE_KEY>`
  - VAPID_PUBLIC_KEY = `<Your setup wizard VAPID_PUBLIC_KEY>`
  - DB_HOST = `mastodongcp-social-pgbouncer.us-central1-a.c.mastodon-tutorial.internal` (Use this format but replace the VM name, project name, and region as appropriate)
  - DB_PORT = `6432`
  - DB_NAME = `mastodon`
  - DB_USER = `mastodon`
  - DB_PASS = `<Your password for the mastodon user within the mastodon-db SQL instance>`
  - REDIS_HOST = `<Your mastodongcp-social-redis Redis instance Primary Endpoint>`
  - REDIS_PORT = `6379`
  - REDIS_PASSWORD = `<LEAVE THIS EMPTY>`
  - S3_ENABLED = `true`
  - S3_PROTOCOL = `https`
  - S3_HOSTNAME = `storage.googleapis.com`
  - S3_ENDPOINT = `https://storage.googleapis.com`
  - S3_MULTIPART_THRESHOLD = `52428800`
  - S3_BUCKET = `mastodongcp-social-storage`
  - S3_REGION = `us-central1`
  - AWS_ACCESS_KEY_ID = `<Your generated Google Cloud Storage HMAC Access Key from earlier>`
  - AWS_SECRET_ACCESS_KEY = `<Your generated Google Cloud Storage HMAC Secret from earlier>`
  - SMTP_SERVER = `smtp.mailgun.org`
  - SMTP_PORT = `2525`
  - SMTP_LOGIN = `<Your Mailgun SMTP username from before, like of the email format @mastodongcp.social>`
  - SMTP_PASSWORD = `<Your Mailgun SMTP password>`
  - SMTP_AUTH_METHOD = `plain`
  - SMTP_OPENSSL_VERIFY_MODE = `none`
  - SMTP_ENABLE_STARTTLS = `auto`
  - SMTP_FROM_ADDRESS = `Mastodon <notifications@mastodongcp.social>`
  - RAILS_LOG_LEVEL = `warn`
  - PREPARED_STATEMENTS = `false`
  - ES_ENABLED = `true`
  - ES_HOST = `mastodongcp-social-es.us-central1-a.c.mastodon-tutorial.internal` (Use this format but replace the VM name, project name, and region as appropriate)
  - ES_PORT = `9200`
  - DB_POOL = `2`
  - MALLOC_ARENA_MAX = `2`
  - LD_PRELOAD = `libjemalloc.so.2`

- **Boot Disk:** Click "CHANGE", then set the Operating system to `Container Optimized OS` and the Version to the latest LTS release of `Container-Optimized OS`. Once done click the blue "SELECT" button.
- **Identity and API access:** For Access scopes select "Allow default access". Keep all the other settings as is.
- **Advanced options:**

- Management:

- Under Metadata click on the "ADD ITEM" button.
- Set the key to `google-logging-enabled` and set the Value 1 to `true`.

### Creating the Instance Groups

Creating the respective instance groups for each of these new instance templates is also fairly straightforward following the workflow that we used for Nginx, but below are some of the changes to take note of. Enter the following details in the "Create Instance Group" page, then hit the blue "CREATE" button:

#### _web_

```
Name: mastodongcp-social-web-mig
Instance template: mastodongcp-social-web-spot
Location: Multiple zones
Region: us-central1
Zones: us-central1-c, us-central1-a, us-central1-f, us-central1-b
Maximum number of instances: 3
Health check -> CREATE A HEALTH CHECK
    Health Check
    Name: mastodongcp-social-web-health-check
    Scope: Regional
    Region: us-central1
    Protocol: TCP
    Port: 3000
    Hit SAVE
```

#### _streaming_

```
Name: mastodongcp-social-streaming-mig
Instance template: mastodongcp-social-streaming-spot
Location: Multiple zones
Region: us-central1
Zones: us-central1-c, us-central1-a, us-central1-f, us-central1-b
Maximum number of instances: 3
Health check -> CREATE A HEALTH CHECK
    Health Check
    Name: mastodongcp-social-streaming-health-check
    Scope: Regional
    Region: us-central1
    Protocol: TCP
    Port: 4000
    Hit SAVE
```

#### _sidekiq_

```
Name: mastodongcp-social-sidekiq-mig
Instance template: mastodongcp-social-sidekiq-spot
Location: Multiple zones
Region: us-central1
Zones: us-central1-c, us-central1-a, us-central1-f, us-central1-b
Maximum number of instances: 3
Health check -> CREATE A HEALTH CHECK
    Health Check
    Name: mastodongcp-social-sidekiq-health-check
    Scope: Regional
    Region: us-central1
    Protocol: TCP
    Port: 22
    Hit SAVE
```

#### _sidekiq scheduler_

```
Name: mastodongcp-social-sidekiq-mig
Instance template: mastodongcp-social-sidekiq-spot
Location: Multiple zones
Region: us-central1
Zones: us-central1-c, us-central1-a, us-central1-f, us-central1-b
Maximum number of instances: 3
Health check: mastodongcp-social-sidekiq-health-check
```

## Load Balancer

### Creating New Load Balancers

One of the last things we will need to do is to create load balancers that will allow the static IP addresses we made from before point to the ehpemeral spot VMs that are periodically created and destroyed by the instance groups. Lets start with creating the external IP load balancer for Nginx:

1. Navigate to Load Balancing:

- In the Cloud Console search bar, type in "Load balancing" and click on the first suggested result.
- Once on the "Load balancing" page, click on the blue text that reads "CREATE LOAD BALANCER" near the top of the page.
- Click on the the blue text that reads "START CONFIGURATION" near the bottom of the card labeled "Network Load Balancer (TCP/SSL)"
- On this new page before hitting the blue "CONTINUE" button, select "Single region only" under the "Multiple regions or single region" section of the page.
- On the "Create external passthrough Network Load Balancer" page, set Load Balancer name to `mastodongcp-social-nginx-load-balancer` and the Region to `us-central1`.

2. Backend configuration:

- Within the "New backend" card, set the Instance group to `mastodongcp-social-nginx-mig`
- Set the Health check to be `mastodongcp-social-nginx-health-check`
- Set the Session affinity to `Client IP`.

3. Frontend configuration:

- Within the "New Frontend IP and port" card, set the Name to `mastodongcp-social-nginx-load-balancer-forwarding-rule`, the IP address to `mastodon-nginx`, and then select "Multiple" under Ports and type in `80,443`.
- Hit "DONE".

4. Review and Create:

- Review your load balancer configuration.
- Click the blue "CREATE" button to create the load balancer.

Now that we have established our public static IP address for our Nginx load balancer, we need to do a similar thing for our private static IP addresses to facilitate communication between our web and streaming VMs. When you are at the "Create a load balancer" page in the process of creating a load balancer for an internap IP address, select "Only between my VMs" from under the Internet facing or internal only section this time. Keep using "Single region only" as well though. This process is almost identical to the previous one, but with these changes for the load balancers of web and streaming respectively:

```
WEB LOAD BALANCER

Load Balancer name: mastodongcp-social-web-load-balancer
Region: us-central1
Network: default

Backend configuration:
Instance group: mastodongcp-social-web-mig
Health check: mastodongcp-social-web-health-check
Session affinity: None

Frontend configuration:
Name: mastodongcp-social-web-load-balancer-forwarding-rule
Subnetwork: default
IP address: Ephemeral (Custom)
Custom empemeral IP address: 10.128.15.200
Ports: Single
Port number: 3000
```

```
STREAMING LOAD BALANCER

Load Balancer name: mastodongcp-social-streaming-load-balancer
Region: us-central1
Network: default

Backend configuration:
Instance group: mastodongcp-social-streaming-mig
Health check: mastodongcp-social-streaming-health-check
Session affinity: None

Frontend configuration:
Name: mastodongcp-social-streaming-load-balancer-forwarding-rule
Subnetwork: default
IP address: Ephemeral (Custom)
Custom empemeral IP address: 10.128.15.201
Ports: Single
Port number: 4000
```

After you do this you should have the three load balancers fully functional, and our work here is done.

## Final Remarks

If you have followed all of the instructions corroectly you should be able to access your Mastodon instance by going to the domain name from your browser. Congrats!
