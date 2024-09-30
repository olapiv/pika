Pika uses a multi-threaded model, using multiple worker threads to perform read and write operations. The underlying blackwidow engine ensures thread safety. There are 12 types of threads:

- PikaServer: main thread
- DispatchThread: listens to 1 port and receives user connection requests
- WorkerThread: There are multiple (user-configured), each thread has several user client connections, is responsible for receiving user commands, and then encapsulates the command into a Task and throws it to ThreadPool for execution. After the task is completed, the thread returns the reply to the user
- ThreadPool: The number of threads in the thread pool is configured by the user, and executes the Task dispatched by WorkerThread. The content of the Task is mainly writing to DB and writing to Binlog
- PikaAuxiliaryThread: auxiliary thread, handles the switching of state machine states during synchronization, sending heartbeats between master and slave, and timeout checks
- PikaReplClient: essentially an Epoll thread (communicating with PikaReplServer of other Pika instances) plus a thread array consisting of several threads (asynchronously processing the tasks of writing Binlog and writing to DB)
- PikaReplServer: It is essentially an Epoll thread (communicating with the PikaReplClient of other Pika instances) plus a thread pool consisting of several threads (processing synchronization requests and updating the Binlog sliding window according to the Ack returned from the library)
- MonitorThread: The client that executes the Monitor command will be assigned to this thread, and this thread returns the command currently being processed by Pika to the client hanging on this thread
- KeyScanThread: In this thread, the task of counting the number of keys triggered by info keyspace 1 is executed
- BgSaveThread: Dump the specified DB, and send the dump data to the slave library during full synchronization (full synchronization of a DB is to throw BgSave and DBSync tasks into the Thread in turn to ensure the order)
- PurgeThread: Used to clean up expired Binlog files
- PubSubThread: Used to support PubSub related functions
