title: 慢SQL案例
author: ming
date: 2025-04-30 10:23:00
tags: 协议
---
### Client
##### java client的报错信息
```
(base) ming@ff2393892154:~/_work/procotol/keng$ java -cp .:./mysql-connector-java-5.1.45.jar Test "jdbc:mysql://192.168.1.66:3306/test?useSSL=false&useServerPrepStmts=true&cachePrepStmts=true&connectTimeout=500&socketTimeout=1700" root 3306 "select sleep(10), id from users where id= ?" 10
com.mysql.jdbc.exceptions.jdbc4.CommunicationsException: Communications link failure

The last packet successfully received from the server was 1,703 milliseconds ago.  The last packet sent successfully to the server was 1,703 milliseconds ago.
	at java.base/jdk.internal.reflect.NativeConstructorAccessorImpl.newInstance0(Native Method)
	at java.base/jdk.internal.reflect.NativeConstructorAccessorImpl.newInstance(NativeConstructorAccessorImpl.java:77)
	at java.base/jdk.internal.reflect.DelegatingConstructorAccessorImpl.newInstance(DelegatingConstructorAccessorImpl.java:45)
	at java.base/java.lang.reflect.Constructor.newInstanceWithCaller(Constructor.java:500)
	at java.base/java.lang.reflect.Constructor.newInstance(Constructor.java:481)
	at com.mysql.jdbc.Util.handleNewInstance(Util.java:425)
	at com.mysql.jdbc.SQLError.createCommunicationsException(SQLError.java:990)
	at com.mysql.jdbc.MysqlIO.reuseAndReadPacket(MysqlIO.java:3559)
	at com.mysql.jdbc.MysqlIO.reuseAndReadPacket(MysqlIO.java:3459)
	at com.mysql.jdbc.MysqlIO.checkErrorPacket(MysqlIO.java:3900)
	at com.mysql.jdbc.MysqlIO.sendCommand(MysqlIO.java:2527)
	at com.mysql.jdbc.ServerPreparedStatement.serverExecute(ServerPreparedStatement.java:1283)
	at com.mysql.jdbc.ServerPreparedStatement.executeInternal(ServerPreparedStatement.java:783)
	at com.mysql.jdbc.PreparedStatement.executeQuery(PreparedStatement.java:1966)
	at Test.main(Test.java:21)
Caused by: java.net.SocketTimeoutException: Read timed out
	at java.base/sun.nio.ch.NioSocketImpl.timedRead(NioSocketImpl.java:288)
	at java.base/sun.nio.ch.NioSocketImpl.implRead(NioSocketImpl.java:314)
	at java.base/sun.nio.ch.NioSocketImpl.read(NioSocketImpl.java:355)
	at java.base/sun.nio.ch.NioSocketImpl$1.read(NioSocketImpl.java:808)
	at java.base/java.net.Socket$SocketInputStream.read(Socket.java:966)
	at com.mysql.jdbc.util.ReadAheadInputStream.fill(ReadAheadInputStream.java:101)
	at com.mysql.jdbc.util.ReadAheadInputStream.readFromUnderlyingStreamIfNecessary(ReadAheadInputStream.java:144)
	at com.mysql.jdbc.util.ReadAheadInputStream.read(ReadAheadInputStream.java:174)
	at com.mysql.jdbc.MysqlIO.readFully(MysqlIO.java:3008)
	at com.mysql.jdbc.MysqlIO.reuseAndReadPacket(MysqlIO.java:3469)
	... 7 more
```
##### client code
```java
import java.sql.Connection;
import java.sql.DriverManager;
import java.sql.ResultSet;
import java.sql.SQLException;
import java.sql.Statement;
import java.sql.PreparedStatement;
public class Test { //不要琢磨代码规范、为什么要这么写，就是为了方便改吧改吧做很多不同的验证试验
   public static void main(String args[]) throws NumberFormatException, InterruptedException, ClassNotFoundException {
     Class.forName("com.mysql.jdbc.Driver");
     String url = args[0];
     String user = args[1];
     String pass = args[2];
     String sql = args[3];
     String interval = args[4];
     try {
       Connection conn = DriverManager.getConnection(url, user, pass);
       while (true) {

         PreparedStatement stmt = conn.prepareStatement(sql);
         stmt.setString(1, interval);
         ResultSet rs = stmt.executeQuery();
         rs.close();
         stmt.close();

         PreparedStatement stmt2 = conn.prepareStatement(sql);
         stmt2.setString(1, interval);
         rs = stmt2.executeQuery();
                while (rs.next()) {
                   System.out.println("fine");
                }
         rs.close();
         stmt2.close();

         Thread.sleep(Long.valueOf(interval));
                break;
       }
            conn.close();
     } catch (SQLException e) {
       e.printStackTrace();
     }
   }
}

```

