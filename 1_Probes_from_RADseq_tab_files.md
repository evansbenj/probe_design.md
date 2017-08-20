# For data in tab format from RADseq

I worked with this file on info:
```
/home/ben/2015_SulaRADtag/good_merged_samples/fastq/GenotypeVCFs_noBSQR_filtered.vcf.gz.tab
```

This is a script that takes as input a tab delimited file from radseq data.  It was be executed as follows:
```
./Makes_capture_probes_from_tab.pl GenotypeVCFs_noBSQR_filtered.vcf.gz.tab 10 1 GenotypeVCFs_noBSQR_filtered.vcf.gz_10.probes
```

Where 10 is the tile_density (the amount of space between overlapping probes) and 1 is the number that the index starts with.  

Here is the script (Makes_capture_probes_from_tab.pl):

```
#!/usr/bin/env perl
use strict;
use warnings;
use List::MoreUtils qw/ uniq /;



#  This program reads in a tab delimited genotype file generated
#  by the perl program "17_adds_outgroup_to_lots_of_tab_files.pl"
#  or from vcftools vcf_to_tab
#  and generates an 52 bp capture arrays

# Execute like this; ./Makes_capture_probes_from_tab.pl INPUTFILE tile_density index_start outputfile

# where tile_density is the amount of overlap for each probe (e.g. 1) and the index_start is the 
# number that you want the index to start on (for example if you already did some others using
# another input file)


my $inputfile = $ARGV[0];
my $tile_density=$ARGV[1];
my $index = $ARGV[2];
my $outputfile = $ARGV[3];

my @temp;
my $previousnumber=0;
my $string="";
my $chr = "";


unless (open DATAINPUT, $inputfile) {
    print "Can not find the input file.\n";
    exit;
}



unless (open(OUTFILE, ">$outputfile"))  {
    print "I can\'t write to $outputfile\n";
    exit;
}

# first make an array with a bunch of microsatellites
my @bp=('A','C','T','G');
my @micros;
my $tempp;
my $x;
my $switch=0;

# onemers
foreach my $basepair (@bp){
    for($x=1; $x<25; $x++){
        $tempp = $tempp.$basepair;
    }
    push(@micros,$tempp);
    $tempp="";
}

#dimers
foreach my $firstbasepair (@bp){
    foreach my $secondbasepair (@bp){
        if($firstbasepair ne $secondbasepair){
            for($x=1; $x<13; $x++){
                $tempp = $tempp.$firstbasepair.$secondbasepair;
            }
            push(@micros,$tempp);
            $tempp="";
        }
    }    
}


#trimers
foreach my $firstbasepair (@bp){
    foreach my $secondbasepair (@bp){
       foreach my $thirdbasepair (@bp){
            if(($firstbasepair ne $secondbasepair)&&($firstbasepair ne $thirdbasepair)&&($secondbasepair ne $thirdbasepair)){
                for($x=1; $x<9; $x++){
                    $tempp = $tempp.$firstbasepair.$secondbasepair.$thirdbasepair;
                }
                push(@micros,$tempp);
                $tempp="";
            }
        }
    }    
}

print "@micros\n";

# Read in datainput file
while ( my $line = <DATAINPUT>) {
    chomp($line);
    @temp=split(/[\/'\t']+/,$line);
    if($temp[0] ne '#CHROM'){
        # extend a new string if one is already established, if the bp is continuous, and if the chr is the same
        if(($temp[1] == ($previousnumber+1))&&($chr eq $temp[0])){
            $string=$string.$temp[2]; # column 3 is the rhemac2 ref
            $previousnumber=$temp[1];
        }
        # otherwise begin building one
        else{  # we skipped a base in the ref, are on a new chr, or just started
            # start building the string (again)
            $string=$temp[2];
            $previousnumber=$temp[1]; # this is used to track the current position and make sure we have no gaps
            $chr=$temp[0];
        }
        # if long enough print the string and delete first $tile_density bp
        if(length $string >= 52){
            # could put a test here to not print it if there are microsats
            foreach my $microsat (@micros){
                if(substr($string, 0, 52) =~ /$microsat/){
                    print substr($string, 0, 52),"\n";
                    $switch=1; # this tells us to not print this probe
                }
            }
            if($switch == 0){    
                print OUTFILE "BJE_rhemac2".$chr."_".($temp[1] - length($string) +1)."_".$index."\t".substr($string, 0, 52)."CACTGCGG\n";
                $index+=1;
            }    
            # after printing, save only the the last (52-$tile_density) bp of the string
            $string=substr($string, -(52-$tile_density));
            $switch=0;                                                       
        }
    } # endif to check for first line
} # end while

close DATAINPUT;

close OUTFILE;
print "Done with input file 1\n";


```

