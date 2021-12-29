# server-perf-analytics
This project is used to capture following type of system informations for anomaly detection , diagonsis or RCA:
1. Application related metrics from Apache Webserver Benchmark(every 1 minute interval) - # of concurrent users, requests/sec response time/requests, # of failed requests 
2. Script generated monitoring data for( 1 min interval) - % cpu usage, % ram usage , % dsik read and write , % network packet in and out , top process names and their resoufce usages 
3. All the logs under /var/log including syslog
4. tracking file when a fault was injected randomly - time and type of fault 
5. Also  from all the metric data has been combined and collected in to a single tabular format for eaase of KPI anomaly analysis

The simulation is done in AWS EC2 - with Apace web server as application , 
Apache benchmark to generate load and Custom scripts to generate the perf metrics. All the scripts and the logs files are present in this repo 
 Training data - all normal instances ( no anomaly) :
 1. App benchmark runs continuously with a target to ensure 90- 100% good response time . That response time should be noted our SLA . RespTime< 2ms . Keep user / no of transaction as your choice of variable.
 2. The output data should be generated continuously , if it takes to x time to generate out put , we should have data for every x interval . lets target x = 10 minutes.
 3. a. Within x =10 minutes , we will collect perf mon metrics 10 records . we will include app resp time as a additional numeric column along with the perf metrics  b. System log , application log c. Process wise perf mon.
 4. Now run this system for  a week  - from this data model will learn normal behaviour.
 Test data randomly abnormal: 
 5. Now we will inject fault . a. Type 1 fault - by increasing application load e.g. user count/ # of transaction etc. drive n > 2ms . This should not be for long but randomly injected. Record the time ( random times) so that we can later know in what duration the fault was injected.  b. Type2 fault - stress ng - CPU fault , RAM fault , disk full , network fault - randomly and record time. This should be also logged in syslog or other logs .
 
 
 The format for the combined  KPI data:  
 "Time
5 min 
Interval"	"Fault 
Injected
(Y/N)"	"Type 
of Fault"	CPU	CPU wait	RAM	"Disk
Sec/Read"	"Disk 
Sec/Write"	"Disk
Read/Sec"	"Disk
Write/Sec"	"N/W In
kb/sec"	"N/W Out 
kb/sec"	# of Uers	"# of 
Response"	"Resp Time 
/ Request"	"Failed 
requests"	Add more ....

Steps involved for cobining the metrics are:

Open benchmark.log and find the line starting with " load test start time"	
Then on the same line extract the time information {{T}} 	"Mon Dec 27 15:54:02 IST 2021"
extract time per request resp time {{RESP_TIME}} and fail requst {{FAIL_REQ}} 	
	Time per request:       361.828 [ms] (mean)
	Failed requests:        4055
Then open output file which is combined KPI.CSV and the resp time and failed request as a new column 	
Now open perf_mon_summ.csv find the time peak T to be reforamtted to match the timestamp perf_mon_sum.csv example	
	27-12 15:54:18|
For each Column (CPU/MEM etc) corrosponding to this minute (15:54) there will 30 values in the 15:54:18, 15:54:19 ... etc	
Will take avg of this value and update the KPI.csv	
Also check during the same time open the random_stress.txt whether any fault injected or not	
If any fault detected during the time then Mark fault injected along with type of fault else mark none	
then iterate this process , 	

 
