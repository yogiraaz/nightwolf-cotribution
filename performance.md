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

 {!inpage-ads.md!}

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

 {!horizontal-ads.md!}

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


{!horizontal-ads.md!}


In case "used"  RAM is 100% then we need to check we need to check which application is consuming that. 

Further, we can check application name in TOP command 

Run "top" then press SHIFT+m to sort w.r.t to Memory utilization 

It will show process taking memory utilization, if that process is taking much memory utilization then we need to act accordingly.  

# OS process: 
if that is worthy then we need to tune OS parameters otherwise there can process which was running from long time and taking much resource, we can simply kill them. 

# Application process:
if that process is worthy then this can be a issue with code within the application 

Further we can tune that application as well by allowing specific RAM to that particular process/app with the help of cgroups. 

# Configure the memory limit to 1000M:  

sudo systemctl set-property .service MemoryLimit=1000M  

# View memory limit configuration: 

 cat /etc/systemd/system.control/.service.d/50-MemoryLimit.conf  

# Configure memory limit to 1000M using a unit file:  

sudo vi /etc/systemd/system/.service -- Add the following entry under the [Service] section MemoryLimit=1000M 

------------------------------------------------------------------------------------------------------------------ 

### High memory utilization 

Excessive memory use on the system can lead to poor performance due to the system's need to move data between the RAM memory and swap storage, memory must be cleared to make room for new allocations, or there is little space available for caching data from the file system. A quick summary of the memory being used on the system can be obtained with the free command. Below is sample output from the free command. 

$ free 
             total       used       free     shared    buffers     cached 
Mem:      15998332   14522616    1475716          0     918548    3673008 
-/+ buffers/cache:    9931060    6067272 
Swap:      5734396     219564    5514832 
 

The Mem row lists how the available memory is being used. The first value on this line is the total amount of memory available on the system. In this case the machine has 15998332 bytes available, about 16GB. The second value in row under the used column indicates how much of that memory is currently been allocated for some use. The free column show that there is 1475716 bytes, a bit less than 1.5GB unallocated memory. The shared column is obsolete on modern Linux kernels and should be ignored. 

The buffers column indicates the amount of memory used to store raw disk blocks while the cached column indcates the amount of memory used to store files from disk. The kernel can dynamically adjust the amount of memory allocated for the buffers and cached files. The -/+ buffers/cache line describes how much memory is used excluding the kernel buffers and cached files. In the example above 9.9GB are being used. This machine is not suffering from excessive memory use and has 6G that could be allocated for other uses if need be. 

Linux support virtual memory, which allows applications to use more memory than physical RAM installed on the machine. Linux will use the swap partitions on the disk drives to store the data that is not currently being used and automatically move data between RAM memory and disk drives as required. However, it is very slow to move data between physical RAM and disk. In the example above there is 219564 bytes, about 220MB, of swap that is in use. However, this is unlikely to be a significant problem because of large amount of free memory (6GB) listed on the -/+ buffers/cache line. 

For systems with large amount of memory the -h (human output) option for free may make it easier to read the output. It provides scaled output describing the amounts in kilobytes (K), megabytes (M), and gigabytes (G). Below is an example output: 

$ free -h 
             total       used       free     shared    buffers     cached 
Mem:           15G       9.7G       5.6G         0B       1.0G       1.6G 
-/+ buffers/cache:       7.0G       8.2G 
Swap:         5.5G       783M       4.7G 
 

In addition to free command, the top command also provides information about overall system memory use. Below is the beginning of the top output which include the overall memory information. It shows that the machine has 15998328 bytes of total memory with 8645132 bytes currently used and an additional 7353196 free. The buffer for devices and cached files occupy 277220 and 3362088 bytes respectively. A relatively small amount of the swap space is being used, only 15864 bytes of the 5734396 bytes of swap available. 

Raw 

top - 12:24:44 up  3:04,  5 users,  load average: 1.96, 2.82, 2.32 
Tasks: 247 total,   1 running, 246 sleeping,   0 stopped,   0 zombie 
%Cpu(s):  3.9 us,  1.3 sy,  0.0 ni, 94.0 id,  0.5 wa,  0.2 hi,  0.1 si,  0.0 st 
KiB Mem:  15998328 total,  8645132 used,  7353196 free,   277220 buffers 
KiB Swap:  5734396 total,    15864 used,  5718532 free,  3362088 cached 


If swap "used" section is 100% or approx then we need to check which application is consuming that. 

## Known Issues 

# Swap memory usage is at 100% 

# Swap memory usage is above the error threshold 

# Swap memory usage is higher than average 

# Why Swap is being used while there is available physical memory? 

Introduction - what is Swap? 

The primary function of Swap space is to utilize disk space from a separate, dedicated partition on the main storage for RAM when the physical main memory fills up and more space is needed. 

