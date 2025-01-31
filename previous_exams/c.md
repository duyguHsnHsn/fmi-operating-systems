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

 Напишете две програми на C (foo и bar), които си комуникират през наименована тръба.
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
                 if (write(output_fd, &byte, 1) != 1) {
                errx(1, "write to output file");
            }
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
        if (found) {
            for (int j = i + 1; j < num_commands; j++) {
                close(pipes[j][0]);
            }
            break;
            }
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

### Зад. 90 2022-SE-01
 Напишете програма на C, която приема два позиционни аргумента – имена на двоични
файлове. Примерно извикване: ./main data.bin comparator.bin


Файлът data.bin се състои от две секции – 8 байтов хедър и данни. Структурата на хедъра е:
- uint32_t, magic – магическа стойност 0x21796F4A, която дефинира, че файлът следва тази спецификация;
- uint32_t, count – описва броя на елементите в секцията с данни.

Секцията за данни се състои от елементи – uint64_t числа.


Файлът comparator.bin се състои от две секции – 16 байтов хедър и данни. Структурата на хедъра е:
- uint32_t, magic1 – магическа стойност 0xAFBC7A37;
- uint16_t, magic2 – магическа стойност 0x1C27;
- комбинацията от горните две magic числа дефинира, че файлът следва тази спецификация;
- uint16_t, reserved – не се използва;
- uint64_t, count – описва броя на елементите в секциата с данни.

Секцията за данни се състои от елементи – наредени 6-торки:
- uint16_t, type – възможни стойности: 0 или 1;
- 3 бр. uint16_t, reserved – възможни стойности за всяко: 0;
- uint32_t, offset1;
- uint32_t, offset2.

Двете числа offset дефинират отместване (спрямо ℕ0) в брой елементи за data.bin; type дефинира
операция за сравнение:
- 0 : “по-малко”;
- 1 : “по-голямо”.

Елементите в comparator.bin дефинират правила от вида:
- “елементът на offset1” трябва да е “по-малък” от “елементът на offset2”;
- “елементът на offset1” трябва да е “по-голям” от “елементът на offset2”.

Програмата трябва да интепретира по ред дефинираните правила в comparator.bin и ако правилото не е изпълнено,
 да разменя in-place елементите на съответните отмествания. Елементи, които са
равни, няма нужда да се разменят

```c
#include <stdio.h>
#include <stdlib.h>
#include <stdint.h>
#include <fcntl.h>
#include <unistd.h>
#include <sys/types.h>
#include <sys/stat.h>
#include <errno.h>

#define DATA_MAGIC 0x21796F4A
#define COMP_MAGIC1 0xAFBC7A37
#define COMP_MAGIC2 0x1C27

void read_exact(int fd, void *buf, size_t count) { // void pointer for the buffer -> accept any type of data
    ssize_t bytes_read = read(fd, buf, count);
    if (bytes_read == = -1 ){
        errx(2, "read")
    }
}

void write_exact(int fd, const void *buf, size_t count) {
    ssize_t bytes_written = write(fd, buf, count);
    if (bytes_written == -1 ){
        errx(2, "write")
    }
}

int main(int argc, char *argv[]) {
    if (argc != 3) {
        errx(1, "Usage: %s <data.bin> <comparator.bin>\n", argv[0]);
    }

    int data_fd =open(argv[1], O_RDWR);
	if(data_fd == -1) { err(1, "%s", argv[1]); }

    int comp_fd = open(argv[2], O_RDONLY);
    if (comp_fd == -1) {
        close(data_fd);
        err(1, "%s", argv[2]); 
    }

    // data.bin headers
    uint32_t data_magic;
    uint32_t data_count;
    read_exact(data_fd, &data_magic, sizeof(data_magic));
    read_exact(data_fd, &data_count, sizeof(data_count));
    if (data_magic != DATA_MAGIC) {
        close(data_fd);
        close(comp_fd);
        errx(2, "Invalid magic number in data.bin\n");
    }

    // comparator.bin header
    uint32_t comp_magic1;
    uint16_t comp_magic2;
    uint16_t comp_reserved;
    uint64_t comp_count;
    read_exact(comp_fd, &comp_magic1, sizeof(comp_magic1));
    read_exact(comp_fd, &comp_magic2, sizeof(comp_magic2));
    read_exact(comp_fd, &comp_reserved, sizeof(comp_reserved));
    read_exact(comp_fd, &comp_count, sizeof(comp_count));
    if (comp_magic1 != COMP_MAGIC1 || comp_magic2 != COMP_MAGIC2) {
        close(data_fd);
        close(comp_fd);
        errx(2,  "Invalid magic number in comparator.bin\n");
    }

    // comparison rules
    for (uint64_t i = 0; i < comp_count; ++i) {
        uint16_t type;
        uint16_t reserved[3];
        uint32_t offset1, offset2;
        read_exact(comp_fd, &type, sizeof(type));
        read_exact(comp_fd, reserved, sizeof(reserved));
        read_exact(comp_fd, &offset1, sizeof(offset1));
        read_exact(comp_fd, &offset2, sizeof(offset2));

        if (offset1 >= data_count || offset2 >= data_count) {
             close(data_fd);
            close(comp_fd);
            errx(2, "Invalid offsets in comparator.bin: %u, %u\n", offset1, offset2);
        }

        // elements from data.bin
        uint64_t element1, element2;
        // considering the offsets relative to the start of the data section
        lseek(data_fd, 8 + offset1 * sizeof(uint64_t), SEEK_SET); // we add 8 because of the header which contains two 4 byte elements
        read_exact(data_fd, &element1, sizeof(element1));
        lseek(data_fd, 8 + offset2 * sizeof(uint64_t), SEEK_SET);
        read_exact(data_fd, &element2, sizeof(element2));

        // comparison and possibly swap
        int should_swap = 0;
        if (type == 0 && element1 >= element2) {
            should_swap = 1;
        } else if (type == 1 && element1 <= element2) {
            should_swap = 1;
        }

        if (should_swap) {
            lseek(data_fd, 8 + offset1 * sizeof(uint64_t), SEEK_SET);
            write_exact(data_fd, &element2, sizeof(element2));
            lseek(data_fd, 8 + offset2 * sizeof(uint64_t), SEEK_SET);
            write_exact(data_fd, &element1, sizeof(element1));
        }
    }

    close(data_fd);
    close(comp_fd);
    return 0;
}
```

### Зад. 74 2017-SE-01
```text
Напишете програма на C, която приема три параметъра – имена на двоични файлове.
Примерно извикване:
$ ./main f1.bin f2.bin patch.bin
Файловете f1.bin и f2.bin се третират като двоични файлове, състоящи се от байтове (uint8_t). Файлът f1.bin e “оригиналният” файл, а f2.bin е негово
копие, което е било модифицирано по някакъв начин (извън обхвата на тази задача). Файлът patch.bin е двоичен файл, състоящ се от наредени
тройки от следните елементи (и техните типове):


- отместване (uint16_t) – спрямо началото на f1.bin/f2.bin
- оригинален байт (uint8_t) – на тази позиция в f1.bin
- нов байт (uint8_t) – на тази позиция в f2.bin

Вашата програма да създава файла patch.bin, на базата на съществуващите файлове f1.bin и f2.bin,
като описва вътре само разликите между двата файла. Ако дадено отместване съществува само в единия от файловете f1.bin/f2.bin,
програмата да прекратява изпълнението си по подходящ начин.

Примерен f1.bin:
00000000: f5c4 b159 cc80 e2ef c1c7 c99a 2fb0 0d8c ...Y......../...
00000010: 3c83 6fed 6b46 09d2 90df cf1e 9a3c 1f05 <.o.kF.......<..
00000020: 05f9 4c29 fd58 a5f1 cb7b c9d0 b234 2398 ..L).X...{...4#.
00000030: 35af 6be6 5a71 b23a 0e8d 08de def2 214c 5.k.Zq.:......!L

Примерен f2.bin:
00000000: f5c4 5959 cc80 e2ef c1c7 c99a 2fb0 0d8c ..YY......../...
00000010: 3c83 6fed 6b46 09d2 90df cf1e 9a3c 1f05 <.o.kF.......<..
00000020: 05f9 4c29 fd58 a5f1 cb7b c9d0 b234 2398 ..L).X...{...4#.
00000030: afaf 6be6 5a71 b23a 0e8d 08de def2 214c ..k.Zq.:......!L

Примерен patch.bin:
00000000: 0200 b159 3000 35af ...Y0.5.

```

