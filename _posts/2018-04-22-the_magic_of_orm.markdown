---
layout: post
title:      "The MAGIC of ORM "
date:       2018-04-22 21:18:00 +0000
permalink:  the_magic_of_orm
---


I've started working through the first half of the ORM curriculum, and the way to translate Ruby objects to raw data and back, is intuitive and practical. There will probably be more lessons to abstract away some of these expressions, but the way a method can use SQL to find the raw data, then operate on the data in Ruby is starting to complete the picture of "real world" vs "terminal world."

For example, we have a Student class represented in our database as a students table. Through our environment file, we can create a connection to that database, expressed as the hash: 
```
require 'sqlite3'
require_relative '../lib/student'

DB = {:conn => SQLite3::Database.new("db/students.db")}
```
The cool thing here is that we can use SQL within a method to query the DB and then operate it in Ruby or just return the data as an array. 

Here, we are simply returning the first x-amount of studnets from a specific grade, in this case, grade 10:

```
def self.first_X_students_in_grade_10(num_of_students)
    sql = <<-SQL
    SELECT * FROM students
    WHERE id <= ?
    SQL
    DB[:conn].execute(sql, num_of_students)
  end
```

What am liking so far about this, is that it stores a string inside a local variable and uses a bound parameter ("?") rather than string interpolation to insert the method argument in the SQL query. Then, we just .execute our sql local variable and our num_of_students argument to return an array of the first (num_of_students) in grade 10. 

Going deeper, we can also turn the array returned to us as a Ruby object with our .new_from_db class method:

```
def self.new_from_db(row)
    student = self.new
    student.id = row[0]
    student.name = row[1]
    student.grade = row[2]
    student
  end
```

Using this method to create an instance from raw data like so:

```
def self.first_student_in_grade_10
    sql = <<-SQL
    SELECT * FROM students
    WHERE grade = 10
    SQL
    DB[:conn].execute(sql).map {|row| self.new_from_db(row)}.first
  end
```

We can return an instance of a Student created from a database. Pretty cool. 
