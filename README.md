# CS 4173 Lab 3
#### MD5 Collision Attack Lab

## Prerequisites
Versions of software used listed here:
- Environment: Ubuntu 16.04.4 LTS, Windows 10 Powershell
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

Note: I am again using the Windows binary to generate hash collision generating binaries. 

In this task we will generate two executables that have the same MD5 hash using the properties we examined previously. First, we will put the given program into `program.c` and compile it into `program`:

```bash
$ make program.c
cc     program.c   -o program
```

Inspecting this `program` binary, we can see that the first multiple of 64 bytes which starts in the array is at the `0x107F` position, or 4223rd byte. Therefore, we can use `head` to make our prefix to generate our two binaries with colliding hashes:

```bash
$ head -c 4224 program > prefix.bin
$ ./md5collgen -p prefix.bin -o P.bin Q.bin
```

Inspecting these outputs, we can see that they end at the `0x1100` position, or 4352nd byte, so we can start from there for our suffix then build our final executables:

```bash
$ tail -c +4352 program > suffix.bin
$ cat ../P.bin suffix.bin > out1.bin
$ cat ../Q.bin suffix.bin > out2.bin
$ ./out1.bin
4141414141414141414141414141414141414141414141414141414141414141e7b07c4ffc1614b1cd4e89a58bdbf1ebc0c6b7ab3bb8e03be4683428c7d2cefe1ebfc4486d46e2ebff4f3b40ea882df79b264cb48ee65c61ce6966e973b9dda4862bb7983a22a96ed2013256b99cc37716b167f881c16c59df6cc7781aded6fb66f1fb48125d644c9699c7c451f1f164f39dc85389c19bc7f924ea9c241414141414141414141414141414141414141414141414141414141414141414141414141414141
$ ./out2.bin
4141414141414141414141414141414141414141414141414141414141414141e7b07c4ffc1614b1cd4e89a58bdbf1ebc0c6b72b3bb8e03be4683428c7d2cefe1ebfc4486d46e2ebff4f3b406a892df79b264cb48ee65c61ce69e6e973b9dda4862bb7983a22a96ed2013256b99cc37716b967f881c16c59df6cc7781aded6fb66f1fb48125d644c9699cfc441f1f164f39dc85389c19bc77924ea9c241414141414141414141414141414141414141414141414141414141414141414141414141414141
```

We verify that the resulting binaries that are pieced together indeed are able to be executed and we can verify that the contents are different if we save the outputs and run a `diff` on them. 

```bash
$ ./out1.bin > 1
$ ./out2.bin > 2
$ diff 1 2
```

Since our outputs are on single lines it can be hard to determine the location of the differences from the output, but `diff` indeed confirms they are different. We can still observe the sequence of `0x41` in the prefix and suffix which tells us that we successfully pieced the binaries together. Finally, we can test whether the MD5 hashes are the same:

```bash
$ md5sum out1.bin
535713123dd6aa0d44e6c2214332fe3b  out1.bin
$ md5sum out2.bin
535713123dd6aa0d44e6c2214332fe3b  out2.bin
```

We can see that they are indeed the same, meaning that the property we examined in the previous task holds true. 