```c
#include <fcntl.h>
#include <unistd.h>
#include <stdint.h>
#include <stdlib.h>
#include <stdio.h>
#include <sys/stat.h>
#include <sys/types.h>
#include <err.h>

void read_exact(int fd, void *buffer, size_t size) {
     ssize_t bytes_read = read(fd, buf, count);
    if (bytes_read == = -1 ){
        errx(2, "read")
    }
}

void write_exact(int fd, const void *buffer, size_t size) {
       ssize_t bytes_written = write(fd, buf, count);
    if (bytes_written == -1 ){
        errx(2, "write")
    }
}

int main(int argc, char *argv[]) {
    if (argc != 4) {
        errx(1, "Usage: %s f1.bin f2.bin patch.bin", argv[0]);
    }

    int fd1 = open(argv[1], O_RDONLY);
    if (fd1 == -1) {
        errx(1, "Error opening %s", argv[1]);
    }

    int fd2 = open(argv[2], O_RDONLY);
    if (fd2 == -1) {
        errx(1, "Error opening %s", argv[2]);
    }

    int patch_fd = open(argv[3],  O_WRONLY | O_TRUNC | O_CREAT, S_IRUSR | S_IWUSR);
    if (patch_fd == -1) {
        err(1, "Error opening %s", argv[3]);
    }

    struct stat st1, st2;
    if (fstat(fd1, &st1) < 0 || fstat(fd2, &st2) < 0) {
        errx(1, "Error getting file size");
    }

    if (st1.st_size != st2.st_size) {
        errx(1, "Files must be of the same size");
    }

    uint16_t offset = 0;
    uint8_t byte1, byte2;

    for (off_t i = 0; i < st1.st_size; i++) {
        read_exact(fd1, &byte1, sizeof(byte1));
        read_exact(fd2, &byte2, sizeof(byte2));

        if (byte1 != byte2) {
            write_exact(patch_fd, &offset, sizeof(offset));
            write_exact(patch_fd, &byte1, sizeof(byte1));
            write_exact(patch_fd, &byte2, sizeof(byte2));
        }

        offset++;
    }

    close(fd1);
    close(fd2);
    close(patch_fd);

    return 0;
}

```

#### Зад. 77 2017-SE-04 

Напишете програма на C, която да работи подобно на командата cat , реализирайки само
следната функционалност:
- програмата извежда на STDOUT
- ако няма подадени параметри, програмата чете от STDIN
- ако има подадени параметри – файлове, програмата последователно ги извежда
- ако някой от параметрите започва с тире (-), програмата да го третира като специално име за
STDIN


Примерно извикване:

```shell
$ ./main f - g
```

- извежда съдържанието на файла f, после STDIN, след това съдържанието на файла g.


```c
#include <stdio.h>
#include <stdlib.h>
#include <fcntl.h>
#include <unistd.h>
#include <string.h>
#include <err.h>

void read_and_write(int fd) {
    char buffer[4096];
    ssize_t bytes_read;
    while ((bytes_read = read(fd, buffer, sizeof(buffer))) > 0) {
        if (write(STDOUT_FILENO, buffer, bytes_read) != bytes_read) {
            errx(1, "write error");
        }
    }
    if (bytes_read < 0) {
        errx(1, "read error");
    }
}

int main(int argc, char *argv[]) {
    if (argc == 1) {
        // read from stdin
        read_and_write(0);
    } else {
        for (int i = 1; i < argc; i++) {
            if (strcmp(argv[i], "-") == 0) {
                //stdin
                read_and_write(0);
            } else {
                int fd = open(argv[i], O_RDONLY);
                if (fd == -1) {
                    errx(2, "Cannot open file %s", argv[i]);
                }
                read_and_write(fd);
                close(fd);
            }
        }
    }
    return 0;
}




```


### Зад. 79 2018-SE-02 
Напишете програма на C, която приема два параметъра – имена на файлове:
- примерно извикване: ./main input.bin output.bin
- файловете input.bin и output.bin се третират като двоични файлове, състоящи се от uint32_t
числа
- файлът input.bin може да съдържа максимум 4194304 числа
- файлът output.bin трябва да бъде създаден от програмата и да съдържа числата от input.bin,
сортирани във възходящ ред
- endianness-ът на машината, създала файла input.bin е същият, като на текущата машина
- ограничения на ресурси: програмата трябва да работи с употреба на максимум 9 MB RAM и 64
MB дисково пространство.

```c    
#include <stdio.h>
#include <stdlib.h>
#include <stdint.h>
#include <fcntl.h>
#include <unistd.h>
#include <err.h>

#define MAX_NUMBERS 4194304
#define MAX_RAM_USAGE 9437184  // 9 MB in bytes

int compare_uint32(const void *a, const void *b) {
    uint32_t arg1 = *(const uint32_t *)a;
    uint32_t arg2 = *(const uint32_t *)b;
    return (arg1 > arg2) - (arg1 < arg2);
}

int main(int argc, char *argv[]) {
    if (argc != 3) {
        errx(1, "Usage: %s input.bin output.bin\n", argv[0]);
    }

    int input_fd = open(argv[1], O_RDONLY);
    if (input_fd == -1) {
        errx(1, "Cannot open input file %s", argv[1]);
    }

    // size of the input file
    off_t input_size = lseek(input_fd, 0, SEEK_END);
    if (input_size == -1) {
        errx(2, "Error seeking input file %s", argv[1]);
    }
    lseek(input_fd, 0, SEEK_SET);

    size_t num_numbers = input_size / sizeof(uint32_t);
    if (num_numbers > MAX_NUMBERS) {
        errx(3 "Input file %s exceeds the maximum number of %d uint32_t numbers", argv[1], MAX_NUMBERS);
    }

    if (num_numbers * sizeof(uint32_t) > MAX_RAM_USAGE) {
        errx(4, "Memory required exceeds maximum allowed RAM usage");
    }

    uint32_t *numbers = malloc(num_numbers * sizeof(uint32_t));
    if (numbers == NULL) {
        errx(4, "Unable to allocate memory");
    }


    if (read(input_fd, numbers, input_size) != input_size) {
        errx(2, "Error reading input file %s", argv[1]);
    }
    close(input_fd);


    qsort(numbers, num_numbers, sizeof(uint32_t), compare_uint32);

    int output_fd = open(argv[2],  O_WRONLY | O_TRUNC | O_CREAT, S_IRUSR | S_IWUSR)
    if (output_fd == -1) {
        errx(2, "Cannot open output file %s", argv[2]);
    }

    if (write(output_fd, numbers, input_size) != input_size) {
        errx(2, "Error writing to output file %s", argv[2]);
    }

    close(output_fd);
    free(numbers);

    return 0;
}

```

