# monitoring
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
dstat -tc 5 10 > dstat.raw
```
or
```
dstat -tcdrgilmns 5 500 > dstat.raw
```

## Create Example gnuplot script, e.g.

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

plot "< tail -n +3 dstat.raw" using 1:3 w lp t "user [%]", \
"" u 1:4 w lp t "system [%]", \
"" u 1:5 w lp t "idle [%]", \
"" u 1:6 w lp t "wait [%]"

set term png
set output 'dstat.png'
replot
####################################
```


## Then run

```
gnuplot -persist gplot.sh
```

You will get something like this



![After processing](https://github.com/schoenemeyer/monitoring/blob/master/figures/dstat.pngfigures/dstat.png)



Systemload 
```
dstat -cndymlp -N total -D total 5 25
dstat -cndymlp -N total --output test
With timestamp
dstat -tcdrgilmns --output dstat.csv --noupdate 5
```




