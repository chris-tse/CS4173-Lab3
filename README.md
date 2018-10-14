# CS 4173 Lab 3
#### MD5 Collision Attack Lab

## Prerequisites
Versions of software used listed here:
- Environment: Ubuntu 16.04.4 LTS
- Environment for Task 1: Windows 10 Powershell
- `diff`: 3.3
- `md5sum`: 8.25

## Task 1: Generate Encryption Key in a Wrong Way

In this task, we use the provided tool to generate two binary files which result in the same hash when using the MD5 hashing algorithm. Since I am not able to use the collision tool I used the Windows binary version through Powershell for generating the outputs. By observing the output, we can see that the prefix is appended to the output binary as a 64 length block. There are 3 prefix files included: `63longprefex.txt`, `64longprefex.txt`, and `65longprefex.txt`. These were used to test the outputs when the prefix is a multiple of 64, less than a multiple of 64, or greater than a multiple of 64. 

### Question 1
If the length of the prefix is a multiple of 64, it will fit perfectly at the start of the output. Otherwise, if it is shorter than 64, the prefix will be padded with hex `00`s until it is. If it is longer than a multiple of 64, a new 64 length block will be used, made up of the remainder of the prefix and padded with hex `00`s. 

### Question 2
Using the `64longprefex.txt` file as the prefix, we see in the output that the hash contents begin immediately after the last byte of the prefix.

### Question 3
For the two binary files generated which create an MD5 collision, they are not completely different. Only 7 bytes in total are different between any pair of outputs that have a hash collision. These are the bytes in the 20th, 109th, 110th, 123rd, 147th, 173rd, and 187th position after the prefix padding. This is the same for every pair of output files regardless of the length or contents of the padding. 

## Task 2: Understanding MD5’s Property 

In this task, we will perform an experiment to test the property of the MD5 hash given in the lab assignment. For testing purposes, input M will be `out1.bin` and N will be `out2.bin` from the previous task. In our first test, we will perform the test using M = T:

```bash
$ cp M.bin T.bin
$ cat M.bin T.bin > MT1.bin
$ cat N.bin T.bin > NT1.bin
$ md5sum MT1.bin
bdfedcd5770d65eac6ccfa4820c7847e  MT1.bin
$ md5sum NT1.bin
bdfedcd5770d65eac6ccfa4820c7847e  NT1.bin
```

We can repeat this a second time with N = T:

```
$ rm T.bin
$ cp N.bin T.bin
$ cat M.bin T.bin > MT2.bin
$ cat N.bin T.bin > NT2.bin
$ md5sum MT1.bin
40096b10eedccd140cc4a10112cbcc76  MT2.bin
$ md5sum NT1.bin
40096b10eedccd140cc4a10112cbcc76  NT2.bin
```

We can also generate a new binary file with random bytes using output from `/dev/urandom` as T for our last test (included in submitted files):

```
$ rm T.bin
$ dd if=/dev/urandom of=T.bin bs=1 count=30
$ cat M.bin T.bin > MT3.bin
$ cat N.bin T.bin > NT3.bin
$ md5sum MT3.bin
87e2ae9ae1577d0bea15bad4b600e425  MT3.bin
$ md5sum NT3.bin
87e2ae9ae1577d0bea15bad4b600e425  NT3.bin
```

## Task 3: Generating Two Executable Files with the Same MD5 Hash