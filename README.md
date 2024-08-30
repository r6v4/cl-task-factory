# cl-task-factory

## Introduction
The Task Factory Pattern is a very new program design pattern,    
partly inspired by smelters, and innovation is the clever combination of simple things.    
Note that this project is not a ready-to-use package, but a program framework composed of pointer arrays and hash tables,    
which will allow you to design software that is more complicated than before but has higher concurrency and lower coupling.    
In other words, this is a more efficient and flexible framework.

## Example
```text
-----------------------------------------
|储运车间                      |安环 |营销 |
-----------------------------------------
|粗炼车间                      |安环 |    |
-----------------------------------------
|电解车间                      |安环 |    |
-----------------------------------------
|精馏车间                      |安环 |    |
-----------------------------------------
```
The core idea of ​​the Task Factory Pattern is to manage resources in a unified manner,    
disassemble the pipelines that need to wait and divide them into corresponding In the workshop,    
each position has clear responsibilities and operates independently, and workers on the same assembly line coordinate with each other.    
It can be seen that the task factory pattern is suitable for the design of asynchronous non-blocking programs.    
Below I will take the smelter as an example to explain several key points in the task factory pattern.    
There are several important workshops and departments in the smelter.    
Raw materials and semi-finished products are supplied by other companies and stored in several mine bins    
and other warehouses in the storage and transportation workshop.    
The materials are sent to the rough refining workshop for preliminary processing to remove some impurities.    
The semi-finished products are sent to the corresponding workshops according to categories.    
Lead is sent to the electrolysis workshop, and zinc is sent to the distillation workshop.    
The finished products are sent to the storage and transportation workshop.    
The storage and transportation workshop is in accordance with the marketing department.    
The production workshop produces according to the plan of the safety and environmental protection department.    
If we regard a web server as the above-mentioned factory, then the raw materials and semi-finished products are the clients,    
the finished products are our services or calculated results,    
the storage and transportation workshop is the place where the socket is input and output,    
and it also manages the client resources, and the marketing department formulates the output plan.    
The safety and environmental protection department manages the user login status to control the production plan,    
and the plan is assigned to the workers in each workshop for execution.    
The crude refining workshop parses the message input by the user to extract valid information,    
and sends the static web page request to the electrolysis workshop, and sends the dynamic web page request to the distillation workshop.    
The production workshop sends the processing to the warehouse of the storage and transportation workshop,    
and the web page waits to be sent step by step.

## Primitive
```text
-------------------------------------
|->queue|->function|->thread|->place|
-------------------------------------

--------------------------------------------------
|->epoll|->function|->thread|->hash-table|->place|
--------------------------------------------------
```
As we have said before, the framework is composed entirely of pointer arrays and hash tables, which we can use to describe the workers and grassroots managers in the factory. The factory worker is a pointer array of length four. The first item of the array is a mailbox, the second item is the function that processes the message each time, the third item is the thread that loops through its own mailbox to retrieve the message and executes the message processing function, and the fourth item is the location of the primitive. We can understand that each worker has his own position, constantly waiting for task messages in the mobile phone, and performing tasks according to the operating procedures in his own manual. I think readers may be confused. Why not merge the second and third items? Because we can let the pointer of the second item point to other functions to modify the behavior of the worker and let him complete other tasks, such as smooth upgrade or restart of the service. After the merger, the worker's work is fixed and cannot be modified in real time. It is not flexible enough. The grassroots manager is responsible for notifying the worker of the task, which is a pointer array of length five. The first item is the epoll handle, and the second item is the notification function, which determines which worker in which workshop or the grassroots manager's mobile phone will be sent the message. The third item is a thread that constantly checks the handle of the first item and passes it in with the ready fd. The second item points to the notification function, and the fourth item points to a hash table. The key of the hash table is fd, and the value is the corresponding object or structure. The workers in charge of the grassroots manager will query the data in the management information hash table, and other workshops and departments may also check it. For example, the safety and environmental protection department will check the login information in the storage and transportation workshop. The fifth item is the location of the primitive, that is, the position of the grassroots manager in the factory. 

## Rule
```text
|->[0,0]|->[0,1]|->[0,2]|->[0,3]|->[0,4]|->[0,5]|->[0,6]|->[0,7]|
|->[1,0]|->[1,1]|->[1,2]|->[1,3]|->[1,4]|->[1,5]|->[1,6]|->[1,7]|
|->[2,0]|->[2,1]|->[2,2]|->[2,3]|->[2,4]|->[2,5]|->[2,6]|->[2,7]|
|->[3,0]|->[3,1]|->[3,2]|->[3,3]|->[3,4]|->[3,5]|->[3,6]|->[3,7]|
|->[4,0]|->[4,1]|->[4,2]|->[4,3]|->[4,4]|->[4,5]|->[4,6]|->[4,7]|
````
1. Clear business. We need to establish a two-dimensional pointer array to represent the factory, and split the waiting part of the assembly line into tasks that can be successfully completed by a single worker during the execution process. Each task is a work section, and the big work is is a workshop, the total number of workshops plus one is the width of the two-dimensional pointer array, each row of the array is a workshop, and each item in the row is a primitive.
2. Identify the device. We need to know the number of CPU threads and determine the length of the two-dimensional pointer array according to the type of business. For example, I use i7-4790k, which is a single-channel 4-core 8-thread machine with 4 cores and 8 vcpus. Static web servers may be computationally intensive, so the length of the two-dimensional array is equal to the number of vcpus, which is 8. For servers for dynamic websites or io-intensive programs that provide data storage services, it is reasonable The length should be twice the number of vcpus, which is 16. If you want to keep a vcpus to handle background tasks such as the functions of the safety and environmental protection department, the reasonable length is the result of the number of vcpus minus one, multiplied by two, and then plus one, which is 15. The number of workers in the largest workshop in the factory is selected as the length of the two-dimensional pointer array.
3. Clarify the positions. Create a corresponding number of workers and grassroots managers, and point the pointers at the corresponding positions of the two-dimensional array to the primitives. Clarify the job responsibilities. Each worker operates independently and cannot go to other posts. Each basic element completes its work according to its own function.
4. Clarify the plan......