### Зад. 80 2018-SE-03 
Напишете програма на C, която да работи подобно на командата cut, реализирайки само
следната функционалност:
- програмата трябва да чете текст от стандартния вход и да извежда избраната част от всеки ред
на стандартния изход;
- ако първият параметър на програмата е низът -c , тогава вторият параметър е или едноцифрено
число (от 1 до 9), или две едноцифрени числа N и M (𝑁 ≤ 𝑀), разделени с тире (напр. 3-5 ).
В този случай програмата трябва да изведе избраният/избраните символи от реда: или само
символа на указаната позиция, или няколко последователни символи на посочените позиции.
- ако първият параметър на програмата е низът -d, тогава вторият параметър е низ, от който е
важен само първият символ; той се използва като разделител между полета на реда. Третият
параметър трябва да бъде низът -f ,а четвъртият - или едноцифрено число (от 1 до 9), или две
едноцифрени числа N и M (𝑁 ≤ 𝑀), разделени с тире (напр. 3-5 ). В този случай програмата
трябва да разглежда реда като съставен от няколко полета (може би празни), разделени от указания символ (първият символ от низа, следващ парметъра -d ),
и да изведе или само указаното поле, или няколко последователни полета на указаните позиции, разделени от същия разделител.
- ако някой ред няма достатъчно символи/полета, за него програмата трябва да изведе каквото
(докъдето) е възможно (или дори празен ред)

```c

```

### Зад. 81 2018-SE-04 
Напишете програма на C, която приема два параметъра – имена на файлове:
- примерно извикване: ./main input.bin output.bin
- файловете input.bin и output.bin се третират като двоични файлове, състоящи се от uint16_t
числа
- файлът input.bin може да съдържа максимум 65535 числа
- файлът output.bin трябва да бъде създаден от програмата и да съдържа числата от input.bin,
сортирани във възходящ ред
- endianness-ът на машината, създала файла input.bin е същият, като на текущата машина
- ограничения на ресурси: програмата трябва да работи с употреба на максимум 256 KB RAM и 2
MB дисково пространство.

```c
#include <stdio.h>
#include <stdlib.h>
#include <stdint.h>
#include <fcntl.h>
#include <unistd.h>
#include <err.h>

#define MAX_NUMBERS 65535
#define MAX_RAM_USAGE (256 * 1024)  // 256 KB in bytes

int compare_uint16(const void *a, const void *b) {
    uint16_t arg1 = *(const uint16_t *)a;
    uint16_t arg2 = *(const uint16_t *)b;
    return (arg1 > arg2) - (arg1 < arg2);
}

int main(int argc, char *argv[]) {
    if (argc != 3) {
        errx(1, "Usage: %s input.bin output.bin\n", argv[0]);
    }

    int input_fd = open(argv[1], O_RDONLY);
    if (input_fd == -1) {
        err(1, "Cannot open input file %s", argv[1]);
    }

    // size of the input file
    off_t input_size = lseek(input_fd, 0, SEEK_END);
    if (input_size == -1) {
        err(2, "Error seeking input file %s", argv[1]);
    }
    lseek(input_fd, 0, SEEK_SET);

    size_t num_numbers = input_size / sizeof(uint16_t);
    if (num_numbers > MAX_NUMBERS) {
        errx(3, "Input file %s exceeds the maximum number of %d uint16_t numbers", argv[1], MAX_NUMBERS);
    }

    if (num_numbers * sizeof(uint16_t) > MAX_RAM_USAGE) {
        errx(4, "Memory required exceeds maximum allowed RAM usage");
    }

    uint16_t *numbers = malloc(num_numbers * sizeof(uint16_t));
    if (numbers == NULL) {
        err(4, "Unable to allocate memory");
    }

    if (read(input_fd, numbers, input_size) != input_size) {
        err(2, "Error reading input file %s", argv[1]);
    }
    close(input_fd);

    qsort(numbers, num_numbers, sizeof(uint16_t), compare_uint16);

    int output_fd = open(argv[2], O_WRONLY | O_TRUNC | O_CREAT, S_IRUSR | S_IWUSR);
    if (output_fd == -1) {
        err(2, "Cannot open output file %s", argv[2]);
    }

    if (write(output_fd, numbers, input_size) != input_size) {
        err(2, "Error writing to output file %s", argv[2]);
    }

    close(output_fd);
    free(numbers);

    return 0;
}

```

### Зад. 112 2024-SE-01

Напишете програма fuzzer, търсеща входове, при които друга дадена програма крашва. Fuzzer-ът приема следните 3 аргумента:
- име (път) на друга програма /тестваната програма/
- цяло число 𝑁 ∈ [0, 2^8) - брой тестове
- име на резултатен файл
Fuzzer-ът трябва да стартира програмата многократно: до завършване на 𝑁 на брой изпълнения или
до достигане на краш. Приемаме, че програмата е крашнала, ако вместо да завърши нормално, нейният процес e убит от сигнал.
При всяко изпълнение на програмата, fuzzer-ът избира случайно число 𝑆∈[0, 216) и подава на stdin
на програмата 𝑆 на брой байта, взети от файла /dev/urandom. Този виртуален файл съдържа безкраен
поток от случайни данни - използвайте ги и за генериране на самото 𝑆. Също така, fuzzer-ът трябва да
се погрижи всякакъв изход от тестваната програма да не стига до терминала - той не ни интересува.
След завършване на 𝑁 на брой итерации без краш, резултатният файл трябва да е празен, а статусът
на завършване на fuzzer-a - успешен. При намерен краш, резултатният файл трябва да съдържа входа,
подаден на програмата при съответното изпълнение, а статусът на завършване на fuzzer-а да е 42.

```c
#include <stdio.h>
#include <stdlib.h>
#include <stdint.h>
#include <unistd.h>
#include <fcntl.h>
#include <sys/wait.h>
#include <signal.h>

// does not test multiple inputs in parallel can be done better with using fork one more time in main? 

int run_test(const char *program, const uint8_t *input, size_t size) {
    int pipefd[2];
    if (pipe(pipefd) == -1) {
        errx(2, "cannot pipe")
    }

    pid_t pid = fork();
    if (pid == -1) {
        errx(2, "cannot fork")
    }

    if (pid == 0) { 
        close(pipefd[1]);
        dup2(pipefd[0], STDIN_FILENO);
        close(pipefd[0]);

        int dev_null = open("/dev/null", O_WRONLY);
        if (dev_null == -1) {
           errx(2, "cannot open /dev/null")
        }

        // output of the program go to dev/null
        dup2(dev_null, STDOUT_FILENO);
        dup2(dev_null, STDERR_FILENO);
        close(dev_null);

        execl(program, program, NULL);  // since we write the input from the parent to the childs stdin this execlp will use that stdin
        errx(2,"cannot execl program");
    } else {
        close(pipefd[0]);
        write(pipefd[1], input, size);
        close(pipefd[1]);

        int status;
        waitpid(pid, &status, 0); 
        // did the program crash
        if (WIFSIGNALED(status)) { 
            return WTERMSIG(status); 
        }
    }
    return 0;
}

int main(int argc, char *argv[]) {
    if (argc != 4) {
        errx(1, "Usage: ./fuzzer <program> <N> <output_file>\n", 45);
    }

    int N = atoi(argv[2]);

    for (int i = 0; i < N; i++) {
        int urandom_fd = open("/dev/urandom", O_RDONLY);
        if (urandom_fd == -1) {
             errx(2, "cannot open /dev/urandom")
        }

        uint16_t S;
        if (read(urandom_fd, &S, sizeof(S)) != sizeof(S)) {
            close(urandom_fd);
            errx(2, "cannot read /dev/urandom")
        }

        S = S % 65536;  // S does not go higher than 2^16

        uint8_t *input = (uint8_t *)malloc(S);
        if (input == NULL) {
            close(urandom_fd);
            errx(2, "cannot malloc")
        }

        if (read(urandom_fd, input, S) != S) {
            free(input);
            close(urandom_fd);
            errx(3, "cannot read /dev/urandom")
        }
        close(urandom_fd);

        int result = run_test(argv[1], input, S);

        if (result) { 
            int output_fd = open(output_file, O_WRONLY | O_CREAT | O_TRUNC,  S_IRUSR | S_IWUSR);
            if (output_fd == -1) {
                free(input);
                errx(2, "cannot open output file")
            }
            if (write(output_fd, input, S) != S) {
                close(output_fd);
                free(input);
                errx(2, "cannot write to output file")
            }
            close(output_fd);
            free(input);
            return 42;
        }

        free(input); 
    }

    // no err; empty file
    int output_fd = open(output_file, O_WRONLY | O_CREAT | O_TRUNC,  S_IRUSR | S_IWUSR);
    if (output_fd == -1) {
        errx(2, "cannot open output file")
    }
    close(output_fd);

    return 0;
}

```

