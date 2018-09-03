#!/usr/athena/bin/perl 

($list,$what) = @ARGV;

if (!$what) {&usage();}


@list = `athrun consult mmblanche $list`;

$len = 0;

foreach $i (@list) {
  $len = max($len, length($i));
}

$len++;

foreach $i (@list) {
  
  if ($i =~ m/\@mit.edu/i) {
    $ii = $i;
    $ii =~ s/\@mit.edu//i;

    chomp $ii;
    $string = "athrun consult ldaps uid=$ii 2>1&";
    @ldap = `$string`;
    
    @ldap = grep /eduPersonAffiliation/, @ldap;
    $ldap = $ldap[0];
    $ldap =~ s/eduPersonAffiliation: //;
    chomp $i;
    print $i;
    for $j (length($i) .. $len) { print " ";}
    chomp $ldap;
    print "$ldap\n";
  }

  else {
    print $i;
  }
}







sub max { $_[0]>$_[1] ? $_[0] : $_[1]; }


sub usage {
  die "mmblanche-ldap <ldap field>

To see what ldap fields are available, type \"athrun consult ldaps <username>\"
Specify the field that you want to look up, and the list will display
with that field for each user entry.
";}