### server抓包
```
(base) ming@ming:~$ sudo tshark -i wlx502b73cb9f9a -Y "tcp.port==3306" -T fields -e frame.number -e frame.time -e frame.time_delta -e tcp.srcport -e tcp.dstport -e tcp.len -e _ws.col.Info -e mysql.query
Running as user "root" and group "root". This could be dangerous.
Capturing on 'wlx502b73cb9f9a'
 ** (tshark:3226) 10:11:29.450593 [Main MESSAGE] -- Capture started.
 ** (tshark:3226) 10:11:29.450692 [Main MESSAGE] -- File: "/tmp/wireshark_wlx502b73cb9f9aQUU052.pcapng"
16	Apr 30, 2025 10:11:34.350350921 CST	0.947355453	60467	3306	0	60467 → 3306 [SYN] Seq=0 Win=65535 Len=0 MSS=1460 WS=64 TSval=2340187876 TSecr=0 SACK_PERM=1
17	Apr 30, 2025 10:11:34.350462259 CST	0.000111338	3306	60467	0	3306 → 60467 [SYN, ACK] Seq=0 Ack=1 Win=65160 Len=0 MSS=1460 SACK_PERM=1 TSval=3615723320 TSecr=2340187876 WS=128
18	Apr 30, 2025 10:11:34.355973421 CST	0.005511162	60467	3306	0	60467 → 3306 [ACK] Seq=1 Ack=1 Win=131776 Len=0 TSval=2340187888 TSecr=3615723320
19	Apr 30, 2025 10:11:34.356373506 CST	0.000400085	3306	60467	78	Server Greeting  proto=10 version=8.0.42
20	Apr 30, 2025 10:11:34.358847563 CST	0.002474057	60467	3306	0	60467 → 3306 [ACK] Seq=1 Ack=79 Win=131712 Len=0 TSval=2340187894 TSecr=3615723326
21	Apr 30, 2025 10:11:34.381972267 CST	0.023124704	60467	3306	242	Login Request user=root db=test
22	Apr 30, 2025 10:11:34.382031245 CST	0.000058978	3306	60467	0	3306 → 60467 [ACK] Seq=79 Ack=243 Win=65152 Len=0 TSval=3615723351 TSecr=2340187914
23	Apr 30, 2025 10:11:34.382161639 CST	0.000130394	3306	60467	48	Auth Switch Request
24	Apr 30, 2025 10:11:34.386851498 CST	0.004689859	60467	3306	0	60467 → 3306 [ACK] Seq=243 Ack=127 Win=131712 Len=0 TSval=2340187921 TSecr=3615723352
25	Apr 30, 2025 10:11:34.386969679 CST	0.000118181	60467	3306	24	Auth Switch Response
26	Apr 30, 2025 10:11:34.387212101 CST	0.000242422	3306	60467	11	Response  OK
27	Apr 30, 2025 10:11:34.389723270 CST	0.002511169	60467	3306	0	60467 → 3306 [ACK] Seq=267 Ack=138 Win=131712 Len=0 TSval=2340187925 TSecr=3615723357
28	Apr 30, 2025 10:11:34.391847790 CST	0.002124520	60467	3306	897	Request Query	/* mysql-connector-java-5.1.45 ( Revision: 9131eefa398531c7dc98776e8a3fe839e544c5b2 ) */SELECT  @@session.auto_increment_increment AS auto_increment_increment, @@character_set_client AS character_set_client, @@character_set_connection AS character_set_connection, @@character_set_results AS character_set_results, @@character_set_server AS character_set_server, @@collation_server AS collation_server, @@init_connect AS init_connect, @@interactive_timeout AS interactive_timeout, @@license AS license, @@lower_case_table_names AS lower_case_table_names, @@max_allowed_packet AS max_allowed_packet, @@net_buffer_length AS net_buffer_length, @@net_write_timeout AS net_write_timeout, @@have_query_cache AS have_query_cache, @@sql_mode AS sql_mode, @@system_time_zone AS system_time_zone, @@time_zone AS time_zone, @@transaction_isolation AS transaction_isolation, @@wait_timeout AS wait_timeout
29	Apr 30, 2025 10:11:34.392261659 CST	0.000413869	3306	60467	1071	Response TABULAR Response  OK
30	Apr 30, 2025 10:11:34.394971212 CST	0.002709553	60467	3306	0	60467 → 3306 [ACK] Seq=1164 Ack=1209 Win=130688 Len=0 TSval=2340187930 TSecr=3615723362
31	Apr 30, 2025 10:11:34.406723844 CST	0.011752632	60467	3306	18	Request Query	SHOW WARNINGS
32	Apr 30, 2025 10:11:34.406912856 CST	0.000189012	3306	60467	203	Response TABULAR Response  OK
33	Apr 30, 2025 10:11:34.409847491 CST	0.002934635	60467	3306	0	60467 → 3306 [ACK] Seq=1182 Ack=1412 Win=130880 Len=0 TSval=2340187945 TSecr=3615723376
34	Apr 30, 2025 10:11:34.410847357 CST	0.000999866	60467	3306	22	Request Query	SET NAMES utf8mb4
35	Apr 30, 2025 10:11:34.411049777 CST	0.000202420	3306	60467	11	Response  OK
36	Apr 30, 2025 10:11:34.413721737 CST	0.002671960	60467	3306	0	60467 → 3306 [ACK] Seq=1204 Ack=1423 Win=131072 Len=0 TSval=2340187949 TSecr=3615723380
37	Apr 30, 2025 10:11:34.414343733 CST	0.000621996	60467	3306	37	Request Query	SET character_set_results = NULL
38	Apr 30, 2025 10:11:34.414548189 CST	0.000204456	3306	60467	11	Response  OK
39	Apr 30, 2025 10:11:34.416990667 CST	0.002442478	60467	3306	0	60467 → 3306 [ACK] Seq=1241 Ack=1434 Win=131072 Len=0 TSval=2340187952 TSecr=3615723384
40	Apr 30, 2025 10:11:34.424098392 CST	0.007107725	60467	3306	21	Request Query	SET autocommit=1
41	Apr 30, 2025 10:11:34.424296835 CST	0.000198443	3306	60467	11	Response  OK
42	Apr 30, 2025 10:11:34.426600459 CST	0.002303624	60467	3306	0	60467 → 3306 [ACK] Seq=1262 Ack=1445 Win=131072 Len=0 TSval=2340187962 TSecr=3615723394
43	Apr 30, 2025 10:11:34.439848404 CST	0.013247945	60467	3306	48	Request Prepare Statement 	select sleep(10), id from users where id= ?
44	Apr 30, 2025 10:11:34.440179423 CST	0.000331019	3306	60467	122	Response
45	Apr 30, 2025 10:11:34.448473451 CST	0.008294028	60467	3306	0	60467 → 3306 [ACK] Seq=1310 Ack=1567 Win=131008 Len=0 TSval=2340187978 TSecr=3615723410
46	Apr 30, 2025 10:11:34.448473631 CST	0.000000180	60467	3306	21	Request Execute Statement
47	Apr 30, 2025 10:11:34.489163416 CST	0.040689785	3306	60467	0	3306 → 60467 [ACK] Seq=1567 Ack=1331 Win=64256 Len=0 TSval=3615723459 TSecr=2340187978
63	Apr 30, 2025 10:11:35.072729238 CST	0.250135043	60302	3306	0	60302 → 3306 [ACK] Seq=1 Ack=1 Win=2048 Len=0
64	Apr 30, 2025 10:11:35.072781476 CST	0.000052238	3306	60302	0	[TCP ACKed unseen segment] 3306 → 60302 [ACK] Seq=1 Ack=2 Win=493 Len=0 TSval=3615724042 TSecr=1597434315
69	Apr 30, 2025 10:11:36.166580140 CST	0.209393801	60467	3306	0	60467 → 3306 [FIN, ACK] Seq=1331 Ack=1567 Win=131072 Len=0 TSval=2340189700 TSecr=3615723459
70	Apr 30, 2025 10:11:36.207164269 CST	0.040584129	3306	60467	0	3306 → 60467 [ACK] Seq=1567 Ack=1332 Win=64256 Len=0 TSval=3615725177 TSecr=2340189700
88	Apr 30, 2025 10:11:39.448950399 CST	0.046271284	3306	60467	113	Response TABULAR Response  OK Response  OK
89	Apr 30, 2025 10:11:39.448999187 CST	0.000048788	3306	60467	55	Response  Error 1158
90	Apr 30, 2025 10:11:39.449046442 CST	0.000047255	3306	60467	0	3306 → 60467 [FIN, ACK] Seq=1735 Ack=1332 Win=64256 Len=0 TSval=3615728418 TSecr=2340189700
93	Apr 30, 2025 10:11:39.455038276 CST	0.001240279	60467	3306	0	60467 → 3306 [ACK] Seq=1332 Ack=1735 Win=130944 Len=0 TSval=2340192988 TSecr=3615728418
96	Apr 30, 2025 10:11:39.457667957 CST	0.001328614	60467	3306	0	60467 → 3306 [ACK] Seq=1332 Ack=1736 Win=130944 Len=0 TSval=2340192988 TSecr=3615728418
^C41 packets captured
```