# MIT6.NULL Course Notes

## Table of Contents

[1. Course Overview and the Shell](#lecture1)

[2. Shell Tools and Scripting](#lecture2)

[3. Editors(vim)](#lecture3)

[4. Data Wrangling](#lecture4)

[5. Command-line Environment](#lecture5)

[6. Version Control(git)](#lecture6)

[7. Debugging and Profiling](#lecture7)

[8. Metaprogramming](#lecture8)

[9. Security and Cryptography](#lecture9)

## <a name="lecture1"></a>Course Overview and the Shell

### Definitions

Shell
: A texture interface that exposes an operating system's services to a human user or other programs.

Kernel Shell and Application
: ![Kernel Shell and Application](./Kernel_Shell_Application.png)
: The kernel is the core of the operating system that controls all the tasks of the system while the shell is the interface that allows the users to communicate with the kernel


### Useful Commands

- `date`
- `echo`

    ```Shell
    ~$ echo Hello
    Hello
    ```

    ```Shell
    ~$ echo $PATH
    /usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin
    ```

- `which`
  
    ```Shell
    ~$ which echo
    /bin/echo
    ~$ /bin/echo $PATH
    /usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin
    ```

- `pwd`: present working directory
- `cd`: change directory

    `.`: current directory
    `..`: parent directory
    `~`: home directory
    `-`: previous directory

- `ls`: list files in current working directory

    `-l`,`-a`,`-h`,...

- `man`: manual pages
- `touch`,`mkdir`,`rm`,`rmdir`

    `rm -rf`

- `chmod`
- `cat` is a program that concatenates files.

### Connecting Programs

- `< file` and `> file` let you rewire the input and output streams of a program to a file respectively:

    ```Shell
    ~$ echo hello > hello.txt
    ~$ cat hello.txt
    hello
    ~$ cat < hello.txt
    hello
    ~$ cat < hello.txt > hello2.txt
    ~$ cat hello2.txt
    hello
    ```

- `>> file` is used to append the contents to the file:

    ```Shell
    ~$ echo hello > hello2.txt
    ~$ cat hello2.txt
    hello
    hello
    ```

- `|` lets you “chain” programs such that the output of one is the input of another:

    ```Shell
    ~$ ls -l / | tail -n1
    drwxr-xr-x 1 root  root  4096 Jun 20  2019 var
    ```

## <a name="lecture2"></a>Shell Tools and Scripting

- `'` or `''`: Strings delimited with `'` are literal strings and will not substitute variable values whereas `"` delimited strings will.

    ```Shell
    foo=bar
    echo "$foo"
    # prints bar
    echo '$foo'
    # prints $foo
    ```

### Arguments

- `$0` - Name of the script
- `$1` to `$9` - Arguments to the script. `$1` is the first argument and so on.
- `$@` - All the arguments
- `$#` - Number of arguments
- `$?` - Return code of the previous command
- `$$` - Process identification number (PID) for the current script
- `!!` - Entire last command, including arguments. A common pattern is to execute a command only for it to fail due to missing permissions; you can quickly re-execute the command with sudo by doing `sudo !!`
- `$_` - Last argument from the last command. If you are in an interactive shell, you can also quickly get this value by typing `Esc` followed by `.` or `Alt+`.

### Operators

- `&&` - and operator.
- `||` - or operator.
- `;` - separate commands within the same line.

## <a name="lecture3"></a>Editors(vim)

## <a name="lecture4"></a>Data Wrangling

## <a name="lecture5"></a>Command-line Environment

## <a name="lecture6"></a>Version Control(git)

## <a name="lecture7"></a>Debugging and Profiling

## <a name="lecture8"></a>Metaprogramming

## <a name="lecture9"></a>Security and Cryptography

