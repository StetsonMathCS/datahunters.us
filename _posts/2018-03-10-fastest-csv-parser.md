---
layout: post
title: "What is the Fastest CSV Parser? Experiments with 1.4 Billion Rows"
author: "Joshua Eckroth"
---

In the world of data science, CSV (comma-separated values) files are a common format for sharing data. The CSV format is simple for the majority of cases: each row of data is a single line in the file, and each field in a row is separated by a single comma. In less simple cases, some fields may be quoted and have internal commas or linebreaks.

In the world of big data, we would like to parse (extract data from) large CSV files as fast as possible. Many CSV parsers are available. These parsers are usually designed to handle all kinds of CSV files, including the complex cases with quotes and internal commas. But if we do not need such complex parsing, what is the fastest tool?

The Data Hunters team addressed this question in Spring, 2018. We benchmarked these tools:

- [Python 3 CSV library](https://docs.python.org/3/library/csv.html).
- [csvkit 1.0.3](https://csvkit.readthedocs.io/en/1.0.3/), a Python-based toolkit with various tools for parsing, aggregating, splitting CSV files.
- An [enhancement](https://github.com/bbelna/NVCSV) of antonmks' [nvParse](https://github.com/antonmks/nvParse) for parsing CSV with CUDA on NVIDIA GPUs.
- Custom C code, single-threaded.
- [fast-cpp-csv-parser](https://github.com/ben-strasser/fast-cpp-csv-parser) by Ben Strasser, multi-threaded.
- Unix tools: cut, awk.
- Perl's awk-like functionality.
- [GNU Parallel](https://www.gnu.org/software/parallel/) for running multiple processes in parallel.

## The Data

We'll calculate a simple sum of a single column of 102 CSV files, totaling 1.4 billion rows of data. These files come from the [New York City Taxi and Limousine Commission](http://www.nyc.gov/html/tlc/html/about/trip_record_data.shtml). We'll use the Yellow Taxi trip data, 2009-2017. The files all have a fifth colum representing distance traveled for each taxi trip. We'll just sum that single column across all files to produce a single answer: 7,140,380,300 miles. But how fast can we do it?

## Python 3 CSV library

## csvkit 1.0.3

```
$ for f in `ls *.csv`; do csvcut -c 5 $f; done | \
  awk '{sum += $1} END {print sum}'
```

Average time: 

```
$ parallel -j8 csvcut -c 5 {} \| awk \' {sum+=\$1} END {print sum}\' ::: *.csv
```

Average time: 55 minutes

This result is corroborated by Trâm Nguyễn, Dearvis Troutman.

## Custom C code

We experimented with a C program custom coded to add up column five in a CSV file. We read the file 1MB at a time and take care to detect line breaks.

Here is the code:

```{c}
#include <stdio.h>
#include <stdlib.h>

#define BUFSIZE 1000000

int main()
{
    char buf[BUFSIZE+1];
    char diststr[100];
    double dist;
    double sumdist = 0.0;

    FILE *file;
    size_t nread;
    
    file = fopen("/ssd/data/taxi/yellow_tripdata_2009-01.csv", "r");
    if(file) {
        int commacnt = 0;
        int i, j;
        while((nread = fread(buf, 1, BUFSIZE, file)) > 0) {
            buf[BUFSIZE] = 0;
            for(i = 0; i < BUFSIZE; i++) {
                if(buf[i] == ',') {
                    commacnt++;
                    j = 0;
                    i++;
                }
                else if(buf[i] == '\n') commacnt = 0;

                if(commacnt == 4) {
                    while(i < BUFSIZE) {
                        if(buf[i] == ',') {
                            diststr[j] = 0;
                            dist = strtod(diststr, NULL);
                            sumdist += dist;
                            commacnt++;
                            break;
                        } else {
                            diststr[j++] = buf[i++];
                        }
                    }
                }
            }
        }
        if(ferror(file)) {
            printf("Error reading file\n");
            return 2;
        }
        fclose(file);
        printf("%f\n", sumdist);
    } else {
        printf("Error opening file\n");
        return 1;
    }

    return 0;
}
```

We observed 28 seconds runtime on a single file.

This code and result were produced by Joshua Eckroth.

## fast-cpp-csv-parser by Ben Strasser

This [tool](https://github.com/ben-strasser/fast-cpp-csv-parser) is written in C++ and multithreaded. So we will not use `parallel` to run multiple processes.

On one file, we observed it takes about two minutes. Thus, we estimate all files may be processed in about 200 minutes.

This result was found by Brandon Belna.

## Unix tools

Only `awk`:

```
$ awk -F "," '{sum += $5} END {print sum}' *.csv
```

Average time: 60 minutes.

First `cut` to keep field #5, then `awk`:

```
$ cut -d "," -f5 | awk '{sum += $1} END {print sum}' *.csv
```

Average time: 23 minutes.

## Perl's awk-like functionality

```
$ perl -F',' -ane '$a += $F[5]; END { print $a }' file.csv
```

This command took 65 seconds on one file, so we estimate it would take 100 minutes on all files.

Another Perl technique wasn't much faster:

```
$ perl -ne 'split /,/ ; $a+= $_[5]; END {print $a."\n";}' -f file.csv
```

These results were found by Eddie White.

## GNU Parallel

Take the cut+awk approach from above (23 minutes), and run eight of these in parallel on different subsets of the input files; then sum their respective results.

```
$ parallel -j8 bash cutawk.sh {} ::: *.csv | \
  awk '{sum += $1} END {print sum}'
```

Average time: 16 minutes.

This result is corroborated by Brandon Belna, Tierney Irwin, Sam Pratico, Maria Anna Shimkovska, Andrew Tompkins, Eddie White.

## Summary

(bar chart here)

