#!/usr/bin/env python
#
# Available device: Tipping-bucket Rain gauge
# Product number: SMR29-C
# Serial communication protocol: RS485
# Purpose:
# Measure rainfall intensity every 10 mins (Unit in mm/hr)
#

# Including
import minimalmodbus
import time
from time import gmtime, strftime
from urllib.request import urlopen
#from urllib2 import urlopen

# Definition of variable
minimalmodbus.BAUDRATE = 19200
rg_slaveAddr = 2
devName = '/dev/ttyUSB0' # RPI    format
comName = 'COM5'        # Window format
id_No = "8999"
countNum_Loop = -1

befVal    = 0
countVal  = 0
bucketVal = 0.2  # Unit in mm
rfInts    = 0    # Unit in mm/hr
rfDep     = 0    # Unit in mm

start = time.time()


def rfRead():
    global befVal, countVal, rfDep
    rf_realVal = 0    
    # Port name, slave address (in decimal)
    instrument_2 = minimalmodbus.Instrument(comName, rg_slaveAddr) 
    try:
        # Register number, number of decimals, function code
        rf_realVal = instrument_2.read_register(0, 0, 4)               
    except IOError:
        print("Failed to read from instrument")
        print('------------------------')    
    
    rf_refVal   = [0, 13107, 26215, 39322, 52429]
    boolTemp1   = rf_realVal in rf_refVal
    while boolTemp1:
        if (rf_realVal != befVal):
            print("Detect additional rainfall")
            print('------------------------')
            befVal = rf_realVal
            countVal += 1
            rfDep = countVal * bucketVal            
            break
        else:
            boolTemp1 = not boolTemp1
            break        
    print("Additional rainfall (T/F):",boolTemp1)
    
    # print("Read value:", rf_realVal)
    # print("Previous value:", befVal)
    print('------------------------')     
    return rfDep

def httpPOST(String0, String1, String2, String3):    
    try:
        global timeStamp
        timeStamp = strftime("%Y%m%d%H%M%S")
        url = 'http://ec2-54-175-179-28.compute-1.amazonaws.com/update_general.php?' + \
              'site=Demo&' + \
              'time='+ repr(timeStamp)+ \
              '&weather=0' + \
              '&id='+ repr(String0) + \
              '&air=0' + \
              '&acceleration=0' + \
              '&cleavage=0' + \
              '&incline=0' + \
              '&field1='+repr(String1)+ \
              '&field2='+repr(String2)+ \
              '&field3='+repr(String3)
        
        url_TT ='http://data.thinktronltd.com/TCGEMSIS/GETMTDATA.aspx?' + \
              'site=Demo&' + \
              'time='+ repr(timeStamp)+ \
              '&weather=0' + \
              '&id='+ repr(String0) + \
              '&air=0' + \
              '&acceleration=0' + \
              '&cleavage=0' + \
              '&incline=0' + \
              '&field1='+repr(String1)+ \
              '&field2='+repr(String2)+ \
              '&field3='+repr(String3) 
        
        resp = urlopen(url).read()
        resp_TT = urlopen(url_TT).read()
        print(resp)
        print(resp_TT)
        print('------------------------')
    except:
        print('We have an error!')
        time.sleep(30) # Wait for 30 sec
        resp = urlopen(url).read()
        print(resp)
        print('------------------------')
try:
    while True:
        end = time.time()
        delSec = end - start # Unit in sec    
        rfDep2 = rfRead() # Unit in mm
        print('Rainfall depth: ',rfDep2,' mm') 
        
        while (countNum_Loop < 0):
            httpPOST(id_No, 0, 0, "Reboot")
            countNum_Loop += 1
            break

        if (delSec - 600) > 0:  
            rfInts = rfDep2/(600/3600) # Unit in mm/hr every 10 mins
            start  = end   
            
            data1 = round(rfInts,3) 
            data2 = 0
            httpPOST(id_No, data1, data2, 0)
            countVal  = 0
            rfDep     = 0
            print('Rainfall intensity: ',rfInts,' mm/hr')
            #print("Upload time : %s" % timeStamp)    
            print('------------------------')        
        else:
            print('Continuous detection.')
            print('------------------------')            
        time.sleep(10) # Wait for 10 sec
        
except KeyboardInterrupt:
    minimalmodbus.CLOSE_PORT_AFTER_EACH_CALL=True
    print('------------------------')
    print("Stop")
    print('------------------------')
