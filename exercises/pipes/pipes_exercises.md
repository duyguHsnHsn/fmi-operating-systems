# Pipes exercises

### 03-a-0200
Сортирайте /etc/passwd лексикографски по поле UserID.
````shell
sort -t: -k 3 /etc/passwd
````
### 03-a-0201
Сортирайте /etc/passwd числово по поле UserID.
(Открийте разликите с лексикографската сортировка)
````shell
sort -n -t: -k 3 /etc/passwd
````
### 03-a-0210
Изведете само 1-ва и 5-та колона на файла /etc/passwd спрямо разделител ":".
````shell
cut -d: -f1,5 /etc/passwd
````

### 03-a-0211
Изведете съдържанието на файла /etc/passwd от 2-ри до 6-ти символ.
````shell
head -n 1 /etc/passwd | cut -c 2-7
````

### 03-a-0212
Отпечатайте потребителските имена и техните home директории от /etc/passwd.
````shell
cut -d: -f1,6 /etc/passwd
````
### 03-a-0213
Отпечатайте втората колона на /etc/passwd, разделена спрямо символ '/'.
````shell
cut -d/ -f2 /etc/passwd
````

### 03-a-1500
Изведете броя на байтовете в /etc/passwd.
Изведете броя на символите в /etc/passwd.
Изведете броя на редовете  в /etc/passwd.

````shell
wc -c /etc/passwd 
wc -m /etc/passwd
wc -l /etc/passwd
````

### 03-a-2000
С отделни команди, извадете от файл /etc/passwd:
- първите 12 реда
- първите 26 символа
- всички редове, освен последните 4
- последните 17 реда
- 151-я ред (или друг произволен, ако нямате достатъчно редове)
- последните 4 символа от 13-ти ред (символът за нов ред не е част от реда)
````shell
head -n 12 /etc/passwd
head -n 1 /etc/passwd | cut -c 1-27
head -n -4 /etc/passwd
tail -n 17 /etc/passwd
head -n 151 /etc/passwd | tail -n 1
head -n 13 /etc/passwd | tail -n 1| rev | cut -z -c 1-4 | rev # -z for not accounting newline
````


### 03-a-3000
Запаметете във файл в своята home директория резултатът от командата `df -P`.

Напишете команда, която извежда на екрана съдържанието на този файл, без първия ред (хедъра), сортирано по второ поле (numeric).
````shell
df -P > df_results.txt
cat df_results.txt | tail -n 8 | sort -nk2 # or tail -n +2
````

### 03-a-3100
Запазете само потребителските имена от /etc/passwd във файл users във вашата home директория.
````shell
cat /etc/passwd | cut -d: -f1 > users
````

### 03-a-3500
Изпишете всички usernames от /etc/passwd с главни букви.
````shell
cat etc/passwd | cut -d: -f1 | tr a-z A-Z
````

### 03-a-5000
* Изведете реда от /etc/passwd, на който има информация за вашия потребител.
* Изведедете този ред и двата реда преди него.
* Изведете този ред, двата преди него, и трите след него.
* Изведете *само* реда, който се намира 2 реда преди реда, съдържащ информация за вашия потребител.
````shell
cat /etc/passwd | grep "s62602"
cat /etc/passwd | grep -B 2 "s62602"
cat /etc/passwd | grep -B 2 -A 3 "s62602"
cat /etc/passwd | grep -B 2 "s62602" | head -n 1
````

### 03-a-5001
Изведете колко потребители не изпозват /bin/bash за login shell според /etc/passwd

(hint: 'man 5 passwd' за информация какъв е форматът на /etc/passwd)
````shell
cat /etc/passwd | grep -v "/bin/bash" | wc -l
````

### 03-a-5002
Изведете само имената на хората с второ име по-дълго от 6 (>6) символа според /etc/passwd
````shell
cat /etc/passwd | cut -d: -f5 | cut -d, -f1 | awk 'length($2) > 6 {print $1 " " $2}'
````

