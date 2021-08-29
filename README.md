# JREN
JREN.BAT  Rename files/folders using regular expressions
JREN.BAT is a hybrid JScript/batch utility that renames files or folders by performing a regular expression search and replace on the names. The utility is pure script that will run natively on any Windows machine from XP forward.

Use JREN /? to get help from the command line.

Important Note: It is a good idea to initially use the /T (test) option when developing a JREN command. This will print out the rename operations that would be attempted, without actually renaming anything.

Below are a few examples of usage. Often there are multiple ways to achieve the same result.

1) Convert all .txt files to lower case in the current directory:

Using the search regular expression and the /L (lower case) and /I (ignore case) options
CODE: SELECT ALL

jren ".*\.txt$" "$&" /l /i

Using the search regular expression and the /J (JScript expression) and /I options
CODE: SELECT ALL

jren ".*\.txt$" "lc($0)" /j /i

Using the /FM (file mask) and /L options
CODE: SELECT ALL

jren "" "" /l /fm "*.txt"

Using the /RFM (regular expression file mask) and /L options
CODE: SELECT ALL

jren "" "" /l /rfm "\.txt$"


2) Rename all 76 ".jpg" files in the "C:\photos\Christmas 2014" directory to an increasing padded number followed by a constant string. Assume the current directory is "C:\photos". Resulting file names should look like "01_Christmas2014.jpg", "02_Christmas2014.jpg", etc.

Using the search regular expression and the /P (root Path), /NPAD ($n pad width), /J, and /I options
CODE: SELECT ALL

jren ".*\.jpg$" "$n+'_Christmas2014.jpg'" /p "Christmas 2014" /npad 2 /j /i

Using the /P, /NPAD, /J, and /FM options
CODE: SELECT ALL

jren "^.*" "$n+'_Christmas2014.jpg'" /p "Christmas 2014" /fm "*.jpg" /npad 2 /j

Using the /P, /NPAD, /J, and /RFM options
CODE: SELECT ALL

jren "^.*" "$n+'_Christmas2014.jpg'" /p "Christmas 2014" /rfm "\.jpg$" /npad 2 /j


3) Rename folders, moving a numeric suffix to the front of the name, padded to 2 digits. Recursively apply this only to folders with "test" as a parent folder, except ignore folders under the root path of "C:\someName\test". Assume the current directory is the root path where recursion starts.

starting folder hierarchy:
CODE: SELECT ALL

C:\someName\test
              proj 1
                someName 1
                  cat 1
                  dog 2
                  zebra 3
                anotherName 2
                test
                  car 1
                  ignore
                  motorcycle 11
                  train 3
              proj 2
                test
                  eel 3
                  fish 2
                  turtle 1
              test
                baseball 1
                basketball 2
                football 3

desired result (default sort order swaps positions of folders)
CODE: SELECT ALL

C:\someName\test
              proj 1
                someName 1
                  cat 1
                  dog 2
                  zebra 3
                anotherName 2
                test
                  01 car
                  03 train
                  11 motorcycle
                  ignore
              proj 2
                test
                  01 turtle
                  02 fish
                  03 eel
              test
                01 baseball
                02 basketball
                03 football

Using the /D (rename Directories), /S (recurse subdirectories), /PM (path mask), /PX (path exclusion), and /J options
CODE: SELECT ALL

  jren "(.*) (\d+)$" "lpad($2,'00')+' '+$1" /j /d /s /pm "**\test" /px "/p:"

Using the /D, /S, /J and /RPM (regular expression Path Mask) options
CODE: SELECT ALL

  jren "(.*) (\d+)$" "lpad($2,'00')+' '+$1" /j /d /s /rpm "c:\\somename\\test.*\\test$"


4) Hack to recursively iterate files or folders using sophisticated filtering

When JREN renames a file, it lists the original full path in quotes, followed by -->, followed by the new name in quotes. For example, using the example from 3), one line of output would look like:
CODE: SELECT ALL

"C:\someName\test\test\baseball 1" --> "001 baseball"

It is fairly easy to add the /T option and process the result with FOR /F to selectively iterate files or folders. Using a Search that returns the entire name, and a Replace of "", coupled with the /T option, would yield results like:
CODE: SELECT ALL

"C:\someName\test\test\baseball 1" --> ""

