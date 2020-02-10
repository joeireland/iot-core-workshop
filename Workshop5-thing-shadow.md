# PART 5: Thing Shadows

### 1. SSH onto Raspberry PI using Chrome SSH Extension

   - Launch the **Google Chrome Secure Shell Extention**
   - Enter a username of **pi**, the IP address displayed on the LCD screen connected to your Raspberry Pi, enter port **80** and press the **[ENTER] Connect** button.

   
   NOTE: The password to use when logging in is written on the brown box of your Raspberry Pi. Also note that the IP address assigned to your Raspberry Pi may differ from the example shown in the screen capture below.


   ![SSH to Raspberry Pi](images/ssh-to-raspberry-pi.png)

### 2. Update your application to use a Thing Shadow

   pi@raspberrypi:~ $ **cd ~/Development/iot-workshop**<br>

   ![npm init](images/nano-edit.png)
   pi@raspberrypi:~ $ **nano index.js**<br>
   Copy and paste the following code into the nano edit session and save the file.<br>
   **IMPORTANT:** Replace the **host:** value with the Custom Endpoint value you took note of earlier<br>

<pre>
const AWSIoT  = require('aws-iot-device-sdk');
const GrovePi = require('node-grovepi').GrovePi;
const Board   = GrovePi.board;
const Sensors = GrovePi.sensors;

const red    = new Sensors.DigitalOutput(2);
const blue   = new Sensors.DigitalOutput(3);
const button = new Sensors.DigitalButton(7);
const buzzer = new Sensors.DigitalOutput(8);
const angle  = new Sensors.RotaryAnalog(0);

let device = null;
<b style="color:red">let ver    = 0;</b>

function main() {
  const board = new Board({
    debug: true,
    onError: onGroveError,
    onInit: onGroveInit
  });

  board.init();

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

function onConnect() {
  console.log('Connected to AWS IoT Core');
  device.subscribe('mything/flash');
  device.subscribe('mything/beep');
  device.subscribe('mything/blue/#');
  <b style="color:red">device.subscribe('$aws/things/mything/shadow/get/accepted');</b>
  <b style="color:red">device.subscribe('$aws/things/mything/shadow/update/delta');</b>
  device.publish('mything/messages', '{ "message": "ONLINE" }');
  <b style="color:red">device.publish('$aws/things/mything/shadow/get', '');</b>
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
  }<b style="color:red">
  else if (topic === '$aws/things/mything/shadow/get/accepted') {
    console.log('shadow/get/accepted');
    updateState(buffer);
  }
  else if (topic === '$aws/things/mything/shadow/update/delta') {
    console.log('shadow/update/delta');
    updateState(buffer);
  }</b>
}

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
  <b style="color:red">reportState('on');</b>
}

function blueOff() {
  console.log('Blue Off');
  blue.turnOff();
  <b style="color:red">reportState('off');</b>
}

function monitorButton() {
  console.log('Monitor Button ...');

  button.on('down', res => {
    console.log('Button: ' + JSON.stringify(res));
    device.publish('mything/button', '{ "message": "button", "value": "' + res + '" }');
  });

  button.watch();
}

function monitorAngle() {
  console.log('Monitor Angle ...');

  angle.on('change', res => {
    console.log('Angle: ' + JSON.stringify(res));
    device.publish('mything/angle', '{ "message": "angle", "value": ' + res + ' }');
  });

  angle.watch(1000);
}
<b style="color:red">
function updateState(buffer) {
  console.log('Update State');

  try {
    let message = JSON.parse(buffer.toString());

    if (ver < message.version) {
      ver = message.version;

      if (message.state.blue === 'on') {
        console.log('State: Blue=ON');
        blue.turnOn();
        reportState('on');
      }
      else {
        console.log('State: Blue=OFF');
        blue.turnOff();
        reportState('off');
      }
    }
  }
  catch (e) {
    console.log('ERROR: Failed to parse state buffer');
  }
}

function reportState(state) {
  device.publish('$aws/things/mything/shadow/update', '{ "state": { "reported": { "blue": "'+ state +'" } } }');
}
</b>
main();
</pre>

   - Start the application and connect your thing to AWS IoT Core<br>
   pi@raspberrypi:~ $ **cd ~/Development/iot-workshop**<br>
   pi@raspberrypi:~ $ **node index.js**


### 3. Test Connected Device Operations Using The Thing Shadow

**In one tab open the Thing Test page and publish Thing Shadow updates**
   - Login to AWS console
   - Go to **Services/IoT Core**
   - Select **Test**
   - Press **Publish to a topic** 
   - Specify a topic = **$aws/things/mything/shadow/update**
     and a body of the following
<pre>
  {
    "state": {
      "desired": {
        "blue": "on"
      }
    }
  }
</pre>
   - Press the **Publish to topic** button<br>
**Notice that the Raspberry Pi's blue LED turns on**


   ![MyThing Publish](images/publish-shadow-blue-on.png)

**In another tab open the Thing Shadow for your 'mything' device**
   - Login to AWS console
   - Go to **Services/IoT Core**
   - Select **Manage/Things**
   - Press **mything**
   - Select **Shadow**<br>
**Notice the "desired" and "reported" states show the blue LED are both "on"**


   ![MyThing Subscribe](images/thing-shadow-blue-on.png)

**Go back to the tab pane where you opened the Thing Test page to publish messages**
   - Specify a topic of **$aws/things/mything/shadow/update**
     and a body of the following
<pre>
  {
    "state": {
      "desired": {
        "blue": "off"
      }
    }
  }
</pre>
   - Press the **Publish to topic** button<br>
**Notice that the Raspberry Pi's blue LED turns off**


   ![MyThing Publish](images/publish-shadow-blue-off.png)

**Go back to the tab pane where you opened the Thing Shadow**

**Notice the "desired" and "reported" states show the blue LED are both "off"**


   ![MyThing Subscribe](images/thing-shadow-blue-off.png)

### 4. Test Disconnected Device Operations Using The Thing Shadow

**Go to the tab pane where you opened the SSH session to your Raspberry Pi**
   - Stop the running program by pressing Control-C (This will simulate a lost network connection)


   ![MyThing Subscribe](images/stop-program.png)

**Go back to the tab pane where you opened the Thing Test page to publish messages**
   - Specify a topic of **$aws/things/mything/shadow/update**
     and a body of the following
<pre>
  {
    "state": {
      "desired": {
        "blue": "on"
      }
    }
  }
</pre>
   - Press the **Publish to topic** button<br>
**Notice that the Raspberry Pi's blue LED remains off**

**Go back to the tab pane where you opened the Thing Shadow**

**Notice the "desired" state is "on" while the "reported" state indicates its last know state is "off"**


   ![MyThing Subscribe](images/thing-shadow-blue-on-off.png)

**Go to the tab pane where you opened the SSH session to your Raspberry Pi**
   - Start the program running again (This will simulate the network restablishing connectivity)


   ![MyThing Subscribe](images/start-program.png)
**Notice that the Raspberry Pi's blue LED turns on**

**Go back to the tab pane where you opened the Thing Shadow**

**Notice the "desired" and "reported" states show the blue LED are both "on"**


   ![MyThing Subscribe](images/thing-shadow-blue-on2.png)