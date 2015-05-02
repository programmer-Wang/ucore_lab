# Lab1 erport

## [��ϰ1]

[��ϰ1.1] ����ϵͳ�����ļ� ucore.img �����һ��һ�����ɵ�?(��Ҫ�Ƚ���ϸ�ؽ��� Makefile ��
ÿһ������������������ĺ���,�Լ�˵������µĽ��)

```	
>
����ucore.img��������Ҫ����bootblock��kernel
>����kernel����Ϊ:
$(kernel): tools/kernel.ld

$(kernel): $(KOBJS)
	@echo + ld $@
	$(V)$(LD) $(LDFLAGS) -T tools/kernel.ld -o $@ $(KOBJS)
	@$(OBJDUMP) -S $@ > $(call asmfile,kernel)
	@$(OBJDUMP) -t $@ | $(SED) '1,/SYMBOL TABLE/d; s/ .* / /; /^$$/d' > $(call symfile,kernel)

����kernel��Ҫkernel.ld init.o readline.o stdio.o kdebug.o
kmonitor.o panic.o clock.o console.o intr.o picirq.o trap.o
trapentry.o vectors.o pmm.o  printfmt.o string.o
������init.o�ļ�Ϊ��,
ʵ������Ϊ
gcc -Ikern/init/ -fno-builtin -Wall -ggdb -m32 \
	-gstabs -nostdinc  -fno-stack-protector \
	-Ilibs/ -Ikern/debug/ -Ikern/driver/ \
	-Ikern/trap/ -Ikern/mm/ -c kern/init/init.c \
	-o obj/kern/init/init.o
����kernelʱ��������:
ld -m    elf_i386 -nostdlib -T tools/kernel.ld -o bin/kernel \
	obj/kern/init/init.o obj/kern/libs/readline.o \
	obj/kern/libs/stdio.o obj/kern/debug/kdebug.o \
	obj/kern/debug/kmonitor.o obj/kern/debug/panic.o \
	obj/kern/driver/clock.o obj/kern/driver/console.o \
	obj/kern/driver/intr.o obj/kern/driver/picirq.o \
	obj/kern/trap/trap.o obj/kern/trap/trapentry.o \
	obj/kern/trap/vectors.o obj/kern/mm/pmm.o \
	obj/libs/printfmt.o obj/libs/string.o
	�����³��ֵĹؼ�����Ϊ
	T <scriptfile>  ��������ʹ��ָ���Ľű�
>����bootblock����ش���Ϊ
# create bootblock
bootfiles = $(call listf_cc,boot)
$(foreach f,$(bootfiles),$(call cc_compile,$(f),$(CC),$(CFLAGS) -Os -nostdinc))

bootblock = $(call totarget,bootblock)

$(bootblock): $(call toobj,$(bootfiles)) | $(call totarget,sign)
	@echo + ld $@
	$(V)$(LD) $(LDFLAGS) -N -e start -Ttext 0x7C00 $^ -o $(call toobj,bootblock)
	@$(OBJDUMP) -S $(call objfile,bootblock) > $(call asmfile,bootblock)
	@$(OBJCOPY) -S -O binary $(call objfile,bootblock) $(call outfile,bootblock)
	@$(call totarget,sign) $(call outfile,bootblock) $(bootblock)

$(call create_target,bootblock)
����bootblock������Ҫ����bootasm.o��bootmain.o��sign.
	>obj/boot/bootasm.o, obj/boot/bootmain.o
	����bootasm.o,bootmain.o�����makefile����Ϊ
	bootfiles = $(call listf_cc,boot) 
	$(foreach f,$(bootfiles),$(call cc_compile,$(f),$(CC),\
			$(CFLAGS) -Os -nostdinc))
		 ʵ�ʴ����ɺ���������
	����bootasm.o��Ҫbootasm.S
	ʵ������Ϊ
	gcc -Iboot/ -fno-builtin -Wall -ggdb -m32 -gstabs \
		-nostdinc  -fno-stack-protector -Ilibs/ -Os -nostdinc \
		-c boot/bootasm.S -o obj/boot/bootasm.o
	���йؼ��Ĳ���Ϊ
		-ggdb  ���ɿɹ�gdbʹ�õĵ�����Ϣ������������qemu+gdb������bootloader or ucore��
		-m32  ����������32λ�����Ĵ��롣�����õ�ģ��Ӳ����32bit��80386������ucoreҲҪ��32λ�������
		-gstabs����stabs��ʽ�ĵ�����Ϣ������Ҫucore��monitor������ʾ�����ڿ������Ķ��ĺ�������ջ��Ϣ
		-nostdinc  ��ʹ�ñ�׼�⡣��׼���Ǹ�Ӧ�ó����õģ������Ǳ���ucore�ںˣ�OS�ں����ṩ����ģ��������еķ���Ҫ�Ը����㡣
		-fno-stack-protector  ���������ڼ�⻺��������Ĵ��롣����for Ӧ�ó���ģ������Ǳ����ںˣ�ucore�ں˺����ò����˹��ܡ�
		-Os  Ϊ��С�����С�������Ż�������Ӳ��spec������������ֻ��512�ֽڣ�����д�ļ�bootloader�����մ�С���ܴ���510�ֽڡ�
		-I<dir>  �������ͷ�ļ���·��
	����bootmain.o��Ҫbootmain.c
	ʵ������Ϊ
	gcc -Iboot/ -fno-builtin -Wall -ggdb -m32 -gstabs -nostdinc \
	-fno-stack-protector -Ilibs/ -Os -nostdinc \
	-c boot/bootmain.c -o obj/boot/bootmain.o
	�³��ֵĹؼ�������
	-fno-builtin  ������__builtin_ǰ׺�����򲻽���builtin�������Ż�
	>����sign���ߵ�makefile����Ϊ
		$(call add_files_host,tools/sign.c,sign,sign)
		$(call create_target_host,sign,sign)
	ʵ������Ϊ:
		gcc -Itools/ -g -Wall -O2 -c tools/sign.c \
		 	-o obj/sign/tools/sign.o
		gcc -g -Wall -O2 obj/sign/tools/sign.o -o bin/sign
	 ��������bootblock.o
	ld -m    elf_i386 -nostdlib -N -e start -Ttext 0x7C00 \
		obj/boot/bootasm.o obj/boot/bootmain.o -o obj/bootblock.o
	 ���йؼ��Ĳ���Ϊ
		-m <emulation>  ģ��Ϊi386�ϵ�������
		-nostdlib  ��ʹ�ñ�׼��
		-N  ���ô���κ����ݶξ��ɶ�д
		-e <entry>  ָ�����
		-Ttext  �ƶ�����ο�ʼλ��
	���������ƴ���bootblock.o��bootblock.out
	objcopy -S -O binary obj/bootblock.o obj/bootblock.out
	    ���йؼ��Ĳ���Ϊ
	    -S  �Ƴ����з��ź��ض�λ��Ϣ
            -O <bfdname>  ָ�������ʽ
	ʹ��sign���ߴ���bootblock.out������bootblock
	bin/sign obj/bootblock.out bin/bootblock
�������bootblock��kernel֮��,������������ɲ���
����һ����10000������ļ���ÿ����Ĭ��512�ֽڣ���0���
dd if=/dev/zero of=bin/ucore.img count=10000
��bootblock�е�����д����һ����s
dd if=bin/bootblock of=bin/ucore.img conv=notrunc 
�ӵڶ����鿪ʼдkernel�е�����
dd if=bin/kernel of=bin/ucore.img seek=1 conv=notrunc

[��ϰ1.2] һ����ϵͳ��Ϊ�Ƿ��Ϲ淶��Ӳ��������������������ʲô?

������sign���ߵĴ���������
 if (st.st_size > 510) {
        fprintf(stderr, "%lld >> 510!!\n", (long long)st.st_size);
        return -1;
    }
    char buf[512];
һ����������������ֻ��512�ֽڡ���
 buf[510] = 0x55;
 buf[511] = 0xAA;
��510���ֽ���0x55��
��511���ֽ���0xAA��
[��ϰ2.1] �� CPU �ӵ��ִ�еĵ�һ��ָ�ʼ,�������� BIOS ��ִ�С�
���ȸ�дMakefile�ļ� 
debug: $(UCOREIMG)
		$(V)$(TERMINAL) -e "$(QEMU) -S -s -d in_asm -D $(BINDIR)/q.log -parallel stdio -hda $< -serial null"
		$(V)sleep 2
		$(V)$(TERMINAL) -e "gdb -q -tui -x tools/gdbinit"
�޸�lab1/tools/gdbmit
set	architecture	i8086
target	remote	:1234

��lab1Ŀ¼��ִ��make debug

ִ������step�ȵ�������ָ����е������ٻ���(next,nexti,stepi)���ܸ�����ͬ.

[��ϰ2.2] �ڳ�ʼ��λ��0x7c00 ����ʵ��ַ�ϵ�,���Զϵ�������
�޸�lab1/tools/gdbmitΪ
set architecture i8086
target	remote	:1234
b *0x7c00
continue
 x /2i $pc
�õ�:
Breakpoint 1, 0x00007c00 in ?? ()
	=> 0x7c00:      cli    
	   0x7c01:      cld    

[��ϰ2.3] �ڵ���qemu ʱ����-d in_asm -D q.log ����������Խ����еĻ��ָ�����q.log �С�
��ִ�еĻ�������bootasm.S �� bootblock.asm ���бȽϣ����������Ƿ�һ�¡�

>ִ�к�q.log������
----------------
IN: 
0xfffffff0:  ljmp   $0xf000,$0xe05b

----------------
IN: 
0x000fe05b:  cmpl   $0x0,%cs:0x65a4
0x000fe062:  jne    0xfd2b9

----------------
IN: 
0x000fe066:  xor    %ax,%ax
0x000fe068:  mov    %ax,%ss

----------------
IN: 
0x000fe06a:  mov    $0x7000,%esp

----------------
IN: 
0x000fe070:  mov    $0xf3c4f,%edx
0x000fe076:  jmp    0xfd12a

----------------
IN: 
0x000fd12a:  mov    %eax,%ecx
0x000fd12d:  cli    
0x000fd12e:  cld    
0x000fd12f:  mov    $0x8f,%eax
0x000fd135:  out    %al,$0x70
0x000fd137:  in     $0x71,%al
0x000fd139:  in     $0x92,%al
0x000fd13b:  or     $0x2,%al
0x000fd13d:  out    %al,$0x92
0x000fd13f:  lidtw  %cs:0x66c0
0x000fd145:  lgdtw  %cs:0x6680
0x000fd14b:  mov    %cr0,%eax
0x000fd14e:  or     $0x1,%eax
0x000fd152:  mov    %eax,%cr0

----------------
IN: 
0x000fd155:  ljmpl  $0x8,$0xfd15d

----------------
IN: 
0x000fd15d:  mov    $0x10,%eax
0x000fd162:  mov    %eax,%ds

----------------
IN: 
0x000fd164:  mov    %eax,%es

----------------
IN: 
0x000fd166:  mov    %eax,%ss

----------------
IN: 
0x000fd168:  mov    %eax,%fs

----------------
IN: 
0x000fd16a:  mov    %eax,%gs
0x000fd16c:  mov    %ecx,%eax
0x000fd16e:  jmp    *%edx

----------------

����bootasm.S��bootblock.asm�еĴ���һ�¡�

## [��ϰ3]
����bootloader ���뱣��ģʽ�Ĺ��̡�

��`%cs=0 $p

