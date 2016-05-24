
Release Notes
#############

2016.01 - January 2016
**********************

Current Revision and Compile Order
==================================

The following table lists the files and revision, starting with the files that 
need to be compiled first.

+---------------------+----------++---------------------+----------+
| File                | Revision || File                | Revision |
+=====================+==========++=====================+==========+
| AlertLogPkg.vhd     | 2016.01  || OsvvmGlobalPkg.vhd  | 2015.01  |
+---------------------+----------++---------------------+----------+
| CoveragePkg.vhd     | 2016.01  || RandomBasePkg.vhd   | 2015.06  |
+---------------------+----------++---------------------+----------+
| MemoryPkg.vhd       | 2016.01  || RandomPkg.vhd       | 2015.06  |
+---------------------+----------++---------------------+----------+
| MessagePkg.vhd      | 2015.01  || SortListPkg_int.vhd | 2015.01  |
+---------------------+----------++---------------------+----------+
| NamePkg.vhd         | 2015.06  || TextUtilPkg.vhd     | 2016.01  |
+---------------------+----------++---------------------+----------+
| OsvvmContext.vhd    | 2015.06  || TranscriptPkg.vhd   | 2016.01  |
+---------------------+----------++---------------------+----------+


.. rubric:: AlertLogPkg 2016.01

* Fixed bug that kept more than 32 bins from being used.

.. rubric:: CoveragePkg 2016.01

* Revised ConcatenateBins so it does not call Alert to allow it to be a pure 
  function.
* Added bounds checking to ICover so that if ICover is called with the 
  wrong length of integer_vector, it will cause an Alert FAILURE.

.. rubric:: MemoryPkg and TextUtilPkg 2016.01

* Changed L.all(1) to L.all(L’left). GHDL does not default objects of type line 
  to have an index of 1.

.. rubric:: TranscriptPkg 2016.01

* Minor reorganization of code so that all calls to TranscriptOpen eventually 
  call the same code.

	
2015.06 - June 2015
*******************

.. rubric:: MemoryPkg (New) 2015.06

* New to OSVVM. Package with protected type for implementing memories.
  Methods MemInit, MemRead, MemWrite, MemErase, FileReadH, FileReadB, FileWriteH, FileWriteB, SetAlertLogID, Deallocate.

.. rubric:: AlertLogPkg 2015.06

* Added AffirmIf.
* Added PASSED log level.
* Added IncAlertCount as a silent alert (Used by CoveragePkg).
* Revised ReportAlerts to print number of affirmations checked.
* Added GetAffirmCheckCount, IncAffirmCheckCOunt.
* ClearAlerts also clears affirmations.
* Added CreateHierarchy final parameter to GetAlertLogID.
* Added a Get for every Set.
* Moved EmptyOrCommentLine to TextUtilPkg and revised.

.. rubric:: CoveragePkg 2015.06

* Implemented Mirroring for WriteBin and WriteCovHoles. When in ILLEGAL_OFF 
  mode, added IncAlertCount (as a silent alert).
* Added SetAlertLogID(Name, ParentID, CreateHierarchy).
* Updated Alert output format.
* Added AddCross(CovMatrix?Type).
* Deprecated AddBins(CovMatrix?Type).
* Moved EmptyOrCommentLine to TextUtilPkg and revised.

.. rubric:: TextUtilPkg (New) 2015.06

* Shared utilities for file reading. SkipWhiteSpace, EmptyOrCommentLine, 
  ReadHexToken, ReadBinaryToken.

.. rubric:: RandomPkg 2015.06

* Revised calls to Alert to 2015.03 preferred format with AlertLogID first.

.. rubric:: NamePkg 2015.06

* Added input parameter to Get to specify a return value when NamePtr is not 
  initialized.

.. rubric:: RandomBasePkg 2015.06

* Changed GenRandSeed to impure since it calls Alert (and Alert is a parent of a 
  call to a protected type method).

.. rubric:: OsvvmContext.pkg 2015.06

* Added MemoryPkg


2015.03 - March 2015
********************

