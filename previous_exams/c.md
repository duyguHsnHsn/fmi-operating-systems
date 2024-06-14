# C exam questions 


### Зад. 71 2016-SE-02
 Двоичните файлове f1 и f2 съдържат 32 битови числа без знак (uint32_t). Файлът f1 е
съдържа 𝑛 двойки числа, нека 𝑖-тата двойка е <𝑥𝑖, 𝑦𝑖>. Напишете програма на C, която извлича
интервалите с начало 𝑥𝑖 и дължина 𝑦𝑖 от файла f2 и ги записва залепени в изходен файл f3.
Пример:
- f1 съдържа 4 числа (2 двойки): 30000, 20, 19000, 10
- програмата записва в f3 две поредици 32-битови числа, взети от f2 както следва:
- най-напред се записват числата, които са на позиции 30000, 30001, 30002, ... 30019.
- след тях се записват числата от позиции 19000, 19001, ... 19009.

Забележка: С пълен брой точки ще се оценяват решения, които работят със скорост, пропорционална
на размера на изходния файл f3.


```c

#include <stdio.h>
#include <stdlib.h>
#include <fcntl.h>
#include <unistd.h>
#include <stdint.h>

#define BUFFER_SIZE 1024  

int get_file_size(int);

int get_file_size(int fd) {
	struct stat st;

	if(fstat(fd, &st) == -1) {
		err(1, "stat");
	}

	return st.st_size;
}

int main(int argc, char *argv[]) {
    if (argc != 4) {
        errx(1, "Usage: %s <f1> <f2> <f3>\n", argv[0]);
    }

   	int f1 = open(argv[1], O_RDONLY);
	if(f1 == -1) {
		err(1, "%s", argv[1]);
	}

    int f2 = open(argv[2], O_RDONLY);
    if(f2 == -1) {
		err(1, "%s", argv[2]);
	}


    int f3 = open(argv[3], O_WRONLY | O_TRUNC | O_CREAT, S_IRUSR | S_IWUSR);
    if(f3 == -1) {
		err(1, "%s", argv[3]);
	}

    
	uint32_t f1_size = get_file_size(f1);
	uint32_t f2_size = get_file_size(f2);

	if(f1_size % (2 * sizeof(uint32_t)) != 0) {
		errx(1, "%s: bad file size", argv[1]);
	}

	if(f2_size % sizeof(uint32_t) != 0) {
		errx(1, "%s: bad file size", argv[2]);
	}

    uint32_t x, y;
    ssize_t read_bytes;
    uint32_t buffer[BUFFER_SIZE];

    while ((read_bytes = read(f1, &x, sizeof(uint32_t))) == sizeof(uint32_t)) {

        read_bytes_second = read(f1, &y, sizeof(uint32_t));
        if(read_bytes_second == -1) {
	    	err(4, "can't read f1");
    	}

        if((x + y) * sizeof(uint32_t) >= f2_size) {
			errx(2, "invalid x %d", x);
		}

        if(lseek(f2, x * sizeof(uint32_t), SEEK_SET) == -1) {
			err(2, "lseek f2");
		}

        while (y > 0) {
            size_t to_read = (y > BUFFER_SIZE) ? BUFFER_SIZE : y;
            read_bytes_third = read(f2, buffer, to_read * sizeof(uint32_t));
            if(read_bytes_third == -1) {
	    	    err(4, "can't read f2");
    	    }

            ssize_t written_bytes = write(f3, buffer, read_bytes);
             if(written_bytes == -1) {
	    	    err(4, "can't write f3");
    	    }

            y -= to_read;
        }
    }

    if(read_bytes == -1) {
		err(4, "can't read f1");
	}


    close(f1);
    close(f2);
    close(f3);

    return 0;
}

```


### Зад. 75 2017-SE-02

 Напишете програма на C, която да работи подобно на командата c a t , реализирайки само
следната функционалност:
- общ вид на изпълнение: . /main [OPTION] [FILE]...
- ако е подаден като първи параметър - n , то той да се третира като опция, което кара програмата
ви да номерира (глобално) всеки изходен ред (започвайки от 1).
- програмата извежда на STDOUT
- ако няма подадени параметри (имена на файлове), програмата чете от STDIN
- ако има подадени параметри – файлове, програмата последователно ги извежда
- ако някой от параметрите е тире (-), програмата да го третира като специално име за STDIN