Then you just need to add FOR /F with the weird delims syntax to set the delimiter to a quote:
CODE: SELECT ALL

  @echo off
  for /f delims^=^" %%F in (
    'jren ".* \d+$" "" /t /d /s /pm "/p:**\test"
  ) do echo %%F

yields:
CODE: SELECT ALL

C:\someName\test\proj 1\test\car 1
C:\someName\test\proj 1\test\motorcycle 11
C:\someName\test\proj 1\test\train 3
C:\someName\test\proj 2\test\eel 3
C:\someName\test\proj 2\test\fish 2
C:\someName\test\proj 2\test\turtle 1
C:\someName\test\test\baseball 1
C:\someName\test\test\basketball 2
C:\someName\test\test\football 3



Here is JREN.BAT version 1
CODE: SELECT ALL

@if (@X)==(@Y) @end /* Harmless hybrid line that begins a JScript comment
::JREN.BAT version 1.0
::
::  Release History:
::    2014-11-30 v1.0: Initial release
::
::************ Documentation ***********
:::
:::JREN  Search  Replace  [/Option  [Value]]...
:::JREN  /?[REGEX|REPLACE|VERSION]
:::
:::  Rename files in the current directory by performing a regular expression
:::  search/replace on the old file name to generate the new file name.
:::  This includes read only, hidden, and system files.
:::
:::  Search  - By default, this is a case sensitive JScript (ECMA) regular
:::            expression expressed as a string. The search is applied globally
:::            to the entire file name.
:::
:::            JScript regex syntax documentation is available at
:::            http://msdn.microsoft.com/en-us/library/ae5bf541(v=vs.80).aspx
:::
:::  Replace - By default, this is the string to be used as a replacement for
:::            each found search expression. Full support is provided for
:::            substituion patterns available to the JScript replace method.
:::
:::            For example, $& represents the portion of the source that matched
:::            the entire search pattern, $1 represents the first captured
:::            submatch, $2 the second captured submatch, etc. A $ literal
:::            can be escaped as $$.
:::
:::            An empty replacement string must be represented as "".
:::
:::            Replace substitution pattern syntax is fully documented at
:::            http://msdn.microsoft.com/en-US/library/efy6s3e6(v=vs.80).aspx
:::
:::  Options:  Behavior may be altered by appending one or more options.
:::  The option names are case insensitive, and may appear in any order
:::  after the Replace argument.
:::
:::      /D  - Rename Directories instead of files.
:::
:::      /I  - Ignore case when matching.
:::
:::      /FM FileOrFolderMask
:::
:::            Only rename files or folders that match any of the pattern(s)
:::            using standard wildcards. Multiple patterns are delimited by a
:::            pipe (|). Only complete name matches count.
:::
:::              * matches any 0 or more characters
:::              ? matches any 0 or 1 character except .
:::
:::      /FX FileOrFolderExclusion
:::
:::            Exclude files or folders that match any of the pattern(s)
:::            using standard wildcards. Multiple patterns are delimited by a
:::            pipe (|). Only complete name matches count.
:::
:::              * matches any 0 or more characters
:::              ? matches any 0 or 1 character except .
:::
:::      /J  - Treat Replace as a JScript expression.
:::            The following variables contain details about each match:
:::
:::              $0 = the substring that matched the Search
:::              $1 through $n = captured submatch strings
:::              $off = the offset where the match occurred
:::              $src = the original source string
:::
:::            The following are also available:
:::
:::              $n = An incrementing number for use in the name. The value
:::                   is reset to the /NBEG value for each directory.
:::                   It increases by the /NINC value for each renamed file.
:::                   The value may be zero padded to the width specified by
:::                   the /NPAD value.
:::
:::              lc(str)
:::
:::                 Convert str to lower case. Shorthand for str.toLowerCase().
:::
:::              uc(str)
:::
:::                 Convert str to upper case. Shorthand for str.toUpperCase().
:::
:::              lpad(string,pad)
:::
:::                 Used to left pad string str to a minimum length. If the
:::                 str already has length >= the pad string length, then no
:::                 change is made. Otherwise it left pads the value with the
:::                 characters of the pad string to the length of pad.
:::
:::                 Examples:
:::                    lpad(15,'0000')    returns "0015"
:::                    lpad(15,'    ')    returns "  15"
:::                    lpad(19011,'0000') returns "19011"
:::
:::              rpad(string,pad)
:::
:::                 Used to right pad string str to a minimum length. If the
:::                 str already has length >= the pad string length, then no
:::                 change is made. Otherwise it right pads the value with the
:::                 characters of the pad string to the length of pad.
:::
:::      /L  - Convert names to Lower case. Entire names can be converted to
:::            lower case without any other changes by using empty strings ("")
:::            for both Search and Replace.
:::
:::      /NBEG BeginValue
:::
:::            Specifies the initial $n value for each directory. The value
:::            must be an integer >= 0. The default value is 1.
:::
:::      /NINC IncrementValue
:::
:::            Specifies the amount $n is incremented after each rename.
:::            The value must be an integer >=1. The default value is 1.
:::
:::      /NPAD MinWidth
:::
:::            Specifies the minimum width for each $n value. If the $n value
:::            has fewer digits than MinWidth, then the value is zero padded
:::            on the left to achieve the MinWidth. The value must be >= 1.
:::            The default value is 3.
:::
:::      /P RootPath
:::
:::            Specifies the path where the rename is to take place.
:::            The default of . represents the current directory.
:::            Wildcards are not allowed.
:::
:::      /PM PathMask
:::
:::            Only rename files or folders whose parent folder path matches
:::            any of the PathMask pattern(s) using augmented wildcards.
:::            Multiple patterns are delimited by a pipe (|). Only full path
:::            matches count.
:::
:::              /P:  matches the root path specified by option /P
:::              **   matches any 0 or more characters
:::              *    matches any 0 or more characters except \
:::              ?    matches any 0 or 1 character except . or \
:::
:::           This option is only useful if the /S option is used.
:::
:::      /PX PathExclusion
:::
:::            Exclude files or folders whose parent folder path matches any
:::            of the PathExclusion pattern(s) using augmented wildcards.
:::            Multiple patterns are delimited by a pipe (|). Only full path
:::            matches count.
:::
:::              /P:  matches the root path specified by option /P
:::              **   matches any 0 or more characters
:::              *    matches any 0 or more characters except \
:::              ?    matches any 0 or 1 character except . or \
:::
:::           This option is only useful if the /S option is used.
:::
:::      /RFM RegexFileOrFoldereMask
:::
:::            Only rename files or folders that match the regular expression,
:::            ignoring case. Partial name matches count.
:::
:::      /RFX RegexFileOrFolderExclusion
:::
:::            Exclude files or folders that match the regular Expression,
:::            ignoring case. Partial name matches count.
:::
:::      /RPM RegexPathMask
:::
:::            Only rename files or folders whose parent folder path matches the
:::            RegexPathMask regular expression, ignoring case. Partial path
:::            matches count. This option is really only useful if the /S
:::            option is used.
:::
:::      /RPX RegexPathExclusion
:::
:::            Exclude files or folders whose parent folder path matches the
:::            RegexPathExclusion regular expression, ignoring case. Partial
:::            path matches count. This option is really only useful if the
:::            /S option is used.
:::
:::      /Q  - Do not list the renamed files/folders (Quiet mode).
:::
:::      /S  - Recurse Subdirectories.
:::
:::      /T  - List the rename operations that would be attempted,
:::            but do not rename anything. (Test mode)
:::
:::      /U  - Convert names to Upper case. Entire names can be converted to
:::            upper case without any other changes by using empty strings ("")
:::            for both Search and Replace.
:::
:::  Help is available by supplying a single argument beginning with /?:
:::
:::      /?        - Writes this help documentation to stdout.
:::
:::      /?REGEX   - Opens up Microsoft's JScript regular expression
:::                  documentation within your browser.
:::
:::      /?REPLACE - Opens up Microsoft's JScript REPLACE documentation
:::                  within your browser.
:::
:::      /?VERSION - Writes the JREPL version number to stdout.
:::
:::  JREN.BAT was written by Dave Benham, and originally posted at
:::  http://www.dostips.com/forum/viewtopic.php?f=3&t=6081
:::

::************ Batch portion ***********
@echo off
setlocal disableDelayedExpansion

if .%2 equ . (
  if "%~1" equ "/?" (
    for /f "tokens=* delims=:" %%A in ('findstr "^:::" "%~f0"') do @echo(%%A
    exit /b 0
  ) else if /i "%~1" equ "/?regex" (
    explorer "http://msdn.microsoft.com/en-us/library/ae5bf541(v=vs.80).aspx"
    exit /b 0
  ) else if /i "%~1" equ "/?replace" (
    explorer "http://msdn.microsoft.com/en-US/library/efy6s3e6(v=vs.80).aspx"
    exit /b 0
  ) else if /i "%~1" equ "/?version" (
    for /f "tokens=* delims=:" %%A in ('findstr "^::JREN\.BAT" "%~f0"') do @echo(%%A
    exit /b 0
  ) else (
    call :err "Insufficient arguments"
    exit /b 2
  )
)

:: Define options
set "options= /D: /FM:"" /FX:"" /G: /I: /J: /L: /NBEG:1 /NINC:1 /NPAD:3 /P:. /PM:"" /PX:"" /RFM:"" /RFX:"" /RPM:"" /RPX:"" /Q: /S: /T: /U: "

:: Set default option values
for %%O in (%options%) do for /f "tokens=1,* delims=:" %%A in ("%%O") do set "%%A=%%~B"

:: Get options
:loop
if not "%~3"=="" (
  set "test=%~3"
  setlocal enableDelayedExpansion
  if "!test:~0,1!" neq "/" (
    call :err "Too many arguments"
    exit /b 2
  )
  set "test=!options:*%~3:=! "
  if "!test!"=="!options! " (
      endlocal
      call :err "Invalid option %~3"
      exit /b 2
  ) else if "!test:~0,1!"==" " (
      endlocal
      set "%~3=1"
      if /i "%~3" equ "/L" set "/U="
      if /i "%~3" equ "/U" set "/L="
  ) else (
      endlocal
      if %4. equ . (
        call :err "Missing %~3 value"
        exit /b 2
      )
      set "%~3=%~4"
      shift /3
  )
  shift /3
  goto :loop
)

:: Execute
cscript //E:JScript //nologo "%~f0" %1 %2
exit /b %errorlevel%

:err
>&2 (
  echo ERROR: %~1
)
exit /b

************* JScript portion **********/
var $n
var _g=new Object();
try {

  _g.defineReplFunc=function() {
    eval(_g.replFunc);
  }

  _g.main=function() {

    function err( msg, rtn ) {
      WScript.StdErr.WriteLine(msg);
      if (rtn) WScript.Quit(rtn);
    }

    function BuildRegex( loc, regex, options ) {
      try {
        return regex ? new RegExp( regex, options ) : false;
      } catch(e) {
        err( 'Invalid '+loc+' regular expression: '+e.message, 1);
      }
    }

    function GetInt( loc, numStr, minVal ) {
      var n = parseInt( numStr );
      if (isNaN(n) || n<minVal) {
        err( 'Error: Invalid '+loc+' value', 1 );
      }
      return n;
    }

    var env = WScript.CreateObject("WScript.Shell").Environment("Process"),
        fso = new ActiveXObject("Scripting.FileSystemObject");

    try {
      var root = fso.GetFolder( env('/P') );
    } catch(e) {
      err( 'Invalid /P path: '+e.message, 1 );
    }

    function MaskRepl($0) {
      switch ($0) {
        case '/P:':
        case '/p:': return root.Path.replace(/[.^$*+?()[{\\|]/g,"\\$&");
        case '**':  return '.*';
        case '*':   return '[^\\\\]*';
        case '?':   return '[^\\\\.]?';
        default:    return '\\'+$0;
      }
    }

    var args=WScript.Arguments,
        search = BuildRegex( 'Search', args.Item(0), env('/G')?'g':'' + env('/I')?'i':'' ),
        replace=args.Item(1),
        rMask = BuildRegex( '/RFM', env('/RFM'), 'i' ),
        rExclude = BuildRegex( '/RFX', env('/RFX'), 'i' ),
        rPathMask = BuildRegex( '/RPM', env('/RPM'), 'i' ),
        rPathExclude = BuildRegex( '/RPX', env('/RPX'), 'i' ),
        regex = new RegExp("/P:|[*][*]|[*]|[?]|[.^$+()[{\\\\]","ig");
        mask = env('/FM') ? new RegExp( '^(?:' + env('/FM').replace(regex,MaskRepl) + ')$', 'i' ) : false;
        exclude = env('/FX') ? new RegExp( '^(?:' + env('/FX').replace(regex,MaskRepl) + ')$', 'i' ) : false;
        pathMask = env('/PM') ? new RegExp( '^(?:' + env('/PM').replace(regex,MaskRepl) + ')$', 'i' ) : false;
        pathExclude = env('/PX') ? new RegExp( '^(?:' + env('/PX').replace(regex,MaskRepl) + ')$', 'i' ) : false;
        dirs = env('/D'),
        upper = env('/U'),
        lower = env('/L'),
        recurse = env('/S'),
        jscript = env('/J'),
        beg = GetInt( '/NBEG', env('/NBEG'), 0 ),
        inc = GetInt( '/NINC', env('/NINC'), 1 ),
        pad = GetInt( '/NPAD', env('/NPAD'), 1 ),
        padStr = Array( pad+1 ).join('0'),
        test = env('/T'),
        quiet = env('/Q');

    function ProcessFolder( folder ) {
      var i, a=[];
      var num = beg;
      if (recurse || dirs) {
        var folders = new Enumerator(folder.SubFolders);
        for( i=0 ; !folders.atEnd(); folders.moveNext()) {
          a[i++]=folders.item()
          if (recurse) ProcessFolder(folders.item());
        }
      }
      if ( (!pathMask || pathMask.test(folder.Path)) &&
           (!rPathMask || rPathMask.test(folder.Path)) &&
           (!pathExclude || !pathExclude.test(folder.Path)) &&
           (!rPathExclude || !rPathExclude.test(folder.Path)) ) {
        if (!dirs) {
          a=[];
          var files = new Enumerator(folder.Files);
          for (i=0; !files.atEnd(); files.moveNext()) a[i++]=files.item();
        }
        for (i=0; i<a.length; i++) {
          var oldName = a[i].Name;
          if ( (!mask || mask.test(oldName)) &&
               (!rMask || rMask.test(oldName)) &&
               (!exclude || !exclude.test(oldName)) &&
               (!rExclude || !rExclude.test(oldName)) ) {
            if (jscript) {
              $n = num.toString();
              if ($n.length<pad) $n = (padStr+$n).slice(-pad);
            }
            try {
              var newName = oldName.replace( search, replace );
            } catch(e) {
              err( 'Replace error: '+e.message, 1 );
            }
            newName=newName.replace( /[ .]+$/, "" );
            if (jscript && newName.search(/[<>|:/\\*?"\x00-\x1F]/)>=0) err('Error: Invalid file name character in Replace',1);
            if (upper) newName = uc(newName);
            if (lower) newName = lc(newName);
            if (newName != oldName) {
              try {
                var oldPath=a[i].Path;
                if (!test) {
                  if (a[i].Name.toUpperCase() == newName.toUpperCase()) a[i].Name = '_{JREN_tempName}_'
                  a[i].Name = newName;
                }
                if (!quiet) WScript.echo( '"'+oldPath+'"  -->  "'+(test?newName:a[i].Name)+'"' );
              } catch(e) {
                err( 'Unable to rename "'+a[i].Path+'"  -->  "'+newName+'" : "'+e.message, 0 );
              }
              num+=inc;
            }
          }
        }
      }
    }

    if (jscript) {
      var regex=new RegExp('.|'+search,''),
          cnt;
      'x'.replace( regex, function(){cnt=arguments.length-2; return '';} );
      _g.replFunc='_g.replFunc=function($0';
      for (var i=1; i<cnt; i++) _g.replFunc+=',$'+i;
      _g.replFunc+=',$off,$src){return eval(_g.replace);}';
      _g.defineReplFunc();
      _g.replace = replace;
      replace = _g.replFunc;
    } else if (replace.search(/[<>|:/\\*?"\x00-\x1F]/)>=0) err('Error: Invalid file name character in Replace',1);

    ProcessFolder( root );
    WScript.Quit(0);
  }

  _g.main();

} catch(e) {
  WScript.StdErr.WriteLine("JScript runtime error: "+e.message);
  WScript.Quit(1);
}

function lc(str) { return str.toLowerCase(); }

function uc(str) { return str.toUpperCase(); }

function lpad( val, pad ) {
  var rtn=val.toString();
  return (rtn.length<pad.length) ? (pad+rtn).slice(-pad.length) : val;
}

function rpad( val, pad ) {
  var rtn=val.toString();
  return (rtn.length<pad.length) ? (rtn+pad).slice(0,pad.length) : val;
}


Dave Benham
