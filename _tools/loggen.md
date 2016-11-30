---
title: logGen
layout: single
---

logGen is a command-line utility (for now) for detecting filesystem changes after a preference change or package installation. This is primarily useful when creating your own .pkg files so you know what you need to package.

You can download the logGen installer here: [loggen-2.2.dmg]({{ site.url }}/downloads/loggen-2.2.dmg) 

LogGen is written by Phil Holland with changes by Dave Pugh, Chris Grieb and J. Billings in 2008. The script was available on http://lsa.umich.edu and when it disappeared I decided to host it for Mac Admins. I still occasionally use it to capture files from hostile package installers.

Please read the updated documentation by typing `perldoc /usr/local/sbin/logGen` .


# Documentation

~~~ text
LOGGEN(1)             User Contributed Perl Documentation            LOGGEN(1)



NAME
       logGen - report filesystem changes

SYNOPSYS
       sudo /usr/local/sbin/logGen [--all|-a] [--fast|-f] [--root|-r <dir>]
       [--user|-u] [orig.dat]

       sudo /usr/local/sbin/logGen [--all|-a] [--fast|-f] [--root|-r <dir>]
       [--user|-u] [new.dat] [orig.dat]

       sudo /usr/local/sbin/logGen [--all|-a] [--fast|-f] [--root|-r <dir>]
       [--user|-u] [new.dat] [orig.dat] > changes.txt

DESCRIPTION
       logGen can be used to detect what files have changed as a result of a
       configuration change or installing a package.  It accomplishes this by
       utilizing a number of methods, but mostly using the modification date
       and a checksum of each file.  Lists will be generated for files that
       are added, changed, or deleted, and will include only the directory if
       everything within it has been added, changed, or deleted.  A number of
       directories are automatically ignored in the search including your home
       directory, temporary directories, network mounts, and non-root volumes.

       As with many tools, logGen cannot accurately detect changes in resource
       forks of files.

       Before performing any changes or installations you'd like to detect,
       take a baseline snapshot of the filesystem by running:

           sudo /usr/local/sbin/logGen orig.dat

       This will write out a data file (orig.dat) containing a listing of each
       file and the information logGen has recorded about each file.  This
       first pass can taken a very long time (even upwards of 30 minutes)
       depending on the speed of your machine and the number of files on your
       disk.

       Next, make the changes you'd like to detect, such as a preference
       change, installing new software, etc, and run logGen a second time.  It
       is recommended (although not required) to redirect STDOUT to a file for
       later examination.

           sudo /usr/local/sbin/logGen new.dat orig.dat > changes.txt

       This will write out a new data file (new.dat) containing a new, current
       listing of each file and the information logGen has recorded about each
       file.  Next, the data is compared between the new.dat and orig.dat
       files and the changes are summarized and printed to STDOUT, and in this
       example saved to changes.txt.  This second execution of logGen gener-
       ally takes much less time than the first.

       All of the filenames are changable.  If you omit a filename for the
       original data file (orig.dat in the above examples) it will default to
       <currentEpochTime>.log, such as "1076949440.log".

OPTIONS
       Three options are available:

       --all (or -a): Check all directories, including /tmp, /var, etc.  This
       still ignores things like /Network and /Volumes, though, to avoid
       checking lots of things you really shouldn't check.

       --fast (or -f): Skips MD5 checks of files - the only downside to this
       is that files whose timestamps have changed but whose contents remain
       identical will be reported as changed even though they didn't.

       --root <dir> (or -r <dir>): Sets the root directory of the search to
       the specified directory.  This could be useful if, for example, you're
       just looking for what preference changed in /Library/Preferences/

       --user (or -u): Includes the /Users directory in the scan.  Normally
       this directory is ignored.

       The order of the options DOES matter - They should be typed in the same
       order as they are listed here.  I felt lazy programming that day...
       Sorry.

EXAMPLE
        % sudo /usr/local/sbin/logGen orig.dat

        logGen  --  version 1.0
        Copyright 2003 - The Regents of the University of Michigan
        All Rights Reserved

        361883 new files:
        ---------------
        /
        ---------------
        0 changed files
        0 deleted files
        0 files with resource forks
        0 files with special permissions

        % sudo /usr/local/sbin/logGen new.dat orig.dat

        logGen  --  version 1.0
        Copyright 2003 - The Regents of the University of Michigan
        All Rights Reserved

        1 new files:
        ---------------
        /Library/NewDir/
        ---------------
        1 changed files:
        ---------------
        /Library/TestDir2/someFile
        ---------------
        1 deleted files:
        ---------------
        /Library/TestDir/
        ---------------
        1 files with resource forks:
        ---------------
        /Library/iconFile
        ---------------
        2 files with special permissions:
        ---------------
        /Library/aFileSetUID ( setuid )
        /Library/anotherFile ( setgid world_writable )
        ---------------

AUTHORS
       Originally written by Phil Holland at the University of Michigan.
       Numerous changes provided by Dave Pugh at the University of Michigan.
       Questions, requests, comments, and code changes should be sent to
       lsa-dev-osx@umich.edu

COPYRIGHT
       COPYRIGHT 2005-2008 THE REGENTS OF THE UNIVERSITY OF MICHIGAN ALL
       RIGHTS RESERVED

       PERMISSION IS GRANTED TO USE, COPY, CREATE DERIVATIVE WORKS AND REDIS-
       TRIBUTE THIS SOFTWARE AND SUCH DERIVATIVE WORKS FOR ANY PURPOSE, SO
       LONG AS NO FEE IS CHARGED, AND SO LONG AS THE COPYRIGHT NOTICE ABOVE,
       THIS GRANT OF PERMISSION, AND THE DISCLAIMER BELOW APPEAR IN ALL COPIES
       MADE; AND SO LONG AS THE NAME OF THE UNIVERSITY OF MICHIGAN IS NOT USED
       IN ANY ADVERTISING OR PUBLICITY PERTAINING TO THE USE OR DISTRIBUTION
       OF THIS SOFTWARE WITHOUT SPECIFIC, WRITTEN PRIOR AUTHORIZATION.

       THIS SOFTWARE IS PROVIDED AS IS, WITHOUT REPRESENTATION FROM THE UNI-
       VERSITY OF MICHIGAN AS TO ITS FITNESS FOR ANY PURPOSE, AND WITHOUT WAR-
       RANTY BY THE UNIVERSITY OF MICHIGAN OF ANY KIND, EITHER EXPRESS OR
       IMPLIED, INCLUDING WITHOUT LIMITATION THE IMPLIED WARRANTIES OF MER-
       CHANTABILITY AND FITNESS FOR A PARTICULAR PURPOSE. THE REGENTS OF THE
       UNIVERSITY OF MICHIGAN SHALL NOT BE LIABLE FOR ANY DAMAGES, INCLUDING
       SPECIAL, INDIRECT, INCIDENTAL, OR CONSEQUENTIAL DAMAGES, WITH RESPECT
       TO ANY CLAIM ARISING OUT OF OR IN CONNECTION WITH THE USE OF THE SOFT-
       WARE, EVEN IF IT HAS BEEN OR IS HEREAFTER ADVISED OF THE POSSIBILITY OF
       SUCH DAMAGES.

       If you do make any changes, it would be appreciated if you submitted
       them back to lsa-dev-osx@umich.edu



perl v5.8.6                       2008-02-12                         LOGGEN(1)
~~~