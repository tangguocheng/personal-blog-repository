---
title: RTFSC uboot 之 宏 
date: 2017-04-06 09:28:07
tags:  RTFSC uboot
toc: true
categories: uboot
thumbnail:
---

# 无副作用的 min 和 max 宏

老的写法：`#define min_old(a,b) ((a < b) ? a :b)`

linux 中使用GUN GCC的拓展语法 `typeof` 消除了参数为如 a++ 时的副作用。

```c
#define min_new(X, Y)				\
	({ typeof (X) __x = (X), __y = (Y);	\
		(__x < __y) ? __x : __y; })

#define max_new(X, Y)				\
	({ typeof (X) __x = (X), __y = (Y);	\
		(__x > __y) ? __x : __y; })
```

# uboot cmd命令

```c
#define Struct_Section  __attribute__ ((unused,section (".u_boot_cmd")))

#ifdef  CFG_LONGHELP
	#define U_BOOT_CMD(name,maxargs,rep,cmd,usage,help) \
	cmd_tbl_t __u_boot_cmd_##name Struct_Section = {#name, maxargs, rep, cmd, usage, help}
#else	/* no long help info */
	#define U_BOOT_CMD(name,maxargs,rep,cmd,usage,help) \
	cmd_tbl_t __u_boot_cmd_##name Struct_Section = {#name, maxargs, rep, cmd, usage}
#endif	/* CFG_LONGHELP */
```

u_boot_cmd 段在uboot的链接脚本中定义：

```s
	. = .;
	__u_boot_cmd_start = .;
	.u_boot_cmd : { *(.u_boot_cmd) }
	__u_boot_cmd_end = .;
```

通过`U_BOOT_CMD`宏，将所有的命令导入到u_boot_cmd段中，在输入命令时，uboot在由`__u_boot_cmd_start` 
与 `__u_boot_cmd_end` 确定的范围内查找命令并执行。

所有的命令原型为：`int (*cmd)(struct cmd_tbl_s *, int, int, char *[])`,定义在结构体`cmd_tbl_s`中：

```c
struct cmd_tbl_s {
	char		*name;		/* Command Name			*/
	int		maxargs;	/* maximum number of arguments	*/
	int		repeatable;	/* autorepeat allowed?		*/
					/* Implementation function	*/
	int		(*cmd)(struct cmd_tbl_s *, int, int, char *[]);
	char		*usage;		/* Usage message	(short)	*/
#ifdef	CFG_LONGHELP
	char		*help;		/* Help  message	(long)	*/
#endif
#ifdef CONFIG_AUTO_COMPLETE
	/* do auto completion on the arguments */
	int		(*complete)(int argc, char *argv[], char last_char, int maxv, char *cmdv[]);
#endif
};
```