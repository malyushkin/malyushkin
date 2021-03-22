---
title: "Курс по Big Data — Часть 1. Первое погружение в HDFS"
date: 2021-02-22T00:25:42+03:00
draft: true
tags: ["course", "hdfs"]
ShowToc: true
TocOpen: true
---

В этом месяце я приступил к прохождению Практического курса по большим данным от [BigData Team](https://bigdatateam.org/big-data-course). Курс состоит из 10 занятий и 10 домашних заданий. Для закрепления результата я буду делать краткую выдержку приобретенных знаний в своей ленте. 

Первя часть — Введение в Большие Данные (Big Data). Распределенные файловые системы.

## Теория



## Практика

Задача практической работы — получить базовые знания об утилите `hdfs dfs` для работы с HDFS-пространством . Базовый список флагов похож на основные команды Linux: `-ls`, `-cat`, `-mkdir` и др. 

Мы использовали версию 2.10.0, документация доступна [по ссылке](https://hadoop.apache.org/docs/r2.10.0/hadoop-project-dist/hadoop-common/FileSystemShell.html). Версию Hadoop можно проверить командой `hdfs version`.

```shell
hdfs version

Hadoop 2.10.0
Subversion ssh://git.corp.linkedin.com:29418/hadoop/hadoop.git -r e2f1f118e465e787d8567dfa6e2f3b72a0eb9194
Compiled by jhung on 2019-10-22T19:10Z
Compiled with protoc 2.5.0
From source with checksum 7b2d8877c5ce8c9a2cca5c7e81aa4026
This command was run using /usr/local/hadoop-2.10.0/share/hadoop/common/hadoop-common-2.10.0.jar
```

Для чтения руководства можно использовать команды `-help` и `-usage`, например:

```shell
hdfs dfs -help ls

-ls [-C] [-d] [-h] [-q] [-R] [-t] [-S] [-r] [-u] [<path> ...] :
  List the contents that match the specified file pattern. If path is not
  specified, the contents of /user/<currentUser> will be listed. For a directory a
  list of its direct children is returned (unless -d option is specified).
  
  Directory entries are of the form:
  	permissions - userId groupId sizeOfDirectory(in bytes)
  modificationDate(yyyy-MM-dd HH:mm) directoryName
  
  and file entries are of the form:
  	permissions numberOfReplicas userId groupId sizeOfFile(in bytes)
  modificationDate(yyyy-MM-dd HH:mm) fileName
  
    -C  Display the paths of files and directories only.
    -d  Directories are listed as plain files.
    -h  Formats the sizes of files in a human-readable fashion
        rather than a number of bytes.
    -q  Print ? instead of non-printable characters.
    -R  Recursively list the contents of directories.
    -t  Sort files by modification time (most recent first).
    -S  Sort files by size.
    -r  Reverse the order of the sort.
    -u  Use time of last access instead of modification for
        display and sorting.
```

### Работа с пространством

#### `-ls`

`-ls` возвращает статистику для указанных файлов и каталогов. Статистика выводится по столбцам и включает: права, фактор репликации, пользователь (владелец), группа пользователей, объём (по умолчанию в байтах), дата последнего изменения и название файла или каталога. Опционально можно добавлять флаги `-R` — recursive (рекурсивный вывод), `-h` — human-readable (человекочитаемый объём памяти). Здесь и дальше для полного списка флагов см. `hdfs dfs -help <flag>`.

Использование `hdfs dfs -ls [-R] [-h] [<path> ...]`.

Например, команда `hdfs dfs -ls -R /data/wiki/` рекурсивно выведет файлы и каталоги в `/data/wiki/`:

```shell
hdfs dfs -ls -R /data/wiki/

drwxr-xr-x   - hdfs hdfs           0 2020-03-13 00:23 /data/wiki/en_articles
-rw-r--r--   3 hdfs hdfs 12328051927 2020-03-13 00:23 /data/wiki/en_articles/articles
drwxr-xr-x   - hdfs hdfs           0 2020-03-12 21:03 /data/wiki/en_articles_part
-rw-r--r--   3 hdfs hdfs    76861985 2020-03-12 21:03 /data/wiki/en_articles_part/articles-part
```

Команда `hdfs dfs -ls -R -h /data/wiki/` сделает тоже самое, но объем файлов будет представлен в МБ и ГБ (11.5 G и 73.3 M):

```shell
hdfs dfs -ls -R -h /data/wiki

drwxr-xr-x   - hdfs hdfs           0 2020-03-13 00:23 /data/wiki/en_articles
-rw-r--r--   3 hdfs hdfs      11.5 G 2020-03-13 00:23 /data/wiki/en_articles/articles
drwxr-xr-x   - hdfs hdfs           0 2020-03-12 21:03 /data/wiki/en_articles_part
-rw-r--r--   3 hdfs hdfs      73.3 M 2020-03-12 21:03 /data/wiki/en_articles_part/articles-part
```

Цифра 3 во втором столбце показывает фактор репликации для файлов в HDFS (файл имеет 3 реплики).

#### `-du`

`-du` возвращает только объём (по умолчанию в байтах) файлов и каталогов. Флаги `-s` — summary (сводная сумма всех файлов и каталогов), `-h` — аналогично `ls`.

Использование `hdfs dfs -du [-s] [-h] [-x] <path>`.

Например, команда `dfs -du -s -h /data/wiki`, покажет объём памяти занимаемый каталогом `/data/wiki` в человекочитаемом виде (11.6 G): 

```shell
hdfs dfs -du -s -h /data/wiki

11.6 G  /data/wiki
```

### Создание и удаление папок

#### `-mkdir`

Флаг `-mkdir` создаёт новый каталог. Флаг `-p` позволяет создавать вложенную структуру каталогов.

Использование `hdfs dfs -mkdir [-p] <path>`.

```shell
hdfs dfs -mkdir -p hdfs_dir/hdfs_subdir/hdfs_subsubdir

# recursive output
hdfs dfs -ls -R hdfs_dir

drwxr-xr-x   - hdfs hdfs          0 2021-02-28 17:06 hdfs_dir/hdfs_subdir
drwxr-xr-x   - hdfs hdfs          0 2021-02-28 17:06 hdfs_dir/hdfs_subdir/hdfs_subsubdir
```

#### `-rm`

Для удаления файлов и каталогов используется флаг `-rm`. Для удаления всех вложенных файлов и каталогов добавляется флаг рекурсии (`-R`).

Использование: `hdfs dfs -rm [-f] [-r|-R] [-skipTrash] [-safely] <src>`

```shell
hdfs dfs -rm -R 12_hdfs_folder

21/02/28 17:18:04 INFO fs.TrashPolicyDefault: Moved: 'hdfs://brain-master.bigdatateam.org:8020/user/x5_malyushkin/hdfs_dir' to trash at: hdfs://localhost:8020/user/hdfs/.Trash/Current/user/hdfs/hdfs_dir
```

Как видно в примере выше, при удалении в HDFS содержимое сначала перемещаются в каталог `/user/<username>/.Trash/`, то есть в "корзину". Их можно восстановить простым переносом (`-mv`)  из `.Trash`. Удаление из корзины происходит спустя регулируется параметром `fs.trash.interval`. Для удаления минуя `.Trash` необходимо истользовать команду `hdfs dfs -rm -R -skipTrash <src>`.

### Работа с файлами

#### `-touchz`

`-touchz` используется для создания файлов в HDFS. Команда `hdfs dfs -touchz hdfs_empty_file` создаст пустой файл.

Использование: `hdfs dfs -touchz <path>`

#### `-put`

Для копирования содержимого на HDFS используется флаг `-put`. В качестве аргументов передаются 2 пути: в локальной файловой системе и на HDFS.

Использование: `hdfs dfs -put [-f] [-p] [-l] [-d] <localsrc> <dst>`.

В примере ниже создается файл в локальной файловой системе, а затем копируется в HDFS.

```shell
# generate some file
printf "Some text\nSome text on 2nd line\nSome text again\nLast line with text" > some_file

# put to hdfs
hdfs dfs -put some_file
```

#### `-cat`

`-cat` выводит содержимое файла на экран. 

Использование: `hdfs dfs -cat [-ignoreCrc] <src>`.

Выведем текст файла `some_file` созданного на предыдущем шаге: 

```shell
hdfs dfs -cat some_file

Some text
Some text on 2nd line
Some text again
Last line with text
```

По умолчанию `-cat` выводит файл полностью. Для вывода первых _n_ строк воспользуемся конвейером и укажем стандартную команду `head -n`.

```shell
hdfs dfs -cat some_file | head -2

Some text
Some text on 2nd line
```

#### `-tail`

Флаг `-tail` выводит конец файла. Стоит отметить, что конец — это 1 КБ последних символов содержимого.

Использование:

#### `-cp`

`-cp` копирует файлы в HDFS. Пример `hdfs dfs -cp some_file some_file_copy`.

Использование `hdfs dfs -cp [-f] [-p | -p[topax]] [-d] <src> <dst>`.

#### `-getmerge`

```shell

```
hdfs dfs -getmerge -nl 12_rand_file 12_rand_file_copy_new 12_rand_file_merged