Примерно извикване:
```txt

$ cat a.txt
a1
a2

$ cat b.txt
b1
b2
b3

$ echo -e "s1\ns2" | ./main -n a.txt - b.txt
1 a1
2 a2
3 s1
4 s2
5 b1
6 b2
7 b3
```

Забележка: Погледнете setbuf(3) и strcmp(3)


```c
#include <fcntl.h>
#include <unistd.h>
#include <string.h>
#include <stdlib.h>
#include <stdio.h>

void print_line(const char *line, size_t length, int line_number, int number_lines);
void process_file(int fd, int number_lines, int *line_counter);

void print_line(const char *line, size_t length, int line_number, int number_lines) {
    ssize_t bytes_written;

    if (number_lines) {
        // Convert the line number to a string and write it to STDOUT
        char num_str[20];
        int num_length = sprintf(num_str, "%d ", line_number); 
        bytes_written = write(1, num_str, num_length); 
        if (bytes_written == -1) {
             err(3, "can't write line number to STDOUT");
        }
    }
    
    bytes_written = write(1, line, length); 
    if (bytes_written == -1) {
        if (written == -1) {
             err(3, "can't write line content to STDOUT");
        }
    }
}

void process_file(int fd, int number_lines, int *line_counter) {
    char buffer[4096];
    ssize_t bytes_read;
    size_t line_length = 0;
    char line[4096];
    int i;

    while ((bytes_read = read(fd, buffer, sizeof(buffer))) > 0) {
        for (i = 0; i < bytes_read; i++) {
            line[line_length++] = buffer[i];
            if (buffer[i] == '\n') {
                print_line(line, line_length, *line_counter, number_lines);
                (*line_counter)++;
                line_length = 0;
            } 
        }
    }

    
    if(bytes_read == -1) {
		err(2, "can't read");
	}


    // Print the last line if the file does not end with a newline
    if (line_length > 0) {
        line[line_length++] = '\n';
        print_line(line, line_length, *line_counter, number_lines);
        (*line_counter)++;
    }
}

int main(int argc, char *argv[]) {
    int number_lines = 0;
    int line_counter = 1;
    int arg_index = 1;

    //  -n option
    if (argc > 1 && strcmp(argv[1], "-n") == 0) {
        number_lines = 1;
        arg_index = 2;
    }

    // STDIN
    if (argc == arg_index) {
        process_file(0, number_lines, &line_counter);
        return 0;
    }

    for (int i = arg_index; i < argc; i++) {
        if (strcmp(argv[i], "-") == 0) {
            // STDIN
            process_file(0, number_lines, &line_counter); 
        } else {
            int fd = open(argv[i], O_RDONLY);
            if (fd == -1) {
                err(1, "can't open file %s", argv[i]);
            }
            process_file(fd, number_lines, &line_counter);
            close(fd);
        }
    }

    return 0;
}

```

### Зад. 78 2018-SE-01 
Напишете програма на C, която да работи подобно на командата tr, реализирайки само
следната функционалност:
- програмата чете от стандартния вход и пише на стандартния изход
- общ вид на изпълнение: ./main [OPTION] SET1 [SET2]
- OPTION би могъл да бъде или -d, или -s, или да липсва
- SET1 и SET2 са низове, в които знаците нямат специално значение, т.е. всеки знак “означава”
съответния знак. SET2, ако е необходим, трябва да е същата дължина като SET1
- ако е подаден като първи параметър -d, програмата трие от входа всяко срещане на знак 𝜇 от
SET1; SET2 не е необходим
- ако е подаден като първи параметър -s, програмата заменя от входа всяка поредица от повторения знак 𝜇 от SET1 с еднократен знак 𝜇; SET2 не е необходим
- в останалите случаи програмата заменя от входа всеки знак 𝜇 от SET1 със съответстващият му
позиционно знак 𝜈 от SET2.


Примерно извикване:

```shell
$ echo asdf | ./main -d ' d123' | ./main 'sm' 'fa' | ./main -s 'f'
af
```

```c

#include <fcntl.h>
#include <unistd.h>
#include <string.h>
#include <stdlib.h>
#include <stdio.h>


int main(int argc, char *argv[]) {
    if (argc < 3) {
        errx(1, "Usage: %s  [OPTION] SET1 [SET2] \n", argv[0]);
    }

    if (argv[1] == "-n")

    return 0;
}

```

### Зад. 101 2020-SE-01

 Напишете две програми на C (foo и bar), които си комуникират през 
Програмата foo приема параметър - име на файл, програмата bar приема параметър - команда като
абсолютен път до изпълним файл.
Примерни извиквания и ред на изпълнение (в отделни терминали):

