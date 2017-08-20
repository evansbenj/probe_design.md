# This script makes probes from a multifasta file - make sure it is unix formatted.

I also made probes from macaque mtDNA and yDNA (SRY and TSPY) seqs with much denser tiling - every base pair.  And I also made probes from Xenopus mtDNA (XL, XB, and ST) and xennie DMRT1 as well.

Execute like this: ./Makes_capture_probes_from_fasta.pl input tiledensity indexstart output

Here is the script (/Makes_capture_probes_from_fasta.pl):
```
#!/usr/bin/env perl
use strict;
use warnings;
use List::MoreUtils qw/ uniq /;



#  This program reads in a multifasta file that can be hard wrapped or not

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
my $name = "";
my $switch=0;


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
    if(substr($line,0,1) ne '>'){
        # extend a new string 
        $string=$string.$line;
        until(length $string < 52){
            # test here to not print it if there are microsats
            foreach my $microsat (@micros){
                if(substr($string, 0, 52) =~ /$microsat/){
                    print substr($string, 0, 52),"\n";
                    $switch=1; # this tells us to not print this probe
                }
            }
            if($switch == 0){    
                #print OUTFILE "BJE_fasta_".$name."_".$index."\t".substr($string, 0, 52)."CACTGCGG\n";
                #print OUTFILE "BJE_macaque_".$index."\t".substr($string, 0, 52)."CACTGCGG\n";
                print OUTFILE "BJE_xenopus_".$index."\t".substr($string, 0, 52)."CACTGCGG\n";
                $index+=1;
            }
            # after printing, delete $tile_density bp of the string
            $string=substr($string, $tile_density, length $string);                                                       
            $switch=0;
        }
    } # endif to check for first line
    else{ # this is a new fasta entry
        $name = substr($line, 1, length $line);
        $string="";
    }
} # end while

close DATAINPUT;

close OUTFILE;
print "Done with input file 1\n";


```
