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

## Count All Open File Handles

````
# lsof | wc -l
````
## List File Descriptors in Kernel Memory
````
# sysctl fs.file-nr
````
Sample outputs:
````
fs.file-nr = 1020	0	70000
````
Where
````
=> 1020 The number of allocated file handles.
=> 0 The number of unused-but-allocated file handles.
=> 70000 The system-wide maximum number of file handles.
````

You can use the following to find out or set the system-wide maximum number of file handles:
````
# sysctl fs.file-max
````
Sample outputs:
````
fs.file-max = 70000
````



