GENSAT-1 CFS Custom App Development Guide
# Table of Contents
` `TOC \o "1-3" \h \z \u [Introduction	 PAGEREF _Toc83626276 \h 1](#_Toc83626276)

[Reference Documentation	 PAGEREF _Toc83626277 \h 1](#_Toc83626277)

[CFS Custom App Development	 PAGEREF _Toc83626278 \h 2](#_Toc83626278)

[Application Source Files	 PAGEREF _Toc83626279 \h 2](#_Toc83626279)

[Public_inc	 PAGEREF _Toc83626280 \h 2](#_Toc83626280)

[Platform_inc	 PAGEREF _Toc83626281 \h 2](#_Toc83626281)

[Src	 PAGEREF _Toc83626282 \h 2](#_Toc83626282)

[Functions	 PAGEREF _Toc83626283 \h 2](#_Toc83626283)

[CmakeList.txt	 PAGEREF _Toc83626284 \h 2](#_Toc83626284)

[Startup Script && Target File	 PAGEREF _Toc83626285 \h 2](#_Toc83626285)



# Introduction
This document is for engineers who are developing GENSAT-1 custom Applications for EPS, ADCS, GadMag, Agile, etc. Before diving into the code, make sure understand the IOs of your payload, CCSDS package, and the basic knowledge of the cFS. This document is focusing on the app itself. If you are unsure about how to build an embedded Linux image for the dev board (Xilinx Zedboard) using Petalinux, or how to build the cFS executable that is compatible with the dev board environment, **please go to the Development Board tab under Wiki on Teams.** 
# Reference Documentation
For those who haven’t have a basic understanding of the architecture of the cFS, here are a list of documents to read. Information of these documents is also referenced in this guide

- [cFS Overview](https://cfs.gsfc.nasa.gov/cFS-OviewBGSlideDeck-ExportControl-Final.pdf) (Official NASA Release)
- [cFE Core Services](https://teams.microsoft.com/l/file/7BEBEC13-F9B4-420B-9F1D-4E3C20E5DD34?tenantId=3bb1544e-a331-4d76-b49d-6f8f6586946e&fileType=docx&objectUrl=https%3A%2F%2Fgenesisesi.sharepoint.com%2Fsites%2FGENSAT%2FShared%20Documents%2FCommand%20and%20Data%20Handling%2FcFS_Information%2FcFE%20Core%20Services.docx&baseUrl=https%3A%2F%2Fgenesisesi.sharepoint.com%2Fsites%2FGENSAT&serviceName=teams&threadId=19:e0f20cf09b0b4b538a93a8b01733a499@thread.tacv2&groupId=aa8cb06c-779a-42cf-90ac-41d70b6c9cc1) (Chris P.)
- [cFE Application Develop Guide](https://github.com/nasa/cFE/blob/main/docs/cFE%20Application%20Developers%20Guide.md) (This document introduces each cFE services, I would ***highly recommend*** you read this document because it uses the ***sample app code*** to show how each cFE service contributes to an application, which is what is this Guide written for)
- [cFE_Users_Guide](https://teams.microsoft.com/l/file/DC1A57E0-78C4-4169-BE9D-0B130223C141?tenantId=3bb1544e-a331-4d76-b49d-6f8f6586946e&fileType=pdf&objectUrl=https%3A%2F%2Fgenesisesi.sharepoint.com%2Fsites%2FGENSAT%2FShared%20Documents%2FCommand%20and%20Data%20Handling%2FcFS_Information%2FcFE_Users_Guide.pdf&baseUrl=https%3A%2F%2Fgenesisesi.sharepoint.com%2Fsites%2FGENSAT&serviceName=teams&threadId=19:e0f20cf09b0b4b538a93a8b01733a499@thread.tacv2&groupId=aa8cb06c-779a-42cf-90ac-41d70b6c9cc1) (API for cFE functions, useful for understanding functionality and IOs)
- [OSAL_Users_Guide](https://teams.microsoft.com/l/file/86A03445-6C1C-4A9B-8C95-CB50ABE4E1AD?tenantId=3bb1544e-a331-4d76-b49d-6f8f6586946e&fileType=pdf&objectUrl=https%3A%2F%2Fgenesisesi.sharepoint.com%2Fsites%2FGENSAT%2FShared%20Documents%2FCommand%20and%20Data%20Handling%2FcFS_Information%2FOSAL_Users_Guide.pdf&baseUrl=https%3A%2F%2Fgenesisesi.sharepoint.com%2Fsites%2FGENSAT&serviceName=teams&threadId=19:e0f20cf09b0b4b538a93a8b01733a499@thread.tacv2&groupId=aa8cb06c-779a-42cf-90ac-41d70b6c9cc1) (API for OSAL functions, useful for understanding functionality and IOs)

# CFS Custom App Development
## Application Source Files

### Public\_inc

### Platform\_inc

### Src
#### *Application Message and Info (xx\_msg.h)*
In the Ap

#### *Version Number and Developments Build (xx\_version.h)*
The version number is a sequence of four numbers, in order, the Major number, the Minor number, the Revision number, and the Mission Revision. This version number is only updated upon official release of tagged versions, NOT on development builds. In order to distinguish between development versions, we also provide a BUILD\_NUMBER. For more detailed explanation on the meaning of each number, please refer to [cFE_Users_Guide](https://teams.microsoft.com/l/file/DC1A57E0-78C4-4169-BE9D-0B130223C141?tenantId=3bb1544e-a331-4d76-b49d-6f8f6586946e&fileType=pdf&objectUrl=https%3A%2F%2Fgenesisesi.sharepoint.com%2Fsites%2FGENSAT%2FShared%20Documents%2FCommand%20and%20Data%20Handling%2FcFS_Information%2FcFE_Users_Guide.pdf&baseUrl=https%3A%2F%2Fgenesisesi.sharepoint.com%2Fsites%2FGENSAT&serviceName=teams&threadId=19:e0f20cf09b0b4b538a93a8b01733a499@thread.tacv2&groupId=aa8cb06c-779a-42cf-90ac-41d70b6c9cc1) **section 7.** 

## Functions 
For more detailed function list and their descriptions, please use the [cFE_Users_Guide](https://teams.microsoft.com/l/file/DC1A57E0-78C4-4169-BE9D-0B130223C141?tenantId=3bb1544e-a331-4d76-b49d-6f8f6586946e&fileType=pdf&objectUrl=https%3A%2F%2Fgenesisesi.sharepoint.com%2Fsites%2FGENSAT%2FShared%20Documents%2FCommand%20and%20Data%20Handling%2FcFS_Information%2FcFE_Users_Guide.pdf&baseUrl=https%3A%2F%2Fgenesisesi.sharepoint.com%2Fsites%2FGENSAT&serviceName=teams&threadId=19:e0f20cf09b0b4b538a93a8b01733a499@thread.tacv2&groupId=aa8cb06c-779a-42cf-90ac-41d70b6c9cc1) as reference. This only lists out the major ones to use.

## CmakeList.txt
Editing the CmakeList.txt allows the developer to cross-compile different applications together. Using **sample\_ap**’s Cmakelist.txt as a reference for now

## Startup Script && Target File
After you edit your sources files and CmakeList.txt, one last thing before moving to make is editing the target and startup script.

You have edit **target.cmake** and **cpu1-cfe\_es\_startup.scr** inside the **sample\_def** directory (**sample**\_**def** directory is located at cFS/cfe/cmake/**sample\_def**, don’t change the files in the original location, instead copy the **sample\_def** directory into cFS home directory)

The **target.cmake** is used for the compiler to search for the corresponding app source folder (an error will be reported if the folder is missing)

# The "MISSION\_GLOBAL\_APPLIST" is a set of apps/libs that will be built

# for every defined and target.  These are built as dynamic modules

# and must be loaded explicitly via startup script or command.

# This list is effectively appended to every TGTx\_APPLIST in targets.cmake.  

# Example:

list(APPEND MISSION\_GLOBAL\_APPLIST sample\_app sample\_lib)

# The "MISSION\_GLOBAL\_STATIC\_APPLIST" is similar to MISSION\_GLOBAL\_APPLIST

# but the apps are statically linked.  

# This list is effectively appended to every TGTx\_STATIC\_APPLIST in targets.cmake.  

# Example:

#   list(APPEND MISSION\_GLOBAL\_STATIC\_APPLIST my\_static\_app)

# FT\_INSTALL\_SUBDIR indicates where the black box test data files (lua scripts) should

# be copied during the install process.

SET(FT\_INSTALL\_SUBDIR "host/functional-test")

# Each target board can have its own HW arch selection and set of included apps

SET(TGT1\_NAME cpu1)

SET(TGT1\_APPLIST ci\_lab to\_lab sch\_lab)

SET(TGT1\_FILELIST cfe\_es\_startup.scr)

To include the custom apps, add their app\_folder names (this must be matched exactly)  to 

This list is appended to 


The **cpu1-cfe\_es\_startup.scr** is the startup file parsed by the cFE core service when you run the cFS executable (runtime errors will be reported if the parsing process encounters error)

Add your new app based on above startup script format 

**Note**: make sure you add a **comma** after each field. **Don’t** write app startup script after the first !** because that indicates stop parsing for the cFE core service (everything after the first ! will be viewed as comments)

Last thing, if you want to exclude certain apps for testing purpose, just remove them from the list in **target.cmake** and comment their lines in **cpu1-cfe\_es\_startup.scr**