### 03-a-5003
Изведете имената на хората с второ име по-късо от 8 (<=7) символа според /etc/passwd // !(>7) = ?
````shell
cat /etc/passwd | cut -d: -f5 | cut -d, -f1 | awk 'length($2) < 8 {print $1 " " $2}'
````

### 03-a-5004
Изведете целите редове от /etc/passwd за хората от 03-a-5003
````shell
cat /etc/passwd | cut -d: -f5 | cut -d, -f1 | awk 'length($2) < 8 {print $1 " " $2}' | xargs -I {} grep {} /etc/passwd
````

### 03-a-6000

Копирайте <РЕПО>/exercises/data/emp.data във вашата home директория.
Посредством awk, използвайки копирания файл за входнни данни, изведете:

- общия брой редове
- третия ред
- последното поле от всеки ред
- последното поле на последния ред
- всеки ред, който има повече от 4 полета
- всеки ред, чието последно поле е по-голямо от 4
- общия брой полета във всички редове
- броя редове, в които се среща низът Beth
- най-голямото трето поле и редът, който го съдържа
- всеки ред, който има поне едно поле
- всеки ред, който има повече от 17 знака
- броя на полетата във всеки ред и самият ред
- първите две полета от всеки ред, с разменени места
- всеки ред така, че първите две полета да са с разменени места
- всеки ред така, че на мястото на първото поле да има номер на реда
- всеки ред без второто поле
- за всеки ред, сумата от второ и трето поле
- сумата на второ и трето поле от всеки ред
````shell
awk 'END {print NR}' emp.data
awk 'NR == 3 {print $0}' emp.data
awk '{print $3}' emp.data # or {print $NF}
awk 'END {print $NF}' emp.data
awk 'NF > 4 {print $0}' emp.data
awk '$NF > 4 {print &0}' emp.data
awk -v total=0 '{total += NF} END{print total}' emp.data
awk -v count=0 '/Beth/ {count++} END{print count}'emp.data
awk -v max=-100 -v line=-1 '$NF > max {max=$NF; line=$R} END{print max " " line}' emp.data
awk 'NF >= 1' emp.data
awk 'length($R) > 17 {print $0}' emp.data
awk '{print NF " " $0}' emp.data
awk '{print $2 " " $1}' emp.data
awk '{print $2 " " $1 " " $3}' emp.data
awk '{print NR " " $2 " " $3}' emp.data
awk '{print $1 " " $3}' emp.data
awk '{print $2+$3}' emp.data
awk -v total=0 '{total +=($2+$3)} END{print total}' emp.data
````

### 03-b-0300
Намерете само Group ID-то си от файлa /etc/passwd.
````shell
cat /etc/passwd | grep "s62602" | cut -d: -f4
# or
awk -F: '/s62602/ {print $4}' /etc/passwd
````

### 03-b-3400
Колко коментара има във файла /etc/services ? Коментарите се маркират със символа #, след който всеки символ на реда се счита за коментар.
````shell
cat /etc/services | grep "#" | wc -l
# or
awk -v total=0 '/#/ {total+=1} END{print total}'/etc/services
````

### 03-b-3500

Колко файлове в /bin са 'shell script'-oве? (Колко файлове в дадена директория са ASCII text?)
````shell

````

### 03-b-3600
Направете списък с директориите на вашата файлова система, до които нямате достъп. 
Понеже файловата система може да е много голяма, търсете до 3 нива на дълбочина.
````shell
find -maxdepth 3 -type d -readable
````

### 03-b-4000
Създайте следната файлова йерархия в home директорията ви:

* dir5/file1
* dir5/file2
* dir5/file3

Посредством vi въведете следното съдържание:

file1:

1

2

3

file2:

s

a

d

f

file3:

3

2

1

45

42

14

1

52