./foo a.txt
./bar /usr/bin/sort

Програмата foo трябва да изпълнява външна команда cat с аргумент името на подадения файл, така
че съдържанието му да се прехвърли през тръбата към програмата bar , която от своя страна трябва
да изпълни подадената и като аргумент команда (без параметри; /usr/bin/sort в примера), която да
обработи получените през тръбата данни, четейки от стандартен вход. Еквивалент на горния пример
би било следното изпълнение:

cat a.txt| /usr/bin/sort


foo.c

```c
#include <stdio.h>
#include <stdlib.h>
#include <fcntl.h>
#include <unistd.h>
#include <sys/types.h>
#include <sys/stat.h>
#include <err.h>

#define FIFO_NAME "/tmp/my_fifo" // наименована тръба

int main(int argc, char *argv[]) {
    if (argc != 2) {
        errx(1, "Usage: %s <filename>\n", argv[0]);
    }

    if (mkfifo(FIFO_NAME, 0666) == -1) {
       errx(1, "mkfifo");
    }

    int fifo_fd = open(FIFO_NAME, O_WRONLY);
    if (fifo_fd == -1) {
        errx(1, "open fifo for writing");
    }

    int file_fd = open(argv[1], O_RDONLY);
    if (file_fd == -1) {
        errx(1, "open input file");
    }

    char buffer[4096];
    ssize_t bytes_read;
    while ((bytes_read = read(file_fd, buffer, sizeof(buffer))) > 0) {
        if (write(fifo_fd, buffer, bytes_read) != bytes_read) {
            errx(1, "write to fifo");
        }
    }

    close(file_fd);
    close(fifo_fd);

    return 0;
}
```

bar.c

```c
#include <stdio.h>
#include <stdlib.h>
#include <fcntl.h>
#include <unistd.h>
#include <sys/types.h>
#include <sys/stat.h>
#include <err.h>

#define FIFO_NAME "/tmp/my_fifo"

int main(int argc, char *argv[]) {
        if (argc != 2) {
        errx(1, "Usage: %s <command>\n", argv[0]);
    }


    int fifo_fd = open(FIFO_NAME, O_RDONLY);
    if (fifo_fd == -1) {
        errx(1,  "open fifo for reading");
    }

    // Redirect stdin to the FIFO
    if (dup2(fifo_fd, 0) == -1) {
        errx(1 "dup2");
    }

    execl(argv[1], argv[1], (char *)NULL);
    err(1, , "execl");

    return 0;
}
```

### Зад. 102 2020-SE-02
```text

При изграждане на система за пренасяне на сериен асинхронен сигнал върху радиопреносна мрежа се оказало, че големи поредици от битове само нули или само единици смущават сигнала, поради нестабилно ниво. Инженерите решили проблема, като:
- в моментите, в които няма сигнал от серийният порт, вкарвали изкуствено байт 0x55 в потока;
- реалните байтове 0x00, 0xFF, 0x55 и 0x7D се кодирали посредством XOR-ване (побитова обработка с изключващо-или) с 0x20, като полученият байт се изпращал през потока, предхождан от 0x7D, който играе ролята на escape character.

Разполагате със запис от такъв поток. Напишете програма на C, която приема два параметъра - имена
на файлове. Примерно извикване:

$./main input.lfld output.bin

Програмата трябва да обработва записа и да генерира output.bin, който да съдържа оригиналните
данни. Четенето на входните данни трябва да става посредством изпълнение на външна shell команда.
```

