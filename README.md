# File-System-Implementation-using-C
Implement a File System Emulator


#include<stdio.h>
#include<stdlib.h>
#include<string.h>
#define MAX 100
char *ptr;//to store the starting address of 256 bytes
int count=16;//to keep track of used and unused chunks
struct chunk_addresses *curr_mem_ptr=NULL;
struct fileTable *files[MAX];

struct chunk_addresses
{
	char *addr;
	struct chunk_addresses *link;
};
struct memory
{
	char *address;
	struct memory *nextMemory;
};
struct version
{
	int timestamp;
	int size;
	int del;
	struct version *nextVersion;
	struct memory *memory_start;
};
struct fileTable
{
	char *fileName;
	struct fileTable *nextFileTable;
	struct version *version_start;
};

void pushMemory(char *x)
{
		struct chunk_addresses *y=curr_mem_ptr,*neww;
		neww=(struct chunk_addresses*)malloc(sizeof(struct chunk_addresses));
		if(y == NULL)
		{
			neww->addr=x;
			neww->link=NULL;
			curr_mem_ptr=neww;
			return ;
		}
		while(y->link != NULL)
		{
			y=y->link;
		}

		neww->addr=x;
		neww->link=NULL;
		y->link=neww;
		return ;

}
int initilizeVirtualDisk(int tot_mem,int chunk_size)
{
	ptr=(char*)malloc(tot_mem);
	if(ptr==NULL)
		return 0;
	else
		return 1;
}

int computeFileIndex(char *str)
{
	int i=0,ans=0;
	char ch;
	while( (ch=str[i]) != '\0')
	{
		switch(ch)
		{
		case 'a' :
			ans = ans*10 + 1;
			break;

		case 'b' :
			ans = ans*10 + 2;
			break;

		case 'f' :
			ans = ans*10 + 3;
			break;

		case '1' :
			ans = ans*10 + 4;
			break;

		case '2' :
			ans=ans*10 + 5;
			break;

		case '3' :
			ans =ans*10 +6;
			break;

		default:

			ans=ans;
		}
		i++;
	}
	return ans;
}
struct version *versionFunc(const char *fileContents,int backUpJobStartTime,struct version *pp)
{

	int len=strlen(fileContents);
	char str[100];
	char arr[17];
	int i=0,j=0,check=1,k;
	strcpy(str,fileContents);
	//printf("%s",str);
	struct version *ver1= (struct version*)malloc(sizeof(struct version));

	struct memory *first,*move,*neww;

	ver1->timestamp=backUpJobStartTime;
	ver1->size=0;
	ver1->del=0;
	ver1->nextVersion=NULL;

	while(len>0)
	{
		while(i<16 && str[j]!='\0')
		{
			arr[i]=str[j];
			i++;j++;
		}
		len=len-16;
		if(str[j] == '\0')
		{
			arr[i]='\0';
		}
		arr[16]='\0';
		//printf("%s",arr);
		strcpy(curr_mem_ptr->addr , arr);

		i=0;
		neww=(struct memory*)malloc(sizeof(struct memory));
		neww->address = curr_mem_ptr->addr;
		if(check==1)
		{
			check=0;
			ver1->memory_start=neww;///////////////////////////////////

			move=first=neww;
		}
		else
		{
			move->nextMemory=neww;
			move=move->nextMemory;
		}
		curr_mem_ptr=curr_mem_ptr->link;
	}
	//printf("yess\n");
	return ver1;


}
struct fileTable* fileTableFunc(char *fileName,const char *fileContents,int backUpJobStartTime,int file_index)
{
	struct fileTable *val;

	if(files[file_index] == NULL)
	{
		val = (struct fileTable*) malloc(sizeof(struct fileTable));
		val->fileName = fileName;
		val->nextFileTable=NULL;
		val->version_start=NULL;
		val->version_start= versionFunc(fileContents,backUpJobStartTime,val->version_start);
		return val;
	}



}
void backupFunc(char *fileName,const char *fileContents,int backUpJobStartTime)
{
	int file_index;
	int len=strlen(fileContents);

	if(len % 16 == 0)
	{
		if((len/16) > count)
		{
			printf("No sufficient memory to allocate\n");
		}
	}
	else
	{
		if((len/16)+1 > count)
		{
			printf("No sufficient memory to allocate\n");
			exit(0);
		}
	}
	file_index = computeFileIndex(fileName);
	//printf("%d\n",file_index);

	files[file_index] = fileTableFunc(fileName,fileContents,backUpJobStartTime,file_index);



}

void displayFunc(int index,char *fname,int time)
{
	struct fileTable *xx;
	struct version *x;
	struct memory *mem;
	xx=files[index];

	while(xx->fileName != fname)
	{
		xx=xx->nextFileTable;
	}

	x=xx->version_start;

	while(x->timestamp!=time)
		x=x->nextVersion;
	mem=x->memory_start;


		printf("%s",mem->address);

	return;


}
void restore(char *fname,int time)
{
	int file_index;
	file_index = computeFileIndex(fname);

	displayFunc(file_index,fname,time);

}
int main(int argc, char* argv[])
{
	char *address;
	char *fileName;
	const char *fileContents;
	int backUpJobStartTime,i;

/*                 0000000001111111111222222222233333333334         5         6         7         8         9         a         b         c         d         e */
/*                 12345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890123456789012345678901234567890 */
const char* F1_0 ="This is F1 v0 -backed up at t0"; //contents of F1 at t0 - version 0 of the file
const char* F1_1 ="This is F1 v1 -backed up at t5"; //contents of F1 at t5 - version 1 of the file
const char* F2_0 ="This is F2 v0 -backed up at t2";
const char* F2_1 ="This is F2 v1 -backed up at t5";
const char* F3_0 ="This is F3 v0 -backed up at t5; this file tells the story of how a simple backup and restore app can be used as an effective design exercise";

if(initilizeVirtualDisk(256, 16) == 0)
{
	printf("Memory not allocated\n");
	exit(0);
}

for(i=0;i<16;i++)
{
	address=ptr + i*16;
	//printf("%ld\t",address);
	pushMemory(address);
}
//Phase 1 - Simple backup, Restore and printing of data structures

//Backup  F1 v0

fileName="f1";
fileContents=F1_0;
backUpJobStartTime=0;

backupFunc(fileName,fileContents,backUpJobStartTime);

restore(fileName,0);

//printf("success\n");
//getch();

return 0;
}
