# 补丁代码 #

```
diff -Naur kqemu-1.3.0pre11/Makefile.winnt kqemu-1.3.0pre11-new/Makefile.winnt
--- kqemu-1.3.0pre11/Makefile.winnt	2007-02-07 05:02:00 +0800
+++ kqemu-1.3.0pre11-new/Makefile.winnt	2008-08-03 22:41:52 +0800
@@ -3,7 +3,7 @@
 # (c) Filip Navara
 #
 OBJECTS = kqemu-win32.o kqemu-mod-i386-win32.o
-CROSS_PREFIX=i386-mingw32-
+CROSS_PREFIX=
 
 TARGET = kqemu.sys
 
diff -Naur kqemu-1.3.0pre11/common/Makefile kqemu-1.3.0pre11-new/common/Makefile
--- kqemu-1.3.0pre11/common/Makefile	2007-02-07 05:02:00 +0800
+++ kqemu-1.3.0pre11-new/common/Makefile	2008-08-03 23:23:45 +0800
@@ -32,8 +32,8 @@
 MON_LD=ld
 ifdef CONFIG_WIN32
 TARGET=../kqemu-mod-$(ARCH)-win32.o
-CC=i386-mingw32-gcc
-LD=i386-mingw32-ld
+CC=gcc
+LD=ld
 else
 TARGET=../kqemu-mod-$(ARCH).o
 CC=gcc
diff -Naur kqemu-1.3.0pre11/common/i386/monitor_asm.S kqemu-1.3.0pre11-new/common/i386/monitor_asm.S
--- kqemu-1.3.0pre11/common/i386/monitor_asm.S	2007-02-07 05:02:00 +0800
+++ kqemu-1.3.0pre11-new/common/i386/monitor_asm.S	2008-08-03 23:01:42 +0800
@@ -151,9 +151,8 @@
         jmp *%eax
                         
 #define SEG_EXCEPTION(label) \
-    .section "seg_ex_table", "a" ; \
-    .long label ; \
-    .previous
+    .section seg_ex_table, "a" ; \
+    .long label
 
 #ifdef USE_SEG_GP        
 /* %ebx contains the kqemu_state pointer, %eax the selector, 
diff -Naur kqemu-1.3.0pre11/common/interp.c kqemu-1.3.0pre11-new/common/interp.c
--- kqemu-1.3.0pre11/common/interp.c	2007-02-07 05:02:00 +0800
+++ kqemu-1.3.0pre11-new/common/interp.c	2008-08-03 23:19:16 +0800
@@ -2358,7 +2358,7 @@
 #endif
  next_byte:
     /* XXX: more precise test */
-    if (unlikely((pc - (unsigned long)&_start) < MONITOR_MEM_SIZE))
+    if (unlikely((pc - (unsigned long)&start) < MONITOR_MEM_SIZE))
         raise_exception(s, KQEMU_RET_SOFTMMU);
     b = ldub_mem_fast(s, pc + s->cpu_state.segs[R_CS].base);
     pc++;
@@ -4819,7 +4819,7 @@
             if (mod == 3 || !(s->cpuid_features & CPUID_FXSR))
                 goto illegal_op;
             addr = get_modrm(s, modrm);
-            if (unlikely((addr - ((unsigned long)&_start - 511)) < 
+            if (unlikely((addr - ((unsigned long)&start - 511)) < 
                          (MONITOR_MEM_SIZE + 511)))
                 raise_exception(s, KQEMU_RET_SOFTMMU);
 #ifdef __x86_64__
@@ -4841,7 +4841,7 @@
             if (mod == 3 || !(s->cpuid_features & CPUID_FXSR))
                 goto illegal_op;
             addr = get_modrm(s, modrm);
-            if (unlikely((addr - ((unsigned long)&_start - 511)) < 
+            if (unlikely((addr - ((unsigned long)&start - 511)) < 
                          (MONITOR_MEM_SIZE + 511)))
                 raise_exception(s, KQEMU_RET_SOFTMMU);
 #ifdef __x86_64__
diff -Naur kqemu-1.3.0pre11/common/kqemu_int.h kqemu-1.3.0pre11-new/common/kqemu_int.h
--- kqemu-1.3.0pre11/common/kqemu_int.h	2007-02-07 05:02:00 +0800
+++ kqemu-1.3.0pre11-new/common/kqemu_int.h	2008-08-03 23:22:17 +0800
@@ -1063,22 +1063,20 @@
 
 #ifdef __x86_64__
 #define MMU_EXCEPTION(label) \
-    ".section \"mmu_ex_table\", \"a\"\n"\
-    ".quad " #label "\n"\
-    ".previous\n"
+    ".section mmu_ex_table, \"a\"\n"\
+    ".quad " #label "\n"
 #else
 #define MMU_EXCEPTION(label) \
-    ".section \"mmu_ex_table\", \"a\"\n"\
-    ".long " #label "\n"\
-    ".previous\n"
+    ".section mmu_ex_table, \"a\"\n"\
+    ".long " #label "\n"
 #endif
 
-extern char _start;
+extern char start;
 
 static inline uint32_t ldub_mem(struct kqemu_state *s, unsigned long addr)
 {
     uint32_t res;
-    if (unlikely((addr - (unsigned long)&_start) < MONITOR_MEM_SIZE))
+    if (unlikely((addr - (unsigned long)&start) < MONITOR_MEM_SIZE))
         raise_exception(s, KQEMU_RET_SOFTMMU);
     asm volatile("1:\n"
                  "movzbl %1, %0\n" 
@@ -1091,7 +1089,7 @@
 static inline uint32_t lduw_mem(struct kqemu_state *s, unsigned long addr)
 {
     uint32_t res;
-    if (unlikely((addr - ((unsigned long)&_start - 1)) < 
+    if (unlikely((addr - ((unsigned long)&start - 1)) < 
                  (MONITOR_MEM_SIZE + 1)))
         raise_exception(s, KQEMU_RET_SOFTMMU);
     asm volatile("1:\n"
@@ -1105,7 +1103,7 @@
 static inline uint32_t ldl_mem(struct kqemu_state *s, unsigned long addr)
 {
     uint32_t res;
-    if (unlikely((addr - ((unsigned long)&_start - 3)) < 
+    if (unlikely((addr - ((unsigned long)&start - 3)) < 
                  (MONITOR_MEM_SIZE + 3)))
         raise_exception(s, KQEMU_RET_SOFTMMU);
     asm volatile("1:\n"
@@ -1120,7 +1118,7 @@
 static inline uint64_t ldq_mem(struct kqemu_state *s, unsigned long addr)
 {
     uint64_t res;
-    if (unlikely((addr - ((unsigned long)&_start - 7)) < 
+    if (unlikely((addr - ((unsigned long)&start - 7)) < 
                  (MONITOR_MEM_SIZE + 7)))
         raise_exception(s, KQEMU_RET_SOFTMMU);
     asm volatile("1:\n"
@@ -1167,7 +1165,7 @@
 
 static inline void stb_mem(struct kqemu_state *s, unsigned long addr, uint32_t val)
 {
-    if (unlikely((addr - (unsigned long)&_start) < MONITOR_MEM_SIZE))
+    if (unlikely((addr - (unsigned long)&start) < MONITOR_MEM_SIZE))
         raise_exception(s, KQEMU_RET_SOFTMMU);
     asm volatile("1:\n"
                  "movb %b0, %1\n" 
@@ -1178,7 +1176,7 @@
 
 static inline void stw_mem(struct kqemu_state *s, unsigned long addr, uint32_t val)
 {
-    if (unlikely((addr - ((unsigned long)&_start - 1)) < 
+    if (unlikely((addr - ((unsigned long)&start - 1)) < 
                  (MONITOR_MEM_SIZE + 1)))
         raise_exception(s, KQEMU_RET_SOFTMMU);
     asm volatile("1:\n"
@@ -1190,7 +1188,7 @@
 
 static inline void stl_mem(struct kqemu_state *s, unsigned long addr, uint32_t val)
 {
-    if (unlikely((addr - ((unsigned long)&_start - 3)) < 
+    if (unlikely((addr - ((unsigned long)&start - 3)) < 
                  (MONITOR_MEM_SIZE + 3)))
         raise_exception(s, KQEMU_RET_SOFTMMU);
     asm volatile("1:\n"
@@ -1203,7 +1201,7 @@
 #if defined (__x86_64__)
 static inline void stq_mem(struct kqemu_state *s, unsigned long addr, uint64_t val)
 {
-    if (unlikely((addr - ((unsigned long)&_start - 7)) < 
+    if (unlikely((addr - ((unsigned long)&start - 7)) < 
                  (MONITOR_MEM_SIZE + 7)))
         raise_exception(s, KQEMU_RET_SOFTMMU);
     asm volatile("1:\n"
@@ -1226,14 +1224,12 @@
 
 #ifdef __x86_64__
 #define SEG_EXCEPTION(label) \
-    ".section \"seg_ex_table\", \"a\"\n"\
-    ".quad " #label "\n"\
-    ".previous\n"
+    ".section seg_ex_table, \"a\"\n"\
+    ".quad " #label "\n"
 #else
 #define SEG_EXCEPTION(label) \
-    ".section \"seg_ex_table\", \"a\"\n"\
-    ".long " #label "\n"\
-    ".previous\n"
+    ".section seg_ex_table, \"a\"\n"\
+    ".long " #label "\n"
 #endif
 
 static inline unsigned long compute_eflags_user(struct kqemu_state *s, 
diff -Naur kqemu-1.3.0pre11/common/monitor.c kqemu-1.3.0pre11-new/common/monitor.c
--- kqemu-1.3.0pre11/common/monitor.c	2007-02-07 05:02:00 +0800
+++ kqemu-1.3.0pre11-new/common/monitor.c	2008-08-03 23:23:09 +0800
@@ -1494,8 +1494,8 @@
     }
 }
 
-extern unsigned long __start_seg_ex_table;
-extern unsigned long __stop_seg_ex_table;
+extern unsigned long _start_seg_ex_table;
+extern unsigned long _stop_seg_ex_table;
 
 static void handle_mon_exception(struct kqemu_state *s, 
                                  struct kqemu_exception_regs *regs,
@@ -1504,7 +1504,7 @@
     unsigned long pc, *p;
     
     pc = regs->eip;
-    for(p = &__start_seg_ex_table; p != &__stop_seg_ex_table; p++) {
+    for(p = &_start_seg_ex_table; p != &_stop_seg_ex_table; p++) {
         if (*p == pc) goto found;
     }
     monitor_panic_regs(s, regs, 
diff -Naur kqemu-1.3.0pre11/common/x86_64/monitor_asm.S kqemu-1.3.0pre11-new/common/x86_64/monitor_asm.S
--- kqemu-1.3.0pre11/common/x86_64/monitor_asm.S	2007-02-07 05:02:00 +0800
+++ kqemu-1.3.0pre11-new/common/x86_64/monitor_asm.S	2008-08-03 23:37:02 +0800
@@ -21,10 +21,9 @@
 #include "kqemu_int.h"
 
 #define SEG_EXCEPTION(label) \
-    .section "seg_ex_table", "a" ; \
+    .section seg_ex_table, "a" ; \
     .align 8 ; \
-    .quad label ; \
-    .previous
+    .quad label ;
 
 /* unsigned long __exec_binary(unsigned long *eflags, int op, 
                                unsigned long a, unsigned long b) */

```