Изведете на екрана:
* статистика за броя редове, думи и символи за всеки един файл
* статистика за броя редове и символи за всички файлове
* общия брой редове на трите файла
````shell
mkdir dir5
cd dir5
touch file{1,2,3}
vi file1 
vi file2
vi file3
wc -l -w -c file{1,2,3}
cat file{1,2,3} | wc -l -w -c # although the above command has output for the total lines, too
 cat file{1,2,3} | wc -l
````


### 03-b-4001
Във file2 (inplace) подменете всички малки букви с главни.
````shell
# assuming we are still in dir5
cat file2 | tr a-z A-Z > temp && mv temp file2 
# or 
cat file2 | awk '{print toupper($0)}' > temp && mv temp file2
# or 
sed -i 's/[[:lower:]]/\u&/' file2
````

### 03-b-4002
Във file3 (inplace) изтрийте всички "1"-ци.
````shell
# assuming we are still in dir5
sed -i '/1/d' file3 # will delete the lines that include 1 
sed -i 's/1//g' file3 # will replace all the 1 with empty string globally 
````

### 03-b-4003
Изведете статистика за най-често срещаните символи в трите файла.
````shell
cat file1 file2 file3 | fold -w1 | sort | uniq -c | sort -nr # fold -w1 makes one line include 1 char
````

### 03-b-4004
Направете нов файл с име по ваш избор, чието съдържание е конкатенирани
съдържанията на file{1,2,3}.
````shell
touch file4
cat file{1,2,3} > file4
````

### 03-b-4005
Прочетете текстов файл file1 и направете всички главни букви малки като
запишете резултата във file2.
````shell
cat file1 | tr A-Z a-z > file2
# or 
awk '{print toupper($0)}' file1 > file2
````

### 03-b-5200
Намерете броя на символите, различни от буквата 'а' във файла /etc/passwd
````shell
cat /etc/passwd | tr -d 'a' | wc -c

````


### 03-b-5300
Намерете броя на уникалните символи, използвани в имената на потребителите от
/etc/passwd.
````shell
cat /etc/passwd | cut -d: -f5 | cut -d, -f1 | tr -d " " | grep -o . | sort | uniq | wc -c
````

### 03-b-5400
Отпечатайте всички редове на файла /etc/passwd, които не съдържат символния низ 'ов'.
````shell
cat /etc/passwd | grep -v 'ов'
````

### 03-b-6100
Отпечатайте последната цифра на UID на всички редове между 28-ми и 46-ред в /etc/passwd.
````shell
cat /etc/passwd | head -46 | tail -n 19 | cut -d: -f3 | rev | cut -c 1 | rev
````

### 03-b-6700
Отпечатайте правата (permissions) и имената на всички файлове, до които имате
read достъп, намиращи се в директорията /tmp. (hint: 'man find', вижте -readable)
````shell
find /tmp -type f -readable -printf "%M\t%p" # %M placeholder for  file's permissions, %p placeholder for file path
````

### 03-b-6900
Намерете имената на 10-те файла във вашата home директория, чието съдържание е
редактирано най-скоро. На първо място трябва да бъде най-скоро редактираният
файл. Намерете 10-те най-скоро достъпени файлове. (hint: Unix time)
````shell
find . -maxdepth 1 -type f -printf '%T@ %p\n' | sort -n | tail -n 10 # %T@ last modification time
# or
ls -ltp | grep -v / | head -n 10
````

### 03-b-7000
да приемем, че файловете, които съдържат C код, завършват на `.c` или `.h`.
Колко на брой са те в директорията `/usr/include`?
Колко реда C код има в тези файлове?
````shell
# to fet the files
find /usr/include/ -type f -name '*.[c\|h]'
# or 
ls -R /usr/include/ | grep -E '\.(c|h)$'
# count 
find /usr/include/ -type f -name '*.[c\|h]' | wc -l
# c code count 
find /usr/include/ -type f \( -name "*.c" -o -name "*.h" \) -exec wc -l {} + | awk '{total += $1} END {print total}'
````

