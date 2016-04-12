## Testing the --test feature

# Using Marcel's C.elegans reads

```
    # in find_circ2/test_data
    PREFIX=/data/BIO3/home/mschilli/repo/global/external/marvin/find_circ2/test_data/
    INDEX=/data/BIO3/indices/WBcel235_bwa_0.7.5a-r405/WBcel235.fa
    
    bwa mem -t16 -k 15 -T 1 $INDEX \
        ${PREFIX}/synthetic_reads.R1.fa.gz \
        ${PREFIX}/synthetic_reads.R2.fa.gz \
        > /scratch/circdetection_test_data/marcel_test.sam
    
    cat /scratch/circdetection_test_data/marcel_test.sam | \
        ../find_circ.py --test -G $INDEX --stdout=test | les
```

# Using my own dm6 reads

```
    # in find_circ2/test_data
    INDEX=/data/rajewsky/indices/dm6_bwa_0.7.12-r1039/dm6.fa
    N_FRAGS=1000

    # generating simulated reads
    time cat ~/circpeptides/fly/dm6.ribocircs_mar2016.ucsc | grep ANNOTATED | \
        ../simulate_reads.py -G $INDEX --n-frags=${N_FRAGS} --read-len=100 --frag-len=350 --mutate=0.00 -o sim/dm6/r100_f350_fpk100/m0.00 --fpk &

    time cat ~/circpeptides/fly/dm6.ribocircs_mar2016.ucsc | grep ANNOTATED | \
        ../simulate_reads.py -G $INDEX --n-frags=${N_FRAGS} --read-len=100 --frag-len=350 --mutate=0.01 -o sim/dm6/r100_f350_fpk100/m0.01 --fpk &

    time cat ~/circpeptides/fly/dm6.ribocircs_mar2016.ucsc | grep ANNOTATED | \
        ../simulate_reads.py -G $INDEX --n-frags=${N_FRAGS} --read-len=100 --frag-len=350 --mutate=0.05 -o sim/dm6/r100_f350_fpk100/m0.05 --fpk &

    time cat ~/circpeptides/fly/dm6.ribocircs_mar2016.ucsc | grep ANNOTATED | \
        ../simulate_reads.py -G $INDEX --n-frags=${N_FRAGS} --read-len=100 --frag-len=350 --mutate=0.10 -o sim/dm6/r100_f350_fpk100/m0.10 --fpk &

    # mapping
    for SAMPLE in dm6/r100_f350_fpk100/m0.00 dm6/r100_f350_fpk100/m0.01 dm6/r100_f350_fpk100/m0.05 dm6/r100_f350_fpk100/m0.10
    do {
        mkdir -p run/${SAMPLE}
        bwa mem -t16 -k 15 -T 1 -p $INDEX sim/${SAMPLE}/simulated_reads.fa.gz > run/${SAMPLE}/aligned.sam
    } done;

    # running find_circ2
    for SAMPLE in dm6/r100_f350_fpk100/m0.00 dm6/r100_f350_fpk100/m0.01 dm6/r100_f350_fpk100/m0.05 dm6/r100_f350_fpk100/m0.10
    do {
        ../find_circ.py run/${SAMPLE}/aligned.sam -o run/${SAMPLE} --test -G $INDEX \
            --known-lin=sim/${SAMPLE}/lin_splice_sites.bed \
            --known-circ=sim/${SAMPLE}/circ_splice_sites.bed &
    } done;

    # making scatter plots
    for SAMPLE in dm6/r100_f350_fpk100/m0.00 dm6/r100_f350_fpk100/m0.01 dm6/r100_f350_fpk100/m0.05 dm6/r100_f350_fpk100/m0.10
    do {
        ../merge_bed.py -6 --score sim/${SAMPLE}/circ_splice_sites.bed run/${SAMPLE}/circ_splice_sites.bed | \
            histogram.py -s -q -b0 --linear-fit -x "simulated junction reads" -y "recovered junction reads" -t "circRNA recovery error" --pdf=run/${SAMPLE}/run_vs_sim.pdf
    } done;
    
```


# Human circRNAs from circbase

```
    # in find_circ2/test_data
    # prepare a sample of human circRNAs
    wget http://www.circbase.org/download/hsa_hg19_circRNA.bed
    cat hsa_hg19_circRNA.bed | unsort.py | head -n 1000 | cut -f 1,2,3,4,5,6 | \
        exon_intersect.py -S hg19 -m hg19_sample.ucsc > hg19_sample.bed

    INDEX=/data/rajewsky/indices/hg19_bwa_0.7.12-r1039/hg19.fa
    N_FRAGS=1000

    # generating simulated reads
    time cat hg19_sample.ucsc | grep ANNOTATED | \
        ../simulate_reads.py -G $INDEX --n-frags=${N_FRAGS} --read-len=100 --frag-len=350 --mutate=0.00 -o sim/hg19/r100_f350_fpk100/m0.00 --fpk &

    time cat hg19_sample.ucsc | grep ANNOTATED | \
        ../simulate_reads.py -G $INDEX --n-frags=${N_FRAGS} --read-len=100 --frag-len=350 --mutate=0.01 -o sim/hg19/r100_f350_fpk100/m0.01 --fpk &

    time cat hg19_sample.ucsc | grep ANNOTATED | \
        ../simulate_reads.py -G $INDEX --n-frags=${N_FRAGS} --read-len=100 --frag-len=350 --mutate=0.05 -o sim/hg19/r100_f350_fpk100/m0.05 --fpk &

    time cat hg19_sample.ucsc | grep ANNOTATED | \
        ../simulate_reads.py -G $INDEX --n-frags=${N_FRAGS} --read-len=100 --frag-len=350 --mutate=0.10 -o sim/hg19/r100_f350_fpk100/m0.10 --fpk &

    # mapping
    for SAMPLE in hg19/r100_f350_fpk100/m0.00 hg19/r100_f350_fpk100/m0.01 hg19/r100_f350_fpk100/m0.05 hg19/r100_f350_fpk100/m0.10
    do {
        mkdir -p run/${SAMPLE}
        bwa mem -t16 -k 15 -T 1 -p $INDEX sim/${SAMPLE}/simulated_reads.fa.gz > run/${SAMPLE}/aligned.sam
    } done;

    # running find_circ2
    for SAMPLE in hg19/r100_f350_fpk100/m0.00 hg19/r100_f350_fpk100/m0.01 hg19/r100_f350_fpk100/m0.05 hg19/r100_f350_fpk100/m0.10
    do {
        ../find_circ.py run/${SAMPLE}/aligned.sam -o run/${SAMPLE} --test -G $INDEX \
            --known-lin=sim/${SAMPLE}/lin_splice_sites.bed \
            --known-circ=sim/${SAMPLE}/circ_splice_sites.bed &
    } done;

    # making scatter plots
    for SAMPLE in hg19/r100_f350_fpk100/m0.00 hg19/r100_f350_fpk100/m0.01 hg19/r100_f350_fpk100/m0.05 hg19/r100_f350_fpk100/m0.10
    do {
        ../merge_bed.py -6 --score sim/${SAMPLE}/circ_splice_sites.bed run/${SAMPLE}/circ_splice_sites.bed | \
            histogram.py -s -q -b0 --linear-fit -x "simulated junction reads" -y "recovered junction reads" -t "circRNA recovery error" --pdf=run/${SAMPLE}/run_vs_sim.pdf
    } done;
    
```