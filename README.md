
![Mobile Jazz Motis](https://raw.githubusercontent.com/mobilejazz/metadata/master/images/banners/mobile-jazz-bluefish-ios.jpg)
# ![Mobile Jazz Badge](https://raw.githubusercontent.com/mobilejazz/metadata/master/images/icons/mj-40x40.png) BlueFish

[![CI Status](http://img.shields.io/travis/Paolo Tagliani/MJBluetoothManager.svg?style=flat)](https://travis-ci.org/Paolo Tagliani/BlueFish)
[![Version](https://img.shields.io/cocoapods/v/BlueFish.svg?style=flat)](http://cocoapods.org/pods/BlueFish)
[![License](https://img.shields.io/cocoapods/l/BlueFish.svg?style=flat)](http://cocoapods.org/pods/BlueFish)
[![Platform](https://img.shields.io/cocoapods/p/BlueFish.svg?style=flat)](http://cocoapods.org/pods/BlueFish)



> CoreBluetooth with block-based APIs

BlueFish is a wrapper around CoreBluetooth concepts like CBCentralManager and CBPeripheral. All the delegate based API are substituted with blocks.

## Installation

BlueFish is available through [CocoaPods](http://cocoapods.org). To install
it, simply add the following line to your Podfile:

```ruby
pod "BlueFish"
```

## Basic Usage
The main objects to interact with are **BFCentralManager** and **BFPeripheral**.

### BFCentralManager

This object is responsible for:

- Scanning for nearby bluetooth devices (with filters)
- Connecting/disconnecting to bluetooth peripheral
- Retrieve bluetooth peripherals from Core Bluetooth cache

Example of use:

```objective-c

NSString *deviceID = @"hdl83h6sd-gl95-bn4f-37gd-jd73hd0tn8za";
BFCentralManager *manager = [[BFCentralManager alloc] init];
BFPeripheral *peripheral = [manager retrievePeripheralWithID:deviceID];
if (peripheral)
{
    //Peripheral found in cache, connect
    [manager connectToPeripheral:peripheral completionBlock:^(NSError *error) {
        //TODO: Manage error or do operation on peripheral
    }];
    return;
}

//If not in cache, search for it in the nearby area
[manager startScanningWithUpdateBlock:^(BFPeripheral *peripheral, NSError *error) {
    if ([peripheral.identifier isEqualToString:deviceID])
    {
        //Stop Scan
        [manager stopScanning];

        //Connect to peripheral
        [manager connectToPeripheral:peripheral completionBlock:^(NSError *error) {
            //TODO: Manage error or do operation on peripheral
        }];
    }
}];
```

### BFPeripheral

This object represents a peripheral, and is responsible for:

- Discovering services and characteristics
- Subscription to notification
- Read/write from/to characteristics
- Notification handling

#### Service and characteristic discovery

After connection, a peripheral does not hold information about characteristics and discovery. To make it ready to use the method `- (void)setupPeripheralForUse:(void (^)(NSError *error))completionBlock` must be called.

```objective-c
//Global discovery
[peripheral setupPeripheralForUse:^(NSError *error) {
    NSLog(@"Peripheral services: %@", peripheral.services.description);
    NSLog(@"Peripheral characteristics: %@", peripheral.characteristics.description);
}];

//Alternative version
[peripheral listServices:^(NSArray<CBService *> *services, NSError *error) {
    NSLog(@"Peripheral services: %@", peripheral.services.description);

    [peripheral listCharacteristics:^(NSError *error) {
        NSLog(@"Peripheral characteristics: %@", peripheral.characteristics.description);
    }];
}];
```
#### Read and write characteristics

```objective-c
NSString *characteristicID = @"sd2343h6sd-gl95-bn4f-37gd-jd73hd0tn8za";
NSData *data = [@"Mobile Jazz" dataUsingEncoding:NSUTF8StringEncoding];

[peripheral writeCharacteristic:characteristicID data:data completionBlock:^(NSError *error) {
    //Handle error if needed ....
}];

[peripheral readCharacteristic:characteristicID completionBlock:^(NSData *data, NSError *error) {
    //Handle error if needed ....
    NSString *string = [[NSString alloc] initWithData:data encoding:NSUTF8StringEncoding];
    NSLog(@"%@", string); //Mobile Jazz
}];
```

#### Notifications

We can subscribe to a peripheral notification on a characteristic value change. When the value changes, the peripheral will notify its notificationDelegate.

```objective-c
NSString *characteristicID = @"sd2343h6sd-gl95-bn4f-37gd-jd73hd0tn8za";
peripheral.notificationDelegate = self;

[peripheral subscribeCharacteristicNotification:characteristicID completionBlock:^(NSError *error) {
    //Handle error if needed ....
}];

#pragma mark - BFNotificationDelegate

- (void)didNotifyValue:(NSData *)value forCharacteristicID:(NSString *)characteristicID
{
    NSLog(@"Received: %@ from characteristic:%@", [value description], characteristicID);
}
```
## Project Maintainer

This open source project is maintained by [Paolo Tagliani](https://github.com/pablosproject).

## TODO

- [] Create example application

## License

    Copyright 2016 Mobile Jazz

    Licensed under the Apache License, Version 2.0 (the "License");
    you may not use this file except in compliance with the License.
    You may obtain a copy of the License at

      http://www.apache.org/licenses/LICENSE-2.0

    Unless required by applicable law or agreed to in writing, software
    distributed under the License is distributed on an "AS IS" BASIS,
    WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
    See the License for the specific language governing permissions and
    limitations under the License.
