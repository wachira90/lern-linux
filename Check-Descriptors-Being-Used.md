# Check Descriptors Being Used

## Find Out PID

````
ps aux | grep mysqld

pidof mysqld

Output:

28290

````

## List File Opened By a PID # 28290
````
# lsof -p 28290
# lsof -a -p 28290

OR

# cd /proc/28290/fd
# ls -l | less

You can count open file, enter:

# ls -l | wc -l

````