### Зад. 110 2023-SE-01

Напишете програма на C, която приема като аргумент име на директория и генерира хешове на файловете в нея, работейки паралелно.
При изпълнение върху директория my-dir, програмата:
- За всеки файл в директорията и нейните поддиректории (напр. my-dir/foo/my-file), трябва
да създаде файл my - dir/foo/my-file.hash, съдържащ хеш-сумата на my-file
- Може да извиква външните команди find (за откриване на файлове) и md5sum (за пресмятане
на хеш-сума)
- Не трябва да обработва файлове, чиито имена вече завършват на .hash
- Трябва да позволява md5sum-процесите да работят паралелно един спрямо друг

```c
#include <stdio.h>
#include <stdlib.h>
#include <stdint.h>
#include <unistd.h>
#include <fcntl.h>
#include <sys/wait.h>
#include <signal.h>


void mdsum_file(char* filename) {
    int mdsum[2];

    if(pipe(mdsum) == -1){
        errx(2, "cannot pipe for mdsum");
    }

    pid_t mdsum_pid = fork();

    if(mdsum_pid == -1 ){
        errx(2, "cannot fork for mdsum");
    }

    if(mdsum_pid == 0) {
        close(mdsum[0]);

        dup2(mdsum[1], 1);
        close(mdsum[1]);

        execlp("md5sum", "md5sum", filename, (char*)NULL);
        errx(2,"cannot execlp md5sum")
    }

    close(mdsum[1]);

    char hash_filename[4096];
	strcpy(hash_filename, filename);
	strncat(hash_filename, ".hash", 4096 - strlen(hash_filename) - 1);

	int hash_fd = open(hash_filename, O_WRONLY | O_TRUNC | O_CREAT, S_IRUSR | S_IWUSR);
	if(hash_fd == -1) {
         err(1, "cannot open %s", hash_filename); 
    }

    ssize_t read_bytes;
    char byte;
    while((read_bytes=read(mdsum[0], &byte, sizeof(byte)) > 0)) {
        if (byte == ' ') {
            break;
        }
        if(write(hash_fd, &byte, sizeof(byte)) == -1){
            errx(2,"cannot write to hash file");
        }
    }
    close(mdsum[0]);
    close(hash_fd);

    int status;
    wait(&status);
}

int main(int argc, char *argv[]) {
    if (argc != 2) {
        errx(1, "Usage: ./main <dir>\n", 1);
    }

    int find[2];
    if(pipe(find) == 0 ){
        errx(1,"cannot pipe");
    }

    pid_t find_pid = fork();
    if(find_pid == -1) {
        errx(2,"cannot fork");
    }

    if(find_pid == 0) {
        close(find[0]);

        dup2(find[1], 1);
        close(find[1]);

        execlp("find", "find", argv[1], "-type", "f", "-not", "-name", "*.hash", "-print0", (char*)NULL);
        errx(2,"cannot execlp find");
    }
    close(find[1]);

    ssize_t read_size;
    char byte;
    char buffer[4096];
    int char_index_in_line;
    int worker_count = 0;

    while((read_size=read(find[0], &byte, sizeof(byte)) > 0)){
        char_index_in_line++;
        if (char_index_in_line > 4095) {
            errx(3,"filename too big to read");
        }

        buffer[char_index_in_line] = byte;

        if(byte == "\0") {
            // we will move to next line
            char_index_in_line = 0;

            pid_t worker = fork();
            if (worker == -1) {
                errx(2, "cannot fork worker");   
            }

            if(worker == 0) {
                // close the pipe for every child process
                close(find[0]);
                mdsum_file(buffer);
                exit(0);
            }
            worker_count++l
        }
    }
    // close the pipe for parent
    close(find[0]);

    if(read_size == -1) {
        errx(2, "couldn't read from pipe");
    }

    while(worker_count) {
        int status;
		wait(&status);

		if(WIFEXITED(status)) {
			if(WEXITSTATUS(status) != 0) {
				warnx("child exited with status %d", WEXITSTATUS(status));
			}
		}else{
			warnx("child was killed");
		}

        worker_count--;
    }

}

```

### Зад. 97 2024-SE-01
Имате файл, съдържащ записи, които са организирани в свързан списък. Някои от записите във файла вече не се използват (били са премахнати от списъка и вече не са част от него). За съжаление, по
GDPR вие нямате право да пазите дълго време потребителски данни, които не се използват – затова
трябва да напишете програма, която цензурира неизползваните записи.
Програмата се извиква с един аргумент (име на файл): ./main tasks.db


Файлът се състои от последователни 512-байтови записи, всеки от които описва възел и има следния
вид:
- next: 8 байта, представящи 64-битово число без знак – индекс на следващия възел (в брой записи
спрямо началото на файла), или 0 , ако след този възел няма следващ
- user_data: 504 байта потребителски данни

  
Първият възел от списъка е винаги първи запис във файла (с индекс 0). Всеки следващ възел може да
е на произволно място във файла.
Програмата трябва да промени така файла, че всеки запис, който не е част от списъка (не е достижим,
обхождайки целия списък), да се замени с 512 байта нули.


Забележки:
- Забележете, че последният възел от списъка (този, който има next==0) най-вероятно няма да
е последният запис във файла.
-  Може да приемете, че възлите, които са в списъка, няма да образуват цикъл.
- Максималният брой записи е 2^40 - може да няма начин да съберем в паметта информация за
всички записи. Ако искате да създадете временен файл, вижте mkstemp(3).

