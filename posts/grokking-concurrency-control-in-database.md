---
title: "Grokking Concurrency Control in Database"
date: 2022-03-13T13:27:41+02:00
draft: true
---

#what can get wrong?
there are many problems that can happen while running more than one transaction concurrently, some of them mentioned by SQL-92 standerd:

 - Dirty read
 - Non-repeatable read
 - Phantom read

 and some aren't but they happen in the real world:
 
 - Read skew
 - Write skew
 - Lost update
 
 and we will explore them one by one.
 for simplicity sake let's call the transaction that has the problem txn1 and the one causing the problme txn2.

### Dirty read
it's when a transaction txn1 start running and in the middle txn2 starts and changed a value without commiting it,  then txn1 make a query based on the new value which maybe rolled back  by txn2, so txn1 made a decision based on a temporary value. like:
![Dirty read](https://docs.google.com/drawings/d/1fLS5Lz97ihHMdMRVHy5izKEv_LhmN4Swf57nzGpXAE0/edit?usp=sharing) 

### Non-repeatable read
it's when a transaction txn1 start running and reads some data, and then txn2 starts running and change the same data and commits, then txn1 reads the same data  with the same select query and gets different resualt from the first one , so we call this a non-repeatable read, like this example:
![Non-repeatable read](https://docs.google.com/drawings/d/1cT--lkzznh-W3XNl0NPHCYDTPhjvc_i2EIxVG1TH8bg/edit?usp=sharing) 

### Phantom read
phantom read happens when the transaction txn1 read some rows with a select query and then transaction txn2 inserts or deletes some rows that matches the where cluase of the first select query, so when txn1 query the same rows with the same where cluase it gets a different number of rows, like this example:
![Phantom read](https://docs.google.com/drawings/d/1ThtxypxFtz2M6fmZAl1bN3ArbYo1zXGwlHzY6PBcskk/edit?usp=sharing)
### Read skew
lets assume we have 2 rows related with each other R1 and R2, so read skew happens when txn1 read R1 and then txn2 starts running and updates R1 and R2, then txn1 continue running and reads R2, so in the end txn1 read the old version of R1 and the new version of R2 which is called read skew phenomena, like this example:
![Read skew](https://docs.google.com/drawings/d/1rGOJfvIC19WNzi7HTGQW45HFjppUwBBotAxKABZu-OI/edit?usp=sharing)
### Write skew


# Isolation levels

### Read uncommited
No isolation, reads every uncommited update.

### Read commited
Read only commited updates even if they are in the middle of the current transaction.

### rebeatable read
transaction only sees the commited changes before the start of the transaction (the updates but new inserted rows from commited transactions in the middle of the primery transaction are seeable).

### serializable
transactions serializable so they don't run in parallel.


----------------



﻿# locks

there is 2 types of locks:
 - Shared lock
 - Exclusive lock

### Shared lock
used when reading data and any number of transactions that trying to read the same data can aquire it. but if the data is locked by exclusive the shared lock can't be aquired and waits until the exclusive lock realesed from the data.

### Exclusive lock
an exclusive lock for the data for writing and it can be aquired only by one transaction for the same data. and it can be aquired on any data locked by shared lock until it released.

so reads can happen concurrently by any number of transactions with shared lock by writes happens only by one transaction that holds the exclusive lock and no other transaction can read it in that time. 
<<? locks

# what can get wrong?
## dirty reads
## nonrepetable reads
# 2 pahse locking
