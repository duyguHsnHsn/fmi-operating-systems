# Processes exercises

### 04-a-5000
Намерете командите на 10-те най-стари процеси в системата.
````shell
ps -ef | tail -n +2 | sort -n -k 2 | head -n 10 | awk '{print $NF}'
````

### 04-a-6000
Намерете PID и командата на процеса, който заема най-много виртуална памет в системата.
````shell
ps -e -o pid= -o vsize= -o cmd= | sort -nr -k 2 | head -n 1 | awk '{print $1, $3}'
````

### 04-a-6300
Изведете командата на най-стария процес
````shell
ps -e -o comm= --sort=start_time | head -n 1
````

### 04-b-5000
Намерете колко физическа памет заемат всички процеси на потребителската група root.
````shell
ps -e -g root -o rss= | awk -v "SUM=0" '{SUM+=$1} END {print SUM}'
````

### 04-b-6100
Изведете имената на потребителите, които имат поне 2 процеса, чиято команда е vim (независимо какви са аргументите й)
````shell
ps -e -o uname= -o comm= | grep "vim" | sort | uniq -c | awk '$1>=2 {print $2}'
````
### 04-b-6200
Изведете имената на потребителите, които не са логнати в момента, но имат живи процеси
````shell
ps -e -o uname= | grep -v "$(who | awk '{print $1}')" | sort | uniq
````

### 04-b-7000
Намерете колко физическа памет заемат осреднено всички процеси на потребителската група root. 
Внимавайте, когато групата няма нито един процес.
````shell
ps -e -g root -o rss= | awk -v"SUM=0" '{SUM+=$1} END { if(NR > 0) print SUM/NR; else print 0}'
````

### 04-b-8000
Намерете всички PID и техните команди (без аргументите), които нямат tty, което ги управлява.
Изведете списък само с командите без повторения.
````shell
ps -e -o tty= -o comm= | awk '$1=="?" {$1=""; print $0}' | cut -c 2- | sort | uniq
````


### 04-b-9000
Да се отпечатат PID на всички процеси, които имат повече деца от родителския си процес.
````shell
ps -e -o pid= -o ppid= | awk '
{
    # save PID children
    children[$2]++;
    # save PID parent
    parent[$1] = $2;
}
END {
    for (pid in parent) {
        ppid = parent[pid];
        if (children[pid] > children[ppid]) {
            print pid;
        }
    }
}'

````