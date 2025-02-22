+++
title = "mnist and the idx file format"
author = ["Nilay Kumar"]
date = 2022-03-05T00:00:00-05:00
lastmod = 2025-02-21T23:04:57-05:00
draft = false
mathjax = true
+++

Boser, Guyon, and Vapnik (BGV) introduce support vector machines (SVMs) in
[<a href="#citeproc_bib_item_1">1</a>] and their immediate application is that of optical character
recognition (OCR). More specifically, given images of handwritten numerals, they
train a classifier to categorize a given image by digit. BGV show that SVMs can
achieve performance comparable to that of a more sophisticated five-layer neural
network classifier. I've been working through their paper to learn about SVMs,
and I thought it might be fun to try to follow along. There is a fairly large
dataset of handwritten numerals due to LeCun, Cortes, and Burges [<a href="#citeproc_bib_item_2">2</a>],
called the MNIST database that's publicly available. What follows are my notes
(mostly for my future self) on how to get started working with this dataset.


## the `idx` file format {#the-idx-file-format}

At the time of writing, unfortunately, the website hosting the MNIST database
and the relevant documentation seems to be locked behind a authentication wall.
The dataset's popular enough, though, that you can find it with a quick internet
search. Once you've downloaded the dataset (and uncompressed it), you should
have 4 files: 2 files for training data and 2 files for testing data.

```sh
ls -lh data/mnist/
```

```text
total 53M
-rw-r--r-- 1 nilay nilay 7.5M Jun 17 17:31 t10k-images.idx3-ubyte
-rw-r--r-- 1 nilay nilay 9.8K Jun 17 17:31 t10k-labels.idx1-ubyte
-rw-r--r-- 1 nilay nilay  45M Jun 17 17:31 train-images.idx3-ubyte
-rw-r--r-- 1 nilay nilay  59K Jun 17 17:31 train-labels.idx1-ubyte
```

You'll notice immediately that the data is stored in a slightly idiosyncratic
manner. A cursory `head data/train-images.idx3-ubyte` will show that we're
working with binary blobs, not images or comma-separated values.[^fn:1] A bit of
elbow grease is necessary, then, in order to work with this dataset. This short
post is an overview of how to work with this file format. If you'd like a quick
and easy solution, I'd suggest taking a look at `idx2numpy` project [<a href="#citeproc_bib_item_3">3</a>].
I found this project halfway through writing up my own code, but it's a pretty
straightforward exercise regardless.

The `idx` format is used to store data that has the shape of an
\\(d\\)-dimensional matrix.[^fn:2] In the files above, the digit following `.idx`
indicates the dimension \\(d\\) and the `ubyte` indicates that the data (the matrix
entries) will be unsigned bytes. Let's fix a bit of notation: bytes can take
values between `\x00` and `\xff`. That is, a byte consists of two hexadecimal
digits. Since each digit can take \\(16=2^4\\) values, two hex digits gives us \\(16\times
16=256=2^8\\) values, which is exactly 8 bits = 1 byte.[^fn:3] The dimension and
the datatype are actually described in the binary, so the filenames are just a
helpful reminder.

An `idx` file is structured as follows (all multibyte sequences are big-endian
[<a href="#citeproc_bib_item_4">4</a>]):

```text
| \x00 | \x00 | dtype | d | n_1 | ... | n_d |

| a_{1, 1, ..., 1}   | ... | a_{1, 1, ..., 1, n_d}   | ... | a_{1, n_2, ..., n_d}   |
| .                  |     | .                       |     | .                      |
| .                  |     | .                       |     | .                      |
| .                  |     | .                       |     | .                      |
| a_{n_1, 1, ..., 1} | ... | a_{n_1, 1, ..., 1, n_d} | ... | a_{n_1, n_2, ..., n_d} |
```

I've written this out as a table, but remember that this is a binary file so the
newlines/formatting here are just for readability. Let's break down what's going
on here:

-   the first two bytes are always zero, `\x00`.[^fn:4]
-   the third byte `dtype` is the datatype:
    ```text
      | value | type          | size (bytes) |
      |-------+---------------+--------------|
      | \x08  | unsigned byte |            1 |
      | \x09  | signed byte   |            1 |
      | \x0b  | short         |            2 |
      | \x0c  | int           |            4 |
      | \x0d  | float         |            4 |
      | \x0e  | double        |            8 |
    ```
    We'll be using unsigned bytes, so we expect to see `\x08`.
