```
package com.tencent.hadoop;


import org.apache.hadoop.conf.Configuration;
import org.apache.hadoop.fs.FileStatus;
import org.apache.hadoop.fs.FileSystem;
import org.apache.hadoop.fs.Path;
import org.apache.hadoop.security.UserGroupInformation;


import java.util.ArrayList;
import java.util.List;

/**
 * Created by leslizhang on 2017/9/8.
 */
public class HdfsAccess {

    public static void main(String[] args) {

        if( args.length != 6 ){
            System.out.println( "[Usage: core-site-path hdfs-site-path authentication-method secureId userName secureKey ]" );
            System.exit(1);
        }

        FileSystem fs = null ;
        try {
            Configuration conf = new Configuration();
            conf.addResource(new Path(args[0]));
            conf.addResource(new Path(args[1]));

            conf.set("hadoop.security.authentication",args[2]);
            conf.set("hadoop_security_authentication_tbds_secureid",args[3]);
            conf.set("hadoop_security_authentication_tbds_username",args[4]);
            conf.set("hadoop_security_authentication_tbds_securekey",args[5]);
            conf.set("fs.hdfs.impl", "org.apache.hadoop.hdfs.DistributedFileSystem");

            UserGroupInformation.setConfiguration( conf );
            UserGroupInformation.loginUserFromSubject(null);
            fs = FileSystem.get(conf) ;

            System.out.println( ">>>>>> List file status..." );
            Path basePath = new Path("/");
            FileStatus[] fileStats = fs.listStatus(basePath) ;

            List<String> fileList = new ArrayList<String>() ;

            if (fileStats != null) {
                for( FileStatus fst : fileStats ){
                    System.out.println( fst.toString() );
                }
            }

            String dir = "/test_demo" + System.currentTimeMillis();
            System.out.println( ">>>>>> mkdir "+dir+" ..." );
            fs.mkdirs(new Path(dir));



        } catch (Exception e){
            e.printStackTrace();
        }
    }
}


```



