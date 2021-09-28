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

The following shows the standard developement directory of each custom application
```
-- xx_app
    |-- fsw
    |   |-- mission_inc
    |   |   |-- public/interface headers for the application
    |   |-- platform_inc
    |   |   |-- public/interface headers for the application
    |   |-- src
    |   |   |-- contains source files and private headers for the application
    |   |-- table
    |   |   |-- contains table defintions that will be registered in the initialization stage
    |   |-- unit-test
    |   |   |-- contains unit test for different functionalities of the application
    |-- CmakeLists.txt
    |   |-- contains a set of directive and instrutions describing the application's source files and target(executable, library, table)

```
| **Files**                             | **Descriptions**                                                                                             |
|:--------------------------------------|:-------------------------------------------------------------------------------------------------------------|
| fsw/src/xx_app.c                      | Main source code for xx_app. Located in src directory.                                                       |
| fsw/src/xx_app.h                      | Main header file for xx_app. It contains your main global typedef, prototypes, and miscellaneous define.     |
| fsw/src/xx_app_events.h               | Defines xx_app event IDs                                                                                     |
| fsw/src/xx_app_msg.h                  | Defines xx_app commands and its structures                                                                   |
| fsw/src/xx_app_version.h              | Defines xx-app verison numbers and development ids                                                           |
| fsw/tables/xx_app_table.c             | Define xx_app table(s)                                                                                       |
| fsw/platform_inc/xx_app_msgids.h      | Define xx_app message IDs                                                                                    |
| fsw/mission_inc/xx_app_perfids.h      | Define xx_app performance IDs                                                                                |

**Note**: 
  * Naming conventions does not have be followed perfectly. However, you should double-check the spelling whenever a file is referenced in a different location
  * Source files are not limited to ones listed. You can add designated header files for specific applications.

## 3.2 Application Source Files

### 3.2.1 Mission Inc

#### 3.2.1.1 Application Performance IDs (xx_perfids.h)

This header file contains the performance ids for application's software performance analysis. Performance Monitor IDs 0-31 are reserved by cFE. Performance ids should be unique for each application. Check all application's performance IDs before assigning a new performance id to a new application.

```

#define CFE_MISSION_ES_PERF_MAX_IDS 128

```

### 3.2.2 Platform Inc

### 3.2.3 Src

The src directory contains the main source files and private header files for the application. Anything not shared across applications should theoritcally be stored in this directory

#### 3.2.3.1 _Version Number and Developments Build (xx_version.h)_

The version number is a sequence of four numbers, in order, the Major number, the Minor number, the Revision number, and the Mission Revision. This version number is only updated upon official release of tagged versions, NOT on development builds. In order to distinguish between development versions, we also provide a BUILD_NUMBER. For more detailed explanation on the meaning of each number, please refer to [cFE User Guide](cFE_Users_Guide.pdf) section 7. 

#### 3.2.3.2 _Application Message and Info (xx_msg.h)_

This header file contains the command and telemetry mesasge definitions, including command code (`__CC`) number assignments

Example Message Definitions for gensat_ci

```
/*
** Type definition (generic "no arguments" command)
** Example CMD: NOOP, ResetCounter
*/
typedef struct/*
** Type definition (generic "no arguments" command)
** Example CMD: NOOP, ResetCounter
*
{
    uint8 CmdHeader[CFE_SB_CMD_HDR_SIZE];

} GENSAT_CI_NoArgsCmd_t;

----------------------------------------------------------------

/*
** Type definition ("arguments" command)
** Example CMD: Report HouseKeeping Telemetry
*/

typedef struct
{
    uint8  CommandErrorCounter;
    uint8  CommandCounter;
    uint8  EnableChecksums;
    uint8  SerialConnected;
    uint8  Spare1[8];
    uint32 IngestPackets;
    uint32 IngestErrors;
    uint32 Spare2;

} GENSAT_CI_HkTlm_Payload_t;

typedef struct
{
    uint8                     TlmHeader[CFE_SB_TLM_HDR_SIZE];
    GENSAT_CI_HkTlm_Payload_t Payload;
} OS_PACK GENSAT_CI_HkTlm_t;

```
A cFS CCSDS command/telemetry package is consisted of a primary header，a secondary header（structed differently for commands and telemetries), and a user data field(usually contains command arguments for command and data for telemtry). For the Noop command message, there is no argument, thus the message definition only includes the `CFE_SB_CMD_HDR`. Instead for the housekeeping telemetry message, it uses the user data field for the housekeeping information. Thus, in addition to the header `CFE_SB_TLM_HDR`, it contains the `GENSAT_CI_HkTlm_Payload_t` as a part of the telemtry message struct.

CMD/TLM header structures are shown to assist you to better understand the message package

