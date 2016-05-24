Functional Coverage
###################

Overview
********

The VHDL package, CoveragePkg, provides subprograms that facilitate 
implementation of functional coverage within VHDL. It is a core part of the 
Open Source VHDL Verification Methodology (OSVVM). While CoveragePkg is just a 
package, it offers a similar conciseness to the language syntax of other 
verification languages, such as SystemVerilog or 'e'. In addition it offers 
capability and flexibility that is a step ahead.

Functional coverage is code we write to track execution of a test plan. It is 
important to any verification approach since it is one of the factors in 
determining when testing is done. I will address this more in the section, 
"What is Functional Coverage and why do I need it?" In this section I will 
also address, "Why can't I just use code coverage?" and "Why you need 
functional coverage, even with directed testing."

Writing functional coverage is concise and flexible. The basics of writing 
functional coverage using coverage package are covered in the section, 
"Writing Functional Coverage using CoveragePkg."

One important, unique feature is the "Intelligent Coverage" that is built 
directly into the coverage data structure. This capability allows us to 
randomly select a hole in the current functional coverage to pass to the 
stimulus generation process. Using "Intelligent Coverage" helps minimize the 
number of test cases generated to achieve complete coverage - resulting in 
fewer simulation cycles and a higher velocity of verification. More details 
are provided in the section, "Intelligent Coverage is 5X or more faster than 
constrained random testing."

Functional coverage with CoveragePkg is captured incrementally using 
sequential code. This provides a great deal of flexibility and capability, and 
facilitates writing high fidelity functional coverage models. More details are
provided in the section, "Flexibility and Capability."

CoveragePkg provides numerous methods (functions and procedures of a protected 
type) that provide a powerful capability. These methods are documented in the 
remaining part of the document. Note the package contains additional 
undocumented methods that are either experimental features or artifacts from 
older use models that are maintained for backward compatibility. Use these at 
your own risk as they may be removed from future revisions.

This documentation is not a substitute for a great training class. CoveragePkg 
was developed and is maintained by Jim Lewis of SynthWorks. It evolved from 
methodology and packages developed for SynthWorks' VHDL Testbenches and 
Verification class. This class includes additional packages that are not yet 
part of OSVVM. Please support our effort in supporting OSVVM by purchasing 
your VHDL training from SynthWorks.

All CoveragePkg features are supported now by many simulators. The only 
required features are protected types (VHDL-2002) and integer_vector (in 
package std.standard in VHDL-2008). In addition, since they are open source, 
the packages are free (download and usage) and will be updated on a regular 
basis.

.. 
   What is Functional Coverage and why do I need it?
   *************************************************

What is Functional Coverage?
*************************************************

.. ============================

Functional coverage is code that observes execution of a test plan. As such, 
it is code you write to track whether important values, sets of values, or 
sequences of values that correspond to design or interface requirements, 
features, or boundary conditions have been exercised.

Functional coverage is important to any verification approach since it is one 
of the factors used to determine when testing is done. Specifically, 100% 
functional coverage indicates that all items in the test plan have been 
tested. Combine this with 100% code coverage and it indicates that testing is 
done.

Functional coverage that examines the values within a single object is called 
either point (SystemVerilog) or item ('e') coverage. I prefer the term item 
coverage since point can also be a single value within a particular bin. One 
relationship we might look at is different transfer sizes across a packet 
based bus. For example, the test plan may require that transfer sizes with the 
following size or range of sizes be observed: 1, 2, 3, 4 to 127, 128 to 252, 
253, 254, or 255.

Functional coverage that examines the relationships between different objects 
is called cross coverage. An example of this would be examining whether an ALU 
has done all of its supported operations with every different input pair of 
registers.

Many think functional coverage is an exclusive capability of a verification 
language such as SystemVerilog. However, functional coverage collection is 
really just a matter of implementing a data structure.

CoveragePkg contains a protected type and methods (procedures and functions of 
a protected type) that facilitates creating the functional coverage data 
structure and writing functional coverage.

Why can't I just use code coverage?
*************************************************

.. ===================================

VHDL simulation tools can automatically calculate a metric called code 
coverage (assuming you have licenses for this feature). Code coverage tracks 
what lines of code or expressions in the code have been exercised.

Code coverage cannot detect conditions that are not in the code. For example, 
in the packet bus item coverage example discussed above, code coverage cannot 
determine that the required values or ranges have occurred - unless the code 
contains expressions to test for each of these sizes. Instead, we need to 
write functional coverage.

In the ALU cross coverage example above, code coverage cannot determine 
whether particular register pairs have been used together, unless the code is 
written this way. Generally each input to the ALU is selected independently of 
the other. Again, we need to write functional coverage.

Code coverage on a partially implemented design can reach 100%. It cannot 
detect missing features (oops forgot to implement one of the timers) and many 
boundary conditions (in particular those that span more than one block). 
Hence, code coverage cannot be used exclusively to indicate we are done testing.

In addition, code coverage is an optimistic metric. In combinational logic 
code in an HDL, a process may be executed many times during a given clock 
cycle due to delta cycle changes on input signals. This can result in several 
different branches of code being executed. However, only the last branch of 
code executed before the clock edge truly has been covered.


Test Done = Test Plan Executed and All Code Executed
****************************************************

.. ====================================================

To know testing is done, we need to know that both the test plan is executed 
and all of the code has been executed. Is 100% functional coverage enough?

Unfortunately a test can reach 100% functional coverage without reaching 100% 
code coverage. This indicates the design contains untested code that is not 
part of the test plan. This can come from an incomplete test plan, extra 
undocumented features in the design, or case statement others branches that do 
not get exercised in normal hardware operation. Untested features need to 
either be tested or removed.

As a result, even with 100% functional coverage it is still a good idea to use 
code coverage as a fail-safe for the test plan.


Why You Need Functional Coverage, even with Directed Testing
************************************************************

.. ============================================================

You might think, "I have written a directed test for each item in the test 
plan, I am done right?"

As design size grows, the complexity increases. A test that completely 
validates one version of the design, may not validate the design after 
revisions. For example, if the size a of FIFO increases, the test may no 
longer provide enough stimulus values to fill it completely and cause a FIFO 
Full condition. If new features are added, a test may need to change its 
configuration register values to enable the appropriate mode.

Without functional coverage, you are assuming your directed, algorithmic, file 
based, or constrained random test actually hits the conditions in your test 
plan.

Don't forget the engineers creed, "In the divine we trust, all others need to 
show supporting data." Whether you are using directed, algorithmic, file 
based, or constrained random test methods, functional coverage provides your 
supporting data.


What is "Coverage" then?
************************

.. ========================

The word coverage can refer to functional coverage, code coverage, or property 
coverage (such as with PSL). Since this document focuses on functional 
coverage, when the word coverage is used by itself, it is functional coverage.