```c
#include <stdio.h>
#include <stdlib.h>
#include <unistd.h>
#include <fcntl.h>
#include <stdint.h>
#include <string.h>

#define RECORD_SIZE 512
#define NEXT_OFFSET 0
#define USER_DATA_OFFSET 8
#define USER_DATA_SIZE (RECORD_SIZE - USER_DATA_OFFSET)

uint64_t read_next(int fd, off_t offset) {
    uint64_t next;
    lseek(fd, offset + NEXT_OFFSET, SEEK_SET);
    if (read(fd, &next, sizeof(next)) != sizeof(next)) {
        errx(1, "Error reading next");
    }
    return next;
}

void mark_used_records(int fd, char* used_records, uint64_t index) {
    while (index != 0) {
        if (used_records[index]) {
            break; // if it is used
        }
        used_records[index] = 1;
        off_t offset = index * RECORD_SIZE;
        index = read_next(fd, offset);
    }
}

void zero_out_record(int fd, off_t offset) {
   char zero_buf[RECORD_SIZE];
    for (int i = 0; i < RECORD_SIZE; i++) {
        zero_buf[i] = 0;
    }

    lseek(fd, offset, SEEK_SET);
    if (write(fd, zero_buf, RECORD_SIZE) != RECORD_SIZE) {
        errx(1, "Error writing zeroes");
    }
}

int main(int argc, char *argv[]) {
    if (argc != 2) {
        errx(1, "Usage: %s <file>\n", argv[0]);
    }

    int fd = open(argv[1], O_RDWR);
    if (fd == -1) {
       errx(1, "cannot open file");
    }

    // find size
    off_t file_size = lseek(fd, 0, SEEK_END);
    if (file_size == -1) {
        errx(2, "cannot seek end")
    }

    if (lseek(fd, 0, SEEK_SET) == -1) {
       errx(2, "cannot seek start")
    }

    uint64_t num_records = file_size / RECORD_SIZE;

    char *used_records = calloc(num_records, sizeof(char));
    if (used_records == NULL) {
        errx(1, "cannot calloc");
    }

    mark_used_records(fd, used_records, 0);

    for (uint64_t i = 0; i < num_records; i++) {
        if (!used_records[i]) {
            zero_out_record(fd, i * RECORD_SIZE);
        }
    }

    free(used_records);
    close(fd);

    return 0;
}

```

### Зад. 93 2023-IN-01 
Вашите колеги от съседната лаборатория са попаднали на вогонски* крипто-вирус и вашата задача е да напишете програма, която възстановява файлове, криптирани от вируса. Примерно
извикване: . /main input.encrypted output.decrypted

Вирусът не бил особенно добре проектиран и след анализ се оказало, че подменя оригиналните файлове с криптирани със следните особенности:
- вогоните явно предпочитат да работят с данни от по 128 бита, като всеки такъв елемент ще наричаме unit (U). Както входния (оригинален) файл, така и изходния (криптиран) файл се третират
като състоящи се от unit-и. Криптираният файл винаги е точен брой unit-и (номерирани спрямо 𝑁0
), като при нужда към оригиналният файл преди криптиране се добавят байтове 0x00
(padding), за да стане точен брой unit-и.
- unit-ите в криптирания файл могат да се ползват за съхранение на следните типове данни (и
техните големини): header (4U), section (2U), cryptdata (1U), unused data (1U).
- типовете данни section и cryptdata са обработени с побитово изключващо или (XOR) със 128 битов
ключ (ключът е голям 1U), като за section ключът се ползва два пъти – за първият и за вторият
unit по отделно.
- криптираният файл винаги започва с header (4U) и има следната структура:
  
		– uint64_t, magic – магическа стойност 0x0000534f44614c47, която дефинира, че файлът е криптиран спрямо тази спецификация
		– uint32_t, cfsb – размер на криптирания файл в байтове
		– uint32_t, cfsu – размер на криптирания файл в брой unit-и
		– uint32_t, ofsb – размер на оригиналния файл в байтове
		– uint32_t, ofsu – размер на оригиналния файл в брой unit-и
		– uint32_t, unused1 – не се използва в тази версия на вируса
		– uint32_t, cksum – checksum-а на оригиналния файл, получена по следния алгоритъм:

				- оригиналният файл се третира като състоящ се от uint32_t елементи
				- при нужда от padding на последния елемент се ползват байтове 0x00
				- всички елементи се XOR-ват един с друг и полученият резултат е cksum
  
		– 128 бита, sectionkey – ключ, с който са кодирани section-ите
		– четири “слота”, описващи начална позиция в U на section

				- uint32_t, s0
				- uint32_t, s1
				- uint32_t, s2
				- uint32_t, s3
				- тъй като unit-и с номера в интервала [0, 3] се ползват за самия header, в горните слотове 0 означава, че този слот не се използва (няма такъв section)
				- unit-ите в изходния (декриптиран) файл са по реда на секциите
  
- section(2U) е кодиран със sectionkey от header-а и след разкодиране има следната структура:
  
		– uint64_t, relative_offset – дефинира от кое U започват unit-ите с данни за тази секция. Числото е относително спрямо първият unit на section, например, ако section е записан в U4/5 и стойността на offset е 2, то първите данни на секцията ще са в U 6.
		– uint64_t, len – дефинира брой unit-и в този section
		– 128 бита, datakey– ключ, с който са кодирани unit-ите с данни в този section
		– unit-ите в изходния (декриптиран) файл са по реда на unit-ите в секцията

- cryptdata (1U) – unit в който има записани данни, кодирани с datakey на секцията, към която е
този unit
- unuse data(1U) – unit, който не се ползва. Колегите ви предполагат, че това е един от начините
чрез които вогоните са вкарвали “шум” в криптирания файл, но без да правят декриптирането
невъзможно.