.. rubric:: AlertLogPkg 2015.03

* Added AlertIfEqual, AlertIfNotEqual, and AlertIfDiff (file).
* Added ReadLogEnables to initialize LogEnables from a file.
* Added ReportNonZeroAlerts.
* Added PathTail to extract an instance name from MyEntity'PathName.
* Added ReportLogEnables and GetAlertLogName.

.. seealso::
   
   See AlertLogPkg_User_Guide.pdf for details.
	 
	 .. TODO:: replace this pdf by a doc link

For hierarchy mode, AlertIfEqual, AlertIfNotEqual, and AlertIfDiff have the 
AlertLogID parameter first. Overloading was added for AlertIf and AlertIfNot 
to make these consistent. Now with multiple parameters, it is easier to 
remember that the AlertLogID parameter is first. The older AlertIf and 
AlertIfNot with the AlertLogID as the second parameter were kept for backward 
compatibility, but are considered bad practice to use in new code.

* Added ParentID parameter to FindAlertLogID. This is necessary to correctly 
  find an ID within an entity that is used more than once.

**Bug Fixes:**

* Bug fix: Updated GetAlertLogID to use the two parameter FindAlertLogID.
  Without this fix, Alerts with the same name incorrectly use the same AlertLogID.
* Bug fix: Updated NewAlertLogRec (called by GetAlertLogID) so a new record gets 
  Alert and Log enables based on its ParentID rather than the ALERTLOG_BASE_ID. 
  Issue, if created an Comp1_AlertLogID, and disabled a level (such as WARNING), 
  and then created a childID of Comp1_AlertLogID, WARNING would not be disabled 
  in childID.
* Bug fix: Updated ClearAlerts to correctly set stop counts (broke since it 
  previously did not use named association). Without this fix, after calling 
  ClearAlerts, a single FAILURE would not stop a simulation, however, a single 
  WARNING would stop a simulation.

	
2015.01 - January 2015
**********************

.. rubric:: OsvvmContext (New) 2015.01

OsvvmContext is a context declaration. Rather than referencing osvvm packages 
with individual use clauses, instead use a single context reference:

library osvvm ;
context osvvm.OsvvmContext ;

.. rubric:: OsvvmGlobalPkg (New) 2015.01

Manages global report settings for CoveragePkg and AlertLogPkg. Provides 
constants and base types for AlertLogPkg.

.. rubric:: TranscriptPkg (New) 2015.01

TranscriptPkg simplifies different parts of a testbench using a common 
transcript file (named TranscriptFile). Also provides overloading for Print 
and WriteLine that use TranscriptFile when it is opened (via TranscriptOpen), 
and otherwise, use std.env.OUTPUT.

.. rubric:: AlertLogPkg (New) 2015.01

New package added to allow catching and counting of assert FAILURE, ERROR, 
WARNING as well as do verbosity filtering for log files.

The package offers either a native OSVVM mode or an interface mode. In the 
interface mode, the package body is used to redirect OSVVM internal calls to a 
separate alert/log package. For example, for BitVis Utility Library (BVUL), 
there is a package body of AlertLogPkg that allows OSVVM to record asserts and 
logs via BVUL. By only changing the package body, the interface mode can be 
recompiled without requiring other elements of the OSVVM library to be 
recompiled. This same methodology allows connection to other packages.

.. rubric:: RandomPkg and RandomBasePkg 2015.01

Replaced all asserts with calls to AlertPkg.

.. rubric:: CoveragePkg 2015.01

4.6.1 Changes

Replaced all asserts and reports with calls to AlertPkg. Added a verbosity 
flag to WriteBin to allow it to handle debug calls.

WriteBin prints a multiple line report. As a result, instead of calling Log in 
the AlertPkg, package, when called with a verbosity flag, WriteBin first 
checks to see if its ScopeID and Verbosity Level are allowed to print. If 
enabled, it then uses write and writeline to print the report.


The undocumented method, DumpBin, now has a LogLevel parameter. Its interface 
is:

procedure DumpBin (LogLevel : LogType := DEBUG) ;

