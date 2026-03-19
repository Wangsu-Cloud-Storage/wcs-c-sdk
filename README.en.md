# wcs-c-sdk
C SDK for wcs

## Language
- [简体中文](README.md)
- [English](README.en.md)

### Overview:

This SDK is implemented in C language that conforms to the C89 standard.
- The SDK mainly includes the following aspects:
    1. Common parts: src/wcs/base.c, src/wcs/conf.c, src/wcs/http.c, src/base/log.c
    2. Client atomic upload for small files: src/wcs/base_io.c io.c
    3. Client shard upload with breakpoint resumption: src/base/threadpoo.c, src/base/inifile.c, src/base/patchfileops.c, src/wcs/multipart_io.c
    4. Data processing (video transcoding, screenshot, etc.): src/wcs/fop.c

### Installation:
```
C-SDK uses cURL and OpenSSL, you need to install the aforementioned dependency libraries by yourself.
GCC compilation options: -lcurl -lssl -lcrypto -lm
If connection errors occur during project compilation, you need to check whether the dependency libraries and compilation options are correct.

Compilation steps:
cd build
rm -rf ./*
cmake ../
make
```
*Note: This version is currently only adapted to Linux environment*

### WCS storage related information:
Calling the interface requires ACCESS_KEY, SECRET_KEY, upload domain name and management domain name, which can be obtained through Wangsu official.

### LOG module:
    When problems need to be analyzed, you can turn on Log, which is divided into 6 levels. You can choose whether to write to a file (for performance, it is not recommended to configure writing to a file normally).
    Configuration file (LogConfig.ini): [SDKLogConfig]
                LOG_LEVEL=5 //0 prints the most, it is recommended to set to 5 normally
                WRITE_FILE=0 //0 does not write to file, 1 writes to file
                LOG_FILE=./SDKTest.log //The name of the file to write log
    Note: If there is no LogConfig.ini, the default is LOG_LEVEL=5 and no file is written
  void wcs_Log_Init(char *logConfigFile, FILE *file); // logConfigFile is the path and name of the above configuration file, file only needs to be defined, do not open the file, mainly used for file closing
  void wcs_close_Logfile(FILE *file);

### Interface call
    Initialization and deinitialization:
wcs_Global_Init (0);
wcs_MacAuth_Init ();
wcs_Client_InitNoAuth (&client, 8192);
。。。。。
wcs_Client_Cleanup (&client);
wcs_Global_Cleanup ();
The interface is called in the ellipsis part. This initialization should be called once in the main thread. It is best not to call it multiple times in multiple threads, otherwise unpredictable errors will occur. This problem is caused by some interfaces in cURL not being thread-safe.

### Detailed examples refer to: demo/test.c



### API overview (Note: the ret parameters passed in each interface need to be initialized):

Interface function | Interface function name | Remarks
---|---|---
Generate upload credential | char *wcs_RS_PutPolicy_Token (wcs_RS_PutPolicy * auth, wcs_Mac * mac) |
Ordinary file upload | wcs_Error wcs_Io_PutFile (wcs_Client * self, wcs_Io_PutRet * ret, const char *uptoken, const char *key, const char *localFile, wcs_Io_PutExtra * extra)| 
Shard upload | wcs_Error wcs_Multipart_PutFile (wcs_Client * self, wcs_Multipart_PutRet * ret, const char *uptoken, const char *key, const char *localFile, wcs_Multipart_PutExtra * extra) | Note: This interface uploads shards through multi-threading, you can change the maximum number of threads: wcs_Multipart_MaxThreadNum|
Breakpoint resumption | wcs_Error wcs_Multipart_UploadCheck(const char *configFile, wcs_Client * self, wcs_Multipart_PutRet * ret)| 
Delete file | wcs_Error wcs_RS_Delete (wcs_Client * self, const char *tableName, const char *key, const char *mgrHost)| 
Get file information | wcs_Error wcs_RS_Stat (wcs_Client * self, wcs_RS_StatRet * ret, const char *tableName, const char *key, const char *mgrHost)| 
List resources | wcs_Error wcs_RS_List (wcs_Client * self, wcs_RS_ListRet * ret, const char *bucketName, wcs_Common_Param * param, const char *mgrHost)| 
Move resources | wcs_Error wcs_RS_Move (wcs_Client * self, const char *tableNameSrc, const char *keySrc, const char *tableNameDest, const char *keyDest, const char *mgrHost)| 
Update mirror resources | wcs_Error wcs_RS_UpdateMirror(wcs_Client * self, wcs_RS_StatRet * ret, const char *bucketName, const char **fileNameList, unsigned int fileNum, const char *mgrHost)| 
Audio and video operations | wcs_Error wcs_Fops_Media (wcs_Client * self, wcs_FOPS_Response * ret, wcs_FOPS_MediaParam *param, const char *mgrHost)| fops assembly refers to wcsAPI documentation
Fetch resources | wcs_Error wcs_Fops_Fetch(wcs_Client * self, wcs_FOPS_Response * ret, wcs_FOPS_FetchParam *ops[], unsigned int opsNum, const char *mgrHost )| fops splicing refers to wcsAPI documentation
Copy resources | wcs_Error wcs_RS_Copy (wcs_Client * self, const char *tableNameSrc, const char *keySrc, const char *tableNameDest, const char *keyDest, const char *mgrHost)| 
Base64 encoding | wcs_Error wcs_Encode_Base64(int argc, char **argv)| 

Fetch resources: https://wcs.chinanetcenter.com/document/API/Fmgr/fetch
Audio and video: https://wcs.chinanetcenter.com/document/API/Video-op

### Examples:
1. Audio and video fops
YXZ0aHVtYi9mbHYvdmIvNjRrfHNhdmVzL2FHRnNaWGwwWlhOME9uUmxjM1JXYVdSbGJ3PT07YXZ0aHVtYi9tNGEvdmIvMTI4S3xzYXZlcy9hR0ZzWlhsMFpYTjBPblJsYzNSQmRXUnBidz09

2. Fetch resources fops
fetchURL/aHR0cDovL2ltYWdlcy53Lndjc2FwaS5iaXoubWF0b2Nsb3VkLmNvbS8xLnBuZw==/bucket/aGFsZXl0ZXN0/key/dGVzdE1lZGlh/prefix/bWVkaWE=/md5/aa36f4e18a13762d77cce9e749976b3c/decompression/zip
Multiple separated by ;