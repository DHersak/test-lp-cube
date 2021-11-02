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

- DF_Run - a Dataframe that has Rows for Serial number of parts, and Columns as sensor readings one-sensor feature a column (i.e. used by RCA). This dataframe contains information about "what happened for a particular part across different sensors before it reaches a target station". (In the future, we will introduce more detailed Line configuration, so that which sensor should be ahead of which other sensor is configured, and those info can be used to create more detailed RUN info. i.e. we will be able to produce RUN for a sub section of the line instead of the full line, which we won't be able to do until we have the line configuration info.)

- DF_TimeSeries - a Dataframe that has time-slice as rows and columns as sensor readings one-sensor feature a column, filtered by min/max values. Part number and Serial number of parts are included as metadata (i.e. used by Single V drift).

- DF_History_Detail - This the a subset of the CUBE with all the features as columns, and all the values filtered with certain criteria (i.e. min/max limit) 

- DF_Path - a dataframe that has Rows for Serial number of parts and Columns as sensor readings. This one is different from DF_Run in the sense that it does not have fixed # of columns while DF_Run has fixed # of columns. Each part only have one PATH even if it has been re-processed several times. Each path includes concatenation of all runs. (It is also easier to think about the differences between RUN and PATH and how they can be used together once we have the line-configuration info: the RUN would tells us sensor readings at each stage of the processing (sequentially according to how a part pass through each stage of processing), and PATH will tell us more info like which particular station actually carried out the operation of each stage, this additional info will give us more insights into difference between parallel stations, or impact of re-processing at some stages.)

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
