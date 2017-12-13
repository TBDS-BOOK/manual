
## 介绍
　　普通ftp协议使用本地磁盘作为数据文件存储，这样ftp存储的文件依赖于服务器本身的磁盘。当服务器磁盘损坏等原因会导致文件丢失。所以套件内部的ftserver改造了服务的后台文件存储，由原来的本地磁盘改为hdfs。这样可以通过部署多ftpserver连接同一套hdfs环境。通过套件内部的服务发现机制保证获取到可用的的服务ip、port。
　　
#### 1.1 客户端连接ftp，进行文件操作
##### 1.1.1 javaclient 连接普通ftpserver
```
import java.io.BufferedInputStream;
import java.io.BufferedOutputStream;
import java.io.File;
import java.io.FileInputStream;
import java.io.FileOutputStream;
import java.io.IOException;
import java.io.InputStream;
import java.io.OutputStream;
import java.io.RandomAccessFile;

import org.apache.commons.net.ftp.FTP;
import org.apache.commons.net.ftp.FTPClient;
import org.apache.commons.net.ftp.FTPFile;
import org.apache.commons.net.ftp.FTPReply;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;

/**
* 支持断点续传的FTP实用类
*
* @version 0.1 实现基本断点上传下载
* @version 0.2 实现上传下载进度汇报
* @version 0.3 实现中文目录创建及中文文件创建，添加对于中文的支持
*/
public class FTPUtils {
private static Logger LOG = LoggerFactory.getLogger(FTPUtils.class);
public FTPClient ftpClient = new FTPClient();

public FTPUtils() {
// 设置将过程中使用到的命令输出到控制台
//this.ftpClient.addProtocolCommandListener(new PrintCommandListener(new PrintWriter(System.out)));
}

/**
* 连接到FTP服务器
*
* @param hostname
* 主机名
* @param port
* 端口
* @param username
* 用户名
* @param password
* 密码
* @return 是否连接成功
* @throws IOException
*/
public boolean connect(String hostname, int port, String username,
String password) throws IOException {
ftpClient.connect(hostname, port);
ftpClient.setControlEncoding("UTF-8");
if (FTPReply.isPositiveCompletion(ftpClient.getReplyCode())) {
if (ftpClient.login(username, password)) {
return true;
}
}
disconnect();
return false;
}

/**
* 从FTP服务器上下载文件,支持断点续传，上传百分比汇报
*
* @param remote
* 远程文件路径
* @param local
* 本地文件路径
* @return 上传的状态
* @throws IOException
*/
public DownloadStatus download(String remote, String local)
throws IOException {
// 设置被动模式
ftpClient.enterLocalPassiveMode();
// 设置以二进制方式传输
ftpClient.setFileType(FTP.BINARY_FILE_TYPE);
DownloadStatus result;

// 检查远程文件是否存在
FTPFile[] files = ftpClient.listFiles(new String(
remote.getBytes("UTF-8"), "iso-8859-1"));
if (files.length != 1) {
System.out.println("远程文件不存在");
return DownloadStatus.Remote_File_Noexist;
}

long lRemoteSize = files[0].getSize();
File f = new File(local);

if (!f.getParentFile().exists()) {
f.getParentFile().mkdirs();
}

// 本地存在文件，进行断点下载
if (f.exists()) {
long localSize = f.length();
// 判断本地文件大小是否大于远程文件大小
if (localSize >= lRemoteSize) {
System.out.println("本地文件大于远程文件，下载中止");
return DownloadStatus.Local_Bigger_Remote;
}

// 进行断点续传，并记录状态
FileOutputStream out = new FileOutputStream(f, true);
ftpClient.setRestartOffset(localSize);
InputStream in = ftpClient.retrieveFileStream(new String(remote
.getBytes("UTF-8"), "iso-8859-1"));
byte[] bytes = new byte[1024];
long step = lRemoteSize / 100;

if(step == 0){
step = 1;
}
long process = localSize / step;
int c;
while ((c = in.read(bytes)) != -1) {
out.write(bytes, 0, c);
localSize += c;
long nowProcess = localSize / step;
if (nowProcess > process) {
process = nowProcess;
//if (process % 10 == 0)
// System.out.println("下载进度：" + process);
// TODO 更新文件下载进度,值存放在process变量中
}
}
in.close();
out.close();
boolean isDo = ftpClient.completePendingCommand();
if (isDo) {
result = DownloadStatus.Download_From_Break_Success;
} else {
result = DownloadStatus.Download_From_Break_Failed;
}
} else {
OutputStream out = new FileOutputStream(f);
InputStream in = ftpClient.retrieveFileStream(new String(remote
.getBytes("UTF-8"), "iso-8859-1"));
byte[] bytes = new byte[1024];
long step = lRemoteSize / 100;

if(step == 0){
step = 1;
}

long process = 0;
long localSize = 0L;
int c;
while ((c = in.read(bytes)) != -1) {
out.write(bytes, 0, c);
localSize += c;
long nowProcess = localSize / step;
if (nowProcess > process) {
process = nowProcess;
//if (process % 10 == 0)
// System.out.println("下载进度：" + process);
// TODO 更新文件下载进度,值存放在process变量中
}
}
in.close();
out.close();
boolean upNewStatus = ftpClient.completePendingCommand();
if (upNewStatus) {
result = DownloadStatus.Download_New_Success;
} else {
result = DownloadStatus.Download_New_Failed;
}
}
System.out.println("完成下载"+remote);
return result;
}


/**
* 从FTP服务器上下载文件,支持断点续传，上传百分比汇报
*
* @param remote
* 远程文件路径
* @param local
* 本地文件路径
* @return 上传的状态
* @throws IOException
*/
public DownloadStatus download(String remote, BufferedOutputStream bos)
throws IOException {
byte[] temp = new byte[1 * 1024 * 10];
// 设置被动模式
ftpClient.enterLocalPassiveMode();
// 设置以二进制方式传输
ftpClient.setFileType(FTP.BINARY_FILE_TYPE);
DownloadStatus result;

// 检查远程文件是否存在
FTPFile[] files = ftpClient.listFiles(new String(remote.getBytes("UTF-8"), "iso-8859-1"));
if (files.length != 1) {
System.out.println("远程文件不存在");
return DownloadStatus.Remote_File_Noexist;
}

long lRemoteSize = files[0].getSize();
// 本地存在文件，进行断点下载
InputStream in = ftpClient.retrieveFileStream(new String(remote
.getBytes("UTF-8"), "iso-8859-1"));
long step = lRemoteSize / 100;
if(step == 0){
step = 1;
}
long process = 0;
long localSize = 0L;
int c;
BufferedInputStream bis = new BufferedInputStream(in);
while ((c = bis.read(temp)) != -1) {
bos.write(temp, 0, c);
localSize += c;
long nowProcess = localSize / step;
if (nowProcess > process) {
process = nowProcess;
//if (process % 10 == 0)
// System.out.println("下载进度：" + process);
// TODO 更新文件下载进度,值存放在process变量中
}
}
in.close();
boolean upNewStatus = ftpClient.completePendingCommand();
if (upNewStatus) {
result = DownloadStatus.Download_New_Success;
} else {
result = DownloadStatus.Download_New_Failed;
}
System.out.println("完成下载"+remote);
bos.flush();
bis.close();
bos.close();
in.close();

return result;
}

public void List(String pathName) throws IOException {
if (pathName.startsWith("/") && pathName.endsWith("/")) {
String directory = pathName;
// 更换目录到当前目录
this.ftpClient.changeWorkingDirectory(directory);
FTPFile[] files = this.ftpClient.listFiles();
for (FTPFile file : files) {
if (file.isFile()) {
System.out.println((directory + file.getName()));
} else if (file.isDirectory()) {
List(directory + file.getName() + "/");
}
}
}
}


public void syncRemoteFile(String pathName,String loaclDir) throws IOException {
if (pathName.startsWith("/") && pathName.endsWith("/")) {
String directory = pathName;
// 更换目录到当前目录
this.ftpClient.changeWorkingDirectory(directory);
FTPFile[] files = this.ftpClient.listFiles();
for (FTPFile file : files) {
if (file.isFile()) {
//System.out.println((directory + file.getName()));
File loacalFile = new File(loaclDir+"/"+file.getName());
if(!loacalFile.exists()){

System.out.println("下载："+pathName+"/"+file.getName());
download(pathName+"/"+file.getName(), loaclDir+"/"+file.getName());
}

} else if (file.isDirectory()) {
syncRemoteFile(directory + file.getName() + "/",loaclDir);
}
}
}
}

/**
*
* <p>
* 删除ftp上的文件
* </p>
*
* @param srcFname
* @return true || false
*/
public boolean removeFile(String srcFname) {
boolean flag = false;
if (ftpClient != null) {
try {
flag = ftpClient.deleteFile(srcFname);
} catch (IOException e) {
e.printStackTrace();

} finally {
try {
this.disconnect();
} catch (IOException e) {
// TODO Auto-generated catch block
e.printStackTrace();
}
}
}
return flag;
}

/**
* 上传文件到FTP服务器，支持断点续传
*
* @param local
* 本地文件名称，绝对路径
* @param remote
* 远程文件路径，使用/home/directory1/subdirectory/file.ext
* 按照Linux上的路径指定方式，支持多级目录嵌套，支持递归创建不存在的目录结构
* @return 上传结果
* @throws IOException
*/
public UploadStatus upload(String local, String remote) throws IOException {
// 设置PassiveMode传输
ftpClient.enterLocalPassiveMode();
// 设置以二进制流的方式传输
ftpClient.setFileType(FTP.BINARY_FILE_TYPE);
ftpClient.setControlEncoding("UTF-8");
UploadStatus result;
// 对远程目录的处理
String remoteFileName = remote;
if (remote.contains("/")) {
remoteFileName = remote.substring(remote.lastIndexOf("/") + 1);
// 创建服务器远程目录结构，创建失败直接返回
if (createDirecroty(remote, ftpClient) == UploadStatus.Create_Directory_Fail) {
return UploadStatus.Create_Directory_Fail;
}
}

// 检查远程是否存在文件
FTPFile[] files = ftpClient.listFiles(new String(remoteFileName.getBytes("UTF-8"), "iso-8859-1"));
if (files.length == 1) {
long remoteSize = files[0].getSize();
File f = new File(local);
long localSize = f.length();
if (remoteSize == localSize) {
return UploadStatus.File_Exits;
} else if (remoteSize > localSize) {
return UploadStatus.Remote_Bigger_Local;
}

// 尝试移动文件内读取指针,实现断点续传
result = uploadFile(remoteFileName, f, ftpClient, remoteSize);

// 如果断点续传没有成功，则删除服务器上文件，重新上传
if (result == UploadStatus.Upload_From_Break_Failed) {
if (!ftpClient.deleteFile(remoteFileName)) {
return UploadStatus.Delete_Remote_Faild;
}
result = uploadFile(remoteFileName, f, ftpClient, 0);
}
} else {
result = uploadFile(remoteFileName, new File(local), ftpClient, 0);
}
return result;
}

/**
* 断开与远程服务器的连接
*
* @throws IOException
*/
public void disconnect() throws IOException {
if (ftpClient.isConnected()) {
ftpClient.disconnect();
}
}

/**
* 递归创建远程服务器目录
*
* @param remote
* 远程服务器文件绝对路径
* @param ftpClient
* FTPClient对象
* @return 目录创建是否成功
* @throws IOException
*/
public UploadStatus createDirecroty(String remote, FTPClient ftpClient)
throws IOException {
UploadStatus status = UploadStatus.Create_Directory_Success;
String directory = remote.substring(0, remote.lastIndexOf("/") + 1);
if (!directory.equalsIgnoreCase("/")
&& !ftpClient.changeWorkingDirectory(new String(directory
.getBytes("UTF-8"), "iso-8859-1"))) {
// 如果远程目录不存在，则递归创建远程服务器目录
int start = 0;
int end = 0;
if (directory.startsWith("/")) {
start = 1;
} else {
start = 0;
}
end = directory.indexOf("/", start);
while (true) {
String subDirectory = new String(remote.substring(start, end)
.getBytes("UTF-8"), "iso-8859-1");
if (!ftpClient.changeWorkingDirectory(subDirectory)) {
if (ftpClient.makeDirectory(subDirectory)) {
ftpClient.changeWorkingDirectory(subDirectory);
} else {
System.out.println("创建目录失败");
return UploadStatus.Create_Directory_Fail;
}
}

start = end + 1;
end = directory.indexOf("/", start);

// 检查所有目录是否创建完毕
if (end <= start) {
break;
}
}
}
return status;
}


public void upload(File file) throws Exception{
if(file.isDirectory()){
ftpClient.makeDirectory(file.getName());
ftpClient.changeWorkingDirectory(file.getName());
String[] files = file.list();
for (int i = 0; i < files.length; i++) {
File file1 = new File(file.getPath()+"\\"+files[i] );
if(file1.isDirectory()){
upload(file1);
ftpClient.changeToParentDirectory();
}else{
File file2 = new File(file.getPath()+"\\"+files[i]);
FileInputStream input = new FileInputStream(file2);
ftpClient.storeFile(file2.getName(), input);
input.close();
}
}
}else{
File file2 = new File(file.getPath());
FileInputStream input = new FileInputStream(file2);
ftpClient.storeFile(file2.getName(), input);
input.close();
}
}

/**
* 上传文件到服务器,新上传和断点续传
*
* @param remoteFile
* 远程文件名，在上传之前已经将服务器工作目录做了改变
* @param localFile
* 本地文件File句柄，绝对路径
* @param processStep
* 需要显示的处理进度步进值
* @param ftpClient
* FTPClient引用
* @return
* @throws IOException
*/
public UploadStatus uploadFile(String remoteFile, File localFile,
FTPClient ftpClient, long remoteSize) throws IOException {
UploadStatus status;
// 显示进度的上传
long step = (localFile.length() / 100) +1;
long process = 0;
long localreadbytes = 0L;
RandomAccessFile raf = null;
OutputStream out = null;
try {
raf = new RandomAccessFile(localFile, "r");
out = ftpClient.appendFileStream(new String(remoteFile.getBytes("UTF-8"), "iso-8859-1"));
// 断点续传
if (remoteSize > 0) {
ftpClient.setRestartOffset(remoteSize);
process = remoteSize / step;
raf.seek(remoteSize);
localreadbytes = remoteSize;
}
byte[] bytes = new byte[1024];
int c;
while ((c = raf.read(bytes)) != -1) {
out.write(bytes, 0, c);
localreadbytes += c;
if (localreadbytes / step != process) {
process = localreadbytes / step;
LOG.info("upload ftp process :" + process);
}
}
} finally{
if(raf!=null){
raf.close();
}
if(out !=null){
out.flush();
out.close();
}

}

boolean result = ftpClient.completePendingCommand();
if (remoteSize > 0) {
status = result ? UploadStatus.Upload_From_Break_Success
: UploadStatus.Upload_From_Break_Failed;
} else {
status = result ? UploadStatus.Upload_New_File_Success
: UploadStatus.Upload_New_File_Failed;
}
return status;
}


public static void main(String[] args) {
FTPUtils myFtp = new FTPUtils();
try {
// portal创建的用用户名密码
myFtp.connect("127.0.0.1", 2121, "user1", "123456");
// myFtp.ftpClient.makeDirectory(new
// String("电视剧".getBytes("GBK"),"iso-8859-1"));
// myFtp.ftpClient.changeWorkingDirectory(new
// String("电视剧".getBytes("GBK"),"iso-8859-1"));
// myFtp.ftpClient.makeDirectory(new
// String("走西口".getBytes("GBK"),"iso-8859-1"));
// System.out.println(myFtp.upload("E:\\yw.flv", "/yw.flv",5));
//myFtp.reName("/upload2/cs1.6.rar","/upload2/cs1.63.rar");
System.out.println(myFtp.upload("e:/安装Eclipse反编译插件1.doc","/admin/安装Eclipse反编译插件1.doc"));
//myFtp.List("/upload/");
//myFtp.disconnect();
} catch (IOException e) {
System.out.println("连接FTP出错：" + e.getMessage());
}
}
}

public enum DownloadStatus {
Remote_File_Noexist, //远程文件不存在
Local_Bigger_Remote, //本地文件大于远程文件
Download_From_Break_Success, //断点下载文件成功
Download_From_Break_Failed, //断点下载文件失败
Download_New_Success, //全新下载文件成功
Download_New_Failed; //全新下载文件失败
}

public enum UploadStatus {
Create_Directory_Fail, //远程服务器相应目录创建失败
Create_Directory_Success, //远程服务器闯将目录成功
Upload_New_File_Success, //上传新文件成功
Upload_New_File_Failed, //上传新文件失败
File_Exits, //文件已经存在
Remote_Bigger_Local, //远程文件大于本地文件
Upload_From_Break_Success, //断点续传成功
Upload_From_Break_Failed, //断点续传失败
Delete_Remote_Faild; //删除远程文件失败
}
```

