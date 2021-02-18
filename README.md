English | [简体中文](./README_cn.md)

# Tuya Device IoT SDK Sample for Android

## Introduction

This repository contains the demo application of the Tuya App SDK for IoT device connection.

The SDK is based on the Tuya IoTOS and is tailored and customized to work on the Android platform. However, the API version of Android OS must be equal to or higher than 19, and your IoT devices can be connected to Tuya IoT.

## Demo usage

The demo provides functions to get activation code, activation, dp point test, status log display, etc.  
[install package experience](./app/release/app-release.apk).

>**Note**: Before starting, you need to configure the `pid`, `uuid`, and `authkey`.

**Three ways to configure:**  

1. Add the following to your `local.properties` file (use this way for compiling demo code by yourself, use the last two for direct APK installation):    

    ```java
    UUID=your uuid  
    AUTHKEY=your key  
    PID=your pid
    ```
2. Edit in the configuration popup window, as shown in the follwoing screenshot.
3. Generate the configuration QR code, scan the code when you enter the demo, as shown in the follwoing screenshot. 

	QR code generation method: Use a [QR generation tool](https://cli.im/text) and the following configuration json to generate a QR code.

    ```json
    {
        "PID": "Your PID",
        "UUID": "Your UUID",
        "AUTHKEY": "your AUTHKEY"
    }
    ```

	<img src="./screenshots/ss0.jpg" width = "30%" height = "20%" align=center /> <img src="./screenshots/ss1.jpg" width = "30%" height = "20%" align=center /> <img src="./screenshots/ss2.jpg" width = "30%" height = "20%" align=center />

## Getting started

[![Download](https://api.bintray.com/packages/tuyasmartai/sdk/tuyasmart-iot_sdk/images/download.svg)](https://bintray.com/tuyasmartai/sdk/tuyasmart-iot_sdk/)

* Dependencies

    ```groovy
    implementation 'com.tuya.smart:tuyasmart-iot_sdk:x.x.x' // use the latest version above
    implementation 'com.tencent.mars:mars-xlog:1.2.3'
    ```

    > Add the repository address to the project root build.gradle

    ```groovy
    maven { url 'https://dl.bintray.com/tuyasmartai/sdk' }
    ```

* Obfuscation  

    If obfuscation is enabled, add in the proguard-rules.pro file

    ```java
    -keep class com.tuya.smartai.iot_sdk.** {*;}
    -keep class com.tencent.mars.** {*;}
    ```

* Permission requirements

    ``` java
    <uses-permission android:name="android.permission.INTERNET" />
    <uses-permission android:name="android.permission.WRITE_EXTERNAL_STORAGE" />
    <uses-permission android:name="android.permission.READ_EXTERNAL_STORAGE" />
    ````

* Initialization

    ``` java
    IoTSDKManager ioTSDKManager = new IoTSDKManager(context);

    /**
        * Initialize the SDK (Note! A UUID cannot be activated on more than one device at the same time)
        * @param basePath storage path Example: "/sdcard/tuya_iot/"
        * @param productId The product ID.
        * @param uuid The user ID.
        * @param authKey The authentication key.
        * @param version The firmware version number (for OTA).
        * @param mCallback The SDK callback method.
        * @return
        */
    ioTSDKManager.initSDK(String basePath, String productId, String uuid, String authorKey, String version, IoTCallback mCallback);

    public interface IoTCallback {

            /**
                    * dp event receiving
                    * @param event
                    * 
                    * event value(event.value)
                    * event id(event.dpid)
                    * event type(event.type)
                    * DPEvent.Type.PROP_BOOL
                    * DPEvent.Type.PROP_VALUE
                    * Type.PROP_STR
                    * PROP_ENUM
                    * Type.PROP_RAW
                    PROP_RAW */
            void onDpEvent(DPEvent event);

            // unbind the device callback (please restart the APP process here, otherwise it will affect the secondary network allocation)
            void onReset();

            //receive the short link of the QR code of the wiring network (null in case of failure to get it)
            void onShorturl(String url);
            
            /**
            * MQTT status change
            * @param status IoTSDKManager.STATUS_OFFLINE Network offline; 
            * IoTSDKManager.STATUS_MQTT_OFFLINE network online MQTT offline; 
            * IoTSDKManager.STATUS_MQTT_ONLINE network online MQTT online
            */
            void onMQTTStatusChanged(int status);
            
            // Device activation
            void onActive();
            
            // First activation of the device
            void onFirstActive();
        
        /**
            * Receive MQTT messages
            * @param protocol protocol number
            * @param msgObj message
            */
        void onMqttMsg(int protocol, org.json.JSONObject msgObj)
            
        }
    ```

* Destroy  

    ```java
    // Destruction operations such as broadcast logout will be performed
    ioTSDKManager.destroy();
    ```

## Testing

It is recommended to turn on the logging service during the testing phase, the SDK logs will be automatically saved in the path you passed in, and you can send the log files to your fellow developers for debugging if you encounter problems.

> **Note**: It is recommended to remove the logging services during the production phase.

```java
/**
     * Enable local logging service
     * @param logPath path to save log files Example: "/sdcard/tuya_log/"
     * @param cacheDays Number of days to cache the log file
     * @return
     */
Log.init(context, logPath, cacheDays);

// Flush log file can be triggered manually when needed. isSync : true is synchronous flush, will return only after the flush is finished. false is asynchronous flush, return without waiting for flush to finish.
Log.flush(isSync)

// Destroy the local logging service, called when the activity ends or the application exits
Log.close();
```

## API

```java
// local unbinding (asynchronous operation, successful unbinding will enter onReset callback)
IoTSDKManager.reset();

/**
     * Send dp event
     * @param id The DP ID.
     * @param type The type of DPEvent.
     * Type.PROP_BOOL boolean
     * Type.PROP_VALUE int
     * Type.PROP_STR string
     * PROP_ENUM int
     PROP_RAW byte[] * DPEvent.
     * @param val The value.
     * @return
     */
IoTSDKManager.sendDP(int id, int type, Object val)

/**
     * Send multiple dp events
     *
     * @param events Multiple DP types
     * @return
     */
IoTSDKManager.sendDP(DPEvent... events)

/**
     * send dp event with timestamp
     *
     * @param id The DP ID.
     * @param type The type of DPEvent.
     * @param val The value.
     * @param timestamp The timestamp in seconds.
     * @return
     */
IoTSDKManager.sendDPWithTimeStamp(int id, int type, Object val, int timestamp)


/**
     * Send multiple dp events with timestamp (timestamp needs to be assigned in DPEvent.timestamp)
     *
     * @param events Multiple dp types
     * @return
     */
IoTSDKManager.sendDPWithTimeStamp(DPEvent... events)

/**
     * Send http request
     * @param apiName Make an API request.
     * @param apiVersion The API version number.
     * @param jsonMsg The parameter in json format.
     * @return
     */
IoTSDKManager.httpRequest(String apiName, String apiVersion, String jsonMsg)

/**
     * Register MQTT messages, called after initializing the SDK
     *
     * @param protocol The protocol number to be registered.
     * @return success: 0;fail: !0
     */
int registMqtt(int protocol)

/**
     * Send an MQTT message
     * @param protocol protocol number
     * @param msg message
     * @return success: 0;fail: !0
     */
int sendMqtt(int protocol, String msg) 

// Get the device id
IoTSDKManager.getDeviceId()

//get the server time
int IoTSDKManager.getUniTime()

/**
 * Pass in token directly to activate the device
 * @param token
 * @return success: 0;fail: !0
 */
int IoTSDKManager.userTokenBind(String token)

// custom implementation of network status monitoring, the return value is whether the network is offline. SDK has provided the default implementation, no need to extend this method if not needed.

ioTSDKManager = new IoTSDKManager(this) {

            @Override
            protected boolean isOffline() {
                // Implement your own network status monitoring
                return super.isOffline();
            }
        }
```

## OTA

>**Note**: the firmware version is specified according to `version` passed in by `ioTSDKManager.initSDK`. Please modify the `version` parameter in 3-digit version format when making a new firmware package, for example, `1.2.3`.

Support device-side detection upgrade and app trigger upgrade, after setting the following callback, upload the new version firmware in the background, and please compress the upgrade file to `.zip` format. After that, you will receive the update information callback, and then you can trigger `ioTSDKManager.startUpgradeDownload ` to start the upgrade download.

```java 
ioTSDKManager.setUpgradeCallback(new UpgradeEventCallback() {
            @Override
            public void onUpgradeInfo(String version) {
                //Receive update information Version: version
                
                //proactively trigger the upgrade file download (can be triggered after receiving the update callback)
                ioTSDKManager.startUpgradeDownload();
            }

            @Override
            public void onUpgradeDownloadStart() {
                //start upgrade download callback
            }

            @Override
            public void onUpgradeDownloadUpdate(int i) {
                //download progress callback i%
            }

            @Override
            public void upgradeFileDownloadFinished(int result, String file) {
                // download completed callback, result == 0 means success, file is the path to the upgrade file archive (recommended to clear after installation)
            }
        });
```

## Technical support

You can get support from Tuya Smart with the following methods:

* [Tuya Smart document center](https://developer.tuya.com/en/docs/iot)
* [Submit a ticket](https://service.console.tuya.com/)

## License

This project is licensed under the MIT License.
