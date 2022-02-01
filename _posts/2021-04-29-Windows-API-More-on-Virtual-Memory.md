---
title: Windows API - More on Virtual Memory.
date: 2021-04-18 11:33:00 +0800
categories: [Programming, Windows, Windows Internals, WinAPI]
tags: [Windows, Windows API Series, Virtual Memory Management]
math: false
mermaid: false
---

In the last post, we learnt about basics of virtual memory management and we also learnt about two Windows API functions that allow us to allocate virtual memory (using `VirtualAlloc`) and free it (using `VirtualFree`).    
In this post, we will continue our exploration of the Windows API functions that allow us to play with the virtual memory.   
The particular function that we are going to learn about in this blog post is `VirtualQuery`.

# GetLastError
Before we start, I would like to introduce you to a function from the Windows API, it is `GetLastError`. It is used to get **the _error code_ of the last error that occurred** and we can get more information about the error code by looking at the error code list which is available at msdn here :
[System Error Codes - Win32 apps](https://docs.microsoft.com/en-us/windows/win32/debug/system-error-codes#system-error-codes-1)    
We will be using this function in the code examples to see if there are any errors in our code.

# 1. VirtualQuery
---
This function is used to query the information of a virtual memory region.

## Function signature
This is the syntax for `VirtualQuery` function:
```c
SIZE_T VirtualQuery(
  LPCVOID                   lpAddress,
  PMEMORY_BASIC_INFORMATION lpBuffer,
  SIZE_T                    dwLength
);
```
## Arguments
The function's return type is `SIZE_T`, it's basically an `unsigned int`.    
**lpAddress**: You might already know the use of this argument if you have read the part one of this blog, it's basically the base address of Virtual Memory region that we allocated which is returned by `VirtualAlloc`.    
**lpBuffer**; This argument is a pointer to a struct that is defined in `winint.h`, it looks like this:
```c
typedef struct _MEMORY_BASIC_INFORMATION {
  PVOID  BaseAddress;
  PVOID  AllocationBase;
  DWORD  AllocationProtect;
  WORD   PartitionId;
  SIZE_T RegionSize;
  DWORD  State;
  DWORD  Protect;
  DWORD  Type;
} MEMORY_BASIC_INFORMATION, *PMEMORY_BASIC_INFORMATION;
```
I'll explain it's members later.    
`dwLength`: This argument is the `size of` the struct from the last argument.

## Return value
Instead of returning something the function just updates the struct that we had created.

# Examples
As we have learnt enough about the function, let's take a look at some examples and see the function and it's working in action.

## Example #1
Now as we have done with understanding of the function, we'll see some code examples of the function. We are going to make a program that will give us the information about a memory region that we'll allocate using the functions that we learnt about in the last blog post. Let me show you the code first, then I will explain it:
```cpp
#include <Windows.h>
#include <stdio.h>

int main()
{
	MEMORY_BASIC_INFORMATION info; // defining the struct
	int ret;
	int *vm = VirtualAlloc(NULL, 8, MEM_COMMIT, PAGE_READONLY); // 8 byte allocation.
	ret = VirtualQuery(vm, &info, sizeof(info));
	if (!ret) // error checking.
	{
		printf("VirtualQuery failed\n");
		printf("The error code for the last error was %d", GetLastError());
		return 1;
	}

	switch (info.AllocationProtect)
	{
		case PAGE_EXECUTE_READ:
			printf("Protection type : EXECUTE + READ\n");
			break;
		case PAGE_READWRITE:
			printf("Protection type : READ + WRITE\n");
			break;
		case PAGE_READONLY:
			printf("Protection type : READ\n");
			break;
		default:
			printf("Not found");
			break;
	}

	switch (info.State)
	{
		case MEM_COMMIT:
			printf("Region State : Committed");
			break;
		case MEM_FREE:
			printf("Region State : Free");
			break;
		case MEM_RESERVE:
			printf("Region State : Reserve");
			break;
		default:
			break;
	}
	VirtualFree(vm, 8, MEM_DECOMMIT); // free the allocated memory.
	return 0;
}
```

I have used `Windows.h` instead of using any other header file because `Windows.h` contains almost everything that we need for doing Windows API programming.   
Let's now understand the code.     
First, we have declared a struct of type `MEMORY_BASIC_INFORMATION`, which is the struct that we talked about, then we committed eight *bytes* of virtual memory which is read-only.    
After that, we have used `VirtualQuery` function to get information about that memory region.    
We gave it the address of the allocated memory region as our first parameter, then we gave the address of the `info` struct that will hold all the returned data from this function, then we gave it the `size of` our info struct.    
Then, we are doing a check if the function is failed, If it's failed then the error code can be found by using the `GetLastError` function.    
Then, we have a switch-case clause, where we are checking the value of `AllocationProtect` member of our `info` struct. This switch-case clause will check for the protection type of the virtual memory region that was specified as the first parameter.    
The constants that are being used to compare in the switch-case clause are defined in the `Windows.h` header file that we included.   
We are then checking the value of `State` member from our `info` struct. This switch-case clause is comparing the state of the allocated virtual memory region. Then, we are just printing information according to the statements. One thing to note is that we cannot compare the value with every type of protection type or every type of memory state, I have tried doing so but I was unsuccessful, so I am have just used the types that can be compared.    
Then we just free the allocated memory.

## Results #1 
Here's the output that I get after running the example:
```console
$ ./vquery-example
Protection type : READ
Region State : Committed
```

The results are expected, we had hardcoded the page protection to be read-only and the page state to committed and the result by the function is precise.     

## Code Example #2
In this example, we are going to implement a functionality to get information from any memory region. This one is more fun. This is the code:
```cpp
#include <Windows.h>
#include <stdio.h>

int main()
{
	MEMORY_BASIC_INFORMATION info;
	int ret;
	const void *location;
	int *vm = VirtualAlloc(NULL, 8, MEM_COMMIT, PAGE_READONLY);
	printf("Address of memory returned by VirtualAlloc is %lu\n", vm);
	printf("Enter the memory address that you want to query: ");
	scanf("%lu", &location);
	ret = VirtualQuery(location, &info, sizeof(info));
	if (!ret)
	{
		printf("VirtualQuery failed\n");
		printf("The error code for the last error was %d", GetLastError());
		return 1;
	}
	printf("Protection type : ");
	switch (info.AllocationProtect)
	{
		case PAGE_EXECUTE_READ:
			printf("EXECUTE + READ\n");
			break;
		case PAGE_READWRITE:
			printf("READ + WRITE\n");
			break;
		case PAGE_READONLY:
			printf("READ\n");
			break;
		default:
			printf("Unknown\n");
			break;
	}

	printf("Region State : ");
	switch (info.State)
	{
		case MEM_COMMIT:
			printf("Committed");
			break;
		case MEM_FREE:
			printf("Free");
			break;
		case MEM_RESERVE:
			printf("Reserve");
			break;
		default:
			printf("Unknown");
			break;
	}
	return 0;
}
```

Not so much changed, let me explain. The code is almost same as the first code but here we first printed the memory address that we allocated using `VirtuaAlloc` and then we asked the user for a memory address. Instead of directly querying the memory address returned by `VirtualAlloc`, We first printed it and then asked for that memory address. By doing this, we can verify that the results of `VirtualQuery` by looking at what we hardcoded. Also, you can give this program any other memory address and it'll query it's information. Here is the output of the code:
```cpp
Address of memory returned by VirtualAlloc is 1572864
Enter the memory address that you want to query: 1572864 
Protection type : READ  
Region State : Committed
```

And as we expected, we hardcoded the protection type as READ-ONLY and Region state as Committed and it gave us correct information. Coding this was fun and I also recommend you to make something with all the Memory API related functions that we learnt about. 

I hope you liked this one, meet you in the next one!
