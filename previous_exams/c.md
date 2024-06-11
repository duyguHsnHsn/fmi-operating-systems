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