# This script makes probes from a multifasta file - make sure it is unix formatted.

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
    if(substr($line,0,1) ne '>'){
        # extend a new string 
        $string=$string.$line;
        until(length $string < 52){
            # could put a test here to not print it if there are microsats
            print OUTFILE "BJE_fasta_".$name."_".$index."\t".substr($string, 0, 52)."CACTGCGG\n";
            $index+=1;
            # after printing, delete $tile_density bp of the string
            $string=substr($string, $tile_density, length $string);                                                       
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