To expand on main memory, it is used by two general categories: Page Cache, which are mostly file contents based in storage that have been stored in the RAM for faster loading; and Anonymous Memory, which is mainly comprised from content not backed by storage, such as program stack and temporary variables. 

The kernel uses a memory management program that detects blocks (AKA: pages) of anonymous memory, in which the contents have not been used recently. 

The memory management program swaps out enough of these relatively infrequently used pages of anonymous memory to a special partition on the main storage specifically designated for “paging”, or swapping. This frees up RAM and makes room for more data. 

Those pages of anonymous memory that were swapped out to the Swap partition are tracked by the kernel’s memory management code and can be paged back into RAM if they are needed (AKA: Swapped-in). 

## INVESTIGATION: 

Review the output of free command 

Example: 

Raw 

# free -m 
              total            used            free            shared       buffers           cached 
Mem:          516544          399419           117124           5427          41               23272 
-/+ buffers/cache:      376105        140438 
Swap:        263167           263144            23   <<<<------ 
 

Review the SAR file output for continuously heavy Swap in/out activity. This is represented by high values of "pswpin" / "pswpout" 

Example of sar output : 

12:00:00 AM  pswpin/s pswpout/s 
<snip> 
05:20:00 AM  0.21      0.00 
05:30:00 AM  0.08      0.85 
05:40:00 AM  0.47      0.00 
05:50:00 AM  3.58      1.71 
06:00:00 AM  2.48      0.00 
06:10:00 AM 39.91      7.17   <<<<----- 
06:20:00 AM  0.21      2.72 
06:30:00 AM 13.30      1.04 

On Runtime, we can refer vmstat command and si/so section to check swapin/swapout.  
 
Resolution 1: Increase the system's physical memory 

Resolution 2: Find process's memory regions that are using the most Swap 

Resolution 3: Increase Swap space 

Resolution 4: Flushing the Swap 

Another option is to flush the Swap by utilizing the following commands: 

Raw 

# swapoff -a   
# swapon -a 
 
Warning: Flushing Swap in this way will force the entire contents of swap back into main memory. If your system is already low on memory this may cause it to go into an Out Of Memory condition (OOM). Care and consideration should be taken before using this option. If the system is somewhat low on memory (and not very low) pages may have to be reclaimed while bringing the contents of Swap into memory - and this may degrade performance for a short time. 

Resolution 5: Clear POSIX shared memory (/dev/shm/) 

Resolution 6 : Tune vm.swappiness kernel parameter. 

KEYPOINTS TO REMEMBER  


The following applies to RHEL 7 : 

Swapping will [only] occur during a low-memory condition 

The value of /proc/sys/vm/swappiness dictates how 'aggressively' the Linux kernel will swap memory pages [during memory reclaim] 

The 'swappiness' value can be between 0 and 100. 

In the context of global memory reclaim (i.e. when there is no suitable page to satisfy an allocation request), swappiness set to 0 does not prevent swapping entirely. 

In the context of cgroup memory reclaim (i.e. when a memory cgroup hits or breaches it's limit_in_bytes), memory.swappiness set to 0 does prevent swapping. 

In the context of memory cgroup and global memory reclaim, when the system is close to OOM condition, a positive swappiness value will force the Linux kernel to consider both anon and file-backed pages equally. 

Now "swapping" will not occur unless the system enters a low-memory condition i.e. when a memory allocation request may put the number of available memory beyond the low water mark in a zone (see __alloc_pages_nodemask() -> __alloc_pages_slowpath()). 

As per Documentation/sysctl/vm.txt 'swappiness' "... is used to define how aggressive the kernel will swap memory pages [during memory reclaim]." The swappiness value can be set between 0 and 100, the default value is 60. 

A value of 0 does not prevent swap. In fact, the kernel will initiate swap when the amount of free and file-backed pages is less than the high water mark in a zone irrespective of the swappiness value (see get_scan_count()). 

If no swap area(s) exists then swappiness is not applicable. 

Now the above is correct in the context of global memory reclaim. 

Under Red Hat Enterprise Linux 7, we build the kernel with Kconfig option CONFIG_MEMCG enabled. Therefore a cgroup which is attached to a Memory Resource Controller can have a unique swappiness i.e. a value different to the global or system-wide value i.e. in /proc/sys/vm/swappiness. In other words, it overrides /proc/sys/vm/swappiness for the particular cgroup. 

The value of 'swappiness' is implicitly used in the memory reclaim code path in the context of global and memory cgroup (memcg) reclaim but as indicated above, a value of 0 in a memcg prevents any swapping even if swap area(s) exists. Albeit this could lead to memcg OOM killer if there are no file pages to reclaim. 

	
## Troubleshooting Linux performance issue happening due to high Disk IO Usage
	
## Troubleshooting Linux OS network performance Issues

{!multiplex-ads.md!}

---

<br>
{!footer.md!}
