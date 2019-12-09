## monitoring
System monitoring tools


Create graphs easily (without excel or python)

## Install Gnuplot and dstat 
```
sudo apt-get update -y
sudo apt-get install gnuplot-x11 dstat
```
I am using these versions (Dstat 0.7.2, gnuplot 4.6 patchlevel 6) on 16.04.5 LTS (Xenial Xerus)
####################################
####################################

## Start dstat 
This will create the raw file


```
dstat -tc 5 10 > data.raw
```
or
```
dstat --time --cpu --mem --load --disk --io > data.raw
```
or
```
dstat --output stats.csv  -tc 5 20
```
## Create Example gnuplot script, for read asci raw file

```
####################################
#!/usr/bin/gnuplot -persist

set xdata time
set timefmt "%d-%m %H:%M:%S"
set format x "%H:%M"

set yrange [0:100]

set grid

set title "dstat CPU Usage"
set xlabel "time"
set ylabel "total-cpu-usage"

#-----time----- ----total-cpu-usage----
# date/time |usr sys idl wai hiq siq
#13-06 01:18:14| 5 1 93 1 0 0

plot "< tail -n +3 data.raw" using 1:3 w lp t "user [%]", \
"" u 1:4 w lp t "system [%]", \
"" u 1:5 w lp t "idle [%]", \
"" u 1:6 w lp t "wait [%]"

set term png
set output 'dstat.png'
replot
####################################
```
Create a file gplot.sh with the following content, if you read the csv file. 
You need to skip the first 5 lines from the header.
```
#!/usr/bin/gnuplot -persist

set xdata time
set timefmt "%d-%m %H:%M:%S"
set format x "%H:%M"

set yrange [0:100]

set grid
set datafile separator ","
set title "dstat CPU Usage"
set xlabel "time"
set ylabel "total-cpu-usage"

#-----time----- ----total-cpu-usage----
# date/time |usr sys idl wai hiq siq
#13-06 01:18:14| 5 1 93 1 0 0

plot 'dstat.csv' every ::6 using 1:3 w lp t "usr [%]", 'dstat.csv' every ::6 using 1:4 w lp t "sys [%]", \
'dstat.csv' every ::6 using 1:5 w lp t "idl [%]"

set term png
set output 'dstat.png'
replot
```

## Then run

```
gnuplot -persist gplot.sh
```
You will get something like this


<img src="https://github.com/schoenemeyer/monitoring/blob/master/figures/dstat.png" width="580"> <img> 



## Systemload 

Other Options
```
dstat -cndymlp -N total -D total 5 25
dstat -cndymlp -N total --output test
```
With timestamp
```
dstat -tcdrgilmns --output dstat.csv --noupdate 5
```

## FIO for K8s
```
apiVersion: batch/v1
kind: Job
metadata:
  name: fiobench-2
spec:
  template:
    spec:
      volumes:
        - name: efs
          nfs:
            server: 192.168.20.80
            path: /volume1/NFSVolume/temp/gaga/efs
            readOnly: true
      containers:
      - name: fio
        image: ivotron/fio:latest
# Mount the NFS volume in the container
        volumeMounts:
            - name: efs
              mountPath: /efs
        resources:
          limits:
            cpu: "2"
          requests:
            cpu: "2"
        command: ["/bin/sh","-c"]
        args: [" fio --readwrite=read --ioengine=psync --direct=0 --iodepth=64 --bs=64k --size=1g --name=test_fio1 --directory=/efs --numjobs=2"]
      restartPolicy: Never
  backoffLimit: 4
```



