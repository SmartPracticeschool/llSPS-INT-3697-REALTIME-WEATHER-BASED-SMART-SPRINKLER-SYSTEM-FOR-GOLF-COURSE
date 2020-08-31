import time
import sys
import ibmiotf.application
import ibmiotf.device
import random
import requests
from pprint import pprint

#Mentioning IBM IOT Credentials
organisation= "yhxaeo"
deviceType= "raspberrypi"
deviceId= "Sensor"
authMethod= "token"
authToken= "9449320176"
data=0

#function to define commands from the Web UI
def myCommandCallback(cmd):
    print("Command received: %s" %cmd.data)

#connecting the IBM device to our python code
try:
    deviceOptions = {"org": organisation, "type": deviceType, "id":deviceId, "auth-method": authMethod, "auth-token": authToken}
    deviceCli= ibmiotf.device.Client(deviceOptions)

except Exception as e:
    print("Caught exception connecting device: %s" %str(e))
    sys.exit()




deviceCli.connect()

while True:
    hum= random.randint(0,100)
    temp= random.randint(0,50)
    
    #code for Soil moisture
    Adcvalue = random.randint(511,1023)
    output_V = Adcvalue/1023
    Moisture = round(100- (output_V*100))
    #Generate Message
    if Moisture <15 and hum >95:
            Message= "Moisture is less but weather will take care."
    elif (Moisture >25 and hum >95) or Moisture >25:
            Message= "Dont turn the sprinklers on even by mistake!"
    elif (Moisture > 15 and Moisture <25):
            Message= "Perfect Soil"
    elif Moisture <15 and hum <60:
            Message= "Turn the sprinklers on"
    elif Moisture <15:
            print("Starting Motor")
    elif Moisture >25:
            print("Stopping Motor")
    


    #print(data)
    def myOnPublishCallback():
        
        print("Soil Moisture= %s %%" %Moisture)
        print("Published temperature= %s C" %temp, "Humidity= %s %%" %hum,"to IBM WATSON")
    #code for generating message and toggling motors    
        if Moisture <15 and hum >95:
            print("Moisture is less but weather will take care." )
        elif (Moisture >25 and hum >95) or Moisture >25:
            print("Dont turn the sprinklers on even by mistake!")
            print("Stopping Motor")
        elif (Moisture > 15 and Moisture <25):
            print("Perfect Soil")
        
        elif Moisture <15 and hum <60:
            print("Turn the sprinklers on")
            print("Starting Motor")
        elif Moisture <15:
            print("Starting Motor")
        elif Moisture >25:
            print("Stopping Motor")

    data= { 'Temperature': temp, 'Humidity': hum,'Soil_moisture': Moisture,'Suggestion': Message }
        
    
    success= deviceCli.publishEvent("Smart","json",data,qos= 0,on_publish= myOnPublishCallback )
    if not success:
        print("Not connected to IoTF")
    time.sleep(10)
    deviceCli.commandCallback= myCommandCallback
deviceCli.disconnect()