4.6.2 Additions

The following methods were added:

procedure SetAlertLogID (A : AlertLogIDType) ;
impure function GetAlertLogID return AlertLogIDType ;
impure function SetName (Name : String) return string ;
impure function GetName return String ;
impure function InitSeed (S : string ) return string ;
procedure WriteBin (LogLevel : LogType ; . . .) ;
procedure WriteCovHoles ( LogLevel : LogType ; . . . ) ;

2014.07a - December 2014
************************

5.1 CoveragePkg, MessagePkg, NamePkg 2014.12

Removed memory leak in CoveragePkg.Deallocate. Removed initialized pointers 
from CoveragePkg, MessagePkg, and NamePkg – when a protected type with 
initialized pointers is abandoned, such as when declared in a subprogram 
exits, a memory leak will occur as there is no destructor to deallocate the 
initialized pointers.

2014.07 - July 2014
*******************

6.1 RandomPkg

No changes were made to RandomPkg. It is still labeled 2014.01.

6.2 CoveragePkg

CoveragePkg now references both MessagePkg and NamePkg.

Added names to bins. When using WriteBin or WriteCovHoles, if a bin name is 
set, it will print. For details, see Setting Bin Names in the Reporting 
Coverage section of the CoveragePkg Users Guide.

Enhanced WriteBin to print "PASSED" if the count is greater than or equal to 
the goal (AtLeast value), otherwise, it prints "FAILED". Added a number of 
parameters to WriteBin to control what fields of a WriteBin report get 
printed. See Enabling and Disabling WriteBin fields in the Reporting Coverage 
section of the CoveragePkg Users Guide.

2014.01 - January 2014
**********************

7.1 RandomPkg

