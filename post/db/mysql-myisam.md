---
layout: page
title: mysql-myisam
category: blog
description: 
date: 2018-09-27
---
# Preface

https://dev.mysql.com/doc/internals/en/myisam-introduction.html

# create table
When you say:

	CREATE TABLE Table1 ... 

MySQL creates files named `Table1.MYD ("MySQL Data")`, `Table1.MYI ("MySQL Index")`, and `Table1.frm ("Format")`. These files will be in the directory:

	/<datadir>/<database>/