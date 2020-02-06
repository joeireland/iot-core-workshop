# PART 6: Greengrass (Machine Learning At The Edge)

### 1. SSH onto Raspberry PI using Chrome SSH Extension
   - Launch the **Google Chrome Secure Shell Extention** in another tab to start a SSH session
   - Enter a username of **pi**, the IP address displayed on the LCD screen connected to your Raspberry Pi, enter port **80** and press the **[ENTER] Connect** button.

   
   NOTE: The password to use when logging in is written on the brown box of your Raspberry Pi. Also note that the IP address assigned to your Raspberry Pi may differ from the example shown in the screen capture below.


   ![SSH to Raspberry Pi](images/ssh-to-raspberry-pi.png)

### 2. Install MXNet Framework
   - REFERENCE: https://docs.aws.amazon.com/greengrass/latest/developerguide/ml-console.html#install-mxnet
   pi@raspberrypi:~ $ **mkdir ~/Development/iot-workshop-greengrass**<br>
   pi@raspberrypi:~ $ **cd ~/Development/iot-workshop-greengrass**<br>
   pi@raspberrypi:~ $ **wget https://d1onfpft10uf5o.cloudfront.net/greengrass-ml-installers/mxnet/ggc-mxnet-v1.2.1-python-raspi.tar.gz**<br>
   pi@raspberrypi:~ $ **tar -xzf ggc-mxnet-v1.2.1-python-raspi.tar.gz**<br>
   pi@raspberrypi:~ $ **cd ggc-mxnet-v1.2.1-python-raspi/**<br>
   pi@raspberrypi:~ $ **./mxnet_installer.sh**<br>
   pi@raspberrypi:~ $ **cp greengrassObjectClassification.zip ..**<br>


   ![Install MXNet](images/install-mxnet.png)

### 3. Create an MXNet Model Package
   - REFERENCE: https://docs.aws.amazon.com/greengrass/latest/developerguide/ml-console.html#package-ml-model
   pi@raspberrypi:~ $ **cd ~/Development/iot-workshop-greengrass**<br>
   pi@raspberrypi:~ $ **wget https://s3.amazonaws.com/model-server/model_archive_1.0/examples/squeezenet_v1.1/squeezenet_v1.1-0000.params**<br>
   pi@raspberrypi:~ $ **https://s3.amazonaws.com/model-server/model_archive_1.0/examples/squeezenet_v1.1/squeezenet_v1.1-symbol.json**<br>
   pi@raspberrypi:~ $ **wget https://s3.amazonaws.com/model-server/model_archive_1.0/examples/squeezenet_v1.1/synset.txt**<br>
   pi@raspberrypi:~ $ **zip squeezenet.zip squeezenet_v1.1-0000.params squeezenet_v1.1-symbol.json synset.txt**<br>


   ![Create MXNet Model](images/create-mxnet-model.png)

### 4. Secure copy your previously saved MXNet model to your PC

   - Launch the **Google Chrome Secure Shell Extention** in another tab to start a SFTP session
   - Enter a username of **pi**, the IP address displayed on the LCD screen connected to your Raspberry Pi, enter port **80** and press the **SFTP** button.

   
   NOTE: The password to use when logging in is written on the brown box of your Raspberry Pi. Also note that the IP address assigned to your Raspberry Pi may differ from the example shown in the screen capture below.


   ![SFTP Raspberry Pi](images/sftp-raspberry-pi.png)

   pi@raspberrypi:~ $ **cd ~/Development/iot-workshop-greengrass**<br>
   pi@raspberrypi:~ $ **get greengrassObjectClassification.zip**<br>
   pi@raspberrypi:~ $ **get squeezenet.zip**<br>

### 5. Installing the AWS IoT Greengrass Core Software onto your Raspberry Pi
   REFERENCE: https://docs.aws.amazon.com/greengrass/latest/developerguide/module2.html
   - Login to AWS console
   - Go to **Services/IoT Core** and select **Greengrass**
   - Select **Manage/Things**
   - Press **Register a thing** button


   (15 mins) Instructions taken from here Part 1 - https://docs.aws.amazon.com/greengrass/latest/developerguide/module2.html
   - Chose Armv7l,Raspbian,Linux
   (20 mins) Instructions taken from here Part 2 - https://docs.aws.amazon.com/greengrass/latest/developerguide/gg-device-start.html
   - sudo su; cd /greengrass/certs; sudo wget -O root.ca.pem https://www.amazontrust.com/repository/AmazonRootCA1.pem

6) (25 mins) Step 4: Create and Publish Lambda Function
   - Instructions from URL - https://docs.aws.amazon.com/greengrass/latest/developerguide/ml-console.html#ml-console-create-lambda

7) (10 mins) Step 5: Add Lambda Function to Group
   - Instructions from URL - https://docs.aws.amazon.com/greengrass/latest/developerguide/ml-console.html#ml-console-config-lambda

8) (30 mins) Step 6: Add Resources to the Greengrass Group
   - Instructions from URL - https://docs.aws.amazon.com/greengrass/latest/developerguide/ml-console.html#ml-console-add-resources

9) (10 mins) Step 7: Add a Subscription to the Greengrass Group
   - Instructions from URL - https://docs.aws.amazon.com/greengrass/latest/developerguide/ml-console.html#ml-console-add-subscription

10) (10 mins) Step 8: Deploy the Greengrass Group
   - Instructions from URL - https://docs.aws.amazon.com/greengrass/latest/developerguide/ml-console.html#ml-console-deploy-group

11) (10 mins) Step 9: Test the Inference App
   - Instructions from URL - https://docs.aws.amazon.com/greengrass/latest/developerguide/ml-console.html#ml-console-test-app
