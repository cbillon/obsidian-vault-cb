---
tags:
  - moodle
  - tools
  - generator
---


In the real world our Moodle has some 30k courses and about 10k users. In this document I want to demonstrate how to get a developer Moodle filled with courses and one teacher assigned to this courses.

## Pre requisites

My setup looks as follows:

- a running Moodle instance
- a docker container with the webserver or the Moodle SDK on a Linux
- a bash compatible command line (the above fulfill this - on Windows this might as well work with the Linux subsystem)

Whenever I use a backslash here at the end of the line, that's for readability in order to continue with the command on the next line. You could omit the backslash and the line break. If there is a line break without backslash at the previous line, then you still can merge the lines into one but must put an semicolon in betweeen.

### Create a course
Moodle has a tool to create a course per command line with some random data and also students that are enrolled in this course. To create 150 courses at once we go to the project root and run the following command:
```

seq 1 120 | while read i; do php admin/tool/generator/cli/maketestcourse.php \
  --shortname="test-$i" --fullname="Make Course Test $i" --size=XS ; done

```
This creates courses with random data in it, one student enroled which is also created with random data.

## Enrol the user to the courses

To enrol the user into the course as a teacher we use the User Upload feature of Moodle.

For that we need to create a csv file to upload the user data. This is a file with two lines, the column names and the actual user data. We do this by simultaneously creating two files first, the header.csv and the firstline.csv.

First we "create" the user:
```
echo -n "username,firstname,lastname,email," >> header.csv
echo -n "teacher1,Walter,White,white@example.com," >> firstline.csv

```
Now we create the course enrolments. For each course the CSV is extended by two columns, the role and the course id. The role column is typeX, the course column is courseX. The "X" is replaced by the course number. We use the same loop like when creating the courses:
```
seq 1 150 | while read i; do
  echo -n "2,test-$i," >> firstline.csv
  echo -n "type$i,course$i," >> header.csv
done

```
It is important that the course name here is the same that we used as the shortname when creating the courses. The 2 before the course short name is the role id so that the user is assigned as a course creator in this case. To get to know which roles are exist on your Moodle, check out the contents of the role table. Mine looks like this:

```
select * from role;
id|name|shortname     |description|sortorder|archetype     |
--+----+--------------+-----------+---------+--------------+
 1|    |manager       |           |        1|manager       |
 2|    |coursecreator |           |        2|coursecreator |
 3|    |editingteacher|           |        3|editingteacher|
 4|    |teacher       |           |        4|teacher       |
 6|    |guest         |           |        6|guest         |
 7|    |user          |           |        7|user          |
 8|    |frontpage     |           |        8|frontpage     |
 5|    |student       |           |        5|student       |

```
Now we have two files and must merge them into one file:
```
echo >> header.csv
cat firstline.csv >> header.csv
```

As a result header.csv now contains two lines, the first with the column names and the second with thr user data.

To make sure that nothing got messed up, check an arbitrary course column to see that the column name and the course short name match:
```
cat header.csv | cut -d, -f240
course118
test-118
```

Before we are finally are able to upload the file we must remove the trailing , at the end of each line.

This file can be used to be uploaded into Moodle now. Go to Site Administration -> Users -> Upload users to upload the file, and then click the button to trigger the import.

When I uploaded the file and it was imorted, even I used the 2 as the type, the role that got assigned to the course was the editingteacher one. I may have misinterpreded something from the docs.