-   the fourth byte `d` is the dimension of the data matrix, and the following
    \\(4d\\) bytes are 4-byte integers \\(n\_1,...,n\_d\\).
-   the rest of the file stores the \\(d\\)-dimensional data matrix, flattened to a
    2-dimensional matrix as the notation above indicates. Each cell has size
    determined by the `dtype` above.

To make things more concrete, let's take a look at the MNIST data.


## the mnist images {#the-mnist-images}

There are a number of standard utilities for displaying the contents of a binary
file as hexadecimal bytes. We'll use `hexdump` to display the first 16 bytes (in
the "canonical" format, hence the `-C` flag):

```sh
hexdump -C --length=16 data/mnist/train-images.idx3-ubyte
```

```text
00000000  00 00 08 03 00 00 ea 60  00 00 00 1c 00 00 00 1c  |.......`........|
00000010
```

The column on the left is a column of hexadecimal offsets (essentially line
numbers), so the first row is displaying the first 16 bytes of our file. The
second row of 16 bytes (represented by the hex `10`) is empty, because we've
only queried 16 bytes with the `--length=16` flag. The 16 middle columns are
what we're after. We'll return to those in a moment. The last column is an
attempt at ASCII representations of the bytes. This is irrelevant for us, as
we're not expecting any plaintext strings in our file, so you can ignore this
column.

The first two bytes are 0, as required by the `idx` format. The next byte `\x08`
tells us that the elements of the data matrix appearing later will be 1-byte
each, to be interpreted directly as unsigned integers between 0 and 255. The
fourth byte `\x03` tells us that the data matrix is 3-dimensional. What are the
dimensions \\(n\_1, n\_2, n\_3\\)? We check the next 12 bytes: `\x0000ea60`,
`\x0000001c`, and `\x0000001c`. Converting these to decimal using the calculator
`bc`,

```sh
echo "ibase=16; EA60; 1C" | bc
```

```text
60000
28
```

we find that

\begin{equation\*}
n\_1 = 60000, \qquad n\_2 = n\_3 = 28.
\end{equation\*}

This means that we're dealing with a \\((60000 \times 28 \times 28)\\) matrix. We have 60000
training images, with each square image having \\(28 \times 28 = 784\\) pixels, and with
each pixel colored in grayscale from 0 (white) to 255 (black). According to the
format specification above, all this data is stored in the file as a \\((60000 \times
784)\\) matrix of bytes. The \\(k\\)th row represents the \\(k\\)th image --- the
\\(k\\)th feature vector.

The first image consists of the subsequent 784 bytes:

```sh
hexdump -C --length=784 --skip=16 data/mnist/train-images.idx3-ubyte
```

```text
00000010  00 00 00 00 00 00 00 00  00 00 00 00 00 00 00 00  |................|
*
000000a0  00 00 00 00 00 00 00 00  03 12 12 12 7e 88 af 1a  |............~...|
000000b0  a6 ff f7 7f 00 00 00 00  00 00 00 00 00 00 00 00  |................|
000000c0  1e 24 5e 9a aa fd fd fd  fd fd e1 ac fd f2 c3 40  |.$^............@|
000000d0  00 00 00 00 00 00 00 00  00 00 00 31 ee fd fd fd  |...........1....|
000000e0  fd fd fd fd fd fb 5d 52  52 38 27 00 00 00 00 00  |......]RR8'.....|
000000f0  00 00 00 00 00 00 00 12  db fd fd fd fd fd c6 b6  |................|
00000100  f7 f1 00 00 00 00 00 00  00 00 00 00 00 00 00 00  |................|
00000110  00 00 00 00 50 9c 6b fd  fd cd 0b 00 2b 9a 00 00  |....P.k.....+...|
00000120  00 00 00 00 00 00 00 00  00 00 00 00 00 00 00 00  |................|
00000130  00 0e 01 9a fd 5a 00 00  00 00 00 00 00 00 00 00  |.....Z..........|
00000140  00 00 00 00 00 00 00 00  00 00 00 00 00 00 00 8b  |................|
00000150  fd be 02 00 00 00 00 00  00 00 00 00 00 00 00 00  |................|
00000160  00 00 00 00 00 00 00 00  00 00 00 0b be fd 46 00  |..............F.|
00000170  00 00 00 00 00 00 00 00  00 00 00 00 00 00 00 00  |................|
00000180  00 00 00 00 00 00 00 00  23 f1 e1 a0 6c 01 00 00  |........#...l...|
00000190  00 00 00 00 00 00 00 00  00 00 00 00 00 00 00 00  |................|
000001a0  00 00 00 00 00 51 f0 fd  fd 77 19 00 00 00 00 00  |.....Q...w......|
000001b0  00 00 00 00 00 00 00 00  00 00 00 00 00 00 00 00  |................|
000001c0  00 00 2d ba fd fd 96 1b  00 00 00 00 00 00 00 00  |..-.............|
000001d0  00 00 00 00 00 00 00 00  00 00 00 00 00 00 00 10  |................|
000001e0  5d fc fd bb 00 00 00 00  00 00 00 00 00 00 00 00  |]...............|
000001f0  00 00 00 00 00 00 00 00  00 00 00 00 00 f9 fd f9  |................|
00000200  40 00 00 00 00 00 00 00  00 00 00 00 00 00 00 00  |@...............|
00000210  00 00 00 00 00 00 2e 82  b7 fd fd cf 02 00 00 00  |................|
00000220  00 00 00 00 00 00 00 00  00 00 00 00 00 00 00 00  |................|
00000230  27 94 e5 fd fd fd fa b6  00 00 00 00 00 00 00 00  |'...............|
00000240  00 00 00 00 00 00 00 00  00 00 18 72 dd fd fd fd  |...........r....|
00000250  fd c9 4e 00 00 00 00 00  00 00 00 00 00 00 00 00  |..N.............|
00000260  00 00 00 00 17 42 d5 fd  fd fd fd c6 51 02 00 00  |.....B......Q...|
00000270  00 00 00 00 00 00 00 00  00 00 00 00 00 00 12 ab  |................|
00000280  db fd fd fd fd c3 50 09  00 00 00 00 00 00 00 00  |......P.........|
00000290  00 00 00 00 00 00 00 00  37 ac e2 fd fd fd fd f4  |........7.......|
000002a0  85 0b 00 00 00 00 00 00  00 00 00 00 00 00 00 00  |................|
000002b0  00 00 00 00 88 fd fd fd  d4 87 84 10 00 00 00 00  |................|
000002c0  00 00 00 00 00 00 00 00  00 00 00 00 00 00 00 00  |................|
*
00000320
```

You'll notice that a number of bytes are omitted because the offsets jump from
`\x10` to `\xa0`. The asterisk `*` denotes that the omitted bytes are identical
(in \\(16 \times 16\\) chunks) to the 16 bytes displayed on the previous line.[^fn:5] This
isn't surprising in the case of handwritten digits: we expect a good chunk of
whitespace towards the top and bottom of the image. It's clearly not easy to
visualize the digit just from this hexdump. We'll write some python code to help
us visualize them later, after we've loaded the images into numpy arrays.

First, though, we'll look at the labels provided for learning supervision. Just
as before, let's look into the training labels.

```sh
hexdump -C --length=8 data/mnist/train-labels.idx1-ubyte
```

```text
00000000  00 00 08 01 00 00 ea 60                           |.......`|
00000008
```

