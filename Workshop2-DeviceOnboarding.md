# PART 2: AWS IoT Device Registration

### 1. Register your device to AWS IoT Core
   - Login to AWS console and ensure your region is **US East (N. Virginia)**


   ![Select Region](images/select-region.png)

   - Go to **Services/IoT Core** by entering **IoT Core** in the Find Services field and pressing **Enter**


   ![IoT Core](images/select-iot-core.png)

   - Select **Manage/Things**
   - Press **Register a thing** button


   ![Register Thing](images/register-thing.png)

   - Press **Create a single thing** button
   

   ![Create Thing](images/create-thing.png)
   
   - Enter **Name=mything** and press **Next** button
   

   ![Create Thing](images/create-thing-next.png)

### 2. Create a certificate for your IoT thing
   - Press **Create certificate** button


   ![Create Certificate](images/create-certificate.png)

   - Download **A certificate for this thing** and save it as **Downloads/certs/cert.pem** on your laptop
   - Download **public key** and save it as **Downloads/certs/public.pem** on your laptop
   - Download **private key** and save it as **Downloads/certs/private.pem** on your laptop
   - Download **A root CA for AWS IoT** save it as **Downloads/certs/rootCA.pem** on your laptop


   ![Download Certificate](images/download-certs.png)
---
   ![Download Root Certificate - Part1](images/get-rootca1.png)
---
   ![Download Root Certificate - Part2](images/get-rootca2.png)
