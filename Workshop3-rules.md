# PART 3: AWS IoT Rules

### 1. Create an IAM Role for use when executing your IoT Rule Actions
   - Select **Services/IAM** and select **Roles**
   - Press **Create role** button


   ![Create Role](images/create-role.png)
   - Select **AWS service** as the trusted entity
   - Select **IoT** as the service using the Role
   - Select **IoT** as your use case
   - Press **Next: Permissions** button


   ![Create Role](images/create-role-service.png)
   - Press **Next: Tags** button


   ![Create Role](images/create-role-policy.png)
   - Press **Next: Review** button


   ![Create Role](images/create-role-tags.png)
   - Enter a **Role name** = **IoTActionRole**
   - Press **Create role** button


   ![Create Role](images/create-role-review.png)
   - Enter **IoTActionRole** under search
   - Select **IotActionRole**


   ![Create Role](images/find-role.png)
   - Select **Add inline policy**


   ![Create Role](images/add-inline-policy.png)
   - Select **JSON** tab
   - Enter the following JSON
<pre>
{
  "Version": "2012-10-17",
  "Statement": {
    "Effect": "Allow",
    "Action": "iot:Publish",
    "Resource": "*"
  }
}
</pre>
  **NOTE: In a production environment you would typically use a more constrained Role**

   - Press **Review policy** button


   ![Create Role](images/save-policy.png)
   - Enter **Name** = **iot-publish**
   - Press **Create policy** button


   ![Create Role](images/create-action-policy.png)

### 2. Create an IoT rule to flash the red LED when the button sensor is pressed
   - Select **Services/IoT Core**, select **Act**, select **Rules**
   - Press **Create a rule** button


   ![Create Rule](images/create-rule.png)
   - Enter **Name = Flash**
   - Enter **Rule query statement = SELECT * FROM 'mything/button'**
   - Press **Add Action** button


   ![Create Rule](images/add-flash-action.png)
   - Select **Republish a message to an AWS IoT topic**
   - Press **Configure action** button


   ![Create Basic Rule](images/select-republish.png)
   - Enter **Topic = mything/flash**
   - Select **Quality of Service = 1**
   - Press **Select** to choose an existing role


   ![Configure Action](images/config-flash-action1.png)
   - Press **Select** next to the role named **IoTActionRole**
   

   ![Configure Action](images/config-flash-action2.png)
   - Press **Add action** button


   ![Configure Action](images/config-flash-action3.png)
   - Press **Create Rule** button


   ![Configure Action](images/create-flash-rule.png)
   - Once the rule indicates it's enabled you may test it (refresh your browser until it indicates "enabled").
   - Test the rule by pressing the button sensor on the Raspberry Pi. The red LED will flash for 1 second each time you press the button.

### 3. Create an advanced IoT rule to beep when the angle sensor is > 80
   - Select **Services/IoT Core**, select **Act**, select **Rules**
   - Press **Create** button


   ![Create Rule](images/create-rule2.png)
   - Enter **Name = Beep**
   - Enter **Rule query statement = SELECT * FROM 'mything/angle' WHERE value > 80**
   - Press **Add Action** button


   ![Create Rule](images/add-beep-action.png)
   - Select **Republish a message to an AWS IoT topic**
   - Press **Configure action** button


   ![Create Basic Rule](images/select-republish.png)
   - Enter **Topic = mything/beep**
   - Select **Quality of Service = 1**
   - Press **Select** to choose an existing role


   ![Configure Action](images/config-beep-action1.png)
   - Press **Select** next to the role named **IoTActionRole**


   ![Configure Action](images/config-beep-action2.png)
   - Press **Add action** button


   ![Configure Action](images/config-beep-action3.png)
   - Press **Create Rule** button


   ![Configure Action](images/create-beep-rule.png)
   - Once the rule indicates it's enabled you may test it (refresh your browser until it indicates "enabled").
   - Test the rule by turning the angle sensor. When the angle sensor is > 80 the buzzer will beep for 1 second.

### 4. Create advanced IoT rule that places a phone call when angle sensor is > 95

   **PART 1:** Create a Lambda function which will act as the rule's action

   - Launch the **Google Chrome Secure Shell App**
   - Enter a username of **pi**, the IP address displayed on the LCD screen connected to your Raspberry Pi and press the **[ENTER] Connect** button. Note that the password to use when logging in is written on the brown box of your Raspberry Pi.


   ![SSH to Raspberry Pi](images/ssh-to-raspberry-pi.png)

   pi@raspberrypi:~ $ **cd ~/Development**<br>
   pi@raspberrypi:~ $ **mkdir iot-workshop-lambda**<br>
   pi@raspberrypi:~ $ **cd iot-workshop-lambda**<br>
   pi@raspberrypi:~ $ **npm init** *(when prompted press enter to select the default value)*<br>
   pi@raspberrypi:~ $ **npm -s install twilio**<br>


   ![Nano Edit](images/edit-lambda1.png)
   - Use your favorite text editor to create a file named **index.js**.


   pi@raspberrypi:~ $ **nano index.js**<br>


   ![Nano Edit](images/nano-edit4.png)
   - Copy and paste the code below into it


   **HINT: Right mouse button may be used to copy and paste when using Google Chrome SSH**

