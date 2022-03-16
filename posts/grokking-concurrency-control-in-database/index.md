---
title: Grokking Concurrency Control in Database
date: 2022-03-13T11:27:41.000Z
draft: true
tags: [Database, Concurrency Control, Isolation levels, Relational Database, DBMS]
---

## What can get wrong?

There are many problems that can happen while running more than one transaction concurrently, some of them mentioned by SQL-92 standard:

- Dirty read
- Non-repeatable read
- Phantom read

And some aren't but they happen in the real world:

- Read skew
- Write skew
- Lost update

And we will explore them one by one. for simplicity sake let's call the transaction that has the problem TXN1 and the one causing the problem TXN2.

### Dirty read

It's when a transaction TXN1 start running and in the middle TXN2 starts and changed a value without committing it, then TXN1 make a query based on the new value which maybe rolled back  by TXN2, so TXN1 made a decision based on a temporary value. like:

{{< figureCupper img="dirty read.jpg" caption="Likes should be 8 but because the dirty read it's 6" command="Resize" options="2000x1300" >}}

So  what happened:

- TXN1 starts then TXN2 starts as well, then TXN2 updates the post likes from 7 to 5.

- TXN1 updates the likes by incrementing them so it's now 6.

- TXN2 rolls back the changes it made (changing likes from 7 to 5).

- TXN1 commits likes value of 6 that was made based on a value already rolled back so it's wrong.

### Non-repeatable read

It's when a transaction TXN1 start running and reads some data, and then TXN2 starts running and change the same data and commits, then TXN1 reads the same data  with the same select query and gets different result from the first one , so we call this a non-repeatable read, like this example:

{{< figureCupper img="Non-repeatable read.jpg" caption="We couldn't get the same read with the same select query because TXN2 changed the row and committed so our reads are non-repeatable" command="Resize" options="2000x1300" >}}

So  what happened:

- TXN1 starts then TXN2 starts as well, then TXN1 reads the post with id 1 and got it and it's title is "DBMS".

- TXN2 updates the title of the same post to "Golang".

- TXN1 reads the post data again and got the title of "Golang", so it couldn't get the same data twice from the same SELECT query because of non-repeatable read phenomena.

### Phantom read

Phantom read happens when the transaction TXN1 read some rows with a select query and then transaction TXN2 inserts or deletes some rows that matches the where clause of the first select query, so when TXN1 query the same rows with the same where clause it gets a different number of rows, like this example:

{{< figureCupper img="phantom read.jpg" caption="We got additional row in the same transaction from the same select query because another transaction inserted it that's called a phantom read" command="Resize" options="2000x1300" >}}

So  what happened:

- TXN1 starts and reads all the posts that have more than 10 likes.

- TXN2 starts and insert new post with likes = 14.

- TXN1 performs the same SELECT query with the same condition (likes > 10) and got a different number of rows because another transaction made an insert or delete (TXN2) and this phenomena called phantom reads.

### Read skew

Lets assume we have 2 rows related with each other R1 and R2, so read skew happens when TXN1 read R1 and then TXN2 starts running and updates R1 and R2, then TXN1 continue running and reads R2, so in the end TXN1 read the old version of R1 and the new version of R2 which is called read skew phenomena, like this example:

{{< figureCupper img="read skew.jpg" caption="We got the old version from post table and the new version from post_details because TXN2 updated both of them before TXN1 retrieved the row from post_details" command="Resize" options="2000x1300" >}}

### Write skew

lets assume we have 2 rows related with each other R1 and R2, so write skew happens when TXN1 read R1 and R2 then decides to update only one of them but before it updates it TXN2 kicks in and updates R1 and R2 then the update from TXN1 happens so we got one row update from one transaction and the other row update from the other transaction which is called read skew phenomena, like this example:

{{< figureCupper img="write skew.jpg" caption="We got a mix of writes from the 2 transactions for to related rows in different tables so we got the post from TXN2 and the post_details from TXN1" command="Resize" options="2000x1300" >}}

### Lost update

It happens when the first transaction TXN1 reads a value and then TXN2 kicks in and do a blind update to this value then TXN1 do an update based on the old value it read so we have a lost update that TXN2 did. as this example:

{{< figureCupper img="Lost update.jpg" caption="The update from TXN1 lost because it happened while TXN2 is running and TXN2 updated it after TXN1 committed" command="Resize" options="2000x1300" >}}

# Isolation levels

### Read uncommitted

No isolation, reads every uncommitted update.

### Read commited

Read only committed updates even if they are in the middle of the current transaction.

### rebeatable read

transaction only sees the committed changes before the start of the transaction (the updates but new inserted rows from committed transactions in the middle of the primary transaction are seeable).

### serializable

transactions serializable so they don't run in parallel.

* * *

# Locks

there is 2 types of locks:

-   Shared lock
-   Exclusive lock

### Shared lock

used when reading data and any number of transactions that trying to read the same data can aquire it. but if the data is locked by exclusive the shared lock can't be acquired and waits until the exclusive lock released from the data.

### Exclusive lock

an exclusive lock for the data for writing and it can be acquired only by one transaction for the same data. and it can be acquired on any data locked by shared lock until it released.

so reads can happen concurrently by any number of transactions with shared lock by writes happens only by one transaction that holds the exclusive lock and no other transaction can read it in that time.
&lt;&lt;? locks

# what can get wrong?

## dirty reads

## nonrepetable reads

# 2 pahse locking