```c
#include <stdio.h>
#include <stdlib.h>
#include <stdint.h>
#include <fcntl.h>
#include <unistd.h>
#include <sys/types.h>
#include <sys/stat.h>
#include <errno.h>

#define DATA_MAGIC 0x0000534f44614c47


void read_exact(int fd, void *buf, size_t count) { // void pointer for the buffer -> accept any type of data
    ssize_t bytes_read = read(fd, buf, count);
    if (bytes_read == = -1 ){
        errx(2, "read")
    }
}

void write_exact(int fd, const void *buf, size_t count) {
    ssize_t bytes_written = write(fd, buf, count);
    if (bytes_written == -1 ){
        errx(2, "write")
    }
}

uint64_t* xor128(uint64_t a[2], uint64_t b[2]) {
    uint64_t result[2];
    result[0] = a[0] ^ b[0];
    result[1] = a[1] ^ b[1];
    return result;
}

uint64_t* extract_section(int fd, uint32_t offset) {
    if (offset == 0 || offset == 1 || offset == 2  || offset == 3 ) { // header
        return NULL;
    }
    // one unit holds 2x uint64_t
    uint64_t unit[2];
    lseek(fd, offset * sizeof(unit) , SEEK_SET); 
    // read 2 units 2x2xuint64 to get all of section
    uint64_t buffer[4];
    read_exact(fd, buffer, sizeof(buffer));
    return buffer;
}


uint64_t* decript_main_section(int fd, uint64_t section[4], uint64_t sectionkey[2], uint64_t absolute_offset) {
    uint64_t* first_decripted_unit = zor128({section[0], section[1]}, sectionkey);
    uint64_t* datakey = zor128({section[2], section[3]}, sectionkey);

    // get data section
    uint64_t relative_offset = first_decripted_unit[0];
    uint64_t data_units_len = first_decripted_unit[1] - 2; // all_section_units - 2(main includes 2)
    uint64_t *data = malloc(data_units_len * 2 * sizeof(uint32_t));
    uint64_t *decripted_data = malloc(data_units_len * 2 * sizeof(uint32_t));
    lseek(fd, (absolute_offset + relative_offset) * sizeof(unit) , SEEK_SET); 
    read_exact(fd, data, sizeof(data));
    for (int i = 0; i < data_units_len * 2;){
        uint64_t* decripted_data_pair = zor128({data[i], data[i+1]}, datakey);
        decripted_data[i] = decripted_data_pair[0];
        decripted_data[i+1] = decripted_data_pair[1];
        i = i + 2;
    }
    free(data);
    return decripted_data;
}

uint64_t* extract_main_section_data(int fd, uint32_t offset, uint64_t sectionkey[2]) {
    uint64_t* buffer = extract_section(fd, offset);
    if (buffer == NULL) {
        return NULL;
    }
    return decript_main_section(fd, buffer, sectionkey, offset);
}

int main(int argc, char *argv[]) {
    if (argc != 3) {
        errx(1, "Usage: %s <input> <output>\n", argv[0]);
    }

    int data_fd =open(argv[1], O_RDONLY);
	if(data_fd == -1) { err(1, "%s", argv[1]); }

    // header
    uint64_t data_magic;
    uint32_t cfsb;
    uint32_t cfsu;
    uint32_t ofsb;
    uint32_t ofsu;
    uint32_t unused1;
    uint32_t cksum;

    uint64_t sectionkey[2];
    uint32_t s0, s1, s2, s3;

    read_exact(data_fd, &data_magic, sizeof(data_magic));
    if (data_magic != DATA_MAGIC) {
        close(data_fd);
        errx(2, "Invalid magic number");
    }

    read_exact(data_fd, &cfsb, sizeof(cfsb));
    // size of the input file
    off_t input_size = lseek(data_fd, 0, SEEK_END);
    if (input_size == -1) {
        err(2, "Error seeking input file %s", argv[1]);
    }
    lseek(data_fd, 0, SEEK_SET);

    if ((uint32_t)input_size != cfsb) {
        err(2, "Error incorect filesize %s", argv[1]);
    }


    read_exact(data_fd, &cfsu, sizeof(cfsu));
    read_exact(data_fd, &ofsb, sizeof(ofsb));
    read_exact(data_fd, &ofsu, sizeof(ofsu));
    read_exact(data_fd, &unused1, sizeof(unused1));
    read_exact(data_fd, &cksum, sizeof(cksum));

    read_exact(data_fd, &sectionkey, sizeof(sectionkey));
    read_exact(data_fd, &s0, sizeof(s0));
    read_exact(data_fd, &s1, sizeof(s1));
    read_exact(data_fd, &s2, sizeof(s2));
    read_exact(data_fd, &s3, sizeof(s3));

    s0_data = extract_main_section_data(data_fd, s0, sectionkey);
    s1_data = extract_main_section_data(data_fd, s1, sectionkey);
    s2_data = extract_main_section_data(data_fd, s2, sectionkey);
    s3_data =extract_main_section_data(data_fd, s3, sectionkey);
    
    int output_fd =  open(argv[2], O_WRONLY | O_TRUNC | O_CREAT, S_IRUSR | S_IWUSR);
    if (output_fd == -1 ) {
        errx(1, "cannot open output dir");
    }

    if(s0_data != NULL) {
        write_exact(output_fd, s0_data, sizeof(s0_data));
        free(s0_data);
    }
    if(s1_data != NULL) {
        write_exact(output_fd, s1_data, sizeof(s1_data));
        free(s1_data);
    }
    if(s2_data != NULL) {
        write_exact(output_fd, s2_data, sizeof(s2_data));
        free(s2_data);
    }
    if(s3_data != NULL) {
        write_exact(output_fd, s3_data, sizeof(s3_data));
        free(s3_data);
    }

    // before end fix sizing of the output dir

    return 0;
}

```

### Зад. 107 2021-SE-01

Ваши колеги - асистенти по ОС се оплакват, че студентите упорито ползват командата
sudo , въпреки че им е обяснено да не я ползват. Вашата задача е да подготвите фалшиво sudo, което:
- добавя ред в текстови файл foo.log с командата, която потребителят се е опитал да изпълни
във формат ДАТА ПОТРЕБИТЕЛ ДАННИ , където:

                - ДАТА е във формат YYYY-MM-DD HH:MM:SS.UUUUUU (UUUUUU са микросекунди)
                - ПОТРЕБИТЕЛ е login name (username) на потребителя
                - ДАННИ е всичко, което потребителят е написал, например . /main ls -l /root
                - горните полета са разделени с интервал
                - пример: 2021-05-10 14:38:17.584344 s99999 ./main ls -l /root
- заключва акаунта, който е изпълнил програмата
- приключва изпълнението на всички процеси, стартирани от този потребител

За да може да изпълни тези неща, фалшивото sudo ще има нужда от root права, а в същото време
трябва да знае кой акаунт изпълнява командата. Традиционно това може да се реши със SUID, което
обаче не важи за shell скриптове. За да заобиколите този проблем, напишете програма на C(main),
която за заключването на акаунта и терминирането на процесите извиква външни shell команди през
pipe() .


Полезни man страници:
- gettimeofay(2)
- localtime(3)
- strftime(3)
- getuid(2)
- getpwuid(3)
- passwd(1)

Опишете в doc.txt как бихте настроили компилираната вече програма така, че студентите да ползват
нея, вместо оригиналното sudo.
Естествено, по време на разработка на програмата можете да я тествате само с вашият акаунт и нямате
root права. За да може все пак да тествате, подменете в кода финалните команди, които програмата
изпълнява (тези, които наистина изискват root права), като им добавите едно echo . Например, ако
трябва да изпълните командата

foo -m pesho

вместо това в кода ви трябва да е дефинирано изпълнение на

echo foo -m pesho

```c

```

### Зад. 101 2017-IN-02 

Напишете програма на C, която приема незадължителен параметър – име на команда.
Ако не е зададена команда като параметър, да се ползва командата echo. Максималната допустима
дължина на командата е 4 знака.
Програмата чете низове (с максимална дължина 4 знака) от стандартния си вход, разделени с интервали (0x20) или знак за нов ред (0x0A). Ако някой низ е с
дължина по-голяма от 4 знака, то програмата да терминира със съобщение за грешка.


Подадените на стандартния вход низове програмата трябва да третира като множество от параметри
за дефинираната команда. Програмата ви трябва да изпълни командата колкото пъти е необходимо с
максимум два низа като параметри, като изчаква изпълнението да приключи, преди да започне ново
изпълнение.
![example](img6.png)

 ```c
#include <stdio.h>
#include <stdlib.h>
#include <stdint.h>
#include <fcntl.h>
#include <unistd.h>
#include <sys/types.h>
#include <sys/stat.h>
#include <errno.h>

void execute_command(const char *command){
    char first_input[4] = {0};
    char second_input[4] = {0};
    char byte;
    int inx = 0;
    int second_inx = 0;
    int try_two_args = 0;
    int ready_1 = 0;
    int ready_2 = 0;

    while((read(0,&byte, sizeof(byte)))){
        if(inx == 0 || first_input[inx -1] != "\0"){
            if (inx + 1 >= 4 ){
                errx(3, "too long string for input");
            }
            if(byte == " " || byte == "\n") {
                first_input[inx]="\0";
                ready1 = 1;
            } else {
                first_input[inx] = byte;
            }
            inx++;
        } else if( byte != " " || byte == !"\n") {
            second_input[second_inx] = byte;
            second_inx++;
            if (second_inx >= 4 ){
                errx(3, "too long string for input");
            }
        } else if (byte == " " || byte == "\n") {
            second_input[second_inx]="\0"; // if no input 2 then second_input will be just "\0"
            ready2 = 1;
        }

        if(ready1 == 1 && ready2 == 1){
            inx = 0;
            second_inx = 0;
            int executed[2];

            if(pipe(executed) == -1){
                errx(2, "cannot pipe");
            }

            pid_t executer_pid = fork();
            if(executer_pid == -1){
                errx(2,"cannot fork");
            }

            if (executer_pid == 0 ){
                close(executed[0]);

                dup2(executed[1], 1);
                close(executed[1]);
    
                execlp(command, command, first_input, second_input,  (char *)NULL);
                errx(2, "cannot execlp command");
            }

            close(executed[1]);
            
            char to_print;
            while((read(executed[0], &to_print, sizeof(to_print))) > 0 ){
                if(write(1, &to_print, sizeof(to_print)) < 1) {
                    errx(2,"cannot write to stdout");
                }
            }
            close(executed[0]);
            wait(NULL); 
        }
    }
}


int main(int argc, char *argv[]) {
    if (argc > 2) {
        errx(1, "wrong usage");
    }
    if(argc == 1) {
        execute_command("echo")
    }
    else {
        if(strlen(argv[1]) > 4 ){
            errx(1, "command is too long")
        }
        execute_command(argv[1];)
    }
    return 0;
}

 ```

 ### Зад. 89 2021-SE-02 

