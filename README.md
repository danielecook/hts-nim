hts-nim
=======

[![badge](https://img.shields.io/badge/docs-latest-blue.svg)](https://brentp.github.io/hts-nim/)


This is a wrapper for [htslib](https://github.com/samtools/htslib) in [nim](https://nim-lang.org). 

Nim is a fast, garbage collected language that compiles to C and has a syntax that's not
too different to python.

Here are examples of the syntax in this library:

## BAM

```nim
import hts/bam

# open a bam and look for the index.
var b:Bam
assert open(b, "test/HG02002.bam", index=true)

for record in b:
  if record.qual > 10:
    echo record.chrom, record.start, record.stop

# regional queries:
for record in b.query('6', 30816675, 32816675):
  if record.flag.proper_pair and record.flag.reverse:
    # cigar is an iterable of operations:
    for op in record.cigar:
      # $op gives the string repr of the operation, e.g. '151M'
      echo $op, op.consumes.reference, op.consumes.query

    # tags are pulled with `aux`
    var mismatches = rec.aux("NM")
    if mismatches != nil and mismatches.integer() < 3:
      var rg = rec.aux("RG")
      echo rg.tostring()

# cram requires an fasta to decode:
var cram:Bam
open_hts(cram, "/tmp/t.cram", fai="/data/human/g1k_v37_decoy.fa")
for record in cram:
  # now record is same as from bam above
  echo record.qname, record.isize
```

## VCF/BCF

```nim
import hts/vcf

var tsamples = @["101976-101976", "100920-100920", "100231-100231", "100232-100232", "100919-100919"]

# VCF and BCF supported
var v:VCF
assert open(v, "tests/test.bcf", samples=tsamples)

for rec in v:
  echo rec, " qual:", rec.QUAL, " filter:", rec.FILTER

echo v.samples

# regional queries look for index. worsk for VCF and BCF
for rec in v.query("1:15600-18250"):
  echo rec.CHROM, ":", $rec.POS
```

## bgzip with csi

A nice thing that is facilitated with this library is creating a .csi index while writing sorted
intervals to a file.  This can be done as:

```nim
import hts
# arguments are 1-based seq-col, start-col, end-col and whether the intervals are 0-based.
var bx = wopen_bgzi("ti.txt.gz", 1, 2, 3, true)
# some duplication of args to avoid re-parsing. args are line, chrom, start, end
check bx.write_interval("a\t4\t10", "a", 4, 10) > 0
check bx.write_interval("b\t2\t20", "b", 2, 20) > 0
check bx.close() == 0
```

After this, `ti.txt.gz.csi` will be usable by tabix.
