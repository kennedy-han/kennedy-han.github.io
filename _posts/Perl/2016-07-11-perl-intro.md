---
layout: post
title: "Perl 1.Introduction"
description: "Perl Introduction"
category: Script
tags: [Perl]
---

This series article that are learning note for *Learning Perl* by Randal L. Schwartz, brian d foy, and Tom Phoenix
published by O'REILLY.

#Perl 1. Introduction

###What's Perl Really Good For?
Perl is optimized for problems which are about 90% working with text.

###What Is Perl Not Good For?
You shouldn't choose Perl if you're trying to make an opaque binary.

###What is CPAN?
CPAN is the Comprehensive Perl Archive Network, your one-stop shopping for Perl.

###Are There Any Other Kinds of Support?
http://www.pm.org/
http://perldoc.perl.org/
http://learn.perl.org/faq/
http://perlmonks.org/
http://learn.perl.org/

###A Simple Program

create and edit a text file

```
$ vi my_program

#!/usr/bin/perl
print "Hello, world!\n";
```

apply permissions:

```
chmod a+x my_program
```

finally run

```
$ ./my_program
```

------

There's another way to write this simple program in Perl 5.10 or later.

Instead of `print`, we use `say`, which does almost the same thing ,but with less typing.

Since it's a new feather and you might not using Perl 5.10 yet, we include 5.010 statement that tells Perl that we used new feathers:

```
#!/usr/bin/perl

use 5.010;

say "Hello, world!";
```

This book covers up to Perl 5.14, so in many of the new features we preface the examples to remind you to add this line:

```
use 5.014;
```

###How Do I Compile My Perl Program?
Just run your Perl program. The `perl` interpreter compiles and runs your program in one user step:

```
$ perl my_program
```