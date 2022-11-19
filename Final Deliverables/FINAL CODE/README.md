import time
import sys
import ibmiotf.application
import ibmiotf.device
import random

#Provide your IBM Watson Device Credentials
organization = "63004g"
deviceType = "MainDevice"
deviceId = "9344022806"
authMethod = "token"
authToken = "9944611970"

# Initialize GPIO
def myCommandCallback(cmd):
    print("Command received: %s" % cmd.data['command'])
    status=cmd.data['command']
    if status == "motoron":
        print ("motor is on")
    elif status == "motoroff":
        print ("motor is off")
    else :
        print ("please send proper command")

try:
	deviceOptions = {"org": organization, "type": deviceType, "id": deviceId, "auth-method": authMethod, "auth-token": authToken}
	deviceCli = ibmiotf.device.Client(deviceOptions)
	#..............................................
	
except Exception as e:
	print("Caught exception connecting device: %s" % str(e))
	sys.exit()

# Connect and send a datapoint "hello" with value "world" into the cloud as an event of type "greeting" 10 times
deviceCli.connect()

while True:
        #Get Sensor Data from DHT11
        animal=random.uniform(0.1, 0.99)
        moisture=random.randint(0,110)
        temperature=random.randint(-20,125)
        Humid=random.randint(0,100)
        
        data = {'animal':animal,'moisture': moisture, 'temperature' : temperature, 'Humid': Humid }
        #print data
        def myOnPublishCallback():
            print ("Published Soil Moisture = %s %%" %moisture,"Temperature = %s C" % temperature, "Humidity = %s %%" % Humid,'animal = %s'%animal, "to IBM Watson")
            if animal>0.98:
                print("Alert")
        success = deviceCli.publishEvent("IoTSensor", "json", data, qos=0, on_publish=myOnPublishCallback)
        if not success:
            print("Not connected to IoTF")
        time.sleep(10)
        
        deviceCli.commandCallback = myCommandCallback

# Disconnect the device and application from the cloud
deviceCli.disconnect()
