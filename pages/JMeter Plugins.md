- https://jmeter-plugins.org/install/Install/
- ![image.png](../assets/image_1696821976659_0.png)
- Basic Graphs
- Additional Graphs
	- ![image.png](../assets/image_1696821949459_0.png)
-
- Active Threads
	- ![image.png](../assets/image_1696822362471_0.png)
- RT
	- ![image.png](../assets/image_1696822375158_0.png)
- TPS
	- ![image.png](../assets/image_1696822408678_0.png)
-
- 资源监控
	- [perfmon-agent](https://github.com/undera/perfmon-agent)
		- ![image.png](../assets/image_1696823077045_0.png)
		- nohup java -jar ./ CMDRunner.jar --tool PerfMonAgent --udp-port 7879 --tcp-port 7879 > log. log 2>&1 &
		- chmod 755 startAgent.sh
		- ![image.png](../assets/image_1696823592436_0.png)