### 03-b-7500
Даден ви е ASCII текстов файл - /etc/services. Отпечатайте хистограма на 10-те най-често срещани думи.
Дума наричаме непразна последователност от букви. Не правим разлика между главни и малки букви.
Хистограма наричаме поредица от редове, всеки от които има вида:
<брой срещания> <какво се среща толкова пъти>
````shell
cat /etc/services | tr -cs '[:alnum:]' '[\n*]' | tr 'A-Z' 'a-z' | sort | uniq -c | sort -nr | head -10

````
### 03-b-8000
Вземете факултетните номера на студентите (описани във файла
<РЕПО>/exercises/data/mypasswd.txt) от СИ и ги запишете във файл si.txt сортирани.


Студент е част от СИ, ако home директорията на този потребител (според
<РЕПО>/exercises/data/mypasswd.txt) се намира в /home/SI директорията.

### 03-b-8500
За всяка група от /etc/group изпишете "Hello, <група>", като ако това е вашата група, напишете "Hello, <група> - I am here!".
````shell
awk -F: -v cg="$(id -gn)" '{print ($1 == cg) ? "Hello, " $1 " - I am here!" : "Hello, " $1}' /etc/group
````

### 03-b-8600
Shell Script-овете са файлове, които по конвенция имат разширение .sh. Всеки
такъв файл започва с "#!<interpreter>" , където <interpreter> указва на
операционната система какъв интерпретатор да пусне (пр: "#!/bin/bash",
"#!/usr/bin/python3 -u").

Намерете всички .sh файлове в директорията `/usr` и нейните поддиректории, и
проверете кой е най-често използваният интерпретатор.
````shell
find /usr -type f -name "*.sh" | xargs -I {} grep -m 1 '^#!' {} | sort | uniq -c | sort -nr | head -1 # -m stop after 1 select
````

### 03-b-8700
1. Изведете GID-овете на 5-те най-големи групи спрямо броя потребители, за които
   съответната група е основна (primary).

2. (*) Изведете имената на съответните групи.

Hint: /etc/passwd
````shell
awk -F: '{print $4}' /etc/passwd | sort | uniq -c | sort -nr -k 1 | head -n 5 | awk '{print $2}'
````

### 03-b-9000
Направете файл eternity. Намерете всички файлове, намиращи се във вашата home
директория и нейните поддиректории, които са били модифицирани в последните
15мин (по възможност изключете .).  Запишете във eternity името (път) на
файла и времето (unix time) на последната промяна.
````shell
find . -mindepth 2 -type f -mmin 15 -printf '%p %T@\n' > eternity

````

### 03-b-9050
Копирайте файл <РЕПО>/exercises/data/population.csv във вашата home директория.

### 03-b-9051
Използвайки файл population.csv, намерете колко е общото население на света
през 2008 година. А през 2016?

### 03-b-9052
Използвайки файл population.csv, намерете през коя година в България има най-много население.

### 03-b-9053
Използвайки файл population.csv, намерете коя държава има най-много население през 2016. А коя е с най-малко население?
(Hint: Погледнете имената на държавите)

### 03-b-9054
Използвайки файл population.csv, намерете коя държава е на 42-ро място по
население през 1969. Колко е населението й през тази година?

### 03-b-9100
В home директорията си изпълнете командата
curl -o songs.tar.gz "http://fangorn.uni-sofia.bg/misc/songs.tar.gz"

### 03-b-9101
Да се разархивира архивът songs.tar.gz в директория songs във вашата home директория.
````shell
mkdir songs
tar -xvf songs.tar.gz --directory=songs
````

### 03-b-9102
Да се изведат само имената на песните.
````shell
find songss -type f -name "*.ogg" | awk -F"- " '{print $2}' | awk -F" (" '{print $1}'
````

