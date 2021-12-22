---
title: VC 中的字符集
date: 2021-10-11
Tags:
  - 字符集
categories:
  - C++
cover: https://pic3.zhimg.com/v2-bcf5d58223b3a55690b9439330ad3c2a_r.jpg
---



### Anis和Unicode

#### 区别：

- ANSI是窄字节，Ansi用char格式表示一个字符，占用一个字节的存储空间，最多表示255个字符，表示英文还可以，但对于中文、日文、韩文等语言来说就不够用了

- Unicode=宽字节，用二个字节去表示一个字符，格式是 unsigned short，被定义成 wchar_t 格式。这样就可以表示世界上绝大多数的语言了！      

#### 开发平台的支持性

- vc++ 6.0 默认为Ansi编码
- vs2005,vs2008,vs2010 都是Unicode编
- Windows 2000 及其以后的 Xp、2003、Vista、Win7 等系统都是使用Unicode从头进行开发的。

- ### vs配置选项“**使用多字节字符集**”和“**使用Unicode字符集**”的区别

  ![VS字符集的选择](https://s3.bmp.ovh/imgs/2021/10/8c44739c17991ac8.png)

- 什么是多字节：

  多字节是指使用多个字节(1-3)表示一个字符。(eg. gbd, utf-8, big5)

  什么是宽字节：

  宽字节一般是固定使用2个字节表示一个字符。（eg. unicode）,

- 使用多字节字符集：不定义`_UNICODE`宏

- 使用Unicode ，定义 `_UNICODE`宏



#### 例子：函数的调用选择

- `MessageBox` 只是一个宏，根据系统是否定义了`_UNICODE`来进行函数的选择
- 如果你的工程没有定义_UNICODE 宏,那么就使用窄字节的` MessageBoxA`，如果定义了，那么就使用宽字节的 `MessageBoxW`.

### 常见的数据类型

- Ansi

  >char  == CHAR,char*
  >
  >//P 代表指针，STR代表字符串，L代表长指针，win32已经废除
  >
  >(PCHAR,PSTR,LPSTR) == char * 
  >
  >
  >
  >LPCSTR = const char *  //C 代表const                      

- Unicode

  >wchar_t == WCHAR
  >
  >wchar_t * ==(PWCHAR,PWSTR,LPWSTR)
  >
  >const wchar_t * == LPCWSTR      

- 通用类型:根据`_UNICODE`宏自动判断使用的类型

  > char  == TCHAR
  >
  > char* == PTCHAR,PTSTR,LPTSTR
  >
  > const char* == LPCTSTR

### 常用数据类型的使用

#### 字符串类型的对象定义

- Ansi: char *pAnsiStr = "hello";
- Unicode: wchar_t * pUnicodeStr = L"hello";
- 通用: TCHAR *pTStr = _T("hello");TCHAR *pTStr2 = _TEXT("hello");
- dynamic memory:TCHAR * pszBuf = new TCHAR[100];

```c++
其中，_TEXT 和 _T 是一样的，定义如下：
#define _T(x)       __T(x)
#define _TEXT(x)    __T(x)

来看看 __T 的最终定义：
#ifdef  _UNICODE
#define __T(x)      L##x  
#else
#define __T(x)      x
#endif
    
##为连接起来的意思
```

#### 字符串的操作函数

字符串长度：
● Ansi：strlen(char *str);
● Unicode：wcslen(wchar_t *str);
● 通用函数：_tcslen(TCHAR *str);

● Ansi：int atoi(const char *str);
● Unicode：int _wtoi(const wchar_t *str);
● 通用函数：_tstoi(const TCHAR *str);

字符串拷贝：
● Ansi：strcpy(char *strDestination, const char *strSource);
● Unicode：wcscpy(wchar_t *strDestination, const wchar_t *strSource);
● 通用函数：_tcscpy(TCHAR *strDestination, const TCHAR *strSource);

以上函数不安全，在vs2003等以上版本的编译器中会有warnning警告提示，以下为安全函数（vc++6.0不支持）：
● Ansi：strcpy_s(char *strDestination, size_t numberOfElements, const char *strSource);
● Unicode：wcscpy_s(wchar_t *strDestination, size_t numberOfElements, const wchar_t *strSource);
● 通用函数：_tcscpy_s(TCHAR *strDestination, size_t numberOfElements, const TCHAR *strSource);

- `numberOfElements` Size of the destination string buffer. 目的缓冲区的大小，以字节为单位，不是字符！

- size_t 是 unsigned integer 即无符号整数。因为不同平台的操作系统(32/64)中 int/long 等类型所占的字节并不一样，而 size_t 在不同的平台下有不同的定义。有点类似于TCHAR类型：
  
  
  
  #ifndef   _SIZE_T_DEFINED
    #ifdef     _WIN64
    typedef   unsigned   __int64         size_t;   //8个字节，64位
    #else
    typedef   _W64   unsigned   int       size_t;   //4个字节，32位
    #endif
    #define   _SIZE_T_DEFINED
  #endif

####  转换

```c++
/*
● 转换用到的最根本的 API 函数：
WideCharToMultiByte 实现宽字节转换到窄字节
MultiByteToWideChar 实现窄字节转换到宽字节
*/

_Ret_opt_z_cap_(nChars) inline LPWSTR WINAPI AtlA2WHelper(_Out_z_cap_(nChars) LPWSTR lpw, _In_z_ LPCSTR lpa, _In_ int nChars, _In_ UINT acp) throw()
{
	ATLASSERT(lpa != NULL);
	ATLASSERT(lpw != NULL);
	if (lpw == NULL || lpa == NULL)
		return NULL;
	// verify that no illegal character present
	// since lpw was allocated based on the size of lpa
	// don't worry about the number of chars
	*lpw = '\0';
	int ret = MultiByteToWideChar(acp, 0, lpa, -1, lpw, nChars); 
	//用 CP_ACP 代码页就实现了 ANSI 与 Unicode 之间的转换；
	//dwFlags 标志为0
	//lpa要转换的窄字节
	//lpw 输出的宽字节
	//nChars 输出宽字节的个数


	if(ret == 0)
	{
		ATLASSERT(FALSE);
		return NULL;
	}		
	return lpw;
}


//-----------------------------------------
_Ret_opt_z_cap_(nChars) inline LPSTR WINAPI AtlW2AHelper(_Out_z_cap_(nChars) LPSTR lpa, _In_z_ LPCWSTR lpw, _In_ int nChars, _In_ UINT acp) throw()
{
	ATLASSERT(lpw != NULL);
	ATLASSERT(lpa != NULL);
	if (lpa == NULL || lpw == NULL)
		return NULL;
	// verify that no illegal character present
	// since lpa was allocated based on the size of lpw
	// don't worry about the number of chars
	*lpa = '\0';
	int ret = WideCharToMultiByte(acp, 0, lpw, -1, lpa, nChars, NULL, NULL);
	if(ret == 0)
	{
		ATLASSERT(FALSE);
		return NULL;
	}
	return lpa;
}

//-----------------------------------
#define A2W(lpa) (\
	((_lpa = lpa) == NULL) ? NULL : (\
		_convert = (lstrlenA(_lpa)+1),\
		(INT_MAX/2<_convert)? NULL :  \
		ATLA2WHELPER((LPWSTR) alloca(_convert*sizeof(WCHAR)), _lpa, _convert, _acp)))

//---------------------------------
#define W2A(lpw) (\
	((_lpw = lpw) == NULL) ? NULL : (\
		(_convert = (lstrlenW(_lpw)+1), \
		(_convert>INT_MAX/2) ? NULL : \
		ATLW2AHELPER((LPSTR) alloca(_convert*sizeof(WCHAR)), _lpw, _convert*sizeof(WCHAR), _acp))))
```



```c++
// A2W、W2A、T2A、T2W 宏的使用：
//头文件:
 #include<atlconv.h>

//加宏：定义用于编码转换如W2A,A2W使用的一些局步变量
USES_CONVERSION
PSTR str = "hello";
PWSTR strW  = A2W(str);

//你可以使用CString对象任意替换const char*和LPCTSTR函数参数。 所以CString转向其他类型的数据时候，就需要明确它等价的类型。

//注意：在宏的调用里面，用alloca 在栈上分配内存，所以不能在多次循环里面调用，会导致栈内存的溢出


```

- 用最基本的函数进行转换

  ```c++
  //宽字节转换到窄字节
  
  int
  WideCharToMultiByte(
      __in UINT     CodePage,
      __in DWORD    dwFlags,
      __in_ecount(cchWideChar) LPCWSTR  lpWideCharStr,
      __in int      cchWideChar,
      __out_bcount_opt(cbMultiByte) __transfer(lpWideCharStr) LPSTR   lpMultiByteStr,
      __in int      cbMultiByte,
      __in_opt LPCSTR   lpDefaultChar,
      __out_opt LPBOOL  lpUsedDefaultChar);
  
  
  
  char *WideCharToAnsi(wchar_t *pWideChar)
  {
  	if (!pWideChar) return NULL;
  	char *pszBuf = NULL;
  	int needBytes = WideCharToMultiByte(CP_ACP, 0, pWideChar, -1, NULL, 0, NULL, NULL);
  	if (needBytes > 0){
  		pszBuf = new char[needBytes+1];
  		ZeroMemory(pszBuf, (needBytes+1)*sizeof(char));
  		WideCharToMultiByte(CP_ACP, 0, pWideChar, -1, pszBuf, needBytes, NULL, NULL);
  	}
  
  	return pszBuf;
  }
  
  //窄字节转换到宽字节
  wchar_t *AnsiCharToWide(char *pChar)
  {
  	if (!pChar) return NULL;
  	wchar_t *pszBuf = NULL;
  	int needWChar = MultiByteToWideChar(CP_ACP, 0, pChar, -1, NULL, 0);
  	if (needWChar > 0){
  		pszBuf = new wchar_t[needWChar+1];
  		ZeroMemory(pszBuf, (needWChar+1)*sizeof(wchar_t));
  		MultiByteToWideChar(CP_ACP, 0, pChar, -1, pszBuf, needWChar);
  	}
  
  	return pszBuf;
  }
  //调用过程
  char* pResult=_T("你好");
  wchar_t * pBuff=AnsiCharToWide(pResult);
  
  wchar_t* pResult=_T("你好");
  wchar_t * pBuff=WideCharToAnsi(pResult);
  
  ```

  









































