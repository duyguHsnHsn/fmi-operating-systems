# Commands exercises

### 02-a-0100
Направете копие на файла /etc/passwd във вашата home директория под името my_passwd.
````shell
cp /etc/passwd my_passwd
````

### 02-a-0500
Направете директория practice-test в home директорията ви. Вътре направете директория test1. 
Можете ли да направите тези две неща наведнъж? Разгледайте нужната man страница.
След това създайте празен файл вътре, който да се казва test.txt,
преместете го в practice-test чрез релативни пътища.
````shell
mkdir -p practice-test/test1
touch practice-test/test1/test.txt
mv practice-test/test1/test.txt practice-test/
````

### 02-a-0600
Създайте директорията practice/01 във вашата home директория.
Създайте 3 файла в нея - f1, f2, f3.
Копирайте файловете f1, f2, f3 от директорията practice/01/ в директория dir1,
намираща се във вашата home директория. Ако нямате такава, създайте я.
````shell
mkdir -p practice/01
cd practice/01
touch f1 f2 f3
cd ../../
mkdir dir1
cp practice/01/f1 practice/01/f2 practice/01/f3 dir1/
````

### 02-a-0601
Нека файлът f2 бъде преместен в директория dir2, намираща се във вашата home директория и бъде преименуван на numbers.
````shell
mkdir dir2
mv practice/01/f2 dir2/numbers
````

### 02-a-1200
Отпечатайте имената на всички директории в директорията /home.
````shell
ls -l | grep ^d | awk '{print $9}'
````

### 02-a-4000
Създайте файл permissions.txt в home директорията си.
За него дайте единствено - read права на потребителя създал файла, 
write and exec на групата, read and exec на всички останали. 
Направете го и с битове, и чрез "буквички".

````shell
touch permissions.txt
# octal (bitwise)
chmod 435 permissions.txt
# букви
chmod u=r,g=xr,o=x permissions.txt
````

### 02-a-4100
За да намерите какво сте правили днес: намерете всички файлове в home директорията ви,
които са променени в последния 1 час.
````shell
find -maxdepth 1 -amin -60 -type f
````

### 02-a-5000
Копирайте /etc/services в home директорията си. Прочетете го с командата cat. 
(Ако този файл го няма, прочетете с cat произволен текстов файл напр. /etc/passwd)
````shell
cp /etc/services .
cat services
````
### 02-a-5200
Създайте symlink на файла /etc/passwd в home директорията ви (да се казва например passwd_symlink).
````shell
ln -s /etc/passwd passwd_symlink
````

### 02-a-5400
Изведете всички обикновени ("regular") файлове, които /etc и нейните преки поддиректории съдържат
````shell
find -maxdepth 2 -type f
````

### 02-a-5401
Изведете само първите 5 реда от /etc/services
````shell
head -n 5 /etc/services
````

### 02-a-5402
Изведете всички обикновени ("regular") файлове, които само преките поддиректории на /etc съдържат
````shell
find -mindepth 1 -maxdepth 2 -type f
````

### 02-a-5403
Изведете всички преки поддиректории на /etc
````shell
# find
find /etc -maxdepth 1 -type d 
# ls -d
ls -d /etc/*/
# ls -l
ls -l /etc | grep ^d | awk '{print $9}'
````

### 02-a-5500
Създайте файл, който да съдържа само последните 10 реда от изхода на 02-a-5403
````shell
ls -l /etc | grep ^d | awk '{print $9}' | tail -n 10 > last_dirs.txt
````

### 02-a-5501
Изведете обикновените файлове по-големи от 42 байта в home директорията ви
````shell
find . -maxdepth 1 -type f -size +42c
````

### 02-a-5504
Изведете всички обикновени файлове в директорията /tmp които са от вашата група,
които имат write права за достъп за група или за останалите(o=w)
````shell
find /tmp -type f -group "students" -perm /0022
````

### 02-a-5505
Изведете всички файлове, които са по-нови от practice/01/f1 ( би трябвало да е създаден като част от по-ранна задача ).
````shell
find . -type f -newer practice/01/f1
````

### 02-a-5506
Изтрийте файловете в home директорията си по-нови от practice/01/f3 
(подайте на rm опция -i за да може да изберете само тези които искате да изтриете).
````shell
find . -maxdepth 1 -type f -newer practice/01/f3 | xargs -I {} rm {} # -i didn't work for me
````

### 02-a-6000
Намерете файловете в /bin, които могат да се четат, пишат и изпълняват от всички.
````shell
find /bin -maxdepth 1 -type f -perm 777
````

### 02-a-8000
Копирайте всички файлове от /etc, които могат да се четат от всички, в
директория myetc в home директорията ви. Направете такава, ако нямате.
````shell
mkdir myetc
find /etc -maxdepth 1 -type -f -perm /0004 | xargs -I {} cp {} myetc
````

### 02-a-9000
От предната задача: когато вече сте получили myetc с файлове, архивирайте
всички от тях, които започват с 'c' в архив, който се казва c_start.tar

изтрийте директорията myetc и цялото и съдържание

изтрийте архива c_start.tar
````shell
find myetc -name "c*" -type f | xargs -I {} tar rfv "c_start.tar" {}
rm -rf myetc
rm x_start.tar
````

### 02-a-9500
Използвайки едно извикване на командата find, отпечатайте броя на редовете във всеки обикновен файл в /etc директорията.
````shell
find /etc -maxdepth 1 -type f | xargs -I {} wc -l {}
````

### 02-b-4000
Копирайте най-малкия файл от тези, намиращи се в /etc, в home директорията си.
````shell
# the smallest file was environment file
# when i want to copy it to the home dir i get "Permission denied"
# so i created a test dir where i copied it
mkdir test
ls -Sr /etc | head -n 1 | xargs -I {} cp /etc/{} test/{}
````