```c
#include <stdio.h>
#include <stdlib.h>
#include <unistd.h>
#include <fcntl.h>
#include <stdint.h>
#include <sys/types.h>
#include <sys/wait.h>
#include <err.h>

#define ESCAPE_CHAR 0x7D
#define XOR_VALUE 0x20
#define BUFFER_SIZE 4096

void decode_and_write(int input_fd, int output_fd) {
    uint8_t buffer[BUFFER_SIZE];
    ssize_t bytes_read;
    while ((bytes_read = read(input_fd, buffer, sizeof(buffer))) > 0) {
        for (ssize_t i = 0; i < bytes_read; i++) {
            uint8_t byte = buffer[i];
            if (byte == ESCAPE_CHAR) {
                // read the next byte and decode it
                i++;
                if (i < bytes_read) {
                    byte = buffer[i] ^ XOR_VALUE;
                } else {
                    // handle case where escape character is the last byte in buffer
                    ssize_t extra_byte = read(input_fd, &byte, 1);
                    if (extra_byte != 1) {
                        errx(1, "Unexpected end of file after escape character");
                    }
                    byte ^= XOR_VALUE;
                }
            }
            if (write(output_fd, &byte, 1) != 1) {
                errx(1, "write to output file");
            }
        }
    }
    if (bytes_read == -1) {
        err(EXIT_FAILURE, "read from input file");
    }
}

int main(int argc, char *argv[]) {
    if (argc != 3) {
        errx(1, "Usage: %s <input_file> <output_file>\n", argv[0]);
    }

    int pipe_fds[2];
    if (pipe(pipe_fds) == -1) {
        errx(2,"pipe");
    }

    pid_t pid = fork();
    if (pid == -1) {
        errx(2, "fork");
    }

    if (pid == 0) {
        // child process executes the cat command
        close(pipe_fds[0]); // close unused read end
        if (dup2(pipe_fds[1], STDOUT_FILENO) == -1) {
            errx(2, "dup2");
        }
        close(pipe_fds[1]); // close the original write end after duplication

        execlp("cat", "cat", argv[1], (char *)NULL);
        errx(1, "execlp cat");

    } else {
        // parent process reads from pipe and writes to output file
        close(pipe_fds[1]); // close unused write end

        int output_fd =  open(argv[2], O_WRONLY | O_TRUNC | O_CREAT, S_IRUSR | S_IWUSR);
        if (output_fd == -1) {
            errx(1, "open output file");
        }

        decode_and_write(pipe_fds[0], output_fd);

        close(pipe_fds[0]);
        close(output_fd);

        int status;
        wait(&status);
        if (!WIFEXITED(status) || WEXITSTATUS(status) != 0) {
            errx(1, "child process failed");
        }
    }

    return 0;
}
```

### Зад. 108 2023-SE-02

Напишете програма, която изпълнява списък от команди в паралелни процеси, но щом някой от процесите изведе ред с текста “found it!”, 
прекратява изпълнението на всички процеси.
За улеснение, командите нямат свои аргументи и се подават като аргументи на вашата програма.
Примерно извикване: ./main ./tests/foo ./tests/bar ./tests/baz

Прекратяването на процес става, като му изпратим SIGTERM и след това го изчакаме да приключи.
Изходът на командите (извън това дали stdout съдържа търсения текст) и техните статуси не ни интересуват. Програмата ви не трябва да извежда нищо, 
а нейният статус трябва да е:
- 0, ако някой процес е извел текста “found it!”
- 1, ако всички процеси са завършили и никой процес не е извел текста
- 26 при друга грешка

```c
#include <stdio.h>
#include <stdlib.h>
#include <unistd.h>
#include <sys/types.h>
#include <sys/wait.h>
#include <signal.h>
#include <string.h>
#include <errno.h>

#define BUFFER_SIZE 4096

void terminate_all(pid_t *pids, int count) {
    for (int i = 0; i < count; i++) {
        if (pids[i] > 0) {
            kill(pids[i], SIGTERM);
        }
    }
}

int main(int argc, char *argv[]) {
    if (argc < 2) {
        errx(26, "Usage: %s <command1> <command2> ...\n", argv[0]);
    }

    int num_commands = argc - 1;
    pid_t pids[num_commands];
    int pipes[num_commands][2];
    int found = 0;

    for (int i = 0; i < num_commands; i++) {
        if (pipe(pipes[i]) == -1) {
            errx(26,"pipe");
        }

        pid_t pid = fork();
        if (pid == -1) {
            errx(26, "fork");
        } else if (pid == 0) {

            close(pipes[i][0]); // close the read end

            dup2(pipes[i][1], 1); // dup stdout 
            close(pipes[i][1]);

            execlp(argv[i + 1], argv[i + 1], (char *)NULL);
            errx(26, "execlp");
        } else {

            pids[i] = pid; //  store the process id of each child process created 
            close(pipes[i][1]); // close the write end
        }
    }

   char buffer[BUFFER_SIZE];
    for (int i = 0; i < num_commands; i++) {
        int bytes_read;
        while ((bytes_read = read(pipes[i][0], buffer, sizeof(buffer))) > 0) {
            buffer[bytes_read] = '\0';
            if (strstr(buffer, "found it!")) {
                found = 1;
                terminate_all(pids, num_commands);
                break;
            }
        }
        close(pipes[i][0]);
        if (found) break;
    }

    for (int i = 0; i < num_commands; i++) {
        int status;
        if (wait(&status) == -1) {
            errx(26, "wait");
        }
        if (!WIFEXITED(status) || WEXITSTATUS(status) != 0) {
            errx(26, "child process failed");
        }
    }

    if (found) {
        exit(0);
    } else {
        exit(1);
    }
}

```

