# <center> Laboratory 2 </center>
# <center> Programming Distributed Applications </center>
## <center> Building a distributed system </center>

#### Microservices Are Distributed Systems
The original definition of a distributed system is: 
"A distributed system is a model in which components 
located on networked computers communicate and coordinate 
their actions by passing messages." (Wikipedia) And this is
exactly what happens in microservices-based architectures.

The individual services are deployed to cloud instances,
physically running somewhere as they exchange messages.
This is a big difference to how we used to build centralized applications. 
Instead of having a bunch of servers in our datacenter that 
handle all kinds of synchronization, transactions, and failover 
scenarios on our behalf, we now have individual services that evolve 
independently and aren't tied to each other. There are some fundamental
challenges that are unique to distributed computing. Among them are fault
tolerance, synchronization, self-healing, backpressure, network splits, and much more.
![proxy](proxy.png)