<pre>
const Twilio = require('twilio');

const accountSid   = process.env.accountSid;
const authToken    = process.env.authToken;
const twilioAPI    = process.env.twilioAPI;
const callingDN    = process.env.callingDN;
const calledDN     = process.env.calledDN;
const twilioClient = Twilio(accountSid, authToken);

function makeCall(from, to) {
  return new Promise((resolve, reject) => {
    twilioClient.calls.create({
      url: twilioAPI,
      from: from,
      to: to
    }).then(call => {
      resolve(call);
    }).catch(err => {
      reject(err);
    });
  });
}

exports.handler = async (event) => {
  try {
    await makeCall(callingDN, calledDN);
    return { statusCode: 200,
             body: 'Success' };
  }
  catch (err) {
    return { statusCode: 400,
             body: JSON.stringify(err) };
  }
};
</pre>

   - If using **nano** to edit the file then press **CTRL-X** to Exit and **Y** when prompted to save it
   - Once the file is saved, create a zip file of the Lambda function so we can deploy it to AWS<br><br>
   pi@raspberrypi:~ $ **zip -r makecall.zip index.js node_modules package.json package-lock.json**<br><br>
   - Launch the **Google Chrome Secure Shell App** in another tab to start a SFTP session
   - Enter a username of **pi**, the IP address displayed on the LCD screen connected to your Raspberry Pi and press the **SFTP** button.
   
   
   **NOTE: The password to use when logging in is written on the brown box of your Raspberry Pi. Also note that the IP address assigned to your Raspberry Pi may differ from the example shown in the screen capture below.**


   ![SFTP Raspberry Pi](images/sftp-raspberry-pi.png)

   nasftp ./ > **cd ~/Development/iot-workshop-lambda**<br>
   nasftp /home/pi/Development/iot-workshop-lambda/ > **get makecall.zip**<br>

   ![SFTP Raspberry Pi](images/sftp-raspberry-pi-get-file-makecall.png)

  - Save file in your Downloads directory of your laptop.


   ![SFTP Raspberry Pi](images/sftp-raspberry-pi-save-file-makecall.png)

  - Deploy Lambda Function using the AWS Console
  - Select **Services/Lambda**
  - Press **Create function** button


   ![Create Lambda](images/create-lambda1.png)

  - Select **Author from scratch**
  - Enter **Funcation name = MakeCall**
  - Select **Node.js 12.x** runtime
  - Press **Create function** button


  ![Enter Lambda Info](images/create-lambda2.png)

  - Select **Code entry type = Upload a .zip file**
  - Press the **Upload** button


  ![Upload ZIP](images/upload-makecall.png)

  - Select **Downloads/makecall.zip** file and press **Open** button


  ![Upload ZIP to Lambda](images/open-makecall.png)

  - Under Environment variables enter the following:
    - accountSid = **WORKSHOP-ORGANIZER-WILL-PROVIDE-THIS-VALUE**
    - authToken = **WORKSHOP-ORGANIZER-WILL-PROVIDE-THIS-VALUE**
    - twilioAPI = http://demo.twilio.com/docs/voice.xml
    - callingDN = **WORKSHOP-ORGANIZER-WILL-PROVIDE-THIS-VALUE**
    - calledDN = **YOUR-CELL-PHONE** *(e.g. +16138675309)*
  - Press the **Save** button at the top of the page


  ![Save Lambda](images/create-lambda3.png)
   - Select **Services/IoT Core**, select **Act**, select **Rules**
   - Press **Create** button


  ![Create Rule](images/create-makecall-rule.png)
   - Enter **Name = MakeCall**
   - Enter **Rule query statement = SELECT * FROM 'mything/angle' WHERE value > 95**
   - Press **Add Action** button


   ![Create Rule](images/create-rule3.png)
   - Select **Send a message to a Lambda function**
   - Press **Configure action** button


   ![Add Action](images/add-makecall-action.png)
   - Under **Function name** press **Select** button


   ![Configure Action](images/config-makecall-action1.png)
   - Press **Select** next to **MakeCall** Lambda function


   ![Configure Action](images/config-makecall-action2.png)
   - Press **Add action** button


   ![Configure Action](images/config-makecall-action3.png)
   - Press **Create rule** button
   
   
   ![Configure Action](images/create-makecall-rule-done.png)
  - Once the rule indicates it's enabled you may test it (refresh your browser until it indicates "enabled").
  - Test the rule by turning the angle sensor. When the angle sensor is > 95 your cell phone will ring and when answered will play a message.
