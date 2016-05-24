
AlertLog Package
################

Overview
********

VHDL assert statements are a limited form of an alert and log filtering utility.
In the simulator GUI, you can set an assertion level that will stop a 
simulation.
In the simulator GUI, you can turn off some assertions from printing. However,
none of this capability can be configured in VHDL, and in addition, at the end
of a test, there is no way to retrieve a count of the ``ERROR`` level assertions
that have occurred.

The ``AlertLogPkg`` provides ``Alert`` and ``Log`` procedures that replace 
VHDL assert
statements and gives VHDL direct access to enabling and disabling of features,
retrieving alert counts, and set stop counts (limits). All of these features can
be used in either a simple global mode or a hierarchy of alerts.

Alert simplifies signaling errors (when an error occurs) and reporting errors
(passed or failed when a test finishes). Alerts have the levels ``FAILURE``, 
``ERROR``,
and ``WARNING``. Each level is counted and tracked in an internal data 
structure.
Within the data structure, each of these can be enabled or disabled. A test can
be stopped if an alert value has been signaled too many times. Stop values for
each counter can be set. The default for ``FAILURE`` is ``0`` and ``ERROR`` 
and ``WARNING`` are
``integer'right``. If all test errors are reported as an alert, at the end of 
the
test, a report can be generated which provides pass/fail and a count of the
different alert values.

Logs provide a mechanism to conditionally print information. This verbosity 
control
allows messages (such as debug, DO254 final test reports, or info) that are too
detailed for normal testing to be printed when specifically enabled. VIA the 
simulator
GUI, assert has this capability to a limited degree.

``AssertLogPkg`` uses ``TranscriptPkg`` to print to either 
``std.textio.OUTPUT``, a file,
or both. When the TranscriptFile is opened, alert and log print to the 
TranscriptFile,
otherwise, they print to ``std.textio.OUTPUT``. For more details on 
``TranscriptPkg`` see
the :doc:`TranscriptPkg User Guide <TranscriptPkg>`.

Already using another package for alerts and verbosity control? The 
AlertLogPkg has an
extensive API that will allow you to retrieve any error information reported 
by the
OSVVM packages and allow you to print a summary of results via another package.

AlertLogPkg Use Models
**********************

Alerts and Logs may be used in either a simple mode or a hierarchy model.

In simple mode, there is single global alert counter that accumulates the 
number of
FAILURE, ERROR, and WARNING level alerts for the entire testbench. When a test 
completes,
a summary of the total number of errors as well as the errors for each level 
can be produced.

In hierarchy mode, there is a hierarchy of alert counters. Each model and/or 
source
of alerts has its own set of alert counters. Counts from lower levels 
propagate up to
the top level counter. Each level in the hierarchy also supports separate 
verbosity
control. When a test completes, an error report can be produced for both the 
top level
and each level in the alert hierarchy.

Simple Mode: Global Alert Counter
*********************************

By default, there is a single global alert counter. All designs that use alert 
or log
need to reference the package AlertLogPkg.

.. code-block:: vhdl
   
   use osvvm.AlertLogPkg.all ;
   architecture Test1 of tb is

Use Alert to flag an error, AlertIf to flag an error when a condition is true, 
or
AlertIfNot to flag an error when a condition is false (similar to assert). 
Alerts
can be of severity FAILURE, ERROR, or WARNING.

.. code-block:: vhdl
   
   -- message, level
   When others => Alert("Illegal State", FAILURE) ;
   . . .
   -- condition, message, level
   AlertIf(ActualData /= ExpectedData, "Data Miscompare …", ERROR) ;
   . . .
   read(Buf, A, ReadValid) ;
   -- condition, message, level
   AlertIfNot( ReadValid, "read of A failed", FAILURE) ;

The output for an alert is as follows. Alert adds the time at which the log 
occurred.

.. code-block:: raw
   
   %% Alert ERROR Data Miscompare … at 20160 ns

When a test completes, use ReportAlerts to provide a summary of errors.

.. code-block:: vhdl
   
   ReportAlerts ;

When a test passes, the following message is generated:

.. code-block:: vhdl
   
   %% DONE PASSED t1_basic at 120180 ns

When a test fails, the following message is generated (on a single line):

.. code-block:: vhdl
   
   %% DONE FAILED t1_basic Total Error(s) = 2 Failures: 0 Errors: 1 Warnings: 
1 at 120180 ns

Similar to assert, by default, when an alert FAILURE is signaled, a test failed
message (see ReportAlerts) is produced and the simulation is stopped. This 
action
is controlled by a stop count. The following call to SetAlertStopCount, causes a
simulation to stop after 20 ERROR level alerts are received.