Вашите колеги от съседната лаборатория са написали програма на C, която може да обработва подаден входен двоичен файл и на негова база генерира изходен
двоичен файл. Програмата работи като encoder, който имплементира вариант на Manchester code, т.е.:
- за всеки входен бит 1 извежда битовете 1 0 , и
- за всеки входен бит 0 извежда битовете 0 1

Например следните 8 бита вход:

1 0 1 1 0 1 1 0 == 0xB6

по описания алгоритъм дават следните 16 бита изход

1 0 0 1 1 0 1 0 0 1 1 0 1 0 0 1 == 0x9A69

Напишете програма на C, която извършва обратния процес, т.е., декодира файлове, създадени от горната програма.

Примерно извикване:

// $ ./main input.bin output.bin

```c
#include <stdio.h>
#include <stdlib.h>
#include <stdint.h>
#include <fcntl.h>
#include <unistd.h>
#include <sys/types.h>
#include <sys/stat.h>
#include <errno.h>

void read_bits(uint16_t num, int bits[16]);

void read_bits(uint16_t num, int bits[16]) {
    for(int i = 15; i > 0; i--){
        bit[15 - i] = (number >> i) & 1;
    }
}

int main(int argc, char *argv[]) {
    if (argc != 3 ) {
        errx(1, "wrong usage");
    }

    fd1 = open(argv[1], O_RDONLY);
    if(fd1 == -1 ) {
        errx(1, "cannot read");
    }

    int fd2 = open(argv[2], O_WRONLY | O_TRUNC | O_CREAT, S_IRUSR | S_IWUSR);
    if(fd2 == -1) {
		err(1, "%s", argv[2]);
	}

    uint16_t two_bytes; // the byte size should be doubled from the previous operation; this wouldn't break
    uint8_t result_byte;
    int bits[16];
    int bit_count = 0;
    while((read(fd1,&two_bytes,sizeof(two_bytes))) > 0 ) {
        iread_bits(two_bytes, bits);
        for(int i = 15; i > 0;) {
            uint8_t result_bit;
            if(bits[i] == 0 && bits[i-1] == 1) {
                result_bit = 1;
            } else if(bits[i] == 1 && bits[i-1] == 0) {
                 result_bit = 0;
            } else {
                errx(3, "the file bits do not match the format");
            }

            // shift the result bit into correct position in the result byte
            result_byte = (result_byte << 1) | result_bit;
            bit_count++;

            if(bit_count == 8 ) {
                if((write(fd2, result_byte, sizeof(result_byte))) < 0 ){
                    errx(2, "cannot open output file to write result");
                }
                bit_count = 0;
                result_byte = 0;
            }
            i = i - 2;
        }
    }

    close(fd1);
    close(fd2);

    return 0;
}

```

### Зад. 98 2024-IN-01

Вашите колеги от съседната лаборатория използват специализирана система за съхранение на данни (СССД), която работи основно с два вида обекти – `volume`-и (текущо използвана версия на блоково устройство) и `snapshot`-и (стара версия на даден обект). Всеки обект има уникален идентификатор `id ∈ ℕ₀`, като стойност 0 е запазена. Текущото състояние на СССД е описано в двоичен файл, който се състои от три секции в следния ред:

#### Секция Header
Състои се от четири `uint16_t` променливи:
- **magic** – всички служебни файлове на СССД имат магическа стойност `0x6963`;
- **ver** – версия и формат на файла, в тази задача разглеждаме файлове с формат `0x6e73`;
- **cp** – брой на елементите в `preamble` секцията;
- **co** – брой на елементите в `objects` секцията;

#### Секция Preamble
Съдържа елементи със следната структура:
- **v1**, `uint16_t`;
- **v2**, `uint16_t`;
- **v3**, `uint32_t`;
- `preamble` елементите не се използват пряко в задачата;

#### Секция Objects
Съдържа елементи със следната структура, които описват обекти от СССД:
- **ctime**, `uint32_t` – време на създаване на обекта (Unix/Epoch time);
- **opt**, `uint16_t` – съдържа флагове, описващи различни характеристики на обекта; двата най-старши бита описват вида на обекта, като `00` се използват за `volume`, а `10` – за `snapshot`;
- **parent_id**, `uint16_t` – идентификатор на предхождащ обект (родител, предишната версия на обекта), като ако обектът няма родител, стойността на `parent_id` е 0;
- **size**, `uint32_t` – размер на обекта в байтове;
- **seize**, `uint32_t` – действително използвани байтове за съхранение на обекта, `seize ∈ [0, size]`.

#### Забележки:
- Съотношението на `seize` към `size` на даден обект ще наричаме **коефициент на запълване**;
- Отместването на елемент от `objects` секцията спрямо началото й (в брой елементи) дефинира `id` на обекта, описан от елемента.

Съществува система за управление на СССД, която спрямо дефинирани критерии автоматично създава дневни `snapshot`-и на някои `volume`-и. Дефиницията за **дневен snapshot** е такъв `snapshot`, който е на време-разстояние един ден плюс-минус 10 минути от родителя си. Например, един `volume` и три негови `snapshot`-а изглеждат по следния начин:

#### Задача
Напишете програма, която приема параметър – име на файл в описания формат. Програмата трябва да пресмята и извежда на `STDOUT` колко е претеглената по размер на обекта средна стойност на коефициента на запълване за дневните `snapshot`-и. Колегите ви искат програмата да минимизира броя на операциите с плаваща запетая (floating-point operations), за да не се натрупва грешка.

