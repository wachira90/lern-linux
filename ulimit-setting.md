# soft, hard, and unlimited

## Resolution
If LIMIT is given, it is the new value of the specified resource; the special LIMIT values soft, hard, and unlimited stand for the current soft limit, the current hard limit, and no limit, respectively.

How to check the Hard limit and soft limit ?

### Check hard limit :

````
docker@test-imac:~$ ulimit -a -H
core file size          (blocks, -c) unlimited
data seg size           (kbytes, -d) unlimited
scheduling priority             (-e) 0
file size               (blocks, -f) unlimited
pending signals                 (-i) 63613
max locked memory       (kbytes, -l) 65536
max memory size         (kbytes, -m) unlimited
open files                      (-n) 1048576
pipe size            (512 bytes, -p) 8
POSIX message queues     (bytes, -q) 819200
real-time priority              (-r) 0
stack size              (kbytes, -s) unlimited
cpu time               (seconds, -t) unlimited
max user processes              (-u) 63613
virtual memory          (kbytes, -v) unlimited
file locks                      (-x) unlimited
````
### Check soft limit :

````
docker@test-imac:~$ ulimit -a -S
core file size          (blocks, -c) 0
data seg size           (kbytes, -d) unlimited
scheduling priority             (-e) 0
file size               (blocks, -f) unlimited
pending signals                 (-i) 63613
max locked memory       (kbytes, -l) 65536
max memory size         (kbytes, -m) unlimited
open files                      (-n) 1024
pipe size            (512 bytes, -p) 8
POSIX message queues     (bytes, -q) 819200
real-time priority              (-r) 0
stack size              (kbytes, -s) 8192
cpu time               (seconds, -t) unlimited
max user processes              (-u) 63613
virtual memory          (kbytes, -v) unlimited
file locks                      (-x) unlimited
docker@test-imac:~$
````

This solution applies to all limits set in the various limits files and with the ulimit command, not just nproc. It will refer to nproc specifically, but that's just for convenience sake, and because that seems to be a commonly tripped upon limit. Please keep in mind that the same sort of ideas apply to all such limits.

A soft limit is still a limit. A user cannot exceed a soft limit. If the user already has, for example, at least as many processes as their nproc soft or hard limit, any attempt to spawn another process (or change the UID of the current process to that user) will fail. A non-root user cannot exceed a soft limit, but what the non-root user can do is increase their soft limit up to the hard their limit. A hard limit cannot be increased by a non-root user. Only root can increase its own hard limit. So, when you see that nproc has a soft limit of, say, 2047 processes, that means that in order to be able to run more than this, the user in question must have issued a ulimit command somewhere (probably in a startup file like ~/.bashrc or similar, maybe even at the command line) and increased their soft limit to at least the number of processes that you see running.

If a non-root user changes a limit such as nproc using ulimit, this will only effect that process, and any child processes that process creates after changing that limit. Therefore, other login sessions, processes spawned before the limit was changed, or anything other than processes that are spawned by that process after this ulimit change will be unaffected. So, let's say you look through your startup scripts and find that the .profile (for instance) for the oracle user has a line in it like:

````
ulimit -u 8192
````
this might be perfectly legal (if your hard limit for oracle is, for example, 16384). Any processes spawned after this command would be subject to that limit, and once reached, attempts to spawn new processes would fail. Now, let's say you see something like:
````
# ps -u oracle  -L  | wc -l
6283
````
The ulimit command found in the oracle startup file would explain how oracle was allowed you to reach the 6K+ processes you see oracle is currently running, but the tricky part is, it also explains why you wouldn't be able to log in or su to the oracle user. You see, before you can get to this command in whatever startup file it is located in, you have to first log in. At that point, the soft limit is still whatever it was before it is increased in your shell's startup file (presumably something less than 6283), which you will not be permitted to exceed. Since you already have more than this running already, you cannot log in or change to the user.

### example docker-compose.yml set ulimits

````
version: '2.2'
services:
  es01:
    image: docker.elastic.co/elasticsearch/elasticsearch:7.16.0
    container_name: es01
    environment:
      - node.name=es01
      - cluster.name=es-docker-cluster
      - discovery.seed_hosts=es02,es03
      - cluster.initial_master_nodes=es01,es02,es03
      - bootstrap.memory_lock=true
      - "ES_JAVA_OPTS=-Xms512m -Xmx512m"
    ulimits:
      memlock:
        soft: -1
        hard: -1
    volumes:
      - data01:/usr/share/elasticsearch/data
    ports:
      - 9200:9200
    networks:
      - elastic
  es02:
    image: docker.elastic.co/elasticsearch/elasticsearch:7.16.0
    container_name: es02
    environment:
      - node.name=es02
      - cluster.name=es-docker-cluster
      - discovery.seed_hosts=es01,es03
      - cluster.initial_master_nodes=es01,es02,es03
      - bootstrap.memory_lock=true
      - "ES_JAVA_OPTS=-Xms512m -Xmx512m"
    ulimits:
      memlock:
        soft: -1
        hard: -1
    volumes:
      - data02:/usr/share/elasticsearch/data
    networks:
      - elastic
  es03:
    image: docker.elastic.co/elasticsearch/elasticsearch:7.16.0
    container_name: es03
    environment:
      - node.name=es03
      - cluster.name=es-docker-cluster
      - discovery.seed_hosts=es01,es02
      - cluster.initial_master_nodes=es01,es02,es03
      - bootstrap.memory_lock=true
      - "ES_JAVA_OPTS=-Xms512m -Xmx512m"
    ulimits:
      memlock:
        soft: -1
        hard: -1
    volumes:
      - data03:/usr/share/elasticsearch/data
    networks:
      - elastic

volumes:
  data01:
    driver: local
  data02:
    driver: local
  data03:
    driver: local

networks:
  elastic:
    driver: bridge
````