.. code-block:: vhdl
   
   SetAlertStopCount(ERROR, 20) ;

Alerts can be enabled by a general enable, SetGlobalAlertEnable (disables all 
alert
handling) or an enable for each alert level, SetAlertEnable. The following 
call to
SetAlertEnable disables WARNING level alerts.

.. code-block:: vhdl
   
   SetGlobalAlertEnable(TRUE) ; -- Default
   SetAlertEnable(WARNING, FALSE) ;

Logs are used for verbosity control. Log level values are ALWAYS, DEBUG, 
FINAL, and INFO.

.. code-block:: vhdl
   
   Log ("A message", DEBUG) ;

Log formats the output as follows.

.. code-block:: raw
   
   %% Log DEBUG A Message at 15110 ns

Each log level is independently enabled or disabled. This allows the testbench 
to support
debug or final report messages and only enable them during the appropriate 
simulation run.
The log ALWAYS is always enabled, all other logs are disabled by default. The 
following
call to SetLogEnable enables DEBUG level logs.

.. code-block:: vhdl
   
   SetLogEnable(DEBUG, TRUE) ;


Hierarchy Mode: Separate Alert Counters
***************************************

In hierarchy mode, each model and/or source of alerts has its own set of alert 
counters.
Counts from lower levels propagate up to the top level counter. The ultimate 
goal of using
hierarchy mode is to get a summary of errors for each model and/or source of 
alerts in the
testbench:

.. code-block:: raw
   
   %% DONE FAILED Testbench Total Error(s) = 21 Failures: 1 Errors: 20 
Warnings: 0 at 10117000 ns
   %% Default Failures: 0 Errors: 4 Warnings: 0
   %% OSVVM Failures: 0 Errors: 0 Warnings: 0
   %% U_CpuModel Failures: 0 Errors: 4 Warnings: 0
   %% Data Error Failures: 0 Errors: 2 Warnings: 0
   %% Protocol Error Failures: 1 Errors: 2 Warnings: 0
   %% U_UART_TX Failures: 0 Errors: 6 Warnings: 0
   %% U_UART_RX Failures: 0 Errors: 6 Warnings: 0

Using hierarchy mode requires a little more work. Behind the scenes within 
AlertLogPkg, there
is a data structure inside shared variable. Each level in a hierarchy is 
referenced with an
AlertLogID - which is currently an integer index into the data structure. As a 
result, each
model must get (allocate) an AlertLogID and then reference the AlertLogID when 
signaling alerts.
Other than referencing the AlertLogID, the usage is identical.
A new AlertLogID is created by calling the function GetAlertLogID. 
GetAlertLogID has two
parameters: Name and ParentID. Name is a string of the ALERT (that prints when 
the alert
prints). ParentID is of type AlertLogIDType.

In the following example, CPU_ALERT_ID uses the instance name of the model as 
its name.
Since it is a top level model, it uses ALERTLOG_BASE_ID (which is also the 
default) as its
ParentID. DATA_ALERT_ID is an alert counter within the CPU. So it uses a 
string as its name
and CPU_ALERT_ID as its ParentID.

