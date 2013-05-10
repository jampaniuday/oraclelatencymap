


OraLatencyMap


see http://dtrace.org/blogs/brendan/2009/03/17/heat-map-analytics/ for some information on heat maps.

see http://externaltable.blogspot.com/2013/05/latency-heat-map-in-sqlplus-with.html for original.

To run an auto refresh color coded heatmap on "log file sync" in sqlplus simple do

    sqlplus / as sysdba
    SQL> @OraLatencyMap_event 3 "log file sync"


where OraLatencyMap_event.sql is in your current directory or sqlpath


Now to play with LGWR latency with cgroup throttles see


https://fritshoogland.wordpress.com/2012/12/15/throttling-io-with-linux/

    # install cgroups on 2.6.24 LINUX or higher
    yum intall cgroup

    # setup /cgroup/blkio
    grep blkio /proc/mounts || mkdir -p /cgroup/blkio ; mount -t cgroup -o blkio none /cgroup/blkio
    cgcreate -g blkio:/iothrottle


    # find the device you want
    df -k
    # my Oracle log file divice was
    ls -l /dev/mapper/vg_source-lv_home
    lrwxrwxrwx. 1 root root 7 May  1 21:42 /dev/mapper/vg_source-lv_home -> ../dm-2

    # my device points to /dev/dm-2
    ls -l /dev/dm-2
    brw-rw----. 1 root disk 253, 2 May  1 21:42 /dev/dm-2

    # my device  major and minor numbers are "253, 2"
    # create a write throtte on this device (for read just replace "write" with "read"
    # this limits it to 10 writers per second
    cgset -r blkio.throttle.write_iops_device="253:2 10" iothrottle

    # look for lgwr
    ps -ef | grep lgwr
    oracle   23165     1  0 13:35 ?        00:00:19 ora_lgwr_o1123

    # put lgwr pid into throttle group
    echo 23165     >  /cgroup/blkio/iothrottle/tasks

    # now play with different throttles
    cgset -r blkio.throttle.write_iops_device="253:2 1" iothrottle
    cgset -r blkio.throttle.write_iops_device="253:2 10" iothrottle
    cgset -r blkio.throttle.write_iops_device="253:2 100" iothrottle
    cgset -r blkio.throttle.write_iops_device="253:2 1000" iothrottle

    # if you are finishe delete it
    cgdelete  blkio:/iothrottle