```c
#include <stdio.h>
#include <stdlib.h>
#include <stdint.h>
#include <fcntl.h>
#include <unistd.h>
#include <sys/types.h>
#include <sys/stat.h>
#include <errno.h>

#define EXPECTED_MAGIC 0x6963
#define EXPECTED_VER 0x6963

void read_exact(int fd, void *buff, size_t count);
int check_msb_bits(uint16_t value, uint8_t num_bits, uint16_t expected);
void write_exact(int fd, void *buff, size_t count);


void read_exact(int fd, void *buff, size_t count){
    ssize_t bytes_read = read(fd, buff, count);
    if (bytes_read == -1 ){
        errx(1, "cannot read file");
    }
}

void write_exact(int fd,const void *buff, size_t count){
    ssize_t bytes_written = write(fd, buff, count);
    if (bytes_read == -1 ){
        errx(1, "cannot write file");
    }
}

int check_msb_bits(uint16_t value, uint8_t num_bits, uint16_t expected){
    uint16_t mask = ((1U << num_bits) - 1) << (16 - num_bits); // mask for last num_bits
    uint16_t msb_bits = (value & mask) >> (16 - num_bits); // isolate and move MSB

    return msb_bits == expected;
}

int main(int argc, char *argv[]) {
	if (argc != 2 ) {
        errx(1, "wrong usage");
	}

    fd = open(argv[1], O_RDONLY)
    if (fd == -1){
       errx(1, "cannot  open provided file") 
    }

    // header values
    uint16_t magic;
    uint16_t ver;
    uint16_t cp;
    uint16_t co;

    read_exact(fd, &magic, sizeof(magic));
    if (magic != EXPECTED_MAGIC) {
        errx(2, "file starts with unexpected magic");
    }
    read_exact(fd, &ver, sizeof(ver));
    if (ver != EXPECTED_VER) {
        errx(2, "file has unexpected version");
    }
    read_exact(fd, &cp, sizeof(cp));
    if ( cp != 3 ) {
        errx(2, "unexpected cp value");
    }
    read_exact(fd, &co, sizeof(co));
    if (co != 5 ) {
        errx(2, "unexpected co value");
    }

    // skip preamble values
    for (uint16_t i = 0; i < cp; i++){
            // preamble values
            //uint16_t v1;
            //uint16_t v2;
            //uint32_t v3;

            //read_exact(fd, &v1, sizeof(v1));
            //read_exact(fd, &v2, sizeof(v2));
            //read_exact(fd, &v3, sizeof(v3));
        lseek(fd, 2 * sizeof(uint32_t), SEEK_CUR);
    }

    uint64_t overall_size;
    uint64_t overall_ssize;
    uint16_t parents_ids[100]; // or use malloc
    uint32_t parents_time[100];

    for (uint16_t j = 0; j < co; j++){
        // objects values
        uint32_t ctime;
        uint16_t opt;
        uint16_t parent_id;
        uint32_t size;
        uint32_t ssize;
        int snapshot = 0;

        read_exact(fd, &ctime, sizeof(ctime));
        parents_time[j] = ctime;
        read_exact(fd, &opt, sizeof(opt));
        if (check_msb_bits(opt, 2, 0b10) == 1) {
            snapshot = 1;
        }
        read_exact(fd, &parent_id, sizeof(parent_id));
        parents_ids[j] = parent_id;
        read_exact(fd, &size, sizeof(size));
        read_exact(fd, &ssize, sizeof(ssize));
        if (ssize > size){
            errx(3, "ssize cannot be larger than size");
        }

        if (snapshot == 1 && parent_id < co) {
            time_t parent_ctime;
            for (int i = 0; i < 100; i++){
                if (parents_ids[i] == parent_id){
                    parent_ctime = parents_ctime[i];
                    break;
                }
            }
            double time_diff = difftime(snapshot_time, parent_time);
            if(time_diff >= 84600 && time_diff <= 87000) { //84600 seconds == 1 day
                  overall_size += size;
                  overall_ssize += ssize;
            }
        }
    }

    uint64_t result = overall_size / overall_ssize; 
    write_exact(1, &result, sizeof(result));

    close(fd);

    return 0;
}

```
### Зад. 93 2022-IN-01

Вашите колеги от съседната лаборатория работят със специализирана система Hoge, която съхранява данните си в binary файлове. За съжаление обаче им се налага да работят с две различни версии на системата (стара и нова) и разбира се, те ползват различни файлови формати.

Вашата задача е да напишете програма на C (./main list.bin data.bin out.bin), която конвертира файлове с данни от стария (list.bin + data.bin) в новия (out.bin) формат.

Всички файлове се състоят от две секции – 8 байтов хедър и данни. Структурата на хедъра е:

- `uint16_t magic` – магическа стойност 0x5A4D, която дефинира, че файлът е от системата Hoge и следва тази спецификация;
- `uint16_t filetype` – дефинира какъв тип файл е това, с допустими стойности 1 за list.bin, 2 за data.bin и 3 за out.bin;
- `uint32_t count` – дефинира броя на елементите в секцията с данни на файла.

Секцията за данни на всички файлове съдържа елементи – цели числа без знак, от съответните типове:

- `list.bin` – `uint16_t`;
- `data.bin` – `uint32_t`;
- `out.bin` – `uint64_t`.

Във файлове `data.bin` и `out.bin` елементи, чиято стойност е 0, са валидни елементи, но се игнорират от системата Hoge (тяхното наличие във файла не пречи на работата на системата).

Всеки елемент в секцията за данни на `list.bin` има две характеристики – позиция и стойност, като позиция е отместването на съответния елемент, докато стойност е неговата стойност. Тези две числа семантично дефинират отмествания във файловете с данни:

- позиция дефинира отместване в `data.bin`;
- стойност дефинира отместване в `out.bin`.

На базата на тези числови характеристики елементите на `list.bin` дефинират правила от вида: “елементът на отместване позиция в `data.bin` трябва да отиде на отместване стойност в `out.bin`”.

_Забележка: Всички отмествания са спрямо N0 в брой елементи._

Програмата трябва да интерретира по ред дефинираните правила и на тяхна база да генерира изходния файл.

Обърнете внимание на проверките, обработката за грешки и съобщенията към потребителя – искаме програмата да бъде удобен инструмент.

_Забележка: Не е гарантирано, че разполагате с достатъчно памет, в която да заредите всички елементи на който и да от файловете._


```c
#include <stdio.h>
#include <stdlib.h>
#include <stdint.h>
#include <fcntl.h>
#include <unistd.h>
#include <sys/types.h>
#include <sys/stat.h>
#include <errno.h>

#define EXPECTED_MAGIC = 0x5A4D

void read_exact(int fd, void *buff, size_t count);
void write_exact(int fd, void *buff, size_t count);
uint32_t read_header_and_get_data_count(int fd, int file_num);


void read_exact(int fd, void *buff, size_t count){
    ssize_t bytes_read = read(fd, buff, count);
    if (bytes_read == -1 ){
        errx(1, "cannot read file");
    }
}

void write_exact(int fd,const void *buff, size_t count){
    ssize_t bytes_written = write(fd, buff, count);
    if (bytes_read == -1 ){
        errx(1, "cannot write file");
    }
}

uint32_t read_header_and_get_data_count(int fd, uint16_t file_num){
    uint16_t magic;
    read_exact(fd, &magic, sizeof(magic));
    if ( magic != EXPECTED_MAGIC ){
        errx(2, "magic mismatch");
    }
    uint16_t filetype;
    read_exact(fd, &filetype, sizeof(filetype));
    if (filetype != file_num ){
        errx(2, "filetype mismatch");
    }
    uint32_t count;
    read_exact(fd, &count, sizeof(count));
    return count;
}

int main(int argc, char *argv[]) {
	if (argc != 4 ) {
        errx(1, "wrong usage");
	}

    fd1 = open(argv[1], O_RDONLY)
    if (fd1 == -1){
       errx(1, "cannot  open provided file") 
    }

    fd2 = open(argv[2], O_RDONLY)
    if (fd2 == -1){
       errx(1, "cannot  open provided file") 
    }

    fd3 = open(argv[3], O_RDWR | O_TRUNC | O_CREAT, S_IRUSR | S_IWUSR);
    if (fd1 == -1){
       errx(1, "cannot  open provided file") 
    }

    input_count = read_header_and_get_data_count(fd1, 1);
    data_count = read_header_and_get_data_count(fd2, 2);
    output_count = read_header_and_get_data_count(fd3, 3);

    for (uint32_t i = 0; i < input_count; i++){
        off_t offset = lseek(fd1, 0, SEEK_CUR)
        uint16_t value;
        read_exact(fd1, &value, sizeof(value));
        lseek(fd2, offset, SEEK_SET);
        uint32_t value_to_read;
        read_exact(fd2, &value_to_read, sizeof(value_to_read));
        lseek(fd3,value, SEEK_SET);
        uint64_t value_to_write = (uint64_t)value_to_read;
        write_exact(fd3, &value_to_write, sizeof(value_to_write));
    }

    close(fd1);
    close(fd2);
    close(fd3);
}



```