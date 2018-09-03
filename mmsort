#!/afs/athena/contrib/perl5/perl
# $Source: /afs/.athena/contrib/consult/arch/common/bin/RCS/mmsort,v $
# $Id: mmsort,v 1.2 2011/02/28 17:39:15 boojum Exp $

use Getopt::Long;
use Getopt::Std;
use LWP::UserAgent;
use HTTP::Request::Common qw(POST);
use filetest 'access';

$baseurl = "https://mailman.mit.edu/mailman/admin/";
$unencrypturl = "http://mailman.mit.edu/mailman/admin/";
$savedir = "$ENV{HOME}/.mmblanche";

# From Programming Perl p 448
$opt_success = &GetOptions("p=s"  => \$password,
			   "S"    => \$setup,
			   "d"    => \$domain,
			   "n"    => \$noquote,
			   "U"    => \$unencrypt
			  );

if (! $opt_success) {
  &usage();
}
if ($#ARGV > 0) {
  &usage();
}

if ($unencrypt) {
  $baseurl=$unencrypturl;
}

$quotemark = "\"";
if ($noquote) {
  $quotemark = "";
}

## the list name is what's left.  Downcase it.  If it doesn't exist,
## print usage. 
$listname = "\L$ARGV[0]";
chomp $listname;
if ($listname eq "") {
  &usage();
}

if (!$password) {
  # Try to load it from file first.
  &lookforpassword();
  # If we didn't get it from the file, prompt. 
  if (!$password) {
    system("stty -echo");
    print STDERR "List Admin Password: ";
    $password = <STDIN>;
    chomp $password;
    system("stty echo");
    print STDERR "\n";
  }
}


# Save the password if the -S flag was set. 
if ($setup) {
  &savepassword();
}


## Membership page
$memberstart = $baseurl . $listname . "/members";
&parsepage($memberstart);

## Time to pull in membership from more links.
print STDERR "Retrieving membership list";
while (!$done) {
  print STDERR ".";
  $nextlink = &undonelinks();
  if ($nextlink eq "NONE") {
    last;
  }
  $alllinks{$nextlink} = 1;
  &parsepage($nextlink);
}
print STDERR " done.\n";

&printmembers();



######################################################################

sub usage() {
  die "Usage: mmblanche listname -p password
";
}

sub addlinks() {
  for $i (@_) {
    ## If it's already there and set to 1, don't reset it to 0. 
    if ($alllinks{$i} != 1) {
      $alllinks{$i} = 0;
    }
  }
}