### Зад. 100 2019-SE-01
 Напишете програма-наблюдател 𝑃, която изпълнява друга програма 𝑄 и я рестартира,
когато 𝑄 завърши изпълнението си. На командния ред на 𝑃 се подават следните параметри:
- праг за продължителност в секунди – едноцифрено число от 1 до 9
- 𝑄
- незадължителни параметри на 𝑄


𝑃 работи по следния алгоритъм:
- стартира 𝑄 с подадените параметри
- изчаква я да завърши изпълнението си
- записва в текстов файл run.log един ред с три полета - цели числа (разделени с интервал):
    * момент на стартиране на 𝑄 (Unix time)
    * момент на завършване на 𝑄 (Unix time)
    * код за грешка, с който 𝑄 е завършила (exit code)
- проверява дали е изпълнено условието за спиране и ако не е, преминава отново към стартирането на 𝑄


Условие за спиране: Ако наблюдателят 𝑃 установи, че при две последователни изпълнения на 𝑄 са
били изпълнени и двете условия:

1. кодът за грешка на 𝑄 е бил различен от 0;
2. разликата между момента на завършване и момента на стартиране на 𝑄 е била по-малка от
подадения като първи параметър на 𝑃 праг; то 𝑃 спира цикъла от изпълняване на 𝑄 и сам завършва изпълнението си.


Текущото време във формат Unix time (секунди от 1 януари 1970 г.) можете да вземете с извикване на
системната функция time() с параметър NULL; функцията е дефинирана в time.h. Ако изпълнената
програма е била прекъсната от подаден сигнал, това се приема за завършване с код за грешка 129

```c
#include <stdio.h>
#include <stdlib.h>
#include <unistd.h>
#include <sys/types.h>
#include <sys/wait.h>
#include <signal.h>
#include <string.h>
#include <errno.h>
#include <fcntl.h>
#include <time.h>

void log_run(time_t start_time, time_t end_time, int exit_code) {
    int log_fd = open("run.log",  O_WRONLY | O_TRUNC | O_CREAT, S_IRUSR | S_IWUSR);
    if (log_fd == -1) {
        errx(1, "open");
    }

    char log_entry[128];
    int log_length = snprintf(log_entry, sizeof(log_entry), "%ld %ld %d\n", start_time, end_time, exit_code);
    if (write(log_fd, log_entry, log_length) == -1) {
        errx(1,"write");
    }

    close(log_fd);
}

int main(int argc, char *argv[]) {
    if (argc < 3 || argc) {
        errx(1, "Usage: %s <threshold> <command> [args...]\n", argv[0]);
    }

    int threshold = atoi(argv[1]);
    if (threshold < 1 || threshold > 9) {
        errx(1, "Threshold must be a single digit between 1 and 9.\n");
    }

    char **command = &argv[2];
    time_t last_start_time = 0, last_end_time = 0;
    int last_exit_code = 0;
    int consecutive_failures = 0;

    
    while (1) {
        time_t start_time = time(NULL);

        pid_t pid = fork();
        if (pid == -1) {
            errx(1, "fork");
        } else if (pid == 0) {
            // child process
            execvp(command, args); // execvp used when the number of arguments is not known at compile time
            errx(1, "execvp");
            /*
             if (argc == 3) {
                execlp(command, command, (char *)NULL);
            } else {
                execlp(command, command, argv[3], argv[4], argv[5], argv[6], argv[7], argv[8], argv[9], argv[10], (char *)NULL);
            }
            */
        } else {
            // parent process
            int status;
            wait(&status);
            time_t end_time = time(NULL);

            int exit_code = WIFEXITED(status) ? WEXITSTATUS(status) : 129;

            log_run(start_time, end_time, exit_code);

            if (exit_code != 0 && (end_time - start_time) < threshold) {
                if (last_exit_code != 0 && (last_end_time - last_start_time) < threshold) {
                    consecutive_failures++;
                } else {
                    consecutive_failures = 1;
                }
            } else {
                consecutive_failures = 0;
            }

            if (consecutive_failures >= 2) {
                break;
            }

            last_start_time = start_time;
            last_end_time = end_time;
            last_exit_code = exit_code;
        }
    }
    return 0;
}

```