# SF-RW_Problem

The Readers-Writers Problem is a standard problem in concurrency.The Model is described as follows,


* We have a shared memory region
* We divide processes accessing them into two types, readers who read from shared memory and writers who write in shared memory
* Any number of readers can read together
* Only one writer can write at a time
* Reading and writing should not occur together

We show the classic solution (First Readers-Writers) first , then we show how we can modify it to make it starve-free

## First Readers-Writers

#### global variables

```
semaphore mutex = 1
semaphore readmutex = 1
int readcount = 0
```

#### Readers

```                             
wait(readmutex)                      
readcount++
if (readcount == 1)
	wait(mutex)                  
signal(readmutex)

/* 
.
.
.
	Critical Section
.
.
.
*/

wait(readmutex)                      
readcount--
if (readcount == 0)
	signal(mutex)
signal(readmutex)
```

#### Writers

```
wait(mutex)	                  

/* 
.
.
.
	Critical Section
.
.
.
*/

signal(mutex)
```

#### Explanation

Here we use two binary semaphores , mutex for exclusive access to resources and readmutex for exclusive access to readcount variable.In writers process we acquire mutex directly because at a time only one writer can access critical section and no other process must access.whereas for reader multiple readers can work together , so we have a readcount variable indicating number of readers executing currently and we acquire and release mutex at first reader and last reader respectively.since modification of readcount variable must be done by only one process at a time , we use another binary semaphore readmutex for the same.

Note that in this solution , if a writer is waiting because other process is reading and readers continously keep coming , then the writer would starve.

## Starve Free Readers Writers

#### global variables


```
semaphore mutex = 1
semaphore readmutex = 1
semaphore queue = 1
int readcount = 0
```

#### Reader

```
wait(Q)                              // wait in queue 
wait(readmutex)                      // exclusive access to readcount
readcount++
if (readcount == 1)
	wait(mutex)                  //first reader,exclusive access of critical section to readers free from queue
signal(Q)
signal(readmutex)

/* 
.
.
.
	Critical Section
.
.
.
*/

wait(readmutex)                      //exclusive access to readcount
readcount--
if (readcount == 0)
	signal(mutex)
signal(readmutex)	 
```

#### Writer

```
wait(Q)                           //wait in queue
wait(mutex)	                  //exclusive access to critical section
signal(Q)

/* 
.
.
.
	Critical Section
.
.
.
*/

signal(mutex)
```

#### Explanation

The previous code suffered from starvation because even though a writer is waiting , new readers when came were allowed to move forward. So here we remedy that by adding an additional binary semaphore Q (assumed to have FIFO scheduling) which maintains the queus of processes in order.Here as soon as they start , they are waited on Q , hence maintainig the order. Note that when a writer is waiting and a reader comes, the reader would have to wait as Q is acquired by the writer process , hence bringing readers and writers on same priority.