The first 8 bytes tell us that this file contains a vector of length 60000,
whose elements are unsigned bytes. Let's take a look at the first 16 elements:

```sh
hexdump -C --length=16 --skip=8 data/mnist/train-labels.idx1-ubyte
```

```text
00000008  05 00 04 01 09 02 01 03  01 04 03 05 03 06 01 07  |................|
00000018
```

As expected for labels indicating digits, all these values are between 0 and 9.

Great, now that we understand how the data is stored, we can get stop messing
around in the command line and load the data into our python scripts.


## reading images into numpy arrays {#reading-images-into-numpy-arrays}

The python implementation of loading the MNIST images is actually pretty
straightforward, now that we know exactly what our datafiles look like. We're
**not** going to write a program to take an arbitrary `idx` datafile and convert
it to a numpy array because that's overkill (just use `idx2numpy`!). We'll stick
to our specific case. That means we just need to read 60000 blocks of size 784
bytes, each (as well as the associated labels). Well okay, let's just read, say,
the first \\(N=2\\) images and the first two labels, because the extension to
general N will be clear.

We make sure to open the datafile for binary reading with the `rb` flag. This
allows us to read data from the file into a `bytes` object, which stores an
immutable sequence of bytes. We throw away the first 16 bytes that we analyzed
above. As it happens, numpy has a super convenient `frombuffer` method (I
learned about this by peeking at `idx2numpy`'s [code](https://github.com/ivanyu/idx2numpy/blob/master/idx2numpy/converters.py#L116)). This function takes any
object that exposes "the buffer interface" and produces from it a numpy array.
This was my first time hearing about python's buffer protocol, but I'll banish
my short description of it to the footnotes, since it's a bit off-topic.[^fn:6]
The upshot is that we can pass in our sequence of bytes and get back out a
vector of size 784. All that's left, then, is to `.reshape((28, 28))` to get it
into a nice square matrix, and follow that up with a bit of custom pretty
printing.

```python
import numpy as np
# let's look at the first two images
N = 2
# 'rb' means that we're reading bytes
with open('data/mnist/train-images.idx3-ubyte', 'rb') as f:
    # discard the first 16 bytes
    f.read(16)
    for n in range(N):
        # read the next 784 bytes into a numpy array and reshape
        image = np.frombuffer(f.read(784), dtype = np.ubyte).reshape((28, 28))

        # draw in ['.', 'o', 'O', '@'] depending on the intensity of a pixel
        for i in range(28):
            for j in range(28):
                if image[i][j] == 0:
                    print('.', end='')
                elif image[i][j] < 64:
                    print('o', end='')
                elif image[i][j] < 128:
                    print('O', end='')
                else:
                    print('@', end='')
            print()
        print()
```

```text
............................
............................
............................
............................
............................
............ooooO@@o@@@O....
........ooO@@@@@@@@@@@@O....
.......o@@@@@@@@@@OOOoo.....
.......o@@@@@@@@@@..........
........O@O@@@o.o@..........
.........oo@@O..............
...........@@@o.............
...........o@@O.............
............o@@@Oo..........
.............O@@@Oo.........
..............o@@@@o........
...............oO@@@........
.................@@@O.......
..............o@@@@@o.......
............o@@@@@@@........
..........oO@@@@@@O.........
........oO@@@@@@Oo..........
......o@@@@@@@Oo............
....o@@@@@@@@o..............
....@@@@@@@o................
............................
............................
............................

............................
............................
............................
............................
...............o@@@o........
..............o@@@@@........
.............o@@@@@@oo......
...........oo@@@@@O@@O......
...........@@@@@@@O@@@......
..........o@@@@O@@oO@@......
.........o@@@@oOOo..@@o.....
........o@@@@O......@@@.....
.......o@@@Ooo......@@@.....
.......o@@o.........@@@.....
.......@@@..........@@@.....
......O@@O..........@@@.....
......O@@o........o@@@o.....
......O@@........o@@@O......
......O@@.......o@@@........
......O@@......O@@@.........
......O@@@ooO@@@@@o.........
......O@@@@@@@@@@...........
......o@@@@@@@@.............
.......o@@@@@o..............
............................
............................
............................
............................

```

That looks like a 5 and a 0 to me. Let's double check by loading in the first
two training labels, which involves more or less the same code as above.

```python
import numpy as np
# let's look at the first two labels
N = 2
# 'rb' means that we're reading bytes
with open('data/mnist/train-labels.idx1-ubyte', 'rb') as f:
    # discard the first 8 bytes
    f.read(8)
    # read the next byte into a numpy array (corresponding to the first label)
    labels = np.frombuffer(f.read(N), dtype = np.ubyte)
    print(labels)
```

```text
[5 0]
```

Looks good!

Well, that about wraps up this post. We haven't looked at the testing data in
the remaining two files, but those are treated exactly the same. The only
difference there is that the testing set is comprised of 10000 images instead of
60000, so there's not much to change.

<hr>

<style>.csl-left-margin{float: left; padding-right: 0em;}
 .csl-right-inline{margin: 0 0 0 1em;}</style><div class="csl-bib-body">
  <div class="csl-entry"><a id="citeproc_bib_item_1"></a>
    <div class="csl-left-margin">[1]</div><div class="csl-right-inline"> <span style="font-variant:small-caps;">Boser</span>, B. E., <span style="font-variant:small-caps;">Guyon</span>, I. M. and <span style="font-variant:small-caps;">Vapnik</span>, V. N. (1992). <a href="https://doi.org/10.1145/130385.130401">A Training Algorithm for Optimal Margin Classifiers</a>. In <i>Proceedings of the Fifth Annual Workshop on Computational Learning Theory</i> COLT ’92 pp 144–52. Association for Computing Machinery, Pittsburgh, Pennsylvania, USA.</div>
  </div>
  <div class="csl-entry"><a id="citeproc_bib_item_2"></a>
    <div class="csl-left-margin">[2]</div><div class="csl-right-inline"> <span style="font-variant:small-caps;">LeCun</span>, Y., <span style="font-variant:small-caps;">Cortes</span>, C. and <span style="font-variant:small-caps;">Burges</span>, C. (2010). <a href="http://yann.lecun.com/exdb/mnist/">MNIST handwritten digit database</a>.</div>
  </div>
  <div class="csl-entry"><a id="citeproc_bib_item_3"></a>
    <div class="csl-left-margin">[3]</div><div class="csl-right-inline"> <span style="font-variant:small-caps;">Yurchenko</span>, I. (2020). <a href="https://github.com/ivanyu/idx2numpy">idx2numpy</a>. <i>GitHub repository</i>.</div>
  </div>
  <div class="csl-entry"><a id="citeproc_bib_item_4"></a>
    <div class="csl-left-margin">[4]</div><div class="csl-right-inline"> <span style="font-variant:small-caps;">Wikipedia contributors</span>. (2022). <a href="https://en.wikipedia.org/w/index.php?title=Endianness&oldid=1073675969">Endianness –- Wikipedia, The Free Encyclopedia</a>.</div>
  </div>
  <div class="csl-entry"><a id="citeproc_bib_item_5"></a>
    <div class="csl-left-margin">[5]</div><div class="csl-right-inline"> <span style="font-variant:small-caps;">Oliphant</span>, T. and <span style="font-variant:small-caps;">Banks</span>, C. (2006). <i><a href="https://www.python.org/dev/peps/pep-3118/">Revising the buffer protocol</a></i>.</div>
  </div>
</div>

[^fn:1]: This is probably done to keep file sizes down. If the numerals were
    stored as images, the dataset would likely be much larger in size. Moreover,
    since the numerals will have large numbers of white pixels, I would guess that
    this binary format, which stores white pixels as `\x00`, yields relatively high
    compression ratios.
[^fn:2]: What is `idx` short for? Image Data matriX? Incredibly Dense Xylophones?
[^fn:3]: Notice that a signed byte uses one bit of information to store the sign,
    and hence can only store integers in \\((-2^7, 2^7)\\). Here we're interested in
    keeping track of how dark a pixel is, for which it's perfectly convenient to
    just use an unsigned byte.
[^fn:4]: Why include the first two bytes if they're always zero? I think it's
    because the first two bytes of a file are often used as a file signature to
    indicate what the filetype is. The creators of the `idx` format were probably
    just adhering to this convention, and indicating that the file is not in a
    commonly used format.
[^fn:5]: You can force `hexdump` to show all bytes explicitly by passing it the
    no-squeezing flag `--no-squeezing`, or `-v` for short.
[^fn:6]: Straight from PEP 3118 [<a href="#citeproc_bib_item_5">5</a>]: the python buffer protocol "allows
    different Python types to exchange a pointer to a sequence of internal buffers.
    This functionality is _extremely_ useful for sharing large segments of memory
    between different high-level objects..." This seems to be quite important for
    heavily performance-reliant code (say in numpy) or code that interacts with
    older legacy software packages written in C or Fortran.
