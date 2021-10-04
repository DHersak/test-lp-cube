# LP-cube
This module transforms incoming data stream to "Cube" format. 


This module will be "headless", there is no direct user interface exposed by this module in the original design.
The main goal is to "breakdown" heavy calculation like pivoting to low cost small operations as streaming data come in.  Every time a Kafka record is received, the consumer would trigger this "incremental pivoting" to get the "Cube" ready for DS query as soon as possible.  It is expected to reduce minutes of data transformation everytime we make a DS call to sub-seconds whenever data arrive.  This will reduce DS data prepartion to seconds each time we need a data frame for inference. 


**Input:** 
This module will rely on the following input:
- JSON messages from streaming system (i.e. Kafka) 
- Line configuration info (Biz schema config) in SQL tables (i.e. Postgresql)
- Technical Schema of how to intepret JSON contents

**Output:**
The outputs are Dataframes to be used by our DataScience functions (should be both a Python Library function API, and an OData API: see https://github.com/SAP/python-pyodata)
 - DF_RUN - a Dataframe that has Rows for Serial number of parts, and Columns as sensor readings one-sensor feature a column (i.e. used by RCA)
 - DF_TimeSeries - a Dataframe that has time-slice as rows, filled with "last-known-good-value", and Columns as sensor readings one-sensor feature a column (Serial number of parts are ignored) (i.e. used by Single V drift)
 - DF_History (full info across the cube that is not filled with "last-known-good-value"

<pre>
Sample Cube 
Time  SN  RUN#  OP1_SIG1	OP1_SIG2	OP2_SIG1	OP2_SIG2	OP3_SIG1	OP3_SIG2	OP3_SIG3	OP4_SIG1	
t0	  P12 1	    1_1_1								
t1	  P12 1	    1_1_2	    1_2_1							
t2	  P12 1		            1_2_2							
t3	  P12 1		            1_2_3							
t4	  P12 1			                    2_1_1						
t5	  P12 1				                            2_2_1	    3_1_1				
t6	  P12 1					                                    3_1_2				
t7	  P12 1						                                            3_2_1	    3_3_1		
t8	  P12 1							                                                    3_3_2	    4_1_1	
t9	  P12 1								                                                            4_1_2	
t10	  P12 2	    1_1_3								
t11	  P12 2			                    2_1_2						
t12	  P12 2			                    2_1_3	    2_2_2					
t13	  P12 2								                                                            4_1_3	
t14	  P13 1	    1_1_1								
t15	  P13 1		            1_2_1							
t16	  P13 1		            1_2_2							
t17	  P13 1			          1_2_3						
t18	  P13 1			          1_2_4						
t19	  P13 1				                            2_2_1					
t20	  P13 1				                            2_2_2					
t21	  P13 1					                                    3_1_1				
t22	  P13 1						                                            3_2_1			
t23	  P13 1							                                                    3_3_1		
t24	  P13 1								                                                            4_1_1	
t25   B14 1     1_1_1
</pre>
