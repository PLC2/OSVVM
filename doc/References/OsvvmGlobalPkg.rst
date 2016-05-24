
OsvvmGlobal Package
###################

Overview
********

OsvvmGlobalPkg provides types and global settings for the OSVVM packages 
CoveragePkg and AlertLogPkg.

Package User Subprogram Reference
*********************************

Package Reference
=================

.. code-block:: vhdl
   
   library osvvm ;
   use osvvm.OsvvmGlobalPkg.all ;


OsvvmOptionsType: Type for OSVVM options
========================================

.. code-block:: vhdl
   
   type OsvvmOptionsType is (OPT_INIT_PARM_DETECT, OPT_USE_DEFAULT, DISABLED, FALSE, ENABLED, TRUE) ;


SetOsvvmGlobalOptions: Configuring OSVVM Options
================================================

Configuration values for the output from Alert, Log, and ReportAlerts.

.. code-block:: vhdl
   
   procedure SetOsvvmGlobalOptions (
     WritePassFail : OsvvmOptionsType := OPT_INIT_PARM_DETECT ;
     WriteBinInfo : OsvvmOptionsType := OPT_INIT_PARM_DETECT ;
     WriteCount : OsvvmOptionsType := OPT_INIT_PARM_DETECT ;
     WriteAnyIllegal : OsvvmOptionsType := OPT_INIT_PARM_DETECT ;
     WritePrefix : string := OSVVM_STRING_INIT_PARM_DETECT ;
     DoneName : string := OSVVM_STRING_INIT_PARM_DETECT ;
     PassName : string := OSVVM_STRING_INIT_PARM_DETECT ;
     FailName : string := OSVVM_STRING_INIT_PARM_DETECT
   ) ;

The following options are for WriteBin and ReportAlerts are:

.. code-block:: vhdl
   
   WritePassFail

WriteBin prints Pass / Fail information

.. code-block:: vhdl
   
   WriteBinInfo

WriteBin prints Bin ranges

.. code-block:: vhdl
   
   WriteCount

WriteBin prints Count

.. code-block:: vhdl
   
   WriteAnyIllegal

WriteBin always prints illegal bins

.. code-block:: vhdl
   
   WritePrefix

Prefix for each line of ReportAlerts. Default "%% "

.. code-block:: vhdl
   
   DoneName

Value printed after ReportPrefix on first line of ReportAlerts. Default "DONE"

.. code-block:: vhdl
   
   PassName

Value printed when a test passes. Default "PASSED".

.. code-block:: vhdl
   
   FailName

Value printed when a test fails. Default "FAILED"

.. code-block:: vhdl
   
   SetOsvvmGlobalOptions will change as the packages are revised. To ensure 

forward compatibility into the future call it using named association.

.. code-block:: vhdl
   
   SetOsvvmGlobalOptions (
     WritePrefix => ">> ",
     DoneName => "FIN"
   ) ;

After setting a value, a string value can be reset using ``OSVVM_STRING_USE_DEFAULT``
and an OsvvmOptionsType value can be reset using ``OPT_USE_DEFAULT``.


OsvvmDeallocate
===============

Deallocates all temporary storage in OsvvmGlobalPkg.

Package Developer Subprogram Reference
**************************************

.. code-block:: vhdl
   
   function IsEnabled (A : OsvvmOptionsType) return boolean ;
   function ResolveOsvvmOption(A, B, C: OsvvmOptionsType) return OsvvmOptionsType;
   function ResolveOsvvmOption(A, B, C, D: OsvvmOptionsType) return OsvvmOptionsType;
   function IsOsvvmStringSet(A : string) return boolean ;
   function ResolveOsvvmOption(A, B : string) return string ;
   function ResolveOsvvmOption(A, B, C : string) return string ;
   function ResolveOsvvmOption(A, B, C, D : string) return string ;
   impure function ResolveOsvvmWritePrefix(A : String) return string ;
   impure function ResolveOsvvmWritePrefix(A, B : String) return string ;
   impure function ResolveOsvvmDoneName(A : String) return string ;
   impure function ResolveOsvvmDoneName(A, B : String) return string ;
   impure function ResolveOsvvmPassName(A : String) return string ;
   impure function ResolveOsvvmPassName(A, B : String) return string ;
   impure function ResolveOsvvmFailName(A : String) return string ;
   impure function ResolveOsvvmFailName(A, B : String) return string ;
   impure function ResolveCovWritePassFail(A, B : OsvvmOptionsType)
   return OsvvmOptionsType ; -- Cov
   impure function ResolveCovWriteBinInfo(A, B : OsvvmOptionsType)
   return OsvvmOptionsType ; -- Cov
   impure function ResolveCovWriteCount(A, B : OsvvmOptionsType)
   return OsvvmOptionsType ; -- Cov
   impure function ResolveCovWriteAnyIllegal(A, B : OsvvmOptionsType)
   return OsvvmOptionsType ; -- Cov


Future Work
***********

OsvvmGlobalPkg.vhd is a work in progress and will be updated from time to time.
