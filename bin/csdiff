#!/usr/bin/perl -w

##############################################################################
#
#                                 csdiff.pl
#
# DESCRIPTION:
# Wrapper that provides color-coding for sdiff output.
#
##############################################################################

##########################
# Initialization section #
##########################

require 5.00;
use strict;
use IO::File;
use Term::ANSIColor qw( :constants );

# Default umask
umask 027;

# Program's name and version number.
my ($BASENAME) = ($0 =~ /([^\/]+)$/);
our $VERSION = "2.0";

# Get login name.
my $USER=getpwuid($<) || getlogin || die "Can't find USER from environment!\n";

#########
# Usage #
#########

sub Usage { # private
    print <<USAGE;
This is $BASENAME $VERSION by Pierre Rioux

Usage: $BASENAME [-Cn] [-F[su|a|mc|d|o]] [-S] [sdiff args]

       The -C option selects how many lines of context to show around changes.

       The -F option selects which parts of the report to show:
         's' or 'u'   Same or Unchanged
         'a'          Added
         'd'          Deleted
         'm' or 'c'   Modified/Changed
         'o'          Omitted (only when -C is specified)

       Note that when using -C and -F at the same time, unselected reports
       in -F become part of the definition of 'context' for -C.

       The -S option triggers a 'summary' mode (no diff is shown, and -F and -C are ignored)

USAGE
    exit 1;
}

##################################
# Global variables and constants #
##################################

my $DEBUG=0;
my $FILTERS = "";
my $CONTEXT = undef; # lines of context for each change.
my $SUMMARY = 0;

# Customize output here
my $INSERT    = ON_GREEN;
my $DELETE    = ON_RED;
my $CHANGED   = ON_BLUE;
my $MISSINGNL = ON_MAGENTA;
my $UNCHANGED = "";
my $OMITTED   = MAGENTA;

my $OFF       = RESET;

##############################
# Parse command-line options #
##############################

for (;@ARGV;) {
    # Add in the regex [] ALL single-character command-line options
    my ($opt,$arg) = ($ARGV[0] =~ /^-([\@FCS])(.*)$/);
    last if ! defined $opt;
    # Add in regex [] ONLY single-character options that
    # REQUIRE an argument, except for the '@' debug switch.
    if ($opt =~ /[FC]/ && $arg eq "") {
        if (@ARGV < 2) {
            print "Argument required for option \"$opt\".\n";
            exit 1;
        }
        shift;
        $arg=$ARGV[0];
    }
    $DEBUG=($arg ? $arg : 1)                     if $opt eq '@';
    $FILTERS=$arg                                if $opt eq 'F';
    $CONTEXT=$arg                                if $opt eq 'C';
    $SUMMARY=1                                   if $opt eq 'S';
    shift;
}

#################################
# Validate command-line options #
#################################

&Usage if @ARGV < 2;
my @SDIFFARGS = @ARGV;
for (my $i=@SDIFFARGS-2;$i>=0;$i--) {
    if ($SDIFFARGS[$i] eq '-L') {
        splice(@SDIFFARGS,$i,2);
    } elsif ($SDIFFARGS[$i] eq '-u') {
        splice(@SDIFFARGS,$i,1);
    }
}

if ($SUMMARY) {
  $CONTEXT=undef;
  $FILTERS=""; # means 'all'
}

if (defined($CONTEXT) && $CONTEXT == 0) {
  $CONTEXT=undef;
  $FILTERS ||= "damco";
  $FILTERS =~ s/u//gi;
}

################
# Trap Signals #
################

sub SigCleanup { # private
     die "\nExiting: received signal \"" . $_[0] . "\".\n";
}
$SIG{'INT'}  = \&SigCleanup;
$SIG{'TERM'} = \&SigCleanup;
$SIG{'HUP'}  = \&SigCleanup;
$SIG{'QUIT'} = \&SigCleanup;
$SIG{'PIPE'} = \&SigCleanup;
$SIG{'ALRM'} = \&SigCleanup;

###############################
#   M A I N   P R O G R A M   #
###############################

# Optimization if no colors chosen for $UNCHANGED
my $UNCHOFF=$UNCHANGED ? $OFF : "";

# Save sdiff args
grep($_ = "'$_'", @SDIFFARGS);

# Get number of columns
my @stty = `stty -a | head -1`;
my ($cols) = $stty[0] =~ m#col(?:umns?)\s+(\d+)#;
   ($cols) = $stty[0] =~ m#(\d+)\s+col# if !defined $cols;
die "Cannot find number of columns in terminal?!? Got: '$cols'.\n"
     unless $cols =~ m#^\d+$#;

# Run sdiff
my @out = `sdiff --expand-tabs -w$cols @SDIFFARGS`;

# Compute position of special sdiff character
my $spos = int($cols/2 + 0.5)-1; # wow, it wasn't easy to figure out
$spos = 0 if $spos < 0;

# Add header and footer.
#my $surround = " " x $cols . "\n";
#substr($surround,$spos,1) = "*";
#unshift(@out,$surround);
#push(@out,$surround);