.. code-block:: vhdl
   
   constant CPU_ALERT_ID : AlertLogIDType :=
   -- Name (string value), Parent AlertLogID
   GetAlertLogID(PathTail(CpuModel'PATH_NAME), ALERTLOG_ BASE_ID) ;
   constant DATA_ALERT_ID : AlertLogIDType :=
   GetAlertLogID("Data Error", CPU_ALERT_ID) ;
   constant PROTOCOL_ALERT_ID : AlertLogIDType :=
   GetAlertLogID("PROTOCOL Error", CPU_ALERT_ID) ;

The AlertLogID is specified first in calls to Alert, Log, SetAlertEnable, 
SetAlertStopCount, and SetLogEnable.

.. code-block:: vhdl
   
   Alert(CPU_ALERT_ID, "CPU Error", ERROR) ;
   AlertIf(PROTOCOL_ALERT_ID, inRdy /= '0', "during CPU Read operation", 
FAILURE);
   AlertIfNotEqual(DATA_ALERT_ID, ReadData, ExpectedData, "Actual /= Expected 
Data");
   -- AlertLogID, Level, Enable
   SetAlertEnable(CPU_ALERT_ID, WARNING, FALSE) ;
   -- AlertLogID, Level, Count
   SetAlertStopCount(CPU_ALERT_ID, ERROR, 20) ;
   Log(UartID, DEBUG, "Uart Parity Received") ;
   -- AlertLogID, Level, Enable, DescendHierarchy
   SetLogEnable(UartID, WARNING, FALSE, FALSE) ;

Printing of Alerts and Logs include the AlertLogID.

.. code-block:: raw
   
   %% Alert FAILURE in CPU_1, Expect data XA5A5 at 2100 ns
   %% Log ALWAYS in UART_1, Parity Error at 2100 ns

Package References
******************

Using AlertLogPkg requires the following package references:

.. code-block:: vhdl
   
   library osvvm ;
   use osvvm.OsvvmGlobalPkg.all ;
   use osvvm.AlertLogPkg.all ;


Alert Method Reference
**********************

AlertType
=========

Alert levels can be FAILURE, ERROR, or WARNING.

.. code-block:: vhdl
   
   type AlertType is (FAILURE, ERROR, WARNING) ;

Simple Alerts
=============

Simple alerts accumulate alerts in the default
AlertLogID (ALERTLOG_DEFAULT_ID). It supports the basic overloading and usage:

.. code-block:: vhdl
   
   procedure Alert( Message : string ; Level : AlertType := ERROR ) ;
   . . .
   Alert("Uart Parity") ; -- ERROR by default

Alert has two conditional forms, AlertIf and AlertIfNot. The following is 
their overloading.

.. code-block:: vhdl
   
   -- without an AlertLogID
   procedure AlertIf( condition : boolean ;
   Message : string ; Level : AlertType := ERROR ) ;
   impure function AlertIf( condition : boolean ;
   Message : string ; Level : AlertType := ERROR ) return boolean ;
   procedure AlertIfNot( condition : boolean ;
   Message : string ; Level : AlertType := ERROR ) ;
   impure function AlertIfNot( condition : boolean ;
   Message : string ; Level : AlertType := ERROR ) return boolean ;

Usage of conditional alerts:

.. code-block:: vhdl
   
   AlertIf(Break='1', "Uart Break", ERROR) ;
   AlertIfNot(ReadValid, "Read Failed", FAILURE) ;

The function form is convenient for use for conditional exit of a loop.

exit AlertIfNot(ReadValid, "in ReadCovDb while reading …", FAILURE) ;

Alert form AlertIfEqual and AlertIfNotEqual to check two values. In the 
following, AType can be std_logic, std_logic_vector, unsigned, signed, 
integer, real, character, or string.

procedure AlertIfEqual( L, R : AType ; Message : string ;
Level : AlertType := ERROR ) ; Message : string ;
procedure AlertIfNotEqual( L, R : AType;
Level : AlertType := ERROR ) ;
Alert form AlertIfDiff is for comparing two files.
procedure AlertIfDiff (Name1, Name2 : string; Message : string := "" ;
Level : AlertType := ERROR ) ;
procedure AlertIfDiff (file File1, File2 : text; Message : string := "" ;
Level : AlertType := ERROR ) ;

Creating Hierarchy: GetAlertLogID
=================================

Each level in a hierarchy is referenced with an AlertLogID. The function, 
GetAlertLogID, creates a new AlertLogID. If an AlertLogID already exists for 
the specified name, GetAlertLogID will return its AlertLogID. It is 
recommended to use the instance label as the Name. The interface for 
GetAlertLogID is as follows.
impure function GetAlertLogID(
Name : string ;
ParentID : AlertLogIDType := ALERTLOG_BASE_ID ;
CreateHierarchy : Boolean := TRUE
) return AlertLogIDType ;
The CreateHierarchy parameter is intended to allow packages to use a unique 
AlertLogID for reporting Alerts without creating hierarchy in ReportAlerts. As 
a function, GetAlertLogID can be called while elaborating the design by using 
it to initialize a constant or signal:
Constant UartID : AlertLogIDType :=
-- Name, Parent AlertLogID
GetAlertLogID("UART_1", ALERTLOG_BASE_ID);

FindAlertLogID: Find an AlertLogID
==================================

The function, FindAlertLogID, finds an existing AlertLogID. If the AlertLogID 
is not found, ALERTLOG_ID_NOT_FOUND is returned. The interface for 
FindAlertLogID is as follows.
impure function FindAlertLogID(Name : string ; ParentID : AlertLogIDType)
return AlertLogIDType ;
impure function FindAlertLogID(Name : string ) return AlertLogIDType ;
Note the single parameter FindAlertLogID is only useful when there is only one 
AlertLogID with a particular name (such as for top-level instance names). As a 
function, FindAlertLogID can be called while elaborating the design by using 
it to initialize a constant or signal.
constant UartID : AlertLogIDType := FindAlertLogID(Name => "UART_1") ;
Caution: only use FindAlertLogID when it is known that the ID has already been 
created - such as in a testbench where the testbench components have already 
been elaborated, as otherwise, it is appropriate to use GetAlertLogID.

PathTail - Used to Discover Instance Name of a Component
========================================================

When used in conjunction with attribute PATH_NAME applied to an entity name, 
PathTail returns the instance name of component.
constant CPU_ALERT_ID : AlertLogIDType :=
-- Name (string value), Parent AlertLogID
GetAlertLogID(PathTail(CpuModel'PATH_NAME), ALERTLOG_ BASE_ID) ;

Hierarchical Alerts
===================

Hierarchical alerts require the AlertLogID to be specified in the call to 
alert. It supports the basic overloading and usage:
procedure alert(
AlertLogID : AlertLogIDType ;
Message : string ;
Level : AlertType := ERROR
) ;
. . .
Alert(UartID, "Uart Parity", ERROR) ;
When an alert is signaled in a lower level of the hierarchy, it increments all 
parent levels until it finds a level whose alert for that level is disabled or 
the top of the hierarchy.
Alert has two conditional forms, AlertIf and AlertIfNot. The following is 
their overloading. The function form is convenient for use for conditional 
exit of a loop.
procedure AlertIf( AlertLogID : AlertLogIDType ; condition : boolean ;
Message : string ; Level : AlertType := ERROR ) ;
impure function AlertIf( AlertLogID : AlertLogIDType ; condition : boolean ;
Message : string ; Level : AlertType := ERROR ) return boolean ;
procedure AlertIfNot( AlertLogID : AlertLogIDType ; condition : boolean ;
Message : string ; Level : AlertType := ERROR ) ;
impure function AlertIfNot( AlertLogID : AlertLogIDType ; condition : boolean ;
Message : string ; Level : AlertType := ERROR ) return boolean ;
Usage of conditional alerts:
AlertIf(Break='1', UartID, "Uart Break", ERROR) ;
AlertIfNot(ReadValid, UartID, "Read", FAILURE);
Alert form AlertIfEqual and AlertIfNotEqual to check two values. In the 
following, AType can be std_logic, std_logic_vector, unsigned, signed, 
integer, real, character, or string.
procedure AlertIfEqual( AlertLogID : AlertLogIDType ; L, R : AType;
Message : string ; Level : AlertType := ERROR ) ;
procedure AlertIfNotEqual( AlertLogID : AlertLogIDType ; L, R : AType;
Message : string ; Level : AlertType := ERROR ) ;
Alert form AlertIfDiff is for comparing two files.
procedure AlertIfDiff (AlertLogID : AlertLogIDType ; Name1, Name2 : string;
Message : string := "" ; Level : AlertType := ERROR ) ;
procedure AlertIfDiff (AlertLogID : AlertLogIDType ; file File1, File2 : text;
Message : string := "" ; Level : AlertType := ERROR ) ;

IncrementAlertCount
===================

Intended as a silent alert. Used by CoveragePkg.
-- Hierarchy
procedure IncAlertCount( -- A silent form of alert
AlertLogID : AlertLogIDType ;

Level : AlertType := ERROR
) ;
-- Global Alert Counters
procedure IncAlertCount( Level : AlertType := ERROR ) ;

ReportAlerts: Reporting Alerts
==============================

At test completion alerts are reported with ReportAlerts.
procedure ReportAlerts (
Name : string := "" ;
AlertLogID : AlertLogIDType := ALERTLOG_BASE_ID ;
ExternalErrors : AlertCountType := (others => 0)
) ;
. . .
ReportAlerts ;
ReportAlerts has 3 optional parameters: Name, AlertLogID, and ExternalErrors. 
Name specifies the test name (and can also be set with SetAlertLogName). 
AlertLogID allows reporting alerts for a specific AlertLogID and its children 
(if any). ExternalErrors allows separately detected errors to be reported. 
ExternalErrors is type AlertCountType and the value (FAILURE => 0, ERROR => 5, 
WARNING => 1) indicates detection logic separate from AlertLogPkg saw 0 
Failures, 5 Errors, and 1 Warning. See notes under AlertCountType.
-- Name, AlertLogID, ExternalErrors
ReportAlerts("Uart1", UartID, (FAILURE => 0, ERROR => 5, WARNING => 1) ) ;
ReportAlerts can also be used to print a passed/failed message for an 
AlertCount that is passed into the procedure call.
procedure ReportAlerts ( Name : String ; AlertCount : AlertCountType) ;
This is useful to accumulate values returned by different phases of a test 
that need to be reported separately.
ReportAlerts("Test1: Final", Phase1AlertCount + Phase2AlertCount) ;
Also see SetAlertLogOptions.

ReportNonZeroAlerts
===================

Within the hierarchy, if a level has no alerts set, then that level will not 
be printed.
procedure ReportNonZeroAlerts (
Name : string := OSVVM_STRING_INIT_PARM_DETECT ;
AlertLogID : AlertLogIDType := ALERTLOG_BASE_ID ;
ExternalErrors : AlertCountType := (others => 0)
) ;

SetAlertLogName: Setting the Test Name
======================================

SetAlertLogName sets the name ReportAlerts prints when called without 
parameters - such as when an internal stop count reached.


procedure SetAlertLogName(Name : string ) ;
. . .
SetAlertLogName("Uart1") ;

GetAlertLogName
===============

GetAlertLogName returns the string value of name associated with an 
AlertLogID. If no AlertLogID is specified, it will return the name set 
bySetAlertLogName.
impure function GetAlertLogName(AlertLogID : AlertLogIDType:= 
ALERTLOG_BASE_ID) return string ;

SetGlobalAlertEnable: Alert Global Enable / Disable
===================================================

SetGlobalAlertEnable allows Alerts to be globally enabled and disabled. The 
intent is to be able to disable all alerts until the system goes into reset. 
Alerts are enabled by default.
procedure SetGlobalAlertEnable (A : EnableType := TRUE) ;
impure function SetGlobalAlertEnable (A : EnableType := TRUE) return 
EnableType ;
Suppress all alerts before reset by turning alerts off during elaboration with 
a constant declaration and then turning them back on later.
InitAlerts : Process
constant DisableAlerts : boolean := SetGlobalAlertEnable(FALSE);
begin
wait until nReset = '1' ; -- Deassertion of reset
SetGlobalAlertEnable(TRUE) ; -- enable alerts

GetGlobalAlertEnable
====================

Get the current value of the global alert enable
impure function GetGlobalAlertEnable return boolean ;

SetAlertEnable: Alert Enable / Disable
======================================

SetAlertEnable allows alert levels to be individually enabled. When used 
without AlertLogID, SetAlertEnable sets a value for all AlertLogIDs.
procedure SetAlertEnable(Level : AlertType ; Enable : boolean) ;
. . .
-- Level, Enable
SetAlertEnable(WARNING, FALSE) ;
When an AlertLogID is used, SetAlertEnable sets a value for that AlertLogID, 
and if DescendHierarchy is TRUE, it’s the AlertLogID's of its children.
procedure SetAlertEnable(AlertLogID : AlertLogIDType ; Level : AlertType ; 
Enable : boolean ; DescendHierarchy : boolean := TRUE) ;

GetAlertEnable
==============

Get the value of the current alert enable for either a specific AlertLogId or 
for the global alert counter.
-- Hierarchy
impure function GetAlertEnable(AlertLogID : AlertLogIDType ; Level : 
AlertType) return boolean ;
-- Global Alert Counter
impure function GetAlertEnable(Level : AlertType) return boolean ;

SetAlertStopCount: Alert Stop Counts
====================================

When an alert stop count is reached, the simulation stops. When used without 
AlertLogID, SetAlertStopCount sets the alert stop count for the top level to 
the specified value if the current count is integer'right, otherwise, it sets 
it to the specified value plus the current count.
procedure SetAlertStopCount(Level : AlertType ; Count : integer) ;
. . .
-- Level, Count
SetAlertStopCount(ERROR, 20) ; -- Stop if 20 errors occur
When used with an AlertLogID, SetAlertStopCount sets the value for the 
specified AlertLogID and all of its parents. At each level, the current alert 
stop count is set to the specified value when the current count is 
integer'right, otherwise, the value is set to the specified value plus the 
current count.
procedure SetAlertStopCount(AlertLogID : AlertLogIDType ;
Level : AlertType ; Count : integer) ;
. . .
-- AlertLogID, Level, Count
SetAlertStopCount(UartID, ERROR, 20) ;
By default, the AlertStopCount for WARNING and ERROR are integer'right, and 
FAILURE is 0.

GetAlertStopCount
=================

Get the value of the current alert stop count for either a specific AlertLogId 
or for the global alert counter.
-- Hierarchy
impure function GetAlertStopCount(
AlertLogID : AlertLogIDType ;
Level : AlertType
) return integer ;
-- Global Alert Stop Count
impure function GetAlertStopCount(Level : AlertType) return integer ;

AlertCountType
==============

Alerts are stored as a value of AlertCountType.

subtype AlertIndexType is AlertType range FAILURE to WARNING ;
type AlertCountType is array (AlertIndexType) of integer ;
CAUTION: When working with values of AlertCountType, be sure to use named 
association as the type ordering may change in the future.

GetAlertCount
=============

GetAlertCount returns the AlertCount value at AlertLogID. GetAlertCount is 
overloaded to return either AlertCountType or integer.
impure function GetAlertCount(AlertLogID : AlertLogIDType := ALERTLOG_BASE_ID)
return AlertCountType ;
impure function GetAlertCount(AlertLogID : AlertLogIDType := ALERTLOG_BASE_ID)
return integer ;
. . .
TopTotalErrors := GetAlertCount ; -- AlertCount for Top of hierarchy
UartTotalErrors := GetAlertCount(UartID) ; -- AlertCount for UartID

GetEnabledAlertCount
====================

GetEnabledAlertCount is similar to GetAlertCount except it returns 0 for 
disabled alert levels. GetEnabledAlertCount is overloaded to return either 
AlertCountType or integer.
impure function GetEnabledAlertCount(AlertLogID : AlertLogIDType :=
ALERTLOG_BASE_ID) return AlertCountType ;
impure function GetEnabledAlertCount (AlertLogID : AlertLogIDType :=
ALERTLOG_BASE_ID) return integer ;
. . .
TopTotalErrors := GetEnabledAlertCount ; -- Top of hierarchy
UartTotalErrors := GetEnabledAlertCount(UartID) ; -- UartID

GetDisabledAlertCount
=====================

GetDisabledAlertCount returns the count of disabled errors for either the 
entire design hierarchy or a particular AlertLogID. GetDisabledAlertCount is 
relevant since a "clean" passing design will not have any disabled alert counts.
impure function GetDisabledAlertCount return AlertCountType ;
impure function GetDisabledAlertCount return integer ;
impure function GetDisabledAlertCount(AlertLogID: AlertLogIDType)
return AlertCountType ;
impure function GetDisabledAlertCount(AlertLogID: AlertLogIDType) return 
integer ;
Note that disabled errors are not added to higher levels in the hierarchy. 
Hence, often GetAlertCount /= GetEnabledAlertCount + GetDisabledAlertCount.

ClearAlerts: Reset Alert and Stop Counts
========================================

ClearAlerts resets all alert counts to 0 and stop counts back to their default.
procedure ClearAlerts ;

Math on AlertCountType
======================

function "+" (L, R : AlertCountType) return AlertCountType ;
function "-" (L, R : AlertCountType) return AlertCountType ;
function "-" (R : AlertCountType) return AlertCountType ;
. . .
TotalAlertCount := Phase1Count + Phase2Count ;
TotalErrors := GetAlertCount - ExpectedErrors ;
NegateErrors := -ExpectedErrors ;

SumAlertCount: AlertCountType to Integer Error Count
====================================================

SumAlertCount sums up the FAILURE, ERROR, and WARNING values into a single 
integer value.
impure function SumAlertCount(AlertCount: AlertCountType) return integer ;
. . .
ErrorCountInt := SumAlertCount(AlertCount) ;

GetAlertLogParentID
===================

Get the AlertLogID of the parent of a specified AlertLogID.
impure function GetAlertLogParentID(AlertLogID : AlertLogIDType) return 
AlertLogIDType ;

Log Method Reference
********************

LogType
=======

Log levels can be ALWAYS, DEBUG, FINAL, or INFO.
type LogType is (ALWAYS, DEBUG, FINAL, INFO) ;

Simple Logs
===========

Simple logs use default AlertLogID. If the log level is enabled, then the log 
message will print. The Enable parameter is an override of the internal 
settings and if true, the log message will print.
procedure log(
Message : string ;
Level : LogType := ALWAYS ;
Enable : boolean := FALSE -- override internal enable
) ;
. . .
Log("Received UART word", DEBUG) ;

Hierarchical Logs
====================
Hierarchical logs use the specified AlertLogID. If the log level is enabled, 
then the log message will print.
procedure log(
AlertLogID : AlertLogIDType ;
Message : string ;

Level : LogType := ALWAYS ;
Enable : boolean := FALSE -- override internal enable
) ;
. . .
Log(UartID, "Uart Parity Received", DEBUG) ;

SetLogEnable: Enable / Disable Logging
======================================

SetLogEnable allows alert levels to be individually enabled. When used without 
AlertLogID, SetLogEnable sets a value for all AlertLogIDs.
procedure SetLogEnable(Level : LogType ; Enable : boolean) ;
. . .
Log(UartID, "Uart Parity Received", DEBUG) ;
When an AlertLogID is used, SetLogEnable sets a value for that AlertLogID, and 
if Hierarchy is true, the AlertLogIDs of its children.
procedure SetLogEnable(AlertLogID : AlertLogIDType ;
Level : LogType ; Enable : boolean ; DescendHierarchy : boolean := TRUE) ;
. . .
-- AlertLogID, Level, Enable, DescendHierarchy
SetLogEnable(UartID, WARNING, FALSE, FALSE) ;

Reading Log Enables from a FILE
===============================

ReadLogEnables read enables from a file.
procedure ReadLogEnables (FileName : string) ;
procedure ReadLogEnables (file AlertLogInitFile : text) ;
The preferred file format is:
U_CpuModel DEBUG
U_UART_TX DEBUG INFO
U_UART_RX FINAL INFO DEBUG
ReadLogEnables will also read a file of the format:
U_CpuModel
DEBUG
U_UART_TX
DEBUG
U_UART_TX
INFO
. . .

IsLoggingEnabled /GetLogEnable
==============================

IsLoggingEnabled returns true when logging is enabled for a particular 
AlertLogID.
impure function IsLoggingEnabled(Level : LogType) return boolean ;
impure function IsLoggingEnabled(AlertLogID : AlertLogIDType ; Level : LogType)
return boolean ;
. . .

If IsLoggingEnabled(UartID, DEBUG) then
. . .
GetLogEnable is a synonym for IsLoggingEnabled.
impure function GetLogEnable(AlertLogID : AlertLogIDType ; Level : LogType) 
return boolean ;
impure function GetLogEnable(Level : LogType) return boolean ;

Affirmation Method Reference
****************************

AffirmIf
====================

Affirmations are a combination of Alerts and Logs. If the Affirmation is true, 
then a log is generated. If an Affirmation is false, then an alert is generated.
-- Hierarchy
procedure AffirmIf(
AlertLogID : AlertLogIDType ;
condition : boolean ;
Message : string ;
LogLevel : LogType := PASSED ;
AlertLevel : AlertType := ERROR
) ;
-- Global Alert Counters
procedure AffirmIf(
condition : boolean ;
Message : string ;
LogLevel : LogType := PASSED ;
AlertLevel : AlertType := ERROR
) ;
Affirmations are intended to be used for self-checking of a test. Each call to 
AffirmIf is counted and reported during ReportAlerts. This provides feedback 
on the amount of self-checking added by a test and is used as a quality metric.

GetAffirmCheckCount
===================

Returns the current affirmation check count.
impure function GetAffirmCheckCount return natural ;

IncAffirmCheckCount
===================

Increments the affirmation check count. Intended to be used only in special 
situations, such as packages that have additional considerations when using 
affirmations.
procedure IncAffirmCheckCount ;


Alert and Log Output Control Options
************************************

SetAlertLogJustify
====================

SetAlertLogJustify justifies name fields of Alerts and Logs. Call after 
setting up the entire hierarchy if you want Alerts and Logs justified (hence 
optional).
SetAlertLogJustify ;

OsvvmOptionsType
================

OsvvmOptionsType defines the values for options. User values are: OPT_DEFAULT, 
DISABLED, FALSE, ENABLED, TRUE. The values DISABLED and FALSE are handled the 
same. The values ENABLED and TRUE are treated the same. The value 
OPT_USE_DEFAULT causes the variable to use its default value. OsvvmOptionsType 
is defined in OsvvmGlobalPkg.
type OsvvmOptionsType is (OPT_INIT_PARM_DETECT, OPT_USE_DEFAULT, DISABLED, 
FALSE, ENABLED, TRUE) ;

SetAlertLogOptions: Configuring Report Options
==============================================

The output from Alert, Log, and ReportAlerts is configurable using 
SetAlertLogOptions.
procedure SetAlertLogOptions (
FailOnWarning : OsvvmOptionsType := OPT_INIT_PARM_DETECT;
FailOnDisabledErrors : OsvvmOptionsType := OPT_INIT_PARM_DETECT;
ReportHierarchy : OsvvmOptionsType := OPT_INIT_PARM_DETECT;
WriteAlertLevel : OsvvmOptionsType := OPT_INIT_PARM_DETECT;
WriteAlertName : OsvvmOptionsType := OPT_INIT_PARM_DETECT;
WriteAlertTime : OsvvmOptionsType := OPT_INIT_PARM_DETECT;
WriteLogLevel : OsvvmOptionsType := OPT_INIT_PARM_DETECT;
WriteLogName : OsvvmOptionsType := OPT_INIT_PARM_DETECT;
WriteLogTime : OsvvmOptionsType := OPT_INIT_PARM_DETECT;
AlertPrefix : string := OSVVM_STRING_INIT_PARM_DETECT;
LogPrefix : string := OSVVM_STRING_INIT_PARM_DETECT;
ReportPrefix : string := OSVVM_STRING_INIT_PARM_DETECT;
DoneName : string := OSVVM_STRING_INIT_PARM_DETECT;
PassName : string := OSVVM_STRING_INIT_PARM_DETECT;
FailName : string := OSVVM_STRING_INIT_PARM_DETECT
) ;
The following options are for ReportAlerts.
FailOnWarning
Count warnings as test errors.
Enabled
FailOnDisabledErrors
Disabled errors are test errors.
Enabled
ReportHierarchy
When multiple AlertLogIDs exist, print an error summary for each level.
Enabled
ReportPrefix
Prefix for each line of ReportAlerts.
"%% "
DoneName
Value printed after ReportPrefix on first line of ReportAlerts.
"DONE"
PassName
Value printed when a test passes.
"PASSED".
FailName
Value printed when a test fails.
"FAILED"

The following options are for alert:
WriteAlertLevel
Print level.
Enabled
WriteAlertName
Print AlertLogID name.
Enabled
WriteAlertTime
Alerts print time.
Enabled
AlertPrefix
Value printed at beginning of alert.
"%% Alert"
The following options are for Log:
WriteLogLevel
Print level.
Enabled
WriteLogName
Print AlertLogID name.
Enabled
WriteLogTime
Logs print time.
Enabled
LogPrefix
Value printed at beginning of log.
"%% Alert"
SetAlertOptions will change as AlertLogPkg evolves. Use of named association 
is required to ensure future compatibility.
SetAlertLogOptions (
FailOnWarning => FALSE,
FailOnDisabledErrors => FALSE
) ;
After setting a value, a string value can be reset using 
OSVVM_STRING_USE_DEFAULT and an OsvvmOptionsType value can be reset using 
OPT_USE_DEFAULT.

ReportAlertLogOptions: Print Report Options
===========================================

Prints out AlertLogPkg Report Options.

Getting AlertLog Report Options
===============================

Report options can be retrieved with one of the Get functions below.
function GetAlertLogFailOnWarning return AlertLogOptionsType ;
function GetAlertLogFailOnDisabledErrors return AlertLogOptionsType ;
function GetAlertLogReportHierarchy return AlertLogOptionsType ;
function GetAlertLogHierarchyInUse return AlertLogOptionsType ;
function GetAlertLogWriteAlertLevel return AlertLogOptionsType ;
function GetAlertLogWriteAlertName return AlertLogOptionsType ;
function GetAlertLogWriteAlertTime return AlertLogOptionsType ;
function GetAlertLogWriteLogLevel return AlertLogOptionsType ;
function GetAlertLogWriteLogName return AlertLogOptionsType ;
function GetAlertLogWriteLogTime return AlertLogOptionsType ;
function GetAlertLogAlertPrefix return string ;
function GetAlertLogLogPrefix return string ;
function GetAlertLogReportPrefix return string ;
function GetAlertLogDoneName return string ;
function GetAlertLogPassName return string ;
function GetAlertLogFailName return string ;

Deallocating and Re-initializing the Data structure
***************************************************

DeallocateAlertLogStruct
========================

DeallocateAlertLogStruct deallocates all temporary storage allocated by 
AlertLogPkg. Also see ClearAlerts.

InitializeAlertLogStruct
========================

InitializeAlertLogStruct is used after DeallocateAlertLogStruct to create and 
initialize internal storage.


Compiling AlertLogPkg and Friends
*********************************

Use of AlertLogPkg requires use NamePkg and OsvvmGlobalPkg. The compile order 
is: NamePkg.vhd, OsvvmGlobalPkg.vhd, TranscriptPkg.vhd, and AlertLogPkg.vhd. 
Compiling the packages requires VHDL-2008.

About AlertLogPkg
*****************

AlertLogPkg was developed and is maintained by Jim Lewis of SynthWorks VHDL 
Training. It originated as an interface layer to the BitVis Utility Library 
(BVUL). However, it required a default implementation and that default 
implementation grew into its own project.
Please support our effort in supporting AlertLogPkg and OSVVM by purchasing 
your VHDL training from SynthWorks.
AlertLogPkg is released under the Perl Artistic open source license. It is 
free (both to download and use - there are no license fees). You can download 
it from http://www.synthworks.com/downloads. It will be updated from time to 
time. Currently there are numerous planned revisions.
If you add features to the package, please donate them back under the same 
license as candidates to be added to the standard version of the package. If 
you need features, be sure to contact us. I blog about the packages at 
http://www.synthworks.com/blog. We also support the OSVVM user community and 
blogs through http://www.osvvm.org.
Find any innovative usage for the package? Let us know, you can blog about it 
at osvvm.org.

Future Work
***********

AlertLogPkg.vhd is a work in progress and will be updated from time to time.
Caution, undocumented items are experimental and may be removed in a future 
version.

.. seealso::
   
   :doc:`AlertLogPkg Reference </References/AlertLogPkg>`
      The complete interface reference for this package.
   :doc:`TranscriptPkg Reference </References/TranscriptPkg>`
      The complete interface reference for the uder laying transcript package.
