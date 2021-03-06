#Hive Active Heating. API.

[![npm package](https://nodei.co/npm/bg-hive-api.png?downloads=true&downloadRank=false&stars=true)](https://nodei.co/npm/bg-hive-api/)

This node.js module provides a wrapper around the REST API provided by British Gas to control your [Hive home heating system](http://www.hivehome.com).

### Note
This software is in **not endorsed by British Gas** and is therefore subject to change at any time. **Use at your own risk**.

##Installation

```
npm install bg-hive-api
```

## Contents

* [Examples](#Examples)
  *  [`Connecting`](#Connecting)
* [Event Handling](#Events)
* [Reference](#Reference)
  * [`Hive`](#Ref-Hive)
  * [`ClimateControl`](#Ref-Heating)
  * [`HotWaterControl`](#Ref-HotWater)
  * [`Temperature`](#Ref-Temp)

<a name="Examples" />
## Examples

<a name="Connecting" />
### Connecting
Before we can anything we first have to authenticate with the remote server and create an active session.

First authenticate by instantiating the 'Hive' object with your login credentials, the same user name and password you use to login to the Hive website and App, then call the Login() method.

If you are logged in successfully the login event handler is called with the session context object. You are now able to control your heating system.

When you're finished call the Logout() method to close the session. The logout event handler will be called whenever the session is closed.

The following program demonstrates these steps by establishing an connection and then immediately logging out again if a successful connection was established. If you don't see *connected* displayed in the console then a connection could not be established, either because your login credentials were wrong or a connection to the remote server could not be established.

```javascript
var Hive = require('bg-hive-api');
var hive = new Hive("<<your login>>", "<<your password>>");

// on successful login this event handler is called
hive.on('login', function(context){
    console.log('Connected');
    hive.Logout();
});

// on logout call this event handler
hive.on('logout', function(){
   console.log('Connection Closed');
});

//Log in
hive.Login();

```
### ClimateController

```javascript
var ClimateControl = require('bg-hive-api/climateControl');
```

Use the ClimateControl object to set and read the current state of your heating. The following program will return the current state of the heating system.

```javascript
var Hive = require('bg-hive-api');
var ClimateControl = require('bg-hive-api/climateControl');
var hive = new Hive("<<your login>>", "<<your password>>");

// on successful login this event handler is called
hive.on('login', function(context){

    // Create an instance of the climate controller
    var climate = new ClimateControl(context);

    // Handle the on complete event.
    climate.on('complete', function(response){
        // write the response state object to the console.
        console.log(response);
        // log out
        hive.Logout();
    });

    climate.GetState();
});

// on logout call this event handler
hive.on('logout', function(){
   console.log('Connection Closed');
});

//Log in
hive.Login();

```

The following *login* event handler will set the temperature to a constant 19 degrees C before closing the session on successful response.

```javascript
// on successful login this event handler is called
hive.on('login', function(context){

    // Create an instance of the climate controller
    var climate = new ClimateControl(context);

    // Handle the on complete event.
    climate.on('complete', function(response){
        // write the response state object to the console.
        console.log(response);
        // log out
        hive.Logout();
    });

    // Set the heating state to Manual
    climate.SetState(climate.Mode.Manual);

    climate.once('accepted', function(response){
        climate.GetState();;
    });

    // Set the temperature to 19 C
    climate.TargetTemperature(19);
});
```
If successful you will see the response output in the console window.

```
{ devices: { '00-AA-BB-CC-DD-EE-FF-11': 'Your Receiver' },
  deviceAvailable: true,
  battery: 'OK',
  mode: 'HEAT',
  control: 'MANUAL',
  on: true,
  isSchedule: false,
  presenceStatus: 'HOME',
  currentTemperature: 19,
  targetTemperature: 19,
  shadowTemperature: 19,
  ...
Connection Closed
```
### HotWaterController

```javascript
var HotWaterControl = require('bg-hive-api/hotwaterControl');
```
The HotWaterControl object is used to set and request the current state of the hot water if your system supports it.

The following event handler sets the state of the hot water to *Scheduled*.

```javascript
// on successful login this event handler is called
hive.on('login', function(context){

    // Create an instance of the hot water controller
    var water = new HotWaterControl(context);

    // Handle the on complete event.
    water.on('complete', function(response){
        // write the response state object to the console.
        console.log(response);
        // log out
        hive.Logout();
    });

    water.once('accepted', function(response){
        water.GetState();;
    });

    // Set the hot water to scheduled
    water.SetState(water.Mode.Schedule);
});
```
If successful you will see the response in the console.

```
{ current: 'SCHEDULE',
  temperature: '200.0',
  temperatureUnit: 'C',
  available: [ 'SCHEDULE', 'MANUAL', 'BOOST', 'OFF' ] }
Connection Closed
```
### Temperature History

```javascript
var Temperature = require('bg-hive-api/temperature');
```
Use the Temperature object to get temperature data recorded by the thermostat over a defined period. In the following example the current days temperature history data is requested.

```javascript
// on successful login this event handler is called
hive.on('login', function(context){

    // Create an instance of the temperature history controller
    var temp = new Temperature(context);

    // Handle the on complete event.
    temp.on('complete', function(response){
        // write the response state object to the console.
        console.log(response);
        // log out
        hive.Logout();
    });

    // Get today's temperature history
    temp.GetState(temp.Period.Day);
});
```
A successful request will display the temperature data in the console.

```
{ period: 'today',
  data:
   [ { date: '2015-01-10 0:00', temperature: 20.29 },
     { date: '2015-01-10 0:30', temperature: 20.08 },
     { date: '2015-01-10 1:00', temperature: 19.65 },
     ...
     { date: '2015-01-10 23:00', temperature: 19.31 },
     { date: '2015-01-10 23:30', temperature: '--' } ],
  temperatureUnit: 'C' }
Connection Closed
```

<a name="Events" />
## Events

### Success events

* complete - Returns response object with the product of the request.
* accepted - Called once a `set` request has changed system state successfully.

### Error events

**Authentication Errors**

* not_authorised - Incorrect user name or password. Returns :- `error`
* locked - Account was locked after 5 failed log in attempts. Returns :- `error`
* invalid - Invalid login attempt. Returns :- `error`
* session_timout - The current session has expired. Typically after 20 minutes.

**General Errors**

* not_available - The requested action is not available or not supported.
* invalid - The requested action was invalid, typically thrown after passing a bad parameter value.
* service_unavailable - Remote service is not available, possibly caused by too many consecutive requests.
* error - Unknown error condition. Return the error object.

### Event Handling

It's generally easier and clearer to create separate handler functions to handle events for each controller object as in the following pattern.

```javascript
...
function HeatingEventHandler(controller) {
    if (controller != undefined)
    {
        controller.on('update', function(data){
            console.log(data);
        });

        controller.on('accepted', function(){
            console.log('OK');
        });

        controller.on('error', function(response){
            console.log(response);
        });

        controller.on('complete', function(response){
            console.log(response);
        });
    }
}

function HotWaterEventHandler(controller) {
    if (controller != undefined)
    {
        controller.on('update', function(data){
             console.log(data);
        });

        controller.on('accepted', function(){
            console.log('OK')
        });

        controller.on('error', function(response){
            console.log(response);
        });

        controller.on('complete', function(response){
            console.log(response);
        });
    }
}

...
hive.on('login', function(context){
    var climate = new HotWaterControl(context);
    var hotwater = new HotWaterControl(context);

    HotWaterEventHandler(hotwater);
    HeatingEventHandler(climate);

    hive.Logout();
});

...

```

<a name="Reference" />
## Reference

<a name="Ref-Hive" />
### Hive(username, password, api)

```javascript
var Hive = require('bg-hive-api');
var hive = new Hive(username, password, api);
```

**Parameters**

*username* -
This will be the same name you use to login into the hive website.

*password* -
Corresponding password you use to authenticate.

*api [optional]* - String value can be either 'Hive' or 'AlertMe'. Defaults to 'HIve'. This parameter specifies the url of the backend rest api.

### Methods

#### Login()

Opens an http connection and authenticates with the remote server.

**Parameters**

*none*

**Events**

* login - On Success. Returns :- Connection `context` object.

#### Logout()

Clears all pending tasks from the command queue, and closes the http connection.

**Parameters**

*none*

**Events**

* logout - On Success.

<a name="Ref-Heating" />
### ClimateControl(`context`)

```javascript
var ClimateControl = require('bg-hive-api/climateControl');
...
hive.on('login', function(context){
    var climate = new ClimateControl(context);
});

```

### Methods

#### GetState()

Return the current state of the heating system.

**Parameters**

*none*

**Events**

* complete - On Complete. Returns :- `response` object.

#### SetState(`Mode`)

Set the current state of the heating system.

**Parameters**

* `ClimateControl.Mode.Off` - Frost protection.
* `ClimateControl.Mode.Manual` - Maintain the current target temperature.
* `ClimateControl.Mode.Schedule` - On scheduled timer.
* `ClimateControl.Mode.Override` - Maintain target temperature until next scheduled event.

**Events**

* accepted - New state has been set.

#### TargetTemperature(`temperature`)

Set the desired target temperature.

**Parameters**

* `temperature` - Numeric temperature value in C

**Events**

* accepted - New temperature has been set.

#### GetSchedule()

Request the programmed schedule.

**Parameters**

*none*

**Events**

* complete - On Complete. Returns :- `response` object.

<a name="Ref-HotWater" />
### HotWaterControl(`context`)

```javascript
var HotWaterControl = require('bg-hive-api/hotWaterControl');
...
Hive.on('login', function(context){
    var hotwater = new HotWaterControl(context);
});
```

### Methods

#### GetState()

Return the current state of the hot water system.

**Parameters**

*none*

**Events**

* complete - On Complete. Returns :- `response` object.

#### SetState(`Mode`)

Set the current state of the heating system.

**Parameters**

* `HotWaterControl.Mode.Schedule` - On pre-programmed scheduled timer.
* `HotWaterControl.Mode.Boost` - Turn on hot water for one hour.

**Events**

* accepted - New state has been set.


#### GetSchedule()

Request the programmed schedule.

**Parameters**

*none*

**Events**

* complete - On Complete. Returns :- `response` object.

<a name="Ref-Temp" />
### Temperature(`context`)

Temperature history.

```javascript
var Temperature = require('bg-hive-api/temperature');
...
hive.on('login', function(context){
    var temperature = new Temperature(context);
});

```

### Methods

#### GetState(`period`)

Get the temperature history recorded by the thermostat over a defined period.

**Parameters**

`Temperature.Period.Hour`, `Temperature.Period.Day`, `Temperature.Period.Week`, `Temperature.Period.Month`, `Temperature.Period.Year`
