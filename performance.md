## Linux System Performance Troubleshooting 
* `Definition: `
  System performance is a measure of the amount of useful work done by a System in a time range. 

* `OS/CPU Load: ` The average amount of processes using or waiting for CPU allocation over a period of time.

Usually System Performance issues can be identified by observing slowness in services offered by OS and it mostly happen due to any of the below reasons. 

        - high CPU Usage 
    	- high Memory Usage
    	- high Disk IO Usage
    	- Network Performance Issues
    	- Software Bugs like memory leaks, Kernel bugs etc. 

We will try to break your investigation steps into major bullet points and helps you to find the root causes of the issue. 

## Tools helpful in your investigation: 

Below listed tools are Linux performance monitoring tools, which will help you find out the root cause of the issue,

![!image](../img/TroubleShooting.jpg)

  - <a href="https://linux.die.net/man/1/top" target="_blank">top</a>
  - <a href="https://linux.die.net/man/1/mpstat" target="_blank">mpstat</a>
  - <a href="https://linux.die.net/man/1/sar" target="_blank">sar</a>
  - <a href="https://linux.die.net/man/1/free" target="_blank">free</a>
  - <a href="https://linux.die.net/man/1/strace" target="_blank">strace</a>
  - <a href="https://linux.die.net/man/1/iotop" target="_blank">iotop</a>
  - <a href="https://www.thegeekstuff.com/2011/07/iostat-vmstat-mpstat-examples/" target="_blank">iostat</a>
  - <a href="https://tldp.org/LDP/nag2/x-087-2-iface.netstat.html" target="_blank">netstat</a>
  - <a href="https://www.thegeekstuff.com/2014/11/pidstat-examples/" target="_blank">pidstat</a>
  - <a href="https://www.thegeekstuff.com/2011/07/iostat-vmstat-mpstat-examples/" target="_blank">vmstat</a>
  - <a href="https://www.tcpdump.org/manpages/tcpdump.1.html" target="_blank">tcpdump</a>
  - <a href="http://iptraf.seul.org/about.html" target="_blank">iptraf</a>
  - <a href="https://git.kernel.org/pub/scm/linux/kernel/git/axboe/blktrace.git/tree/README" target="_blank">blktrace</a>
  - <a href="https://www.tecmint.com/10-lsof-command-examples-in-linux/" target="_blank">lsof</a>
  - <a href="https://www.networkworld.com/article/3633889/looking-at-your-linux-systems-network-interface-with-ethtool.html" target="_blank">ethtool</a>

  - [type-clip](https://github.com/v-nightwolf/nightwolf-cotribution/raw/4fc6bb408de57e9776d2d041e5573990928fbecc/TypeClip%20-%201.0.5933.22582.exe) : A very good tool when you have to paste something into virtual consoles. 
  - <a href="https://github.com/rackerlabs/recap#installation" target="_blank">recap</a> : To capture historic stats of a server. This tool is almost harmless and captures a lot of information. You can add this tool to your default package list in your environment.
<br>

You can install these tools  very easily using below command:

`      yum install -y  lsof sysstat iptraf tcpdump procps-ng net-tools strace iotop ethtool blktrace       `

<br>

#### You can start your investigation by executing small script, which will gather a lot of system stats for you:
	
	
	bash <(curl -s https://raw.githubusercontent.com/v-nightwolf/nightwolf-cotribution/main/server_stats.sh)

The output of above script will look like this: 

	############################################

	Server Uptime: 1days 6:43:23

	Load Average: Current 0.26
	Load Average: 15min Average 0.26

	#### Printing CPU stats: ####
	01:15:26 PM  CPU    %usr   %nice    %sys %iowait    %irq   %soft  %steal  %guest  %gnice   %idle
	01:15:26 PM  all    1.47    0.00    0.48    0.02    0.00    0.12    0.00    0.00    0.00   97.91


	###RAM usage###
	Free Ram: 569.93MB
	Used RAM: 3171.23MB

	15% ram left
	RAM ALERT: Low! 

	TOP RAM CONSUMER: /opt/mongodb/mms/jdk/bin/mms-app
	RAM Usage (RSS): 1857.43 MB
	Total Number of RAM Processes: 2
	Total RAM Usage for all /opt/mongodb/mms/jdk/bin/mms-app processes =  2287.56 MB
	61.15% used by /opt/mongodb/mms/jdk/bin/mms-app


	###CPU usage###
	Top Process:  /opt/mongodb/mms/jdk/bin/mms-app
	CPU % for SINGLE Top Process =  4.2
	number of processes this is running: 2
	Total CPU % for /opt/mongodb/mms/jdk/bin/mms-app =  4.8

	############################################

## Troubleshooting Linux perfomance isssue happening due to high CPU Usage

   Every performance issue troubleshooting should starts with `top` command:

   and the most useful output of `top` command is: 

   	top - 16:18:32 up  2:52,  1 user,  load average: 0.09, 0.20, 0.22
   	Tasks:  99 total,   1 running,  98 sleeping,   0 stopped,   0 zombie
   	%Cpu(s):  3.2 us,  0.0 sy,  0.0 ni, 96.8 id,  0.0 wa,  0.0 hi,  0.0 si,  0.0 st
   	KiB Mem :  2889532 total,    73328 free,  2707940 used,   108264 buff/cache
   	KiB Swap:  2097148 total,  1206268 free,   890880 used.    48052 avail Mem 
<br>

    a). 1st line contains: UPTIME and LOAD Averages
           current time and length of time since last boot
           total number of users logged in.
           system load avg over the last 1, 5 and 15 minutes
<br>

    b). 2nd line contains: TASKs
        This line shows total tasks or threads, depending on the state of the Threads-mode toggle.  
        That total is further classified as: running; sleeping; stopped; zombie