#### 1.2 javaclient 连接启用ftps的 ftpserver
##### 1.2.1启用ftps
1、进入portal运维中心->系统运维->本集群包含服务（右上角）->搜索ftp,进入并点击服务->组件：FTPServer,点击主机ip-> 服务配置（右上角）->左侧服务列表选择ftponhdfs，置界面修改ftponhdfs.open.ssl，默认是false，不开启ssl on ftp，设置为true，开启ssl on ftp，端口是2226. 

2、在安装ftponhdfs服务的机器中，/usr/local/hdfs-over-ftp/conf 下有ssl 证书。ftp.jks是服务端证书，ftpclient.jks 客户端连接证书

##### 1.2.2 javaclient ftps方式连接ftpserver
```
import org.apache.commons.net.PrintCommandListener;
import org.apache.commons.net.ftp.FTPFile;
import org.apache.commons.net.ftp.FTPSClient;

import javax.net.ssl.KeyManager;
import javax.net.ssl.KeyManagerFactory;
import java.io.*;
import java.security.KeyStore;


public class FTPSTest {
private static String key_path = "/data/home/tbds/ftpclient.jks";
private static String key_pw = "39604a9b0c816abfb971b16701825811";
private static final int defaultTimeout = 10000;
private static final int soTimeout = 900000;
private static final int dataTimeout = 5000;

public static void main(String[] args) throws Exception {
connect(args[0], args[1]);
}

private static void connect(String active, String host) throws Exception {
FTPSClient ftpsClient = new FTPSClient(true);
ftpsClient.addProtocolCommandListener(new PrintCommandListener(new PrintWriter(System.out)));
ftpsClient.setKeyManager(getKeyManager());
ftpsClient.setDefaultTimeout(defaultTimeout);
System.out.println("ftps client created.");
ftpsClient.connect(host, 2226);
System.out.println("ftps connected");
ftpsClient.setSoTimeout(soTimeout);
ftpsClient.getReplyCode();
//ftpsClient.execPBSZ(0);
//ftpsClient.execPROT("P");
ftpsClient.login("user1", "123456"); // portal创建的用户名、密码
System.out.println("ftps login success");
if(active.equalsIgnoreCase("active")){// 主动模式
ftpsClient.enterLocalActiveMode();
} else {//被动模式
ftpsClient.enterLocalPassiveMode();
}
long startTime = System.currentTimeMillis();
System.out.println("List file length:" + ftpsClient.listFiles().length);
long endTime = System.currentTimeMillis();
System.out.println("List file time:" + (endTime - startTime)/1000);

FileInputStream fs = new FileInputStream(key_path);
System.out.println("storeFile: " + ftpsClient.storeFile("key_path.jks", fs));

ftpsClient.changeWorkingDirectory("/ftp/yiwen2/all/20170615");//cd dir
FTPFile[] files = ftpsClient.listFiles();
System.out.println("List file length of /ftp/yiwen2/all/20170615:" + files.length);
for (FTPFile file : files){
System.out.println(file.getName());
File localFile = new File("/data/home/tbds/" + file.getName());
OutputStream os = new FileOutputStream(localFile);
ftpsClient.retrieveFile(file.getName(), os);
}
System.out.println("download file success");
ftpsClient.disconnect();
}

private static KeyManager getKeyManager() throws Exception {
KeyStore key_ks = KeyStore.getInstance("JKS");
key_ks.load(new FileInputStream(key_path), key_pw.toCharArray());
KeyManagerFactory kmf = KeyManagerFactory.getInstance(KeyManagerFactory.getDefaultAlgorithm());
kmf.init(key_ks, key_pw.toCharArray());
KeyManager[] km = kmf.getKeyManagers();
return km[0];
}
}
```
