# 介绍 #

在程序开发中，如果说C/C++、汇编是程序的血肉的话，那么数据结构与算法可以说是程序的骨骼，正是不同的模块化的数据及其维护函数充当了整个程序内部整体的工作框架与协调基准。

在现代程序开发中，因为基本上每一个应用都非常复杂，我们不可能完全靠简单的语法拼凑完成各种功能，而需要首先需要面向对象用模型进行描述，也就是各种数据结构，然后根据模型抽象出对这些数据的宏观外部操作方法，让其他程序可以调用以完成所需要的任务，这就是算法。

我们可以想象，大型程序好比都是由许多个很大的箱子堆砌起来的，每一个箱子完成独立的功能，而箱子内部有着自己不为人知的运作机制，但是它对外总是能够魔术般的展示出它神秘的功能。

在下面笔者给出一个简单的面向类似磁盘文件系统管理的数据结构及其算法，虽然是非常原始的代码，但是实际上这是目前文件系统领域最前沿的B+树管理模式，下面我们贴出代码并进行介绍。

# 模型描述与操作方法 #

根据我们日常对于文件系统的了解，每一个文件自身如果不论是文件还是目录的话，拥有如下的结构：
```
      父结点
        |
兄弟-->自己-->兄弟
        |
      子节点
```

我们将以上模型实现为下面的C语言数据结构：
```
struct inode
{
        // 父结点指针、子节点指针、下一个兄弟指针
	struct inode *parent, *child, *next;

        // 节点名字（文件名）
	char name[20];
};
```

对于这样的数据模型，我们还需要设计一整套功能齐全的函数集，以实现对文件系统的各种操作：
```
// 创建指定路径的节点
struct inode *mknode(char *path);

// 打印目前的节点树结构
void printree(struct inode *ip);

// 获得相对ip节点path指定路径的节点指针
struct inode *iget(char *path, struct inode *ip);
```

# 代码实现 #

下面是我们具体的代码实现：

```
/****************************************************************************
 * 简易内核用文件系统树（B+树）数据结构
 *
 * 本代码基于GPL协议发布
 ****************************************************************************/
#include <stdio.h>

struct inode
{
	struct inode *parent, *child, *next;
	char name[20];
};

struct inode root = {0, 0, 0, 0};

int slen(char *str)
{
	int len = 0;
	while((*str != '/') && (*str != '\0')) {
		len++;
		str++;
	}

	if(*str == '/') {
		return len+1;
	} else {
		return 0;	// possibly not the end of path
	}

	return 0;
}

struct inode *iget(char *path, struct inode *ip)
{
	int len = 0;

	// process the '/' root identifier
	if(*path == '/') {
		ip = &root;	// point to the root node
		path++;		// get over '/'
	}

	// get the current dir name length (including '/')
	len = slen(path);	// get the length of current dir
	printf("%d:%s\n", len, path);

	if(len == 0) {
		return ip;
	}

	// do the search
	for(ip = ip->child; ip != 0; ip=ip->next) {
		if(!strncmp(path, ip->name, len-1)) {
			break;
		}
	}

	// check if we have a valid ip value
	if(ip == 0) {
		printf("error: node not exist\n");
		return (struct inode*) -1;
	}

	// do recursion
	if(len == 0) {
		return ip;
	} else {
		return iget(path + len, ip);
	}

	return 0;
}

struct inode *mknode(char *path)
{
	printf("\nthe address is:  %p\n",path);
	struct inode *ip;

	// get the parent node to create
	if((ip = iget(path, &root)) == (struct inode *)-1) {
		printf("error: target directory not exist, cannot create\n");
		return (struct inode *) -1;
	}
	printf("\nthe later address is :  %p\n",path);

	// get the target's name to create (bottom-up search)
	printf("The string is :  %s\n\n",path);
	int i, len = strlen(path);
	for(i=len; (i!=0) && (path[i-1] != '/'); i--);

	// allocate new node
	struct inode *nip;
	nip = malloc(sizeof(struct inode));

	// set linkage
	nip->parent = ip;
	nip->next   = ip->child;
	nip->child  = 0;
	 ip->child  = nip;

	// set name (and other values if exist)
	strcpy(nip->name, &path[i]);

	return 0;
}

int sem = 0;

void printree(struct inode *ip)
{
	int semno;

	semno = sem;
	while(semno--) printf("\t");
	printf("%s\n", ip->name);

	sem++;

	for(ip = ip->child; ip!=0; ip=ip->next) {
		printree(ip);
	}

	sem--;
}

int main()
{
	strcpy(root.name, "root");
	mknode("/home");
	mknode("/home/martin");
	mknode("/home/luojun");
	mknode("/home/luojun/newnode");
	mknode("/home/luojun/newnode2");
	mknode("/home/luojun/newnode3");
	mknode("/home/joysom");
	mknode("/abc");
	mknode("/abc/hello");
	mknode("/abc/world");

	printf("\n---------tree structure---------\n");
	printree(&root);
	return 0;
}
```

这里只是一个对数据结构与算法概念初步的印象，如果需要深入学习数据结构与算法，推荐大家去阅读各种教材。