������������������flag��0�ͽ��μĴ�����0
```
	.code16
	    cli
	    cld
	    xorw %ax, %ax
	    movw %ax, %ds
	    movw %ax, %es
	    movw %ax, %ss
```

 ����A20 Gate��ȡ����A20��ַ�ߵĽ�ֹ,ʹ���Է���4G���ڴ�ռ䡣
```
seta20.1:
    inb $0x64, %al                                  # Wait for not busy(8042 input buffer empty).
    testb $0x2, %al
    jnz seta20.1

    movb $0xd1, %al                                 # 0xd1 -> port 0x64
    outb %al, $0x64                                 # 0xd1 means: write data to 8042's P2 port

seta20.2:
    inb $0x64, %al                                  # Wait for not busy(8042 input buffer empty).
    testb $0x2, %al
    jnz seta20.2
    movb $0xdf, %al                                 # 0xdf -> port 0x60
    outb %al, $0x60                                 # 0xdf = 11011111, means set P2's A20 bit(the 1 bit) to 1
```

��ʼ��GDT��һ���򵥵�GDT������������Ѿ���̬�������������У����뼴��
```
	    lgdt gdtdesc
```

���뱣��ģʽ��ͨ����cr0�Ĵ���PEλ��1�㿪���˱���ģʽ
```
	    movl %cr0, %eax
	    orl $CR0_PE_ON, %eax
	    movl %eax, %cr0
```