```
/*----- CCSDS packet primary header. -----*/

typedef struct {

   uint8   StreamId[2];  /* packet identifier word (stream ID) */
      /*  bits  shift   ------------ description ---------------- */
      /* 0x07FF    0  : application ID                            */
      /* 0x0800   11  : secondary header: 0 = absent, 1 = present */
      /* 0x1000   12  : packet type:      0 = TLM, 1 = CMD        */
      /* 0xE000   13  : CCSDS version:    0 = ver 1, 1 = ver 2    */

   uint8   Sequence[2];  /* packet sequence word */
      /*  bits  shift   ------------ description ---------------- */
      /* 0x3FFF    0  : sequence count                            */
      /* 0xC000   14  : segmentation flags:  3 = complete packet  */

   uint8  Length[2];     /* packet length word */
      /*  bits  shift   ------------ description ---------------- */
      /* 0xFFFF    0  : (total packet length) - 7                 */

} CCSDS_PriHdr_t;

/*----- CCSDS command secondary header. -----*/

typedef struct {

   uint8 FunctionCode; /* Command Function Code */
                       /* bits shift ---------description-------- */
                       /* 0x7F  0    Command function code        */
                       /* 0x80  7    Reserved                     */

   uint8 Checksum;     /* Command checksum  (all bits, 0xFF)      */

} CCSDS_CmdSecHdr_t;

/*----- CCSDS telemetry secondary header. -----*/

typedef struct {

   uint8  Time[CCSDS_TIME_SIZE];

} CCSDS_TlmSecHdr_t;

typedef struct
{
    CCSDS_SpacePacket_t  SpacePacket;   /**< \brief Standard Header on all packets  */
    CCSDS_CmdSecHdr_t    Sec;
} CCSDS_CommandPacket_t;

/*----- Generic combined telemetry header. -----*/

typedef struct
{
    CCSDS_SpacePacket_t  SpacePacket;   /**< \brief Standard Header on all packets */
    CCSDS_TlmSecHdr_t    Sec;
} CCSDS_TelemetryPacket_t;

/** \brief Generic Software Bus Message Type Definition */
typedef union {
    CCSDS_PriHdr_t      Hdr;   /**< \brief CCSDS Primary Header #CCSDS_PriHdr_t */
    CCSDS_SpacePacket_t SpacePacket;
    uint32              Dword; /**< \brief Forces minimum of 32-bit alignment for this object */
    uint8               Byte[sizeof(CCSDS_PriHdr_t)];   /**< \brief Allows byte-level access */
}CFE_SB_Msg_t;

/** \brief Generic Software Bus Command Header Type Definition */
typedef union {
    CCSDS_CommandPacket_t   Cmd;
    CFE_SB_Msg_t            BaseMsg; /**< Base type (primary header) */
} CFE_SB_CmdHdr_t;

/** \brief Generic Software Bus Telemetry Header Type Definition */
typedef union {
    CCSDS_TelemetryPacket_t Tlm;
    CFE_SB_Msg_t            BaseMsg; /**< Base type (primary header) */
} CFE_SB_TlmHdr_t;

#define CFE_SB_CMD_HDR_SIZE     (sizeof(CFE_SB_CmdHdr_t))/**< \brief Size of #CFE_SB_CmdHdr_t in bytes */
#define CFE_SB_TLM_HDR_SIZE     (sizeof(CFE_SB_TlmHdr_t))/**< \brief Size of #CFE_SB_TlmHdr_t in bytes */

```

#### 3.2.3.3 _Application Events (xx_events.h)_

This header file contains the Event ID(`__EID`) number assignments for events generated while the application is running. Event statements will be downlinked to the ground station, so be aware when you want to save a log while the cFS is running, making sure whether you want it to be saved locally on the satellite or tobe downlinked to the ground.

```
#define GENSAT_CI_RESERVED_EID         0
#define GENSAT_CI_INF_EID              1
#define GENSAT_CI_STARTUP_INF_EID      2
#define GENSAT_CI_COMMANDNOP_INF_EID   3
#define GENSAT_CI_COMMANDRST_INF_EID   4
#define GENSAT_CI_INGEST_INF_EID       5
#define GENSAT_CI_ERR_EID              6
#define GENSAT_CI_SERIALCREATE_ERR_EID 7
#define GENSAT_CI_COMMAND_ERR_EID      8
#define GENSAT_CI_INGEST_ERR_EID       9
#define GENSAT_CI_LEN_ERR_EID          10
#endif /* _gensat_ci_events_h_ */

```

#### 3.2.3.4 _Application Header (xx_app.h)_

#### 3.2.3.5 _Application Main File (xx_app.c)_

### 3.2.4 Tables 

### 3.2.5 Unit-tests

Don't worry about them for now

## 3.3 Common Functions

## 3.4 CmakeList.txt

Editing the CmakeList.txt allows the developer to cross-compile different applications together. Using the **sample_app**’s Cmakelist.txt as a reference for now

## 3.5 Startup Script && Target Files

After you edit your sources files and CmakeList.txt, one last thing before moving to make is editing the target and startup script.

You have to edit **target.cmake** and **cpu1-cfe_es_startup.scr** inside the **sample_def** directory (**sample_def** directory is located at cFS/cfe/cmake/**sample_def**, don’t change the files in the original location, instead copy the **sample_def** directory into cFS home directory)

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

The **cpu1-cfe_es_startup.scr** is the startup file parsed by the cFE core service when you run the cFS executable. The startup script is a text file, written by the user that contains a list of entries (one entry for each application) and is used by the ES application for automating the startup of applications. (runtime errors will be reported if the parsing process encounters error) please refer to [cFE User Guide](cFE_Users_Guide.pdf) section 9.1.3 **"Startup Script"**. 

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
