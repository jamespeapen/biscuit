---
title: Read Mapping
nav_order: 2
has_children: true
permalink: docs/alignment
---

# Read Mapping
{: .no_toc }

## Table of Content
{: .no_toc .text-delta }

1. TOC
{:toc}

### Index reference for mapping

```bash
biscuit index GRCh38.fa
```

The index of BISCUIT composed of the 2-bit packed reference
(`.bis.pac`, `.bis.amb`, `.bis.ann`). The suffix array and
FM-index of the parent strand (`.par.bwt` and `.par.sa`) and
the daughter strand (`.dau.bwt` and `.dau.sa`).

### Read mapping

The following snippet shows how BISCUIT can be used in conjunction
with [samtools](https://github.com/samtools/samtools) to produce
indexed alignment BAM file.
```bash
$ biscuit align -t 10 GRCh38.fa fastq1.fq.gz fastq2.fq.gz | 
    samtools sort -T . -O bam -o output.bam
$ samtools index output.bam
```

### Map one sequence

One does not need to prepare a fastq file to map the reads. The `-1`
option let you map one read on the fly.

```bash
$ biscuit align GRCh38.fa -1 AATTGGCC
```

One can also just map one pair of reads with an extra `-2` option.

### Which strand to map?

Depending on the library construction strategy, bisulfite-converted
reads can come from one of the four types of strands, depending on
whether the targeted strand is the Waston or Crick strand and whether
the DNA is the original strand that underwent bisulfite conversion
(__parent strand__), or the synthesized strand during PCR
amplification (__daughter strand__). Understanding which strand reads
come from or can be generated is critical to proper use of a read
mapper. In BISCUIT, whether mapping occurs at parent or daughter
strand is controled using `-b` option.

#### Single-End library

By default, BISCUIT map read to both strands (`-b 0`). This works for
most cases including conventional library (such as TCGA WGBS and
Roadmap Epigenome Project) and PBAT library and single cell libraries
(where tagging happens after amplification). When `-b 1` is specified,
reads are forced to map to parent strands. This is the behavior with
the conventional libraries (such as from TCGA and Roadmap
Project). `-b 1` makes mapping more efficient and less error-prone for
the conventional libraries. `-b 3` (rarely used) forces reads to be
mapped to daughter strands.

#### Paired-End library

By default, BISCUIT map read 1 to one strand (parent or daughter) and
read 2 to the other strand (different from the read 1 strand). This is
the behavior with `-b 0`. The `-b 1` forces read 1 to be mapped to
parent strand and read 2 to be mapped to the daughter strand. You
could map read 1 to the daughter and read 2 to the parent by simply
swapping the read 1 and read 2 fastq file.

#### An example

Let's look at one example:

```bash
biscuit align mm10.fa -1
    TTGGTGTGTGGGTTTTGATGTTGGGTGGAGGGTTT
```

```
inputread	0	chr10	3386516	3	35M	*	0	0
    TTGGTGTGTGGGTTTTGATGTTGGGTGGAGGGTTT	*
	NM:i:0	MD:Z:35	ZC:i:1	ZR:i:0	AS:i:35
	XS:i:34	XL:i:35	XA:Z:chr10,+3386516,35M,1	XB:Z:1,0
	YD:A:f
```

One can see without specifying the strand BISCUIT sends the read
to bisulfite Waston (`YD:A:f`) automatically without 
mismatches (`NM:i:0`).

```bash
biscuit align -b 3 mm10.fa -1
    TTGGTGTGTGGGTTTTGATGTTGGGTGGAGGGTTT
```

```
inputread	0	chr10	3386516	60	35M	*	0	0
    TTGGTGTGTGGGTTTTGATGTTGGGTGGAGGGTTT	*
	NM:i:1	MD:Z:34C0	ZC:i:0	ZR:i:17	AS:i:34
	XS:i:0	XL:i:35	YD:A:r
```

With `-b 3`, BISCUIT sends the read to Bisulfite Crick (`YD:A:r`)
with 1 mismatch (`NM:i:1`).

### Distinguish decoy chromosomes in human and mouse

A la BWA, BISCUIT makes a distinction of primary chromosomes from
alternative/decoy chromosomes. For human and mouse, the inference is
turned on by default. This results a preferred alignment of read to
the primary chromosomes. This can be turned off through a `-i` option.
The logic of inference is the following: If the chromosome names are
like chr1, chr2, ..., then chromosomes with name pattern `chrUn`,
`_random`, `_hap`, `_alt` are set as ALT chromosomes.

### Other useful options

- `-F` suppresses SAM header output

### Other alignment features worth mentioning

- assymmetric scoring for C to T and G to A in local mapping.
- produce consistent mapping quality calculation with destination
  strand specification.
- produce consistent NM and MD tags under assymmetric scoring.
- produce ZR and ZC tags for retention count and conversion count
- separate seeding for parent and daughter strands for mapping
  efficiency
- disk-space-economic indices with no need to store a bisulfite
  converted reference.
- BWA-mem-like parameters visible to the users.
- rare dependencies
- robust to OS build, stable multi-threading
- Optional parent strand and daughter strand restriction for both
  single- and paired-end reads.
- Optional BSW/top/BSC/bottom strand restriction, tightly integrated
  in mapping.