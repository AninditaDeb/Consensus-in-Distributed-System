# Consensus-in-Distributed-System

Goal - To design a fault tolerable system where a cluster of servers are working in consensus.Used Raft Algorithm to achieve consensus between the nodes.

1)Using Python Flask Framework and Python Import Request package designed a application which allows user to connect and query the database for an ecommerce system using GET, POST, PUT, PATCH and DELETE HTTP requests.

2)The first phase of the project consists of a single node that is acting as both the server and the client.

3)This node is then scaled up to a cluster of 5 nodes, created a docker image built on the top of the application in step1 and 5 different containers using Docker Compose Yaml file which were using the same docker image but were binded to five different ports and these conatiners are comunicating using UDP socket programming.

4)In Raft there two main functionalitues using which consensus is achieved first is leader election using log matching property and the seond property manages commiting a client request safe by the leader only after all the other servers present in the cluster are brought into consensus to the leaders log.

5)In Raft there are three possible states of a node, one is it could be  a leader - a leader is the one as the name suggests is gin=ving instructions to the other nodes present in the cluster, a follower - a follower is a passive state in which the node just follows and executes the commands of the leader and third state is of a candidate, a candidate state is a very important one and it comes into existence only when a leader election takes place.A leader election can happen at the very begin of the cluster formation when all the nodes start as followers and none of them is the leader amongst them, or when a follower node finds out that the leader has died.
So, Basically when a node becomes a leader it sends Append Entry RPC's to the other nodes present in the cluster to establish its authority over others and at the same time the follower nodes continue to wait for the heart beats for a particluar period of time.If the leader crashes then the follower doesn't receive this heart beat within a that time period that is supposed to receive and then some of the followers will time out and switch their state from follower to candidate.

6)Two main implementations - 
Leader election - So when a leader dies or a cluster has just been formed some of the follower nodes will switch their state to candiadte as expalined above and start sending Reuqest vote RPC's to the other follower nodes which are yet to time out(RAFT uses randomized time outs for each of the follower nodes)to get votes in their favor to win the election,its an equally important for the follower nodes to elect one the candidates as the leader.So,Request vote RPC contains the candidate ID and the most recent client request that the candidate carries in its log, the follower tries to identify whether the candidates log is atleast as uptodate as that of the follower's by using the candidate log ID(present in the Request vote RPC) or if it is more updated than the candidate's , in both situations the foolower grants a vote other wise the Request Vote RPC fails.

There are three possible outcomes of the leader election phase.

1)The candidate wins - using this log matching property one of the candidates having the most complete log establishes itself as the leader.

2)Reelection - Many followers nodes switches their state to candidate at a time, and hence the votes get spliut between them, so no one can be the leader now, its rarely likely to happenn as RAFT uses randomized timeouts,even if it happens the RAFT will start a reelection.

3)The candidate starts receiving Append Entry RPCs(heart beats)- when this happens the candidates realizes that some node has already established itself as the leader and hence it switches its state back to being a follower.

Client Request Processing - When a client sends a request to a cluster of servers, it doesn't know who is the actual leader at that point of time, so this request might hit any of the servers present in that cluster, if the server picking up the request happens to be a follower it will reject the client request and and provide the client leader id of the node from which it received the most recent append entry heart beats.Eventually the client request would be forwarded to the leader. The leader as soon as it recieves the client request, this request is appended to the the leaders log. The leader now tries to convince the followers to copy this client request.The leader sends a Append Entry RPC which unlike the heart beat Append ENtry RPC carries  the identification ID of the leader,the new client request and the client request just preceding the current one.It depends on the follower whether it accepts the Append Entry RPC or  not.The follower tries to match the last log index of the leader sent thorugh that Append Entry RPC with its own log, if its log is as updated as that of the leaders it gives  a positive acknowledgement to the leader otherwies the Append Entry RPC fails.If the Append ENtry RPC fails ,since RAFT knows that the leader is the one which has the supreme authority because the way The RAFT elects the leader gurantees the fact that the leader is having the most comnplete logs, so all the followers are bound to follow the leader.So, when a append entry RPC fails the leader -every node when it becomes a leader maintains a next index state position which is basically the log index of the client request just preceding the client request number correponding to every  follower its an integer number and is the same for every follower, on failure of a Append Entry RPC the leader decerement that next index state position number and keeps on decrement and at every decrement of this next index state position number the leader sends a Append Entry RPS, eventually a point is reached when the followers's log matches that of the leader's log and at that very point the follower provides a successful acknowledgement back to the leader. The leader tries to get as many positive responses from the followers nodes, which gives the leader confidence to finally commit that entry as safe.This is exactly how consensus is achieved.Eventually the commited client request will be executed by the state machine.