### 03-b-9103
Имената на песните да се направят с малки букви, да се заменят спейсовете с
долни черти и да се сортират.
````shell
find songss -type f -name "*.ogg" | awk -F"- " '{print $2}' | awk -F" (" '{print $1}' | tr A-Z a-z | tr " " "_" | sort
````

### 03-b-9104
Да се изведат всички албуми, сортирани по година.
````shell
find songs -type f -name *.ogg | awk -F '(' '{print $2}' | awk -F ')' '{print $1}' | sort -n -t',' -k 2 | awk -F"," '{print $1}'
# or 
find songs -type f -name *.ogg | awk -F '(' '{print $2}' | awk -F ')' '{print $1}' | sort -n -t',' -k 2 | cut -d, -f 1
````

### 03-b-9105
Да се преброят/изведат само песните на Beatles и Pink.
````shell
# Pink != Pink Floyd
# count 
find songs -type f -name *.ogg | grep "Beatles\|Pink" | grep -v "Floyd" | wc -l
# song names
find songs -type f -name *.ogg | grep "Beatles\|Pink" | grep -v "Floyd" | awk -F"- " '{print $2}' | awk -F"(" '{print $1}'
````

### 03-b-9106
Да се направят директории с имената на уникалните групи. За улеснение, имената
от две думи да се напишат слято:

Beatles, PinkFloyd, Madness
````shell
# working in home this will create the directories in songs dir
find songs -type f -name *.ogg | awk -F"-" '{print $1}' | sort | uniq | xargs -I {} mkdir {}
````


### 03-b-9200
Напишете серия от команди, които извеждат детайли за файловете и директориите в
текущата директория, които имат същите права за достъп както най-големият файл
в /etc директорията.


### 03-b-9300
Дадени са ви 2 списъка с email адреси - първият има 12 валидни адреса, а
вторията има само невалидни. Филтрирайте всички адреси, така че да останат
само валидните. Колко кратък регулярен израз можете да направите за целта?

Валидни email адреси (12 на брой):
```
email@example.com
firstname.lastname@example.com
email@subdomain.example.com
email@123.123.123.123
1234567890@example.com
email@example-one.com
_______@example.com
email@example.name
email@example.museum
email@example.co.jp
firstname-lastname@example.com
unusually.long.long.name@example.com
```

Невалидни email адреси:
```
#@%^%#$@#$@#.com
@example.com
myemail
Joe Smith <email@example.com>
email.example.com
email@example@example.com
.email@example.com
email.@example.com
email..email@example.com
email@-example.com
email@example..com
Abc..123@example.com
(),:;<>[\]@example.com
just"not"right@example.com
this\ is"really"not\allowed@example.com
```

### 03-b-9400

Посредством awk, използвайки emp.data (от 03-a-6000.txt) за входнни данни,
изведете:

- всеки ред, като полетата са в обратен ред

(Разгледайте for цикли в awk)

### 03-b-9500

Копирайте <РЕПО>/exercises/data/ssa-input.txt във вашата home директория.
Общият вид на файла е:

- заглавна част:
  Smart Array P440ar in Slot 0 (Embedded)

- една или повече секции за масиви:

  Array A

  Array B

  ...

  като буквата (A, B, ...) е името на масива

- във всяка таква секция има една или повече подсекции за дискове:
  
physicaldrive 2I:0:5
  
physicaldrive 2I:0:6

...

като 2I:0:5 е името на диска

- във всяка подсекция за диск има множество параметри във вида:

key name: value

като за нас са интересни само:

      Current Temperature (C): 35
      Maximum Temperature (C): 36

Напишете поредица от команди която обработва файл в този формат, и генерира
следният изход:

```
A-2I:0:5 35 36
A-2I:0:6 34 35
B-1I:1:1 35 50
B-1I:1:2 35 49
```

x-yyyyyy zz ww

където:
- x е името на масива
- yyyyyy е името на диска
- zz е current temperature
- ww е max temperature