# Add the enclosed members to the hash of members
sub membershere() {
  @page = @_;
  @names = grep /INPUT name/, @page;
  @names = grep /user/, @names;
  foreach $i (@names) {
    $realname = $i;
    # the address is the only thing not in angle brackets
    $i =~ s/<[^<>]+>//g;
    $i =~ s/[	 ]//g;
    chomp $i;
    # Meanwhile, the name is stuffed in a realname field
    $realname =~ s/^.+realname.+?value=\"//;
    $realname =~ s/\".+$//;
    $namemap{$i} = $realname;
  }
  return @names;
}
  
    
# Add the enclosed links to the hash of links
sub linksinpage() {
  @letterlinks = @_;

  @letterlinks = grep m/href/, @letterlinks;
  @letterlinks = grep m/members\?letter/, @letterlinks;

  foreach $i (@letterlinks) {
    $i =~ s/^.+href//;
    $i =~ s/^[^\"]+\"//;
    $i =~ s/\".+//;
    if (!$unencrypturl) {
      $i =~ s/http:/https:/;
    }
  }
  return @letterlinks;
}

# Print membership, sorted
sub printmembers() {
  print "Sorting...\n";
  if ($domain) {
      foreach $i (sort bydomain keys %allmembers) {
	print "$quotemark$namemap{$i}$quotemark <$i>\n";
      }
    }
  else {
    foreach $i (sort bylastname keys %allmembers) {
      print "$quotemark$namemap{$i}$quotemark <$i>\n";
    }
  }
}

sub bydomain() {
  $tmpa = $a;
  $tmpb = $b;
  $tmpa =~ s/^.+@//;
  $tmpb =~ s/^.+@//;
  $tmpa cmp $tmpb
      or $a cmp $b;
}

sub bylastname() {
  $tmpa = $namemap{$a};
  $tmpb = $namemap{$b};
  $tmpa =~ s/^.+ //;
  $tmpb =~ s/^.+ //;
  $tmpa cmp $tmpb;
}


# Return the specified page.  Using the password. 
sub findpage() {
  $ua = LWP::UserAgent->new();
  $url = @_[0];
  $req = POST $url,
  [ 'adminpw' => $password];
  $content = $ua->request($req)->as_string;
#  print $content;
  if ($content =~ m/Administrator Authentication/) {
    die "Wrong password.\n";
  }
  if ($content =~ m/mailman.mit.edu mailing lists - Admin Links/) {
    die "No such list found.\n";
  }

  return $content;
}

## Look for an undone link.  If there aren't any, return "NONE"
sub undonelinks() {
  foreach $i (keys %alllinks) {
    if ($alllinks{$i} == 0) {
      return $i;
    }
  }
  # No zeros found
  return "NONE";
}


# Grab the membership from this page, and grab the links to other
# pages too.  
sub parsepage() {
  $memberstart = @_[0];
#  print "Slurping $memberstart\n";
  $content = &findpage($memberstart);
  if ($content =~ m/501 \(Not Implemented\) Protocol scheme \'https\' is not supported/) {
    die "ERROR: Unable to setup a secure https connection with the Mailman server.
You can run mmsort with the -U flag to use an unencrypted http connection.\n";
  }
#  print $content;
  @startlines = split /\n/, $content;
  
  @title = grep m/title/i, @startlines;
#  print "@title";

  @theselinks = &linksinpage(@startlines);
  @thesemembers = &membershere(@startlines);
  &addlinks(@theselinks);
  &addmembers(@thesemembers);
}


## Look in .mmblanche/listname and load password from that file.     
sub lookforpassword() {
  if (-d $savedir) {
    if (-e "$savedir/$listname") {
      open (FILE, "$savedir/$listname");
      @filecontents = <FILE>;
      $password = $filecontents[0];
      chomp $password;
    }
  }
}

# Create .mmblanche if it doesn't exist.  Write the password to
# .mmblanche/listname 
sub savepassword() {
  if (!-e $savedir) {
    print "Creating private $savedir directory\n";
    system("mkdir $savedir");
    system("fs sa $savedir $ENV{USER} all -clear");
  }
  open (OUTFILE, ">$savedir/$listname");
  print OUTFILE $password;
  close OUTFILE;
}

sub addmit() {
  foreach $i (@_) {
    if ($i !~ m/\@/) {
      $i =~ s/^(.+)$/$1\@mit.edu/;
    }
  }
  return @_;
}

# This is just a hash (so we don't accidentally duplicate anyone)
sub addmembers() {
  for $i (@_) {
      $allmembers{$i} = 1;
    }
}


sub readfile() {
  $filename = $_[0];
  open FILE, $filename or die "Cannot open $filename\n";
  @input = <FILE>;
  return @input;
}

sub usage() {
  die "mmsort listname [-p password] [-S] [-d] [-n] [-U]

mmsort does not add or remove members; it only gets the list of
members, both email address and name, and prints them in alphabetical
order by last name (well, last word of the name).  Unless the -d flag
is used, in which case it sorts by the domain of the email address.

Addresses are printed as 
\"Firstname Lastname\" <email@wherever.com>
unless the -n flag is used, in which case the quote marks are omitted.

-U requests the unencrypted pages instead. 

If the -S option is included, the password will be saved into a file
in a private .mmblanche directory, and subsequent use of mmsort or
mmblanche will check there for the password.

I advise against using the -p flag if you're on a shared machine. 
If you do not use -p, and the password was not previously saved using
-S, then mmsort will prompt for a password.

\n"; 
}
