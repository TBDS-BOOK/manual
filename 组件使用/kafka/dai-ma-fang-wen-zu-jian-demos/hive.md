```
package com.tencent.hive;

/**
 * Created by leslizhang on 2017/11/14.
 */
import org.apache.hive.service.auth.HiveAuthFactory;
import org.apache.hive.service.auth.PlainSaslHelper;
import org.apache.hive.service.cli.thrift.*;
import org.apache.thrift.TException;
import org.apache.thrift.protocol.TBinaryProtocol;
import org.apache.thrift.transport.TTransport;

import javax.security.sasl.SaslException;
import java.sql.*;
import java.util.Iterator;
import java.util.List;
    /*
     *  to make it work add the hive-service and hadoop-client dependencies
     *  in pom.xml
     */
    public class HiveThriftClient {
        private static String driverName = "org.apache.hive.jdbc.HiveDriver";

        public static void main(String[] args) throws SQLException {

            if( args.length != 2 ){
                System.out.println( "Usage:[ hiveserver2:port tableName ]" );

            }

            try {
                Class.forName(driverName);
            } catch (ClassNotFoundException e) {
                e.printStackTrace();
                System.exit(1);
            }

            Connection con = DriverManager.getConnection("jdbc:hive2://"+args[0]+"/default", "hive", "hive@Tbds.com");
            Statement stmt = con.createStatement();
            String tableName = args[1];
            stmt.execute("drop table if exists " + tableName);
            stmt.execute("create table " + tableName +  " (key int, value string)");
            System.out.println(">>>>>>>>>> Create table success!");
            // show tables
            String sql = "show tables '" + tableName + "'";
            System.out.println(">>>>>>>>>> Running: " + sql);
            ResultSet res = stmt.executeQuery(sql);
            if (res.next()) {
                System.out.println(res.getString(1));
            }

            // describe table
            sql = "describe " + tableName;
            System.out.println(">>>>>>>>>> Running: " + sql);
            res = stmt.executeQuery(sql);
            while (res.next()) {
                System.out.println(res.getString(1) + "\t" + res.getString(2));
            }


            sql = "select * from " + tableName;
            res = stmt.executeQuery(sql);
            while (res.next()) {
                System.out.println(String.valueOf(res.getInt(1)) + "\t" + res.getString(2));
            }

            sql = "select count(1) from " + tableName;
            System.out.println(">>>>>>>>>> Running: " + sql);
            res = stmt.executeQuery(sql);
            while (res.next()) {
                System.out.println(res.getString(1));
            }
        }

    }
```