ͨ������ת����cs�Ļ���ַ
```
	 ljmp $PROT_MODE_CSEG, $protcseg
	.code32
	protcseg:
```

���öμĴ�������������ջ
```
	    movw $PROT_MODE_DSEG, %ax
	    movw %ax, %ds
	    movw %ax, %es
	    movw %ax, %fs
	    movw %ax, %gs
	    movw %ax, %ss
	    movl $0x0, %ebp
	    movl $start, %esp
```
ת������ģʽ��ɣ�����boot������
```
	    call bootmain
```
## [��ϰ4]
����bootloader����ELF��ʽ��OS�Ĺ��̡�

���ȿ�readsect������
�����õ���GCC�������

`readsect`���豸�ĵ�secno������ȡ���ݵ�dstλ��
```
	static void
	readsect(void *dst, uint32_t secno) {
	    waitdisk();   //waitdisk(void) {    while ((inb(0x1F7) & 0xC0) != 0x40)/* do nothing */

         // outb(uint16_t port, uint8_t data) {
         // asm volatile ("outb %0, %1" :: "a" (data), "d" (port));
         // }
	    outb(0x1F2, 1);                         // ���ö�ȡ��������ĿΪ1
	    outb(0x1F3, secno & 0xFF);
	    outb(0x1F4, (secno >> 8) & 0xFF);
	    outb(0x1F5, (secno >> 16) & 0xFF);
	    outb(0x1F6, ((secno >> 24) & 0xF) | 0xE0);// ��������ָ�������ƶ���������
	        
	    outb(0x1F7, 0x20);                      // 0x20�����ȡ����
	
	    waitdisk();

	    insl(0x1F0, dst, SECTSIZE / 4);         // ��ȡ��dstλ�ã�
	}
```
�������� ��readseg,����readsect��ʵ�ִ��豸��ȡ���ⳤ�ȵ����ݡ�
```
	readseg(uintptr_t va, uint32_t count, uint32_t offset) {
        uintptr_t end_va = va + count;

    		// round down to sector boundary
    	va -= offset % SECTSIZE;

    	// translate from bytes to sectors; kernel starts at sector 1
    		uint32_t secno = (offset / SECTSIZE) + 1;

        // If this is too slow, we could read lots of sectors at a time.
        // We'd write more to memory than asked, but it doesn't matter --
        // we load in increasing order.
       for (; va < end_va; va += SECTSIZE, secno ++) {
         readsect((void *)va, secno);
       }
}
```

