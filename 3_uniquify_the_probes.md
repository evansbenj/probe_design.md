# Save only unique probez

This script (make_uniq_probez.pl) saves only the unique probes:

```
#!/usr/bin/env perl
use strict;
use warnings;
use List::MoreUtils qw/ uniq /;
 
# This script takes a redundant list of probes and makes a new list of unique ones.

# Execute like this: ./make_uniq_probez.pl input indexstart output

my $inputfile = $ARGV[0];
my $index = $ARGV[1];
my $outputfile = $ARGV[2];



unless (open DATAINPUT, $inputfile) {
    print "Can not find the input file.\n";
    exit;
}



unless (open(OUTFILE, ">$outputfile"))  {
    print "I can\'t write to $outputfile\n";
    exit;
}

my @temp;
my $previousnumber=0;
my $string="";
my $name = "";
my $switch=0;

my @probez;


while ( my $line = <DATAINPUT>) {
    chomp($line);
    @temp=split("\t",$line);
    push (@probez,$temp[1]);
}
close DATAINPUT;

my @unique_probez = uniq @probez;


foreach(@unique_probez){
	print OUTFILE "BJE_".$index."\t".$_."\n";
	$index +=1;
}



close OUTFILE;
```
