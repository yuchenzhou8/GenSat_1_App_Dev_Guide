# GENSAT_cFS_APP_Development_Guide

Table of Contents
-----------------

    * [Table of Contents](#table-of-contents)
    * [1. Inroduction](#1-introduction) 
# 1. Introduction

This document is for engineers who are developing GENSAT-1 custom Applications for EPS, ADCS, GadMag, Agile, etc. Before diving into the code, make sure understand the IOs of your payload, CCSDS package, and the basic knowledge of the cFS. This document is focusing on the app itself. If you are unsure about how to build an embedded Linux image for the dev board (Xilinx Zedboard) using Petalinux, or how to build the cFS executable that is compatible with the dev board environment, **please go to the Development Board tab under Wiki on Teams**. 

# 2. Reference Documentation
  * [cFS Overview](https://cfs.gsfc.nasa.gov/cFS-OviewBGSlideDeck-ExportControl-Final.pdf)</br> Official NASA Release
  * [cFE Core Service](cFE_Core_Services.pdf)</br> Info collected by Chris P.
  * [cFE Application Develop](https://github.com/nasa/cFE/blob/main/docs/cFE%20Application%20Developers%20Guide.md)</br> This document introduces each cFE services, I would ***highly recommend*** you read this document because it uses the ***sample app*** code to show how each cFE service contributes to an application, which is what is this Guide written for
  * [cFE User Guide](cFE_Users_Guide.pdf)</br> (API for cFE functions, useful for understanding functionality and IOs)
  * [OSAL User Guide](OSAL_Users_Guide.pdf)</br> (API for OSAL functions, useful for understanding functionality and IOs)


# 3. CFS Custom App Developmeent
## 3.1 Application Directory Tree

## 3.2 Application Source Files

### 3.2.1 Public Inc

### 3.2.2 Platform Inc

### 3.2.3 Src

#### 3.2.3.1 _Version Number and Developments Build (xx_version.h)_

The version number is a sequence of four numbers, in order, the Major number, the Minor number, the Revision number, and the Mission Revision. This version number is only updated upon official release of tagged versions, NOT on development builds. In order to distinguish between development versions, we also provide a BUILD_NUMBER. For more detailed explanation on the meaning of each number, please refer to [cFE User Guide](cFE_Users_Guide.pdf) section 7. 

#### 3.2.3.2 _Application Message and Info (xx_msg.h)_

#### 3.2.3.3 _Application Events (xx_events.h)_

### 3.2.4 Tables 

### 3.2.5 Unit-tests

Don't worry about them for now

## 3.3 Common Functions

## 3.4 CmakeList.txt

Editing the CmakeList.txt allows the developer to cross-compile different applications together. Using the **sample_app**’s Cmakelist.txt as a reference for now

## 3.5 Startup Script && Target Files

After you edit your sources files and CmakeList.txt, one last thing before moving to make is editing the target and startup script.

You have edit **target.cmake** and **cpu1-cfe_es_startup.scr** inside the **sample_def** directory (**sample_def** directory is located at cFS/cfe/cmake/**sample_def**, don’t change the files in the original location, instead copy the **sample_def** directory into cFS home directory)

### 3.5.1 target.cmake

The **target.cmake** is used for the compiler to search for the corresponding app source folder (an error will be reported if the folder is missing)

```
# The "MISSION_GLOBAL_APPLIST" is a set of apps/libs that will be built
# for every defined and target.  These are built as dynamic modules
# and must be loaded explicitly via startup script or command.
# This list is effectively appended to every TGTx_APPLIST in targets.cmake.  
# Example:
list(APPEND MISSION_GLOBAL_APPLIST sample_app sample_lib)

# The "MISSION_GLOBAL_STATIC_APPLIST" is similar to MISSION_GLOBAL_APPLIST
# but the apps are statically linked.  
# This list is effectively appended to every TGTx_STATIC_APPLIST in targets.cmake.  
# Example:
# list(APPEND MISSION_GLOBAL_STATIC_APPLIST my_static_app)

# FT_INSTALL_SUBDIR indicates where the black box test data files (lua scripts) should
# be copied during the install process.
SET(FT_INSTALL_SUBDIR "host/functional-test")

# Each target board can have its own HW arch selection and set of included apps
SET(TGT1_NAME cpu1)
SET(TGT1_APPLIST ci_lab to_lab sch_lab)
SET(TGT1_FILELIST cfe_es_startup.scr)

```
To include the custom apps, add their app_folder names (this must be matched exactly) to

```list(APPEND MISSION_GLOBAL_APPLIST sample_app sample_lib)``` 

For example, to include **gensat_ci** and **gensat_to** applications, the list will become 

```list(APPEND MISSION_GLOBAL_APPLIST sample_app sample_lib gensat_ci gensat_to gensat_io_lib)```

### 3.5.2 cpu1-cfe_es_start.scr

The **cpu1-cfe_es_startup.scr** is the startup file parsed by the cFE core service when you run the cFS executable (runtime errors will be reported if the parsing process encounters error)

```
CFE_LIB, cfe_assert,  CFE_Assert_LibInit, ASSERT_LIB,    0,   0,     0x0, 0;
CFE_LIB, sample_lib,  SAMPLE_LIB_Init,    SAMPLE_LIB,    0,   0,     0x0, 0;
CFE_APP, sample_app,  SAMPLE_APP_Main,    SAMPLE_APP,   50,   16384, 0x0, 0;
CFE_APP, ci_lab,      CI_Lab_AppMain,     CI_LAB_APP,   60,   16384, 0x0, 0;
CFE_APP, to_lab,      TO_Lab_AppMain,     TO_LAB_APP,   70,   16384, 0x0, 0;
CFE_APP, sch_lab,     SCH_Lab_AppMain,    SCH_LAB_APP,  80,   16384, 0x0, 0;
!
! Startup script fields:
! 1. Object Type      -- CFE_APP for an Application, or CFE_LIB for a library.
! 2. Path/Filename    -- This is a cFE Virtual filename, not a vxWorks device/pathname
! 3. Entry Point      -- This is the "main" function for Apps.
! 4. CFE Name         -- The cFE name for the APP or Library
! 5. Priority         -- This is the Priority of the App, not used for Library
! 6. Stack Size       -- This is the Stack size for the App, not used for the Library
! 7. Load Address     -- This is the Optional Load Address for the App or Library. Currently not implemented
!                        so keep it at 0x0.
! 8. Exception Action -- This is the Action the cFE should take if the App has an exception.
!                        0        = Just restart the Application
!                        Non-Zero = Do a cFE Processor Reset
!
```

Add your new app's startup script based on above format

**Note**: make sure you add a **comma(,)** after each field. **Don’t** write app startup script after the first ! because that indicates stop parsing for the cFE core service (everything after the first ! will be viewed as comments)

Last but not least, if you want to exclude certain apps for testing purpose, just remove them from the list in **target.cmake** and comment their lines in **cpu1-cfe_es_startup.scr**
