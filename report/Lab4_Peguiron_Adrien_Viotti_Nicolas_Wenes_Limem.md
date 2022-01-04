# AIT - Labo 4 : Docker

Authors : Peguiron A., Viotti N., Wènes L. - 03.01.2022

#### Introduction



> ### Task 0: Identify issues and install the tools
>
> #### Identify issues
>
> 1. **[M1]** Do you think we can use the current solution for a production environment? What are the main problems when deploying it in a production environment?
>  No, we cannot use our current solution for a production environment. There is no automated process for adding dynamically new server nodes when the traffic hits the roof. 
>  
> 2. **[M2]** Describe what you need to do to add new `webapp` container to the infrastructure. Give the exact steps of what you have to do without modifiying the way the things are done. Hint: You probably have to modify some configuration and script files in a Docker image.
>  First of all, we need to reconfigure (edit config file and .env file) HA proxy so that it can serve the new webapp.  Secondly, we edit the _docker-compose_ file so that we can build a new container that contains the files of the new _webapp_ . Last step is to rebuild the containers in the docker compose file and execute them as a final step . 
>  
> 3. **[M3]** Based on your previous answers, you have detected some issues in the current solution. Now propose a better approach at a high level.
>  	We can add a mechanism that detects the load rate on the HAProxy, if we reach a certain level on both nodes , we launch an automated script that adds new node till the load is back to normal rates.
>  	
> 4. **[M4]** You probably noticed that the list of web application nodes is hardcoded in the load balancer configuration. How can we manage the web app nodes in a more dynamic fashion?
>  	We have to edit HAProxy config file, that it detects new nodes. There exists a set of discovery tools that detects a new node is added | deleted  to |from the pool. 
>  	
> 5. **[M5]** In the physical or virtual machines of a typical infrastructure we tend to have not only one main process (like the web server or the load balancer) running, but a few additional processes on the side to perform management tasks.
>
>    For example to monitor the distributed system as a whole it is common to collect in one centralized place all the logs produced by the different machines. Therefore we need a process running on each machine that will forward the logs to the central place. (We could also imagine a central tool that reaches out to each machine to gather the logs. That's a push vs. pull problem.) It is quite common to see a push mechanism used for this kind of task.
>
>    Do you think our current solution is able to run additional management processes beside the main web server / load balancer process in a container? If no, what is missing / required to reach the goal? If yes, how to proceed to run for example a log forwarding process?
>   The central tenet of Docker design architecture is one service per container. Hence we can not run additional management processes beside the main web server / load      balancer. We have to install a process supervisor.  
>
> 6. **[M6]** In our current solution, although the load balancer configuration is changing dynamically, it doesn't follow dynamically the configuration of our distributed system when web servers are added or removed. If we take a closer look at the `run.sh` script, we see two calls to `sed` which will replace two lines in the `haproxy.cfg` configuration file just before we start `haproxy`. You clearly see that the configuration file has two lines and the script will replace these two lines.
>
>    What happens if we add more web server nodes? Do you think it is really dynamic? It's far away from being a dynamic configuration. Can you propose a solution to solve this?
>   One line in the script adds a server to the pool of HAProxy. To be able to add more servers , we have to add more **sed** commands to the scripts (server per command). 
	But this solution, automated it might seem, but it's dynamic because we have to run the script, and we have to check everytime if we have to add another server or not. 
  In this case, we have to write a script the generate  HAProxy config file.   
>    #### Install the tools
>
>    1. Take a screenshot of the stats page of HAProxy at [http://192.168.42.42:1936](http://192.168.42.42:1936/). You should see your backend nodes.

![0_1](figures\0_1.png)

>  		  2. Give the URL of your repository URL in the lab report.

https://github.com/AdrienPeg/Teaching-HEIGVD-AIT-2020-Labo-Docker

> ### Task 1: Add a process supervisor to run several processes
>
> **Deliverables:**
>
> 1. Take a screenshot of the stats page of HAProxy at [http://192.168.42.42:1936](http://192.168.42.42:1936/). You should see your backend nodes. It should be really similar to the screenshot of the previous task.

![1_1](figures\1_1.png)

> 2. Describe your difficulties for this task and your understanding of what is happening during this task. Explain in your own words why are we installing a process supervisor. Do not hesitate to do more research and to find more articles on that topic to illustrate the problem

In this task, we installed a process supervisor called `S6`. This allows multiple process to be ran at once in a docker container. Normally, a docker container can only run one process and, when it's over, the container stops as well.

To use this process supervisor, we must first change the dockerfiles of the webapps and HAProxy to install `s6` , start it as the main process in the containers and give it a starting script. Once it is done, the images must be rebuilt and the containers rerun.

We had no real difficulties during this task as it was well documented.

> ### Task 2: Add a tool to manage membership in the web server cluster
>
> **Deliverables:**
>
> 1. There is different way to implement the sticky session. One possibility is to use the SERVERID provided by HAProxy. Another way is to use the NODESESSID provided by the application. Briefly explain the difference between both approaches (provide a sequence diagram with cookies to show the difference).

The difference is the entity who will deliver the cookie. for SERVERID it is HAProxy who takes care of it while for NODESESSID, it will be the application. 

![task2_1](C:\Users\Don Peg\Downloads\AIT-LAB3-NICO\figures\task2_1.png)

> - Choose one of the both stickiness approach for the next tasks.

We used the SERVERID approach for this lab

> 2. Provide the modified `haproxy.cfg` file with a short explanation of the modifications you did to enable sticky session management.

![image-20211207171938117](C:\Users\Don Peg\AppData\Roaming\Typora\typora-user-images\image-20211207171938117.png)

These are the three lines used to make `sticky session` work.

The first one enables cookies and insert the name of the server into it.

The second and third lines add both servers to the load balancer, assign them a name, bind them to an ip address and set the server name in the cookie.

> 3. Explain what is the behavior when you open and refresh the URL [http://192.168.42.42](http://192.168.42.42/) in your browser. Add screenshots to complement your explanations. We expect that you take a deeper a look at session management.

First request : 

![image-20211207173559102](C:\Users\Don Peg\AppData\Roaming\Typora\typora-user-images\image-20211207173559102.png)

Second request : 

![image-20211207173638148](C:\Users\Don Peg\AppData\Roaming\Typora\typora-user-images\image-20211207173638148.png)

The `sessionViews` parameter has gone up as expected. We can also see that the second request was redirected to the same server has the first request. This means that the session is now persistent.

The cookie SRVNAME contains the name of the server as configured in the previous point : 

![image-20211207173829116](C:\Users\Don Peg\AppData\Roaming\Typora\typora-user-images\image-20211207173829116.png)



> 4. Provide a sequence diagram to explain what is happening when one requests the URL for the first time and then refreshes the page. We want to see what is happening with the cookie. We want to see the sequence of messages exchanged (1) between the browser and HAProxy and (2) between HAProxy and the nodes S1 and S2. We also want to see what is happening when a second browser is used.

![task2_4](C:\Users\Don Peg\Downloads\AIT-LAB3-NICO\figures\task2_4.png)

> 5. Provide a screenshot of JMeter's summary report. Is there a difference with this run and the run of Task 1?

![image-20211207173951850](C:\Users\Don Peg\AppData\Roaming\Typora\typora-user-images\image-20211207173951850.png)

Every request is redirected to server1. This means the load balancer is working since it redirects requests from the same client to the same server.

They will either all be redirected to server1 or all to server2 depending on which one received the first request.

> - Clear the results in JMeter.
> - Now, update the JMeter script. Go in the HTTP Cookie Manager and verify that the box `Clear cookies each iteration?` is unchecked.
> - Go in `Thread Group` and update the `Number of threads`. Set the value to 2.
>
> 6. Provide a screenshot of JMeter's summary report. Give a short explanation of what the load balancer is doing.

![image-20211207174536463](C:\Users\Don Peg\AppData\Roaming\Typora\typora-user-images\image-20211207174536463.png)

Both server got half of the requests. Both threads were assigned to a different server via cookies and contacted only this server.



> ### Task 3: Drain mode
>
> **Deliverables:**
>
> 1. Take a screenshot of the Step 5 and tell us which node is answering.

![image-20211208134813602](C:\Users\Don Peg\AppData\Roaming\Typora\typora-user-images\image-20211208134813602.png)

Under the `nodes `section, all the activities happened on s1 and s2 has no last known requests. We can deduce that the answering node is s1.

> 2. Based on your previous answer, set the node in DRAIN mode. Take a screenshot of the HAProxy state page.

To set up the node in DRAIN mode, a connexion with HAproxy has been established with the help of socat and then the following command has been used :  `set server nodes/s1 state drain`. 

![image-20211208140603153](C:\Users\Don Peg\AppData\Roaming\Typora\typora-user-images\image-20211208140603153.png)

On the statistics report, the node status has changed from `UP` to `DRAIN`

> 3. Refresh your browser and explain what is happening. Tell us if you stay on the same node or not. If yes, why? If no, why?

Yes, we stay on the same node. DRAIN mode will not redirect requests to another node if a connection has been established before the drain mode has been set up. Since we were already connected to s1 beforehand, any request will still be directed towards s1.

> 4. Open another browser and open `http://192.168.42.42`. What is happening?

![image-20211208141227633](C:\Users\Don Peg\AppData\Roaming\Typora\typora-user-images\image-20211208141227633.png)

Upon accessing `http://192.168.42.42` with a new browser, the request will be redirected towards s2 since s1 is only accessible to those who were connected to it before the DRAIN mode

> 5. Clear the cookies on the new browser and repeat these two steps multiple times. What is happening? Are you reaching the node in DRAIN mode?

No, since the cookies have been deleted, there is no way for the server to know if there was a connection established before. Therefore, no request from this browser will be able to access s1 except if the cookie value is manually changed..

> 6. Reset the node in READY mode. Repeat the three previous steps and explain what is happening. Provide a screenshot of HAProxy's stats page.

The command is `set server node/s1 state ready`

![image-20211208141740619](C:\Users\Don Peg\AppData\Roaming\Typora\typora-user-images\image-20211208141740619.png)

The node is up and running again.

The request will be distributed evenly between the two nodes. If a connection from a browser has already been established, the browser will always access the same server. Once the cookies have been reset, it will be redirected towards one of the two servers in a round robin fashion.

> 7. Finally, set the node in MAINT mode. Redo the three same steps and explain what is happening. Provide a screenshot of HAProxy's stats page.

The command is `set server node/s1 state maint`

![image-20211208142250849](C:\Users\Don Peg\AppData\Roaming\Typora\typora-user-images\image-20211208142250849.png)

Every request will be redirected towards s2. Even if a connection to s1 has been established beforehand, upon refresh it is s2 who will be accessed. Clearing the cookie or changing browser will not change this.

> ### Task 4: Round robin in degraded mode
>
> **Deliverables:**
>
> *Remark*: Make sure you have the cookies are kept between two requests.
>
> 1. Make sure a delay of 0 milliseconds is set on `s1`. Do a run to have a baseline to compare with in the next experiments.

To set the delay : `curl -H "Content-Type: application/json" -X POST -d '{"delay": 0}' http://192.168.42.11:3000/delay`

Run results : 

![image-20211208153048045](C:\Users\Don Peg\AppData\Roaming\Typora\typora-user-images\image-20211208153048045.png)

Requests are evenly distributed between the two servers.

> 2. Set a delay of 250 milliseconds on `s1`. Relaunch a run with the JMeter script and explain what is happening.

To set the delay : `curl -H "Content-Type: application/json" -X POST -d '{"delay": 250}' http://192.168.42.11:3000/delay`

![image-20211208153912736](C:\Users\Don Peg\AppData\Roaming\Typora\typora-user-images\image-20211208153912736.png)

Requests are still evenly distributed. The only difference is that since there is a delay on s1 before sending the response, it's throughput dropped to 3.1 requests per second 

> 3. Set a delay of 2500 milliseconds on `s1`. Same than previous step.

To set the delay : `curl -H "Content-Type: application/json" -X POST -d '{"delay": 2500}' http://192.168.42.11:3000/delay`

![image-20211208154411413](C:\Users\Don Peg\AppData\Roaming\Typora\typora-user-images\image-20211208154411413.png)

Only two response have been received from s1, every other request has been redirected to s2.

The delay was too long and the node was considered down since it took too long to receive a response.

> 4. In the two previous steps, are there any errors? Why?

`jmeter` doesn't show any error, but on the terminal where the servers and proxy are initiated, we see the following warning :

![task4_4](C:\Users\Don Peg\Downloads\AIT-LAB3-NICO\figures\task4_4.png)

The server s1 is considered DOWN because no response has been sent after 2001ms. This is because of the delay of 2500ms. 

Since it is considered down, every requests will be forwarded towards s2.

> 5. Update the HAProxy configuration to add a weight to your nodes. For that, add `weight [1-256]` where the value of weight is between the two values (inclusive). Set `s1` to 2 and `s2` to 1. Redo a run with a 250ms delay.

In the config file, the last two lines have been modified to add weight to the servers : 

![image-20211208155401729](C:\Users\Don Peg\AppData\Roaming\Typora\typora-user-images\image-20211208155401729.png)

![image-20211208163725086](C:\Users\Don Peg\AppData\Roaming\Typora\typora-user-images\image-20211208163725086.png)

The results are the same as the first time we did it in step 2. This can be explained by the fact that the sticky sessions are still in action and, since both threads were assigned a different server, they will use it for every request.

> 6. Now, what happens when the cookies are cleared between each request and the delay is set to 250ms? We expect just one or two sentence to summarize your observations of the behavior with/without cookies.

![image-20211208164622381](C:\Users\Don Peg\AppData\Roaming\Typora\typora-user-images\image-20211208164622381.png)

Since the cookies are cleared, the server doesn't know if a session is established and two requests from the same browser wont necessarily be redirected to the same server.

Now it is the weight of the servers who determines which server will get the request, and since s1 has a weight of 2 while s2 has only a weight of 1, s1 will receive 2/3 of the request while s2 receives only 1/3

> ### Task 5: Balancing strategies
>
> **Deliverables:**
>
> 1. Briefly explain the strategies you have chosen and why you have chosen them.

We choose to use the strategies `leastconn` and`first`.

We looked at the list of strategies available here : http://cbonte.github.io/haproxy-dconv/2.2/configuration.html#balance.

The first one on the list not recommended for the lab and the second one is very similar to the first so we decided to pick the next two.

`Leastconn` will check which server as the least number of connection and connect to this one. Round robin is used when two or more servers have the same load.

`First` will connect to the first server with available connection slots. The maximum number of possible connections to a server must be specified in the config file in the `maxconn` parameter.

> 2. Provide evidence that you have played with the two strategies (configuration done, screenshots, ...)

#### Leastconn :

First we need to change the HAproxy config file `haproxy.cfg`

![image-20211208170313742](C:\Users\Don Peg\AppData\Roaming\Typora\typora-user-images\image-20211208170313742.png)

At the end of the file, change the balance to leastconn, and remove the extra options and parameter we added to make the sticky session work.

The end of the config file should look like the picture above.

Then rebuild the two containers and connect to `192.168.42.42`.

If refresh the page, it will always change server on each session. This is because the connection is closed once the response is sent and then when the next request comes, both server have 0 connections and the round robin method is used.

To have a better understanding on how it works, we can add some delay to the response and use `jmeter`. Here's the result with 250ms of delay on s1 :

![image-20211208171355852](C:\Users\Don Peg\AppData\Roaming\Typora\typora-user-images\image-20211208171355852.png)

s2 receives a lot more requests than s1. It is because while s1 is busy responding, every request is redirected to s2. The difference with round robin is that it is way faster to redirect to another server while the first is occupied, than to distribute requests evenly and wait longer if you are assigned to the slower server.

#### First :

Like for `Leastconn` we need to edit `haproxy.cfg`

![image-20211208172135436](C:\Users\Don Peg\AppData\Roaming\Typora\typora-user-images\image-20211208172241900.png)

We change the balance to `first` and we add a new parameter `maxconn` to both our servers. This parameter represents how many connections can be established before having to redirect requests to another server. 

Here are the results with 0 delay :

![image-20211208172740195](C:\Users\Don Peg\AppData\Roaming\Typora\typora-user-images\image-20211208172740195.png)

since we are currently testing with 2 threads, there will be at most 2 requests at the same time and since s1 accept 2 connections and is the first one available, no request will ever go to s2.

if we add one more thread to the tests :

![image-20211208173010428](C:\Users\Don Peg\AppData\Roaming\Typora\typora-user-images\image-20211208173010428.png)

We see that roughly one out of three request are now sent to s2. That is because the first request will see that s1 is free and be redirected to it, the second one will do the same, and once a third request comes, it will see that s1 has reached the maximum number of connections (2) and will be redirected to s2.

If we add a 250ms delay on s1 as well as 2 more threads :

![image-20211208173633571](C:\Users\Don Peg\AppData\Roaming\Typora\typora-user-images\image-20211208173633571.png)

When 2 connections on s1 are waiting for the delay, every request is redirected to s2 who can produce responses way faster and be able to accept a new request.

> 3. Compare the two strategies and conclude which is the best for this lab (not necessary the best at all).

They are both good for long connections. `first` purpose is to use as little server as possible while `leastconn` goal is to distribute connection evenly among every server. 

For this lab, `leastconn` seems more adapted since we want to use both server. 

Furthermore, `first `would require a lot of modification of the `maxconn` parameter and could result in some unwanted results if it is forgotten. 

#### Conclusion

In this lab, we learned a lot about load balancers, how they work and their different strategies. Having a well configured load balancer in a complex infrastructure can really optimize performances and avoid issues. There is of course a lot more to them, but having a general understanding is whiteout a doubt very useful.
