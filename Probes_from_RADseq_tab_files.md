# FOr data in tab format from RADseq
This is a script that takes as input a tab delimited file from radseq data.  It can be executed as follows:
```
./Makes_capture_probes_from_tab.pl input.tab tile_density index_number probes.out
```
Where  tile_density is the amount of space between overlapping probes (e.g. 2) and the index number is the number the index starts with.  Here is the script (Makes_capture_probes_from_tab.pl):
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
            print OUTFILE "BJE_rhemac2".$chr."_".($temp[1] - length $string+1)."_".$index."\t".substr($string, 0, 52)."CACTGCGG\n";
            $index+=1;
            # after printing, save only the the last (52-$tile_density) bp of the string
            $string=substr($string, -(52-$tile_density));                                                       
        }
    } # endif to check for first line
} # end while

close DATAINPUT;

close OUTFILE;
print "Done with input file 1\n";

```

The output looks like this (not that the last 8 bp of all probes are the same:
```
BJE_rhemac2chr1_2607738_1	CGCCAGGCATTTTGGCAGCAGCTCTCGAATGAAGCCGTGAGTGTCCCTTGTGCACTGCGG
BJE_rhemac2chr1_2607740_2	CCAGGCATTTTGGCAGCAGCTCTCGAATGAAGCCGTGAGTGTCCCTTGTGGCCACTGCGG
BJE_rhemac2chr1_2607742_3	AGGCATTTTGGCAGCAGCTCTCGAATGAAGCCGTGAGTGTCCCTTGTGGCGCCACTGCGG
BJE_rhemac2chr1_2607744_4	GCATTTTGGCAGCAGCTCTCGAATGAAGCCGTGAGTGTCCCTTGTGGCGCGGCACTGCGG
BJE_rhemac2chr1_2607746_5	ATTTTGGCAGCAGCTCTCGAATGAAGCCGTGAGTGTCCCTTGTGGCGCGGCACACTGCGG

```


