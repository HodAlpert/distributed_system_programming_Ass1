# Distributed Systems Programming

### Requirements

* **awscli:**
```
sudo pip3 install --upgrade awscli
```
than ask me for credentials and use 
```
aws configure
```
to configure them

* **maven:**

[Maven](https://www.javahelps.com/2017/10/install-apache-maven-on-linux.html)- Instructions on installing maven

### Installing

in order to install the package first clone it using 

```
git clone --recursive https://github.com/hod246/distributed_system_programming_Ass1.git
```
than go to LocalApplication directory:
```
cd distributed_system_programming_Ass1/LocalApplication/
```
checkout branch master
```
git checkout master
```
install the maven project
```
mvn compile package
```

## Getting Started

start LocalApplication with input file of your selection
```
java -jar target/Local-application-1.0-SNAPSHOT.jar <inputFileName> <outputFileName> <n> <terminate (optionatl)>
```
## Security:

* Local application can be started only if access-key, secret and region was configured using aws cli command- credentials are hard-coded into the repository.
* The Manager and Worker instances has instance-profile with role which gives them permission specifically on EC2, SQS, S3, and IAM for session of 2 hours

## Scalability:

### Manager:

In order to handle threads I used _Executors.newCachedThreadPool()_.

There are two main threads submitted on initialization of the manager: **ClientListener** and **WorkerListener**.

#### ClientListener:

Recieve messages from *manager-queue.fifo* and submit a new **ReceiveNewClientTask** to the executor each time a new client arrives.

The **ReceiveNewClientTask** download the input-file from the client and submits a new **SendNewPDFTask** task to the executor on each line in the input file, which sends an SQS message via *worker-queue.fifo* to the worker.

#### WorkerListener:

Recives messages from *manager-queue-from-worker.fifo* and submit a **ReceiveNewDonePDFTask** task to the executor on each message.

Each message from the worker has uniqe client ID with which the manager knows which client requested the task.
If all tasks of this client are done- than **ReceiveNewDonePDFTask** will submit a **WrapThisUp** task which will generate summary file and send it back to the specific client.

### Worker:

The worker is a single-threaded process (which can be easily changed if needed). since the LocalApplication can decide how many workers it wants for it's tasks, worker scalability concern are up to the client.
## persistence:

The manager is a Single Point of Failure in case it crashes. but since all **ClientListener** and **WorkerListener** does is receive 
messages and submit tasks it is very unlikely to happen. 
if other thread failed- it will be submitted again, until it finishes.

In case a worker fails- than another worker will be able to take it's place in all unfinished tasks after two minutes

## Versioning

We use [SemVer](http://semver.org/) for versioning. For the versions available, see the [tags on this repository](https://github.com/your/project/tags). 

## Authors

**Name**: *Hod Alpert*

**ID**: *305452146*

See also the list of [contributors](https://github.com/your/project/contributors) who participated in this project.

## License

This project is licensed under the MIT License - see the [LICENSE.md](LICENSE.md) file for details

## Acknowledgments

* Hat tip to anyone whose code was used
* Inspiration
* etc
