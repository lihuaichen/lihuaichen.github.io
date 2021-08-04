---
title: qsub,qdel,qmgr,qstat,qhost
date: 2017-12-27 16:36:56
tags: CSDN迁移
---
 版权声明：本文为carson_li原创文章，未经博主允许不得转载。更多文章请访问个人网站https://www.lihuaichen.cn https://blog.csdn.net/lihuaichen/article/details/78913792   
  qsub,qdel,qmgr,qstat,qhost  
 PBS 是公开源代码的作业管理系统，在此环境下运行，用户不需要指定程序在哪些节点上运行，程序  
 所需的硬件资源由PBS 管理和分配。  
 1、PBS 命令  
 PBS 提供4 条命令用于作业管理。  
 (1) qsub 命令—用于提交作业脚本  
 命令格式：  
 qsub [-a date_time] [-c interval] [-C directive_prefix]  
 [-e path] [-I] [-j join] [-k keep] [-l resource_list] [-m mail_options]  
 [-M user_list][-N name] [-o path] [-p priority] [-q destination] [-r c]  
 [-S path_list] [-u user_list][-v variable_list] [-V]  
 [-W additional_attributes] [-z]  
 [script]  
 参数说明：因为所采用的选项一般放在pbs 脚本中提交，所以具体见PBS 脚本选项。  
 例：# qsub aaa.pbs 提交某作业，系统将产生一个作业号  
 qsub -cwd -S /bin/bash -l vf=1.5G,p=8,h=compute-0-15 -P project -q all.q -p 100 -N test -o std.o -e std.e  
 run.sh  
 -cwd #指定当前路径为工作目录，sge的日志会输出到当前路径。  
 -S #指定远程计算节点的shell路径  
 -l #指定资源请求，多个请求用逗号(,)隔开  
 vf=1.5G #任务的预估内存，内存估计的值应稍微大于真实的内存，内存预估偏小可能会导致节点跑挂。  
 h=compute-0-15 #指定任务跑在compute-0-15节点上  
 p=8 #指定要申请的CPU核心数  
 -q #指定要投递到的队列，如果不指定的话，SGE会在用户可使用的队列中选择一个。  
 -P #参数指明任务所属的项目  
 -p #设置优先级，优先级高的优先执行  
 -N #指定任务名称  
 -o #指定标准输出路径  
 -e #指定标准错误路径  
 run.sh #为任务脚本  
 (2) qdel 命令—用于删除已提交的作业  
 命令格式：qdel [-W 间隔时间] 作业号  
 命令行参数：  
 例：# qdel -W 15 211 15 秒后删除作业号为211 的作业  
 (3) qmgr 命令—用于队列管理  
 qmgr -c "create queue batch queue_type=execution"  
 qmgr -c "set queue batch started=true"  
 qmgr -c "set queue batch enabled=true"  
 qmgr -c "set queue batch resources_default.nodes=1"  
 qmgr -c "set queue batch resources_default.walltime=3600"  
 qmgr -c "set server default_queue=batch"  
 (4) qstat 命令—用于查询作业状态信息  
 命令格式：qatat [-f][-a][-i] [-n][-s] [-R] [-Q][-q][-B][-u]  
 参数说明：  
 -f jobid 列出指定作业的信息  
 -a 列出系统所有作业  
 -i 列出不在运行的作业  
 -n 列出分配给此作业的结点  
 -s 列出队列管理员与scheduler 所提供的建议  
 -R 列出磁盘预留信息  
 -Q 操作符是destination id，指明请求的是队列状态  
 -q 列出队列状态，并以alternative 形式显示  
 -au userid 列出指定用户的所有作业  
 -B 列出PBS Server 信息  
 -r 列出所有正在运行的作业  
 -Qf queue 列出指定队列的信息  
 -u 若操作符为作业号，则列出其状态。  
 若操作符为destination id，则列出运行在其上的属于user_list 中用户的作业状态。  
 例：# qstat -f 211 查询作业号为211 的作业的具体信息。  
 这个命令默认显示当前用户的任务状态(r/qw/Eqw)，不显示资源消耗情况  
 $ qstat -u zhandl  
 job-ID prior name user state submit/start at queue slots  
 ja-task-ID  
 --------------------------------------------------------------------------------------------  
 591148 0.55000 test.sh zhandl r 01/27/2016 08:54:16 rna.q@compute-0-26.local 1  
 如果要查看任务详细信息可以使用qstat -j job-ID参数或者qsee(之前分享的脚本）  
 $ qsee -u zhandl  
 job-name job-ID node@queue memory_host memory_job  
 cpu_time pro_code  
 -------- ------ ---------- ----------- ----------  
 test.sh 591148 compute-0-26@rna.q (3.9G/62.8G, 17.5M/62.5G) vmem=1.2G/1.2G (VF=15G)  
 01:28:49  
 可以看到消耗的资源、cpu时间等。另qstat -f可以查看节点状态（a/au/o）和load信息  
 $ qstat -f|grep compute-0-1  
 reseq.q@compute-0-1.local BIP 0/5/38 28.63 linux-x64  
 reseq.q@compute-0-10.local BIP 0/10/38 19.23 linux-x64  
 reseq.q@compute-0-11.local BIP 0/5/38 35.23 linux-x64  
 reseq.q@compute-0-12.local BIP 0/6/38 35.39 linux-x64  
 reseq.q@compute-0-13.local BIP 0/6/38 28.47 linux-x64  
 reseq.q@compute-0-14.local BIP 0/5/38 30.65 linux-x64  
 reseq.q@compute-0-15.local BIP 0/4/38 27.32 linux-x64  
 rna.q@compute-0-16.local BIP 0/21/38 -NA- linux-x64 au  
 rna.q@compute-0-17.local BIP 0/8/38 31.26 linux-x64  
 rna.q@compute-0-18.local BIP 0/9/38 25.13 linux-x64  
 rna.q@compute-0-19.local BIP 0/10/38 7.09 linux-x64  
 如上面看到compute-0-16已被跑挂，状态为au，load为-NA-.  
 qstat -f 结果中的states  
 (a)larm, (u)nreachable, (E)rror  
 state (au) whenever:  
 - A node is down  
 - A node is hung/frozen  
 - Network problems  
 (4) qhost 命令—用于看集群资源  
 这个命令是查看集群资源的，如直接qhost会列出节点信息  
 $ qhost|tail -n 5  
 compute-0-8 linux-x64 40 28.80 63.0G 28.3G 62.5G 23.3M  
 compute-0-9 linux-x64 40 23.98 63.0G 16.2G 62.5G 20.9M  
 mysql linux-x64 12 4.01 63.0G 9.8G 62.5G 78.6M  
 super-0-1 linux-x64 80 10.13 1009.9G 227.0G 128.0G 0.0  
 test linux-x64 12 12.74 63.0G 17.0G 62.5G 8.5M  
 也可以查看节点的任务信息  
 $ qhost -j |grep super  
 super-0-1 linux-x64 80 9.40 1009.9G 227.1G 128.0G 0.0  
 585895 0.55000 denovo.sh fufangni r 01/26/2016 16:51:06 super.q@su MASTER  
 587179 0.55000 denovo.sh fufangni r 01/26/2016 16:52:25 super.q@su MASTER  
 591156 0.55000 denovo_lat yukaicheng r 01/27/2016 09:17:48 super.q@su MASTER   
 