��bootmain�����У�
```
	void
	bootmain(void) {
	    // ���ȶ�ȡELF�ļ���ͷ��
	    readseg((uintptr_t)ELFHDR, SECTSIZE * 8, 0);
	    
	    // �ж��Ƿ��ǺϷ���ELF�ļ�
	    if (ELFHDR->e_magic != ELF_MAGIC) {
	        goto bad;
	    }
	
	    struct proghdr *ph, *eph;
	
	    // ELFͷ��������ELF�ļ�Ӧ���ص��ڴ�ʲôλ�õ�������
	    // �Ƚ��������ͷ��ַ����ph
	    ph = (struct proghdr *)((uintptr_t)ELFHDR + ELFHDR->e_phoff);
	    eph = ph + ELFHDR->e_phnum;

	    // ����������ELF�ļ������������ڴ�
	    for (; ph < eph; ph ++) {
	        readseg(ph->p_va & 0xFFFFFF, ph->p_memsz, ph->p_offset);
	    }
	    ((void (*)(void))(ELFHDR->e_entry & 0xFFFFFF))();
	
	bad:
	    outw(0x8A00, 0x8A00);
	    outw(0x8A00, 0x8E00);
	    while (1);
	}
```
## [��ϰ5] 
ʵ�ֺ������ö�ջ���ٺ��� 

ss:ebpָ��Ķ�ջλ�ô�����caller��ebp���Դ�Ϊ�������Եõ�����ʹ�ö�ջ�ĺ���ebp��
ss:ebp+4ָ��caller����ʱ��eip��ss:ebp+8���ǣ����ܵģ�������
>���Ϊ:  
Special kernel symbols:
  entry  0x00100000 (phys)
  etext  0x001032e0 (phys)
  edata  0x0010ea16 (phys)
  end    0x0010fd20 (phys)
Kernel executable memory footprint: 64KB
ebp:0x00007b08 eip:0x001009a6 args:0x00010094  0x00000000  0x00007b38  0x00100092  
    kern/debug/kdebug.c:305: print_stackframe+21
ebp:0x00007b18 eip:0x00100cb2 args:0x00000000  0x00000000  0x00000000  0x00007b88  
    kern/debug/kmonitor.c:125: mon_backtrace+10