Added randomization for time (RandTime), additional overloading for type real 
(RandReal), and sets of values for types (integer_vector, real_vector, and 
time_vector. Made Sort and RevSort from SortListPkg_int visible using aliases.

7.2 CoveragePkg

Revised ReadCovDb to support merging of coverage models (from different test 
runs).

Revised RandCovPoint and RandCovBinVal to log the bin index in the LastIndex 
variable. Revised ICover to look in bin referenced by LastIndex first. Added 
method GetLastIndex to get the variable value. Added GetLastBinVal to get the 
BinVal of LastIndex.

Revised AddBins and AddCross bin merging to allow arbitrary CountBin overlap. 
With the addition of LastIndex, the overlap is not an issue.

Split SetName into SetMessage (headers) and SetName (printing illegal bins)

Added method GetItemCount to return the count of the number of randomizations 
and method GetTotalCovGoal to return the sum of the individual coverage goals 
in the coverage model.


2013.05 - May 2013
*******************

8.1 RandomPkg

Added big vector randomization.

8.2 CoveragePkg

No substantial changes. Removed extra variable declaration in functions 
GetHoleBinval, RandCovBinVal, RandCovHole, GetHoleBinVal. Now referencing 
NULL_RANGE type from RandomPkg to remove NULL range warnings.


2013.04 - April 2013
********************

9.1 RandomPkg

Changed DistInt return value. The return value is now determined by the range 
of the input array. For literal values, this produces the same value as it did 
previously. Also added better error checking for weight values.

Added better min, max error handling in Uniform, FavorBig, FavorSmall, Normal, 
Poisson.


9.2 CoveragePkg

Revised AddBins and AddCross such that bin merging is off by default. Added 
SetMerging to enable/disable merging. Note: Merging is an experimental feature 
and still evolving.

Revised AddBins and AddCross to check for changes in BinVal size (different 
size bin).

Added RandCovPoint for integer.

Added SetThresholding and SetCovThreshold (Percent) to enable/disable(default) 
thresholding. Revised RandCovPoint and RandCovBinVal to use the new mechanism.

Added SetCovTarget to increase/decrease coverage goals for longer/shorter 
simulation runs. Made CovTarget the default percentage goal (via overloading) 
for methods RandCovPoint, RandCovBinVal, IsCovered, CountCovHoles, 
GetHoleBinVal, and WriteCovHoles.

Revised SetIllegalMode and ICover to support ILLEGAL_FAILURE (severity FAILURE 
on illegal bin).

Added manual bin iteration support. Added the following methods that return a 
bin index value: GetNumBins, GetMinIndex, and GetMaxIndex. Added the following 
methods that return bin values: GetBinVal(BinIndex), GetMinBinVal, and 
GetMaxBinVal. Added the following methods that return point values: 
GetPoint(BinIndex), GetMinPoint, and GetMaxPoint.

Added GetCov to return the current percent done of the entire coverage model.

Added FileOpenWriteBin and FileCloseWriteBin to specify default file for 
WriteBin, WriteCovHoles, and DumpBin.

Added CompareBins to facilitate comparing two coverage models. Added 
CompareBins to facilitate comparing two coverage models.

Revised WriteBin, WriteCovHoles, and WriteCovDb to check for uninitialized 
model.

Revised WriteBins and WriteCovHoles to only print weight if the selected 
WeightMode uses the weight.

Added IsInitialized to check if a coverage model is initialized.

Added GetBinInfo and GetBinValLength to get bin information

Changed WriteCovDb default for File_Open_Kind to WRITE_MODE. Generally only 
one WriteCovDb is needed per coverage model.



Revised WriteCovDb and ReadCovDb for new internal control/state variables, in 
the order of ThresholdingEnable, CovTarget, and MergingEnable. To manually 
edit old file, add FALSE, 100.0, FALSE to end of first line.

Removed IgnoreBin with AtLeast and Weight parameters. These are zero for 
ignore bins.

Revised method naming for consistency. The following have changed:

New Name
Old Name
Why
GetErrorCount
CovBinErrCnt
Consistency between packages
GetMinCount
GetMinCov[return integer]
Naming clarity
GetMaxCount
GetMaxCov[return integer]
Naming clarity
SetName
SetItemName
SetName now does multi-line messages
RandCovBinVal
RandCovHole
Naming consistency (2.4)
GetHoleBinVal
GetCovHole
Naming consistency (2.4)

Deprecated usage of the AtLeast parameter (integer) with the following 
methods: RandCovPoint, RandCovBinVal, IsCovered, CountCovHoles, GetHoleBinVal, 
and WriteCovHoles.

2.4 - January 2012
******************

10.1 RandomPkg

No changes

10.2 CoveragePkg

Added bin merging and deletion for overlapping bins.

Working on consistency of naming. Renamed RandCovHole to RandCovBinVal. 
Renamed GetCovHole to GetCovBinVal. Old names maintained for backward 
compatibility.

New Name
Old Name
Why
RandCovBinVal
RandCovHole
Naming consistency
GetCovBinVal
GetCovHole
Naming consistency


2.3 - January 2012
******************

11.1 RandomPkg

No changes

11.2 CoveragePkg

Revision 2.3 adds the function GetBin. GetBin is an accessor function that 
returns a bin in the form of a record. It is only intended for debugging. In 
particular, the return value of this function may change as the internal data 
types evolve.


2.2 - July 2011
***************

12.1 RandomPkg

Removed '_' in the name of subprograms FavorBig and FavorSmall to make more 
consistent with other subprogram names.

12.2 CoveragePkg

Revision 2.2 adds AtLeast and Weights to the coverage database. The AtLeast 
value allows individual bins to have a specific coverage goal. A conjunction 
of the AtLeast and Weight (depending on the WeightMode) are used to weight the 
random selection of coverage holes. These features are at the heart of 
intelligent coverage.

2.1 - June 2011
***************

13.1 RandomPkg

Bug fix to convenience functions for slv, unsigned, and signed.

13.2 CoveragePkg

Removed signal based coverage support.

2.0 - April 2011
****************

14.1 CoveragePkg

Coverage modeled in a protected type.

1.X - June 2010
***************

15.1 CoveragePkg

Coverage modeled in signals of type integer_vector. The signal based coverage 
methodology is available in the package, CoverageSigPkg, however, it is 
recommended that you use CoveragePkg instead.
