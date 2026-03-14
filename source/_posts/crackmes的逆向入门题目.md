title: crackmes的逆向入门题目
author: ming
tags:
  - 逆向
categories:
  - 逆向
date: 2026-03-13 10:56:00
---
# cbmhackers

https://crackmes.one/crackme/5b8a37a433c5d45fc286ad83

### 1. 下载解压
zip解压密码：crackmes.one

### 2. objdump反汇编
```
objdump -M intel -d rev50_linux64-bit

00000000000011c4 <main>:
    11c4:	55                   	push   rbp
    11c5:	48 89 e5             	mov    rbp,rsp
    11c8:	48 83 ec 10          	sub    rsp,0x10
    11cc:	89 7d fc             	mov    DWORD PTR [rbp-0x4],edi
    11cf:	48 89 75 f0          	mov    QWORD PTR [rbp-0x10],rsi
    11d3:	83 7d fc 02          	cmp    DWORD PTR [rbp-0x4],0x2
    11d7:	75 7e                	jne    1257 <main+0x93>
    11d9:	48 8b 45 f0          	mov    rax,QWORD PTR [rbp-0x10]
    11dd:	48 83 c0 08          	add    rax,0x8
    11e1:	48 8b 00             	mov    rax,QWORD PTR [rax]
    11e4:	48 89 c7             	mov    rdi,rax
    11e7:	e8 54 fe ff ff       	call   1040 <strlen@plt>
    11ec:	48 83 f8 0a          	cmp    rax,0xa
    11f0:	75 54                	jne    1246 <main+0x82>
    11f2:	48 8b 45 f0          	mov    rax,QWORD PTR [rbp-0x10]
    11f6:	48 83 c0 08          	add    rax,0x8
    11fa:	48 8b 00             	mov    rax,QWORD PTR [rax]
    11fd:	48 83 c0 04          	add    rax,0x4
    1201:	0f b6 00             	movzx  eax,BYTE PTR [rax]
    1204:	3c 40                	cmp    al,0x40
    1206:	75 2d                	jne    1235 <main+0x71>
    1208:	48 8d 3d 16 0e 00 00 	lea    rdi,[rip+0xe16]        # 2025 <_IO_stdin_used+0x25>
    120f:	e8 1c fe ff ff       	call   1030 <puts@plt>
    1214:	48 8b 45 f0          	mov    rax,QWORD PTR [rbp-0x10]
    1218:	48 83 c0 08          	add    rax,0x8
    121c:	48 8b 00             	mov    rax,QWORD PTR [rax]
    121f:	48 89 c6             	mov    rsi,rax
    1222:	48 8d 3d 07 0e 00 00 	lea    rdi,[rip+0xe07]        # 2030 <_IO_stdin_used+0x30>
    1229:	b8 00 00 00 00       	mov    eax,0x0
    122e:	e8 1d fe ff ff       	call   1050 <printf@plt>
    1233:	eb 31                	jmp    1266 <main+0xa2>
    1235:	48 8b 45 f0          	mov    rax,QWORD PTR [rbp-0x10]
    1239:	48 8b 00             	mov    rax,QWORD PTR [rax]
    123c:	48 89 c7             	mov    rdi,rax
    123f:	e8 46 ff ff ff       	call   118a <usage>
    1244:	eb 20                	jmp    1266 <main+0xa2>
    1246:	48 8b 45 f0          	mov    rax,QWORD PTR [rbp-0x10]
    124a:	48 8b 00             	mov    rax,QWORD PTR [rax]
    124d:	48 89 c7             	mov    rdi,rax
    1250:	e8 35 ff ff ff       	call   118a <usage>
    1255:	eb 0f                	jmp    1266 <main+0xa2>
    1257:	48 8b 45 f0          	mov    rax,QWORD PTR [rbp-0x10]
    125b:	48 8b 00             	mov    rax,QWORD PTR [rax]
    125e:	48 89 c7             	mov    rdi,rax
    1261:	e8 24 ff ff ff       	call   118a <usage>
    1266:	b8 00 00 00 00       	mov    eax,0x0
    126b:	c9                   	leave
    126c:	c3                   	ret
    126d:	0f 1f 00             	nop    DWORD PTR [rax]
```

### 3. gdb调试