ebp:0x00007b38 eip:0x00100092 args:0x00000000  0x00007b60  0xffff0000  0x00007b64  
    kern/init/init.c:48: grade_backtrace2+33
ebp:0x00007b58 eip:0x001000bb args:0x00000000  0xffff0000  0x00007b84  0x00000029  
    kern/init/init.c:53: grade_backtrace1+38
ebp:0x00007b78 eip:0x001000d9 args:0x00000000  0x00100000  0xffff0000  0x0000001d  
    kern/init/init.c:58: grade_backtrace0+23
ebp:0x00007b98 eip:0x001000fe args:0x001032fc  0x001032e0  0x0000130a  0x00000000  
    kern/init/init.c:63: grade_backtrace+34
ebp:0x00007bc8 eip:0x00100055 args:0x00000000  0x00000000  0x00000000  0x00010094  
    kern/init/init.c:28: kern_init+84
ebp:0x00007bf8 eip:0x00007d68 args:0xc031fcfa  0xc08ed88e  0x64e4d08e  0xfa7502a8  
    <unknow>: -- 0x00007d67 --
++ setup timer interrupts

�����,��ջ�����һ��Ϊ
���Ӧ���ǵ�һ��ʹ�ö�ջ�ĺ�����bootmain.c�е�bootmain��
bootloader���õĶ�ջ��0x7c00��ʼ��ʹ��"call bootmain"ת��bootmain������callָ��ѹջ������bootmain��ebpΪ0x7bf8��  


## [��ϰ6]
�����жϳ�ʼ���ʹ���

[��ϰ6.1] �ж���������һ������ռ�����ֽڣ������ļ�λ�����жϴ���������ڣ�

�ж�������һ������ռ��8�ֽڣ�����2-3�ֽ��Ƕ�ѡ���ӣ�0-1�ֽں�6-7�ֽ�ƴ��λ�ƣ�
�������ϱ����жϴ���������ڵ�ַ��

[��ϰ6.2] ��������kern/trap/trap.c�ж��ж���������г�ʼ���ĺ���idt_init��

������

[��ϰ6.3] ��������trap.c�е��жϴ�����trap���ڶ�ʱ���жϽ��д���Ĳ�����дtrap����

������



## [��ϰ7]

����syscall���ܣ�������һ�û�̬��������ִ��һ�ض�ϵͳ���ã����ʱ�Ӽ���ֵ����
���ں˳�ʼ��Ϻ󣬿ɴ��ں�̬���ص��û�̬�ĺ��������û�̬�ĺ�����ͨ��ϵͳ���õõ��ں�̬�ķ���

��idt_init�У����û�̬����SWITCH_TOK�жϵ�Ȩ�޴򿪡�
	SETGATE(idt[T_SWITCH_TOK], 1, KERNEL_CS, __vectors[T_SWITCH_TOK], 3);

��trap_dispatch�У���iretʱ��Ӷ�ջ�����ĶμĴ��������޸�
	��TO User
```
	    tf->tf_cs = USER_CS;
	    tf->tf_ds = USER_DS;
	    tf->tf_es = USER_DS;
	    tf->tf_ss = USER_DS;
```
	��TO Kernel

```
	    tf->tf_cs = KERNEL_CS;
	    tf->tf_ds = KERNEL_DS;
	    tf->tf_es = KERNEL_DS;
```

��lab1_switch_to_user�У�����T_SWITCH_TOU�жϡ�
ע����жϷ���ʱ�����pop��λ����������λ��ֵ����ss,sp���𻵶�ջ��
����Ҫ�Ȱ�ջѹ��λ�����ڴ��жϷ��غ��޸�esp��
```
	asm volatile (
	    "sub $0x8, %%esp \n"
	    "int %0 \n"
	    "movl %%ebp, %%esp"
	    : 
	    : "i"(T_SWITCH_TOU)
	);
```

��lab1_switch_to_kernel�У�����T_SWITCH_TOK�жϡ�
ע����жϷ���ʱ��esp����TSSָʾ�Ķ�ջ�С�����Ҫ�ڴ��жϷ��غ��޸�esp��
```
	asm volatile (
	    "int %0 \n"
	    "movl %%ebp, %%esp \n"
	    : 
	    : "i"(T_SWITCH_TOK)
	);
```

������������������ı���������ʾ����trap_dispatch��תUser̬ʱ��������io����Ȩ�޽��͡�
```
	tf->tf_eflags |= 0x3000;
```
