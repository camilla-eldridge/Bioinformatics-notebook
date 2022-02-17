# How to recompile autodock for docking of large complexes 

- Compiled autogrid and autodock version 4.2 from source and edited the line in constants.h (in autodock directory) to :


#define MAX_ATOMS    100000000
#define MAX_RECORDS  100000000

** not sure if this one made a difference or not!



- Also edited the line in autogrid.h to :

#define AG_MAX_ATOMS    200000

(numbers over a million didn't work so used 200,000) 

- Also added the files NEW and ChangeLog by using touch as these were required for compilation.


-Before compiling , made a directory for the new build (called x86_64)

then ran autoreconf -i  , cd to x64_86 then built normally (../configure, make) 


-Next added the following line to the file AD4_parameters.dat file in the autodock subdirectory:(adds charges etc for K-Potassium)



atom_par K      3.81  0.035  12.000   -0.00110  0.0  0.0  0  -1  -1  1  # Non H-bonding Potassium


- Made a sym link of the autogrid executable from the autodocksuite_4.2 directory to the mgltools/bin directory (named autogrid4)


*Note need to install csh dependancy (sudo-apt get install csh)


Everything is run in the /home/mgltools/bin directory which contains:


autodock4 executable, autogrid4 link to executable, Vina (although we didnt use it) adt(and other autodock executables)

Input PDB files are also  put in this directory


After the .gpf and dpf files are made, added the following line at the top of each file:


parameter_file AD4_parameters.dat

So that changes in the parameter file are used.
