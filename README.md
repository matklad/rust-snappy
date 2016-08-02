snap
====
A pure Rust implementation of the
[Snappy compression algorithm](http://google.github.io/snappy/).
Includes streaming compression and decompression using the Snappy frame format.
This implementation is ported from both the
[reference C++ implementation](https://github.com/google/snappy)
and the
[Go implementation](https://github.com/golang/snappy).

[![Linux build status](https://api.travis-ci.org/BurntSushi/snap.png)](https://travis-ci.org/BurntSushi/snap)
[![Windows build status](https://ci.appveyor.com/api/projects/status/github/BurntSushi/snap?svg=true)](https://ci.appveyor.com/project/BurntSushi/snap)
[![](http://meritbadge.herokuapp.com/snap)](https://crates.io/crates/snap)

Dual-licensed under MIT or the [UNLICENSE](http://unlicense.org).

### Documentation

[http://burntsushi.net/rustdoc/snap/](http://burntsushi.net/rustdoc/snap/)

### Usage

Add this to your `Cargo.toml`:

```toml
[dependencies]
snap = "0.1"
```

and this to your crate root:

```rust
extern crate snap;
```

### Example: compress data on `stdin`

This program reads data from `stdin`, compresses it and emits it to `stdout`.
This example can be found in `examples/compress.rs`:

```rust
extern crate snap;

use std::io;

fn main() {
    let stdin = io::stdin();
    let stdout = io::stdout();

    let mut rdr = stdin.lock();
    // Wrap the stdout writer in a Snappy writer.
    let mut wtr = snap::Writer::new(stdout.lock());
    io::copy(&mut rdr, &mut wtr).expect("I/O operation failed");
}
```

### Example: decompress data on `stdin`

This program reads data from `stdin`, decompresses it and emits it to `stdout`.
This example can be found in `examples/decompress.rs`:

```rust
extern crate snap;

use std::io;

fn main() {
    let stdin = io::stdin();
    let stdout = io::stdout();

    // Wrap the stdin reader in a Snappy reader.
    let mut rdr = snap::Reader::new(stdin.lock());
    let mut wtr = stdout.lock();
    io::copy(&mut rdr, &mut wtr).expect("I/O operation failed");
}
```

### Example: the szip tool

`szip` is a tool with similar behavior as `gzip`, except it uses Snappy
compression. It can be installed with Cargo:

```
$ cargo install szip
```

To compress a file, run `szip file`. To decompress a file, run `szip -d
file.sz`. See `szip --help` for more details.

### Testing

This crate is tested against the reference C++ implementation of Snappy.
Currently, compression is byte-for-byte equivalent with the C++ implementation.
This seems like a reasonable starting point, although it is not necessarily
a goal to always maintain byte-for-byte equivalence.

Tests against the reference C++ implementation can be run with
`cargo test --features cpp`. Note that you will need to have the C++ Snappy
library in your `LD_LIBRARY_PATH` (or equivalent).

### Performance

The performance of this implementation should roughly match the performance of
the C++ implementation on x86_64. Below are the results of the microbenchmarks
(as defined in the C++ library):

```
name             cpp ns/iter           rust ns/iter         diff ns/iter   diff %
uflat00_html     45,097 (2270 MB/s)    44,372 (2307 MB/s)           -725   -1.61%
uflat01_urls     496,988 (1412 MB/s)   475,693 (1475 MB/s)       -21,295   -4.28%
uflat02_jpg      4,800 (25644 MB/s)    4,935 (24942 MB/s)            135    2.81%
uflat03_jpg_200  144 (1388 MB/s)       127 (1574 MB/s)               -17  -11.81%
uflat04_pdf      6,699 (15285 MB/s)    6,586 (15548 MB/s)           -113   -1.69%
uflat05_html4    187,082 (2189 MB/s)   184,941 (2214 MB/s)        -2,141   -1.14%
uflat06_txt1     152,245 (998 MB/s)    152,185 (999 MB/s)            -60   -0.04%
uflat07_txt2     134,235 (932 MB/s)    135,057 (926 MB/s)            822    0.61%
uflat08_txt3     407,234 (1047 MB/s)   418,990 (1018 MB/s)        11,756    2.89%
uflat09_txt4     563,671 (854 MB/s)    580,281 (830 MB/s)         16,610    2.95%
uflat10_pb       42,207 (2809 MB/s)    41,624 (2849 MB/s)           -583   -1.38%
uflat11_gaviota  159,276 (1157 MB/s)   153,006 (1204 MB/s)        -6,270   -3.94%
zflat00_html     108,043 (947 MB/s)    104,306 (981 MB/s)         -3,737   -3.46%
zflat01_urls     1,416,005 (495 MB/s)  1,305,846 (537 MB/s)     -110,159   -7.78%
zflat02_jpg      8,260 (14902 MB/s)    8,372 (14702 MB/s)            112    1.36%
zflat03_jpg_200  329 (607 MB/s)        247 (809 MB/s)                -82  -24.92%
zflat04_pdf      12,279 (8339 MB/s)    11,351 (9021 MB/s)           -928   -7.56%
zflat05_html4    465,677 (879 MB/s)    448,619 (913 MB/s)        -17,058   -3.66%
zflat06_txt1     461,344 (329 MB/s)    442,385 (343 MB/s)        -18,959   -4.11%
zflat07_txt2     409,416 (305 MB/s)    393,293 (318 MB/s)        -16,123   -3.94%
zflat08_txt3     1,194,880 (357 MB/s)  1,178,756 (362 MB/s)      -16,124   -1.35%
zflat09_txt4     1,638,914 (294 MB/s)  1,614,618 (298 MB/s)      -24,296   -1.48%
zflat10_pb       100,514 (1179 MB/s)   97,523 (1216 MB/s)         -2,991   -2.98%
zflat11_gaviota  358,002 (514 MB/s)    326,086 (565 MB/s)        -31,916   -8.92%
```

Notes: These benchmarks were run with Snappy/C++ on commit `32d6d7` with debug
assertions disabled. Both the C++ and Rust benchmarks were run with the same
benchmark harness. Benchmarks were run on an Intel i7-6900K.