---
   - Close the CA Certificates tab pane once you've dowloaded your root CA cert file.
   - Press **Done** button *(make sure you've downloaded your certs as shown above before you do this)*


   ![Download Certificate](images/download-certs2.png)


### 3. Create an IoT Policy for your Thing
   - Select **Secure/Policies**
   - Press "Create a policy" button


   ![Create Policy](images/create-policy1.png)
   
   - Enter **Name=mything-policy**
   - Enter **Action=iot:\***
   - Enter **Resource ARN=\***
   - Select **Effect=Allow**
   - Press **Create** button<br>
     *NOTE: Typically you would make a more constrained policy in a production environment*


   ![Create Policy](images/create-policy2.png)
---
   ![Create Policy](images/create-policy-done.png)

### 4. Attach your IoT Policy to the device certificate
   - Select **Secure/Certificates**
   - Select **...** pulldown menu of your certificate, and select **Attach policy**


   ![Create Policy](images/attach-policy1.png)

   - select your policy **mything-policy**
   - press **Attach** button


   ![Create Policy](images/attach-policy2.png)
   ![Create Policy](images/attach-policy-done.png)

### 5. Attach your certificate to your Thing and activate the certificate
   - Select **Secure/Certificates**
   - Select **...** pulldown menu of your certificate, and select **Attach thing**


   ![Attach Thing](images/attach-thing1.png)

   - Select your thing **mything**
   - Press **Attach** button


   ![Attach Thing](images/attach-thing2.png)

   - Select **...** pulldown menu of your certificate, and select **Activate**


   ![Activate Certificate](images/activate-cert.png)

### 6. Locate your IoT Custom Endpoint value
   - Select **Settings** and take note of your **Custom Endpoint** (your value may differ from the example shown below).<br>
   

   **NOTE: Save this value in your notes for the future. It will be used when you configure the host which your IoT code will connect to below**


   ![Custom Endpoint](images/custom-endpoint.png)

### 7. Secure copy your previously saved **mything** IoT certificates onto your Raspberry Pi

   - Launch the **Google Chrome Secure Shell App** in another tab to start a SFTP session
   - Enter a username of **pi**, the IP address displayed on the LCD screen connected to your Raspberry Pi, enter port **80** and press the **SFTP** button.

   
   **NOTE: The password to use when logging in is written on the brown box of your Raspberry Pi. Also note that the IP address assigned to your Raspberry Pi may differ from the example shown in the screen capture below.**


   ![SFTP Raspberry Pi](images/sftp-raspberry-pi.png)

  nasftp ./ > **cd ~/Development/iot-workshop**<br>
  nasftp /home/pi/Development/iot-workshop/ > **mkdir certs**<br>
  nasftp /home/pi/Development/iot-workshop/ > **cd certs**<br>
  nasftp /home/pi/Development/iot-workshop/certs/ > **put**<br>

   ![SFTP Raspberry Pi](images/sftp-raspberry-pi-put-file.png)

  - Select your **mything** IoT certs that you previously stored in **Downloads/certs**


   ![SFTP Raspberry Pi](images/sftp-raspberry-pi-save-put.png)

### 8. SSH onto Raspberry PI using Chrome SSH App

   - Launch the **Google Chrome Secure Shell Extention** in another tab to start a SSH session
   - Enter a username of **pi**, the IP address displayed on the LCD screen connected to your Raspberry Pi, enter port **80** and press the **[ENTER] Connect** button.

   
   **NOTE: The password to use when logging in is written on the brown box of your Raspberry Pi. Also note that the IP address assigned to your Raspberry Pi may differ from the example shown in the screen capture below.**


   ![SSH to Raspberry Pi](images/ssh-to-raspberry-pi.png)

### 9. Update your application to connect to AWS IoT Core as your **mything** you previously created using the AWS Console

   pi@raspberrypi:~ $ **cd ~/Development/iot-workshop**<br>
   pi@raspberrypi:~ $ **npm install -s aws-iot-device-sdk**<br>

   ![npm init](images/aws-iot-device-sdk.png)
   - Use your favorite editor to modify the test program to connect to AWS IoT Core.<br><br>

   pi@raspberrypi:~ $ **nano index.js** *(press control-X to exit and press Y to save the file)*<br>
   

   ![npm init](images/nano-edit2.png)

   **IMPORTANT: Replace the host: value with the Custom Endpoint value you took note of earlier**<br>
   **HINT: When using nano use shift and arrow keys to select text and control-K to delete text**<br>
   **HINT: Right mouse button may be used to copy and paste when using Google Chrome SSH**

<pre>
<b style="color:red">const AWSIoT  = require('aws-iot-device-sdk');</b>
const GrovePi = require('node-grovepi').GrovePi;
const Board   = GrovePi.board;
const Sensors = GrovePi.sensors;

const red    = new Sensors.DigitalOutput(2);
const blue   = new Sensors.DigitalOutput(3);
const button = new Sensors.DigitalButton(7);
const buzzer = new Sensors.DigitalOutput(8);
const angle  = new Sensors.RotaryAnalog(0);

<b style="color:red">let device = null;</b>

function main() {
  const board = new Board({
    debug: true,
    onError: onGroveError,
    onInit: onGroveInit
  });

  board.init();
<b style="color:red">
  device = AWSIoT.device({
    keyPath:  './certs/private.pem',
    certPath: './certs/cert.pem',
    caPath:   './certs/rootCA.pem',
    clientId: 'mything',
    region:   'us-east-1',
    protocol: 'mqtts',
    // IMPORTANT: REPLACE THE HOST VALUE BELOW WITH YOUR CUSTOM ENDPOINT VALUE
    host:     'a26erfmc9c0soi-ats.iot.us-east-1.amazonaws.com',
    debug:    'true'
  });

  device.on('connect', onConnect);
  device.on('message', onMessage);</b>
}
<b style="color:red">
function onConnect() {
  console.log('Connected to AWS IoT Core');
  device.subscribe('mything/flash');
  device.subscribe('mything/beep');
  device.subscribe('mything/blue/#');
  device.publish('mything/messages', '{ "message": "ONLINE" }');
}

function onMessage(topic, buffer) {
  if (topic === 'mything/flash') {
    flash();
  }
  else if (topic === 'mything/beep') {
    beep();
  }
  else if (topic === 'mything/blue/on') {
    blueOn();
  }
  else if (topic === 'mything/blue/off') {
    blueOff();
  }
}
</b>
function onGroveError(err) {
  console.log('ERROR: ' + JSON.stringify(err));
}

function onGroveInit(res) {
  console.log('GrovePi initialized');

  flash();
  monitorButton();
  monitorAngle();
}

function flash() {
  console.log('Flash');

  red.turnOn();
  setTimeout(() => { red.turnOff(); }, 1000);
}

function beep() {
  console.log('Beep');

  buzzer.turnOn();
  setTimeout(() => { buzzer.turnOff(); }, 1000);
}

function blueOn() {
  console.log('Blue On');
  blue.turnOn();
}

function blueOff() {
  console.log('Blue Off');
  blue.turnOff();
}

function monitorButton() {
  console.log('Monitor Button ...');

  button.on('down', res => {
    console.log('Button: ' + JSON.stringify(res));
    <b style="color:red">device.publish('mything/button', '{ "message": "button", "value": "' + res + '" }');</b>
  });

  button.watch();
}

function monitorAngle() {
  console.log('Monitor Angle ...');

  angle.on('change', res => {
    console.log('Angle: ' + JSON.stringify(res));
    <b style="color:red">device.publish('mything/angle', '{ "message": "angle", "value": ' + res + ' }');</b>
  });

  angle.watch(1000);
}

main();
</pre>

### 10. Show connectivity to AWS IoT Core and test your **mything**
   - Login to AWS console
   - Go to **Services/IoT Core**
   - Select **Test**
   - Subscribe **topic="mything/#** and press **Subscribe to topic** button


   ![MyThing Subscribe](images/mything-subscribe.png)

   - Once subscribed, you'll see a message "Hello from AWS IoT console"


   ![MyThing Subscribed](images/mything-subscribed.png)

### 11. Connect your Raspberry Pi as **mything** to AWS IoT Core

   - Go back to your tab where you're SSH-ed onto your Raspberry Pi and run the program<br>
   pi@raspberrypi:~ $ **cd ~/Development/iot-workshop**<br>
   pi@raspberrypi:~ $ **node index.js**<br>


   ![Connect to AWS IoT Core](images/connect-to-AWS.png)

   - The application indicates "Connected to AWS IoT Core"
   - The IoT Test page shows an incoming **ONLINE** message from the Raspberry Pi


   ![Connect to AWS IoT Core](images/connected-publications.png)

   - Turn angle sensor on Raspberry Pi and see incoming **Angle** messages


   ![Angle Tests SSH](images/angle-test-ssh.png)
   ![Angle Tests Publications](images/angle-test-pubs.png)

   - Press the button sensor and see incoming button press messages (**singlepress** & **longpress**)


   ![Button Tests SSH](images/button-test-ssh.png)
   ![Button Tests Publications](images/button-test-pubs.png)

   - Publish a message to flash red LED on the Raspberry Pi
   - Enter a Publish **topic** of **mything/flash**
   - Press **Publish to topic** button


   ![MyThing Publish](images/publish-flash.png)

   - Publish a message to beep the buzzer on the Raspberry Pi
   - Enter a Publish **topic** of **mything/beep**
   - Press **Publish to topic** button


   ![MyThing Publish](images/publish-beep.png)

   - Publish a message to turn on the blue LED attached to the Raspberry Pi


   ![MyThing Publish](images/publish-blue-on.png)

   - Publish a message to turn off the blue LED attached to the Raspberry Pi


   ![MyThing Publish](images/publish-blue-off.png)
