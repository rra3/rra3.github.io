---
layout: post
title: "The Poor Man's Mongo Interface for Ruby"
date: 2014-03-22 11:53:19 -0400
comments: true
categories: [ruby, mongodb, javascript, vim]
---
> We will encourage you to develop the three great virtues of a _programmer: laziness, impatience_, and hubris.
- Larry Wall, Programming Perl

I've been working in Mongo a lot lately. Initially jazzed by the thrill of using javascript for the query language, I've since learned that the queries themselves can quite quickly become verbose and ungainly to work with from within the confines of the little CLI mongo shell environment. For longer queries, you do have the option of writing standalone js scripts, resorting to the mongo shell prompt only for smaller, simpler queries. It feels much more natural to think of mongo queries as tiny scripts than something analogous to a SQL query. In fact, Mongo [exposes many javascript equivalents](http://docs.mongodb.org/manual/tutorial/write-scripts-for-the-mongo-shell/) that allows you to do just about anything in native javascript that you can do manually from the mongo shell prompt.  After creating and saving your javascript file, you can then pass the saved file directly to mongo:

```
 mongo [options] [db address] [file names (ending in .js)]
```

I use vim, and I was delighted to learn that while crafting an involved query I could test it quickly by mapping a key to something like this:

```
  :w !mongo
```

Why does this work? Because mongo accepts input from stdin. In your life as a programmer you often have to make compromises for the sake of pragmatism. It can often be quicker to open a system call to external programs rather than taking the time to learn an new api. Depending upon the size and nature of the task, this can be preferable. What is the cost/benefit ratio to doing something in pure Ruby versus delegating to an external program? You have to make that call, but for small tasks, leaning on external tools and programs can be a blessing - especially those that accept stdin. In my former life as a perl programmer, I learned to open a filehandle directly against a subprocess. This was handy for programs that accept data on stdin, such as sendmail:

```perl
open(SENDMAIL, "|/usr/sbin/sendmail -oi -t  -odq")
  or die "cannot fork for sendmail: $!\n";

print SENDMAIL << "EOF";
From: Bob <bob\@localhost>
To: Joe <Joe\@somewherez>
Subject: Lunch?

Hey Joe, whaddya know? Got lunch plans?

EOF
```

...I also love heredocs - which as you can see above, allow me to write directly to the open SENDMAIL filehandle. Ruby, of course, offers the same flexibility with filehandles. I had nearly forgotten about this until recently when I needed to send an email report from data I had stored in Mongo. I wanted to write the report generator in Ruby, but also didn't want to bother learning a new api for interfacing with Mongo when I had all of my necessary scripts at the ready as javascript. That's when I recalled the above sendmail trick. Here's a variation in Ruby for writing queries to mongo:

```ruby
   def fork_mongo
    open("|mongo --quiet -u #{ENV['MONGO_USER']} -p #{ENV['MONGO_PW']} your.mongo.server:27017/some_db", 'w+') do |p|
      p.write('db.orders.aggregate({$match: {status: "A}"}, {$group: {_id: "$cust_id", total: {$sum: "$amount"}}})')
      p.close_write
      p.read
    end
  end
```

Notice that the parameter passed to the open method is preceded by a "|" symbol, which indicates you are opening a subprocess. The syntax may look strange, but if you have any familiarity with unix pipes, it makes more sense. I've opened the IO stream here in write mode ("w+") as I wish to write my query to mongo. It turns out this also gives me read access to the stream, so you can be fancy and say that the IO stream is _duplexed_. For further information on opening IO streams with open, refer to the documenation on [Kernel#open](http://ruby-doc.org/core-2.1.1/Kernel.html#method-i-open).

I've hardcoded the javascript into the write method to emphasize the point, but you could make this much more dynamic with variable interpolation. Heredocs are nice because you don't have to fuss with quoting and you can retain formatting for readability.  

One potential drawback is that the output will be returned in string format, so you will have to parse/split/coerce that into whatever format you need, which - depending upon your needs, may be more trouble than necessary, so YMMV. 