<br>

    c). 3rd line contains CPU state percentages: This will guide your investigation into a specific direction. 

        As a default, percentages for these individual categories are displayed.
           us, user    : time running un-niced user processes
           sy, system  : time running kernel processes
           ni, nice    : time running niced user processes
           id, idle    : time spent in the kernel idle handler
           wa, IO-wait : time waiting for I/O completion
           hi : time spent servicing hardware interrupts
           si : time spent servicing software interrupts
           st : time stolen from this vm by the hypervisor
<br>
    Above stats might guide your investigation into a specific direction. Depending on above CPU stats, you will be able to decide what to check next. 
	
	* If only "%us"(time running un-niced user processes) is high that means your application is consuming more
	  CPU. Now you should deep dive into process stats and process trace. We will discuss this in more detail soon.  

	* If only "%sy"(time running kernel processes) is high that means you kernel level system call are consuming
	  the CPU. This usually happens when there is a bug in kernel packages. 

	* If "%ni"(time running niced user processes) is high that means the  prioritized processes are consuming
	  the CPU. You should try to depriritize the process.

	* If "%id"(time spent in the kernel idle handler) is high that means your system is ideal and is not doing
	  anything. 

	* If "%wa"(time waiting for I/O completion) is high that means your most CPU time is being spent on waiting for
	  I/O completion. These waits can be due to Disk slowness or Network slowness. 

	* If "%hi"(time spent servicing hardware interrupts) is high that means CPU is very busy in servicing the 
	  Hardware interrupts. Hardware interrupt will cause CPU to stop its current processing and go handle the 
	  Hardware interrupt. Ususally hardware interrupts was generated by physical devices like disk, NIC,computer
	  peripherals. You should check the hardwares attached to the machine and see if they are working fine. 

	* If "%si"(time spent servicing software interrupts) is high that CPU is busy serving Software interrupts.
	  Usually Software interrupt occure at kernel level. 

	* "%st"(time stolen from this vm by the hypervisor) means that virtual CPU is being spent waiting for the 
	  Hypervisor to allocate CPU to virtual machine. This stat is only applicable to Virtual machines. 
<br>
    
  If %us is high, that mean some process in user space is consuming the CPU. Next step would be to deep dive into
    process stats: 

    1. First identify the process consuming high CPU. This can be identified using "top" command. 

    2. Once you have identified the process, note down its process id(pid).

    3. Try tracing the process using stace command. This will throw a lot output onto the screen. 
       starce -t -p PID_OF_PROCESS  => This will print wall clock time of each system call. 

    4. Please check if process is getting stuck at any system call and note down system call and the resource system 
       call is using. 

    5. If system call is waiting for a File related I/O, then next step is to check file system call is waiting for.

    6. Go to /proc/{$PID_OF_PROCESS}/
<br>

## Troubleshooting Linux performance issue happening due to high Memory Usage

    d). 4th and 5th line in 'top' output contains Memory stats in Kibibytes(kib).
        There is slight difference between kilibyte and kibibyte i.e 1 kB = 1000 bytes. 1 KiB = 1024 bytes.

        As a default, line 4 reflects physical memory, classified as:
           total, free, used and buff/cache

        line 5 reflects mostly virtual memory(swap), classified as:
           total, free, used and avail

        total: Total size of memory available to system. 

        free: Size of memory which in un-utilized.  

        used: Size of memory currently utilized by processes+System.

        buff/cache: Size of memory utilized by system for kernel buffers(i.e. buff) and page cache and slabs(i.e. cache)



## Troubleshooting Linux performance issue happening due to high Disk IO Usage

## Troubleshooting Linux OS network performance Issues
