## OSL Windows Build Guide

This guide aims to help you install all the necessary dependencies OSL needs, and then build OSL from source on your Windows machine. We will use VCPKG to get us most of the dependencies and then use a custom build script to build Bison and Flex. We will then get OSL from source and build it using CMake and the dependencies we installed.

Requirements: CMake, Visual Studio

### **Step 0: Installing vcpkg**

The vcpkg repo: [https://github.com/microsoft/vcpkg](https://github.com/microsoft/vcpkg)

* Vcpkg is a C/C++ package manager. Here is a [guide](https://learn.microsoft.com/en-us/vcpkg/get_started/get-started?pivots=shell-powershell#1---set-up-vcpkg) to install and configure VCPKG with your environment variables  
* After setting up your environment variables, you can run vcpkg commands from the terminal  
* For our purpose, we will be using the “vcpkg install \[dependency\]” command. Here is [documentation](https://learn.microsoft.com/en-us/vcpkg/commands/install) on this command.

#### Potential Issues:

* CMake Error: CMake was unable to find a build program corresponding to "Ninja".  
    
  **Fix:**  
  By default, vcpkg uses the Ninja build system. Make sure you have the Ninja build system installed on your machine. Run ninja \--version to make sure that you have ninja installed and set up with your environment variables.

### **Step 1: Installing Dependencies**

 Overview of Dependencies we will need:

* OpenImageIO  
* LLVM  
* Flex  
* Bison

**This is NOT an exhaustive list of dependencies for OSL. For the first build we will skip optional dependencies like Python, Qt, etc.**

##### **Specifying triplets when installing dependencies:**

 You will probably need to specify the architecture triplet when installing your dependencies. For example, someone on windows x64 who wants to install the OpenImageIO dependency would use the following:  
vcpkg install openimageio:x64-windows

It is advised to specify the triplet to avoid linker issues later on when building with CMake. If you don’t specify the triplet, it will default to the following:  
Windows: x64-windows  
Linux: x64-linux  
OSX: x64-osx

#### **Step 1.1:**  Install OpenImageIO

* Run the following command from your terminal:

	vcpkg install openimageio:\[your triplet\]

* Installing openimageio gets us a lot of the dependencies we will need for OSL


#### **Step 1.2:**  Install LLVM

* Run the following command from your terminal:

	vcpkg install llvm:\[your triplet\]

* Installing LLVM will take time\!

#### **Step 1.3:**  Install Flex and Bison

* Flex and Bison don’t have a port on vcpkg. We will install them using this [repo](https://github.com/lexxmark/winflexbison).  
* Clone the repo and make a build folder in the root directory.  
* Cd into the build folder and configure your project using CMake:  
  * mkdir build  
  * cd build  
  * cmake ..  
    

  (I ended up using Visual Studio to build this project and generate the executables. I believe that there is a way to do this using CMake?)  
    
* Open the winflexbison.sln file using Visual Studio.  
* Build the solution (set the configuration to release and the architecture of your machine)  
* This should generate the executables for flex and bison in the specified path.  
  \[Path to repo\]\\winflexbison\\bin\\Release  
* This path will be crucial later when we tell CMake to find these executables to build OSL.

	

This completes the installation of the necessary dependencies\! (HOORAY\!\!\!)

#### Some helpful vcpkg commands:

* vcpkg install \[dependency\]:\[your triplet\]  
  Installs the dependency for the specified architecture. You can verify that the dependencies were installed by navigating to the installed folder in your vcpkg directory:  
  	\[PATH TO VCPKG DIR\]\\vcpkg\\installed\\x64-windows\\  
* vcpkg list  
  It is advised to specify the triplet to avoid linker issues later on when building with CMake. You can verify this (and other info) for your installed dependencies using vcpkg list.  This gives you a list of your dependencies along with their target triplet, version and description  
* vcpkg search \[dependency\]

	Returns all the existing ports for the specified dependencies that can be installed

### Step 2: Building OSL

Now that we have our dependencies, we will clone the OSL GitHub repository and build it using CMake.. We will use CMake arguments to customize the build process without modifying the CMakeLists.txt file.

#### **Step 2.1: Clone OSL and set Build Directory**

* Clone the [OSL github repository](https://github.com/AcademySoftwareFoundation/OpenShadingLanguage.git) using the following command:


git clone https://github.com/AcademySoftwareFoundation/OpenShadingLanguage.git 

* Cd into the repo and make a build folder  
  * mkdir build  
  * cd build

#### **Step 2.2: Configuring CMake**

Instead of simply calling cmake.. in the build repo, we will configure our project to make sure that our installed dependencies are linked properly. We will do so by setting additional CMake variables and flags in the configure command. The final CMake configure command should look like this:

cmake .. ^  
 \-DCMAKE\_TOOLCHAIN\_FILE=${VCPKG\_PATH}/scripts/buildsystems/vcpkg.cmake ^  
  \-DCMAKE\_BUILD\_TYPE=Release ^  
  \-DFLEX\_EXECUTABLE=${FLEX\_PATH}/bin/win\_flex.exe ^  
  \-DBISON\_EXECUTABLE=${BISON\_PATH}/bin/win\_bison.exe ^  
  \-DUSE\_PYTHON=0 \-DUSE\_QT6=0 \-DUSE\_QT5=0 \-DUSE\_PARTIO=0 ^  
  \-DCMAKE\_CXX\_FLAGS="/utf-8"  
  \-DCMAKE\_VERBOSE\_MAKEFILE=ON

* To specify that we are using vcpkg to manage dependencies, we set the \-DCMAKE\_TOOLCHAIN\_FILE as follows:

\-DCMAKE\_TOOLCHAIN\_FILE=${VCPKG\_PATH}/scripts/buildsystems/vcpkg.cmake 

* We then set the BUILD\_TYPE variable to Release mode as follows:

  \-DCMAKE\_BUILD\_TYPE=Release

* We then specify the paths to the Flex and Bison executables:

  \-DFLEX\_EXECUTABLE=${FLEX\_PATH}/bin/win\_flex.exe ^  
  \-DBISON\_EXECUTABLE=${BISON\_PATH}/bin/win\_bison.exe ^

* We then set the Python, Qt and PartIO flags to 0 so that we can build without them:

\-DUSE\_PYTHON=0 \-DUSE\_QT6=0 \-DUSE\_QT5=0 \-DUSE\_PARTIO=0 ^

* I had an error where my files were not being interpreted correctly because the utf-8 flag wasn’t set up correctly. So I added the flag:

\-DCMAKE\_CXX\_FLAGS="/utf-8"

* Set the CMake verbose command if you need more information to debug

\-DCMAKE\_VERBOSE\_MAKEFILE=ON

If you have any missing dependencies, you will get errors after completing this step. Install those dependencies, verify existing variables, add additional variables if needed, and try the CMake command again.

**Delete the contents of your build folder before you are re-configuring/ re-building your project.**

#### **Step 2.3: Building OSL**

After the configure command works without raising any errors, you’re ready to build OSL. This step might give you errors. You can use the CMake verbose command to get more information that will be helpful for debugging.

* Enter the following build command from your build directory:

\-DCMAKE\_VERBOSE\_MAKEFILE=ON

* OSL will start building. This might take some time.  
* We can verify that our build succeeded by running the executables generated in the release folder:

\[PATH TO OSL\]\\OpenShadingLanguage\\build\\bin\\Release

* For example, running  .\\oslinfo.exe should output:

“oslinfo \-- list parameters of a compiled OSL shader Open Shading Language 1.14.2.0dev…”

### Potential Build Issues:

* #### LINK : fatal error LNK1181: cannot open input file 'zstd.dll.lib' 

    
  **FIX:**   
  Look for the strange zstd.dll.lib file in the vcxproj that had the linker issue. We found four instances of zstd.dll.lib which we renamed to zstd.lib (the library that was installed and present in the vcpkg installed lib repo). This had to be done for both the oslexec.vcxproj and oslcomp.vcxproj files. After doing this, OSL was able to link to the zstd.lib library and the build succeeded.  
    
  We're still not sure exactly where the "zstd.dll.lib" file came from.  
    
  Link to a similar issue: [https://gitlab.com/taricorp/llvm-sys.rs/-/issues/68](https://gitlab.com/taricorp/llvm-sys.rs/-/issues/68)  
    
  Link explaining the ".dll.lib" ext: [https://gitlab.kitware.com/cmake/cmake/-/issues/25478](https://gitlab.kitware.com/cmake/cmake/-/issues/25478)

* #### 

### Resources:

* Join the OSL channel in the ASWF Slack: [https://slack.aswf.io/](https://slack.aswf.io/)  
* The [Install guide](https://github.com/AcademySoftwareFoundation/OpenShadingLanguage/blob/main/INSTALL.md).  
* 
