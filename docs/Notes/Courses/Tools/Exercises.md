# MIT6.NULL Exercises

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

 1. For this course, you need to be using a Unix shell like Bash or ZSH. If you are on Linux or macOS, you don’t have to do anything special. If you are on Windows, you need to make sure you are not running cmd.exe or PowerShell; you can use Windows Subsystem for Linux or a Linux virtual machine to use Unix-style command-line tools. To make sure you’re running an appropriate shell, you can try the command `echo $SHELL`. If it says something like `/bin/bash` or `/usr/bin/zsh`, that means you’re running the right program.

    ```Shell
    ~$ echo $SHELL
    /usr/bin/zsh
    ```

 2. Create a new directory called `missing` under `/tmp`.

    ```Shell
    ~$ cd /tmp
    /tmp$ sudo mkdir missing 
    ```

 3. Look up the `touch` program. The `man` program is your friend.

    ```Shell
    /tmp$ man touch
    ```

 4. Use `touch` to create a new file called `semester` in `missing`.

    ```Shell
    /tmp$ touch /missing/semester
    ```

 5. Write the following into that file, one line at a time:

    ```Shell
    #!/bin/sh
    curl --head --silent https://missing.csail.mit.edu
    ```

    ```Shell
    /tmp/missing$ cd missing
    /tmp/missing$ echo '#!/bin/sh' > semester
    /tmp/missing$ echo 'curl --head --silent https://missing.csail.mit.edu' >> semester
    /tmp/missing$ cat semester
    #!/bin/sh
    curl --head --silent https://missing.csail.mit.edu
    ```

 6. Try to execute the file, i.e. type the path to the script (`./semester`) into your shell and press enter. Understand why it doesn’t work by consulting the output of `ls` (hint: look at the permission bits of the file).

    ```Shell
    /tmp/missing$ ./semester
    zsh: permission denied: ./semester
    /tmp/missing$ ls -l
    -rw-r--r-- 1 ubuntu ubuntu  61 Oct 15 12:44 semester
    ```

 7. Run the command by explicitly starting the `sh` interpreter, and giving it the file `semester` as the first argument, i.e. `sh semester`. Why does this work, while `./semester` didn’t?

    ```Shell
    /tmp/missing$ sh semester
    ```

 8. Look up the `chmod` program (e.g. use `man chmod`).

    ```Shell
    /tmp/missing$ man chmod
    ```

9. Use chmod to make it possible to run the command ./semester rather than having to type sh semester. How does your shell know that the file is supposed to be interpreted using sh? See this page on the shebang line for more information.

    ```Shell
    /tmp/missing$ chmod +x semester
    /tmp/missing$ ls -l
    -rwxr-xr-x 1 ubuntu ubuntu  61 Oct 15 12:44 semester
    /tmp/missing$ ./semester
    ```

10. Use `|` and `>` to write the “last modified” date output by `semester` into a file called `last-modified.txt` in your home directory.

    ```Shell
    /tmp/missing$ ./semester | grep last-modified > ~/last-modified.txt
    /tmp/missing$ cat ~/last-modified.txt
    last-modified: Sat, 17 Sep 2022 10:59:54 GMT
    ```

11. Write a command that reads out your laptop battery’s power level or your desktop machine’s CPU temperature from `/sys`. Note: if you’re a macOS user, your OS doesn’t have sysfs, so you can skip this exercise.

    ```Shell
    /tmp/missing$ cat /sys/class/power_supply/battary
    100
    ```

## <a name="lecture2"></a>Shell Tools and Scripting

## <a name="lecture3"></a>Editors(vim)

## <a name="lecture4"></a>Data Wrangling

## <a name="lecture5"></a>Command-line Environment

## <a name="lecture6"></a>Version Control(git)

## <a name="lecture7"></a>Debugging and Profiling

## <a name="lecture8"></a>Metaprogramming

## <a name="lecture9"></a>Security and Cryptography