# Create colored lines
my @spec = ();
for (my $n=0;$n<@out;$n++) {
    my $line = $out[$n];
    chomp $line;
    my $spec = length($line) >= $spos ? substr($line,$spos,1) : " ";
    if ($spec eq ">") {
        $line .= (" " x ($cols - length($line)));
        $line = $INSERT.$line.$OFF."\n";
    } elsif ($spec eq "<") {
        $line .= (" " x ($cols - length($line)));
        $line = $DELETE.$line.$OFF."\n";
    } elsif ($spec eq "|") {
        $line .= (" " x ($cols - length($line)));
        $line = $CHANGED.$line.$OFF."\n";
    } elsif (($spec eq "/") || ($spec eq "\\")) {
        $line .= (" " x ($cols - length($line)));
        $line = $MISSINGNL.$line.$OFF."\n";
    } else {
        $line = $UNCHANGED.$line.$UNCHOFF."\n";
        $spec = " ";
    }
    $out[$n]  = $line;
    $spec[$n] = $spec;
}

my %SHOW_SPECS=();
$FILTERS ||= "udamco"; # all
$FILTERS .= "u" if defined($CONTEXT);
$SHOW_SPECS{" "}=1 if $FILTERS =~ /[us]/i; # unchanged/same
$SHOW_SPECS{"<"}=1 if $FILTERS =~ /d/i;    # deleted
$SHOW_SPECS{">"}=1 if $FILTERS =~ /a/i;    # added
$SHOW_SPECS{"|"}=1 if $FILTERS =~ /[mc]/i; # modified/changed
$SHOW_SPECS{"o"}=1 if $FILTERS =~ /o/i;    # omitted

if (defined($CONTEXT)) {
    my @newout  = ();
    my @newspec = ();
    my $prevend = -1;
    for (my $n=0;$n<@out;$n++) {
        my $spec = $spec[$n] || " ";
        next if $spec eq " " || !$SHOW_SPECS{$spec};
        my $start = $n - $CONTEXT; $start = 0 if $start < 0;
        $start = $prevend + 1 if $start <= $prevend;
        my $end   = $n;
        for (;;) {
            $n++ while $n < @out-1 && ($spec[$n+1] ne " " && $SHOW_SPECS{$spec[$n+1]});
            $end = $n + $CONTEXT;
            $end = @out-1 if $end >= @out;
            last if $end == $n;
            my @endspecs = @spec[$n+1 .. $end];
            if (scalar(grep($_ eq " " || !$SHOW_SPECS{$_},@endspecs)) == $end - $n) {
                last if $end == @out-1;
                last if $spec[$end+1] eq " " || !$SHOW_SPECS{$spec[$end+1]};
                $n=$end+1;
                next;
            }
            $end-- while $end > $n && ($spec[$end] eq " " || !$SHOW_SPECS{$spec[$end]});
            $n = $end;
            next;
        }
        my $numskipped = $start-$prevend-1;
        if ($numskipped > 1 && $SHOW_SPECS{"o"}) {
            my $line = "($numskipped lines omitted " . ($prevend == -1 ? "at start of file)" : "here)");
            $line .= (" " x ($spos - length($line))) . "- " . $line;
            $line .= (" " x ($cols - length($line)));
            $line = $OMITTED.$line.$OFF."\n";
            push(@newout,$line);
            push(@newspec,"o");
        } elsif ($numskipped == 1) {
            push(@newout,$out[$start-1]);
            push(@newspec,$spec[$start-1]);
        }
        push(@newout, @out[$start .. $end]);
        push(@newspec,@spec[$start .. $end]);
        $n=$end;
        $prevend = $end;
    }
    my $numskipped = @out-$prevend-1;
    if ($numskipped > 1 && $SHOW_SPECS{"o"}) {
        my $line = "($numskipped lines omitted " . ($prevend == -1 ? "totally in file)" : "at end of file)");
        $line .= (" " x ($spos - length($line))) . "- " . $line;
        $line .= (" " x ($cols - length($line)));
        $line = $OMITTED.$line.$OFF."\n";
        push(@newout,$line);
        push(@newspec,"o");
    } elsif ($numskipped == 1) {
        push(@newout,$out[-1]);
        push(@newspec,$spec[-1]);
    }
    @out  = @newout;
    @spec = @newspec;
}

if ($SUMMARY) {
   my %spec_cnts = ();
   foreach my $spec (@spec) {
       $spec_cnts{$spec}++;
   }
   printf "Unchanged: %d\n",($spec_cnts{' '} || 0);
   printf "Modified:  %d\n",($spec_cnts{'|'} || 0);
   printf "Added:     %d\n",($spec_cnts{'>'} || 0);
   printf "Deleted:   %d\n",($spec_cnts{'<'} || 0);
   exit 0;
}

if (defined($CONTEXT)) {
    print @out;
    exit 0;
}

for (my $n=0;$n<@out;$n++) {
    my $spec = $spec[$n] || " ";
    next unless $SHOW_SPECS{$spec};
    print $out[$n];
}

exit 0;

#############################
#   S U B R O U T I N E S   #
#############################

# None.

