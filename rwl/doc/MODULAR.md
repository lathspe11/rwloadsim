## Modular programming
The RWP\*Load Simulator has several features that make it suited for 
modular programming:
 * Separation of code into several source files
 * Variables can be declared private causing them to only be known in one file
 * Inclusion of source files in other source files
 * Search for source files via RWLOADSIM_PATH or in special public directory
 * Conditional compilation
Your simulation should consist of a set of rwl input files potentially 
with inclusion of other files via the $include directive, which allows 
for modular development.
You could e.g. have files that contain database declarations, files 
that contain SQL and procedure declarations, and files that contain the 
actual simulation.
Although possible, you are advised to only run one simulation (that is 
one thread execution using the run keyword).  

Unless -no-nameexpand option or the $nameexpand:off directive is used, 
environment names in either $NAME or ${NAME} syntax will be expanded 
before opening files.
This allows for advanced scripting.

## Including source files 
As an alternative to having a long list of file names provided to 
rwloadsim, you can create one "master" file that includes other files 
using a $include directive.
Such files can themselves contain a $include directive for multiple 
levels of inclusion.
You can only use $include when you are not parsing a statement, 
declaration or thread execution and the file included must similarly 
contain complete rwl syntax.
The filename to be included is either put between double quote 
characters or between '<' and '>' characters similar to how it is done 
in C.
In the first case, the file will first be attempted opened in the 
current directory, then in the directories provided in the 
RWLOADSIM_PATH environment variable.
In the latter case, the current directory will not be searched first.
Using the -u option, the rwl/public directory will be searched before 
any directories mention in RWLOADSIM_PATH.

Environment variable expansion takes place as it does for files name on 
the command line.

## Conditional execution
You can use the three directives $if $then, $else, $endif similar to 
how #if, #else, #endif are used in C to conditionally include or 
exclude code.
The $if ... $then directive takes a simple expression that does not 
include function calls (neither most standard functions nor user 
defined functions); the simplified expression must be between $if and 
$then and all of these three parts must be on the same source line.
If the expression is non-zero, the code until a subsequent $else is 
included, otherwise the code after $else and until $endif is included.
The $else part can be omitted.

The expression that is between the $if and $then directives can only 
consist of constants or global or public variables of type integer, 
double or string (including environment variables such as $USER), the 
function call access() or the function defined(), which takes an 
identifier as its single argument.
No other standard or declared functions can be used.

Some examples are shown here:
```
integer one := 1;

$if one $then
# This code is included
printline "this will be printed";
$else
# This code is not included
$include:"subfile.rwl"
$endif

$if one-1 $then printline "nothing will be printed"; $endif

$if  defined(one   ) $then 
   printline "one="||one;
$endif

$if $PRECISION="high" $then $dformat:%.8f $endif
printline "sqrt(2) =", sqrt(2);
```
If the above is given to rwloadsim with PRECISION=high in the environment,
what really will be parsed is only 
the following:
```
# This code is included
printline "this will be printed";

   prinitline "one="||one;

printline "sqrt(2) =", sqrt(2);

The file name 'subfile.rwl' will not be recursively read and the last 
printline will not be included.

The $dformat directive will be processed only if the environment 
variable $PRECISION has the value "high", causing the last printline to 
show the square root of 2 with 8 digits precision.

Note the following points about these directives:

 * The $if and $then must be on the same line and as an exception to the general rule about directives, white-space is allowed between $if and $then.  Effectively, the text between $if and the $then is the expression that evaluates to true (non-zero) or false.  
 * Only global or private variables are used in the $if $then directive expression.  
 * You are recommended not to use the variables used in $if .. $then for other purposes in your rwloadsim program.  
 * The directives are not line-oriented (unlike #if etc in C), but the $if $then directive itself cannot span more than one line.  
 * The source code in the part that is not included is still scanned, and it must therefore be legal rwloadsim input; only parsing is avoided by these directives.  
 * Other directives found in the excluded part of $if/$else/$endif are not processed.  
 * Each source file provided to rwloadsim must have matching $if $then/$else/$endif directives.

One potential use-case for these directives is to comment out large 
parts of code by surrounding if with $if 0 $then and $endif