The output looks like this (not that the last 8 bp of all probes are the same:
```
BJE_rhemac2chr1_947_1	TTCTACGTCCCGTCTGCTTCCCCTAAAGCGAGCTCCTTGAGGCATACAGAGCCACTGCGG
BJE_rhemac2chr1_957_2	CGTCTGCTTCCCCTAAAGCGAGCTCCTTGAGGCATACAGAGCATCCTCTGCTCACTGCGG
BJE_rhemac2chr1_1615_3	TTGAATTTATCAGTGCCGGTCACCAGCCCCTCCCTGCGCTGCACAGGCTGAGCACTGCGG
BJE_rhemac2chr1_1625_4	CAGTGCCGGTCACCAGCCCCTCCCTGCGCTGCACAGGCTGAGGTGTAACCTTCACTGCGG
BJE_rhemac2chr1_1635_5	CACCAGCCCCTCCCTGCGCTGCACAGGCTGAGGTGTAACCTTCTGTGTGCCTCACTGCGG
BJE_rhemac2chr1_1645_6	TCCCTGCGCTGCACAGGCTGAGGTGTAACCTTCTGTGTGCCTGCAGGGGAGGCACTGCGG
BJE_rhemac2chr1_1655_7	GCACAGGCTGAGGTGTAACCTTCTGTGTGCCTGCAGGGGAGGAAAATCAGGTCACTGCGG
BJE_rhemac2chr1_1665_8	AGGTGTAACCTTCTGTGTGCCTGCAGGGGAGGAAAATCAGGTATTGGGAGCACACTGCGG
BJE_rhemac2chr1_1675_9	TTCTGTGTGCCTGCAGGGGAGGAAAATCAGGTATTGGGAGCACTAGTCATGTCACTGCGG

```

and the end of this part looks like this:
```
BJE_rhemac2chrX_153925675_570378	TCTGGTGCTCAAGGAGACACTGGGTGGAGTGGGAGCCACAGACCTTCTGCTGCACTGCGG
BJE_rhemac2chrX_153925685_570379	AAGGAGACACTGGGTGGAGTGGGAGCCACAGACCTTCTGCTGATGGCAAAGACACTGCGG
BJE_rhemac2chrX_153925695_570380	TGGGTGGAGTGGGAGCCACAGACCTTCTGCTGATGGCAAAGACGGGGTTCCTCACTGCGG
BJE_rhemac2chrX_153925705_570381	GGGAGCCACAGACCTTCTGCTGATGGCAAAGACGGGGTTCCTGCAGGTGCTGCACTGCGG
BJE_rhemac2chrX_153925715_570382	GACCTTCTGCTGATGGCAAAGACGGGGTTCCTGCAGGTGCTGGCTCCCTCTGCACTGCGG
BJE_rhemac2chrX_153925725_570383	TGATGGCAAAGACGGGGTTCCTGCAGGTGCTGGCTCCCTCTGTGGGCTGAGGCACTGCGG
BJE_rhemac2chrX_153925735_570384	GACGGGGTTCCTGCAGGTGCTGGCTCCCTCTGTGGGCTGAGGACCCGGCTGGCACTGCGG
BJE_rhemac2chrX_153925745_570385	CTGCAGGTGCTGGCTCCCTCTGTGGGCTGAGGACCCGGCTGGTAACTCCCCGCACTGCGG
BJE_rhemac2chrX_153925755_570386	TGGCTCCCTCTGTGGGCTGAGGACCCGGCTGGTAACTCCCCGTTCTGAACCCCACTGCGG
```
Then I added to these probes the macaque and xenopsu probes that were designed from the other script and using fasta files as input.




