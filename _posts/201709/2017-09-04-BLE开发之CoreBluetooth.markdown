---
layout:     post
title:      "BLE开发之CoreBluetooth"
subtitle:   ""
date:       2017-09-04 15:41:00
author:     "易博"
header-img: "img/201705/18/head_bg.JPG"
tags:
    - iOS
---

####  一、前言

BLE，全称蓝牙低能耗(Bluetooth Low Energy)技术是低成本、短距离、可互操作的鲁棒性无线技术，工作在免许可的2.4GHz ISM射频频段，隶属于蓝牙4.0规范。它从一开始就设计为超低功耗(ULP)无线技术。它利用许多智能手段最大限度地降低功耗。蓝牙低能耗技术采用可变连接时间间隔，这个间隔根据具体应用可以设置为几毫秒到几秒不等。另外，因为BLE技术采用非常快速的连接方式，因此平时可以处于“非连接”状态（节省能源），此时链路两端相互间只是知晓对方，只有在必要时才开启链路，然后在尽可能短的时间内关闭链路

iOS开发中常用的蓝牙框架也就下面这几个。**GameKit.framework**，一般用于游戏开发中iOS设备之间的连接；**MultipeerConnectivity.framework**，iOS7后推出，个人理解可以实现airdrop类似的功能；**ExternalAccessory.framework**，可用于基于MFI认证的设备连接；**CoreBluetooth.framework**，低功耗蓝牙，基于蓝牙4.0，开放度高。大部分场景下采用的都是CoreBluetooth.framework，比如现在常见的物联网设备

CoreBluetooth.framework框架结构如下图所示，基于CoreBluetooth.framework做蓝牙开发，我们首先需要了解几个概念

![](http://www.xttxqjfg.cn/img/201709/04/01001.png)

**中心设备(central)、外设(peripheral)**：这两个概念放在一起说明比较容易理解。比如当你用摩拜App去解锁自行车的时候，你的手机就是中心设备，摩拜车就是外设。如果是从mac利用airdrop传文件到手机，那么电脑就是中心设备，你的手机则又变成了外设。对应结构图中就有中心模式(CBCentralManager)和外设模式(CBPeripheralManager)两种

**服务(service)**：每个外设一般都会设计若干个服务，每个服务对应一个事件。比如摩拜车，就可能会有一个解锁的服务。每个服务都会有一个唯一的标识(UUID)用于区分。这个标识一般为16bit、32bit或者128bit

**特征(characteristic)**：每个服务下面可以有若干个特征，设备之间的通讯就是通过这些特征值来实现的。比如在解锁摩拜车的时候，可能会有一个车锁状态的特征，解锁或者锁车之后回去修改这个特征值。同样每个特征也会有一个唯一的标识(UUID)用于区分

**特征的属性**：对应特征的作用域，某些特征是只读、某些是可读可写、某些是广播等等

基于上面几个概念和结构图，在说明一下使用的流程

**1、中心模式**：创建对象->搜索外设->连接外设->扫描服务->获取特征->基于特征做数据交互->完成后断开连接

**2、外设模式**：创建对象->设置服务、特征、属性->发布广播->设置读写等委托方法

下面详细介绍这两种模式的使用方法。其中外设模式在iOS开发中一般使用频率不高，主要为中心模式

####  二、外设模式

**1、引入框架、定义标识**

```
#import <CoreBluetooth/CoreBluetooth.h>

#define notiyCharacteristicUUID         @"FFF1"
#define readwriteCharacteristicUUID     @"FFF2"
#define readCharacteristicUUID          @"FFE1"

#define ServiceUUID1                    @"FFF0"
#define ServiceUUID2                    @"FFE0"
#define LocalNameKey                    @"MyBlueDevice"
```

**2、创建外设模式管理，初始化并设置CBPeripheralManagerDelegate代理**

```
@property (nonatomic,strong) CBPeripheralManager *peripheralManager;

self.peripheralManager = [[CBPeripheralManager alloc] initWithDelegate:self queue:nil];

/*
蓝牙设备打开需要一定时间，打开成功后会进入委托方法
- (void)peripheralManagerDidUpdateState:(CBPeripheralManager *)peripheral;
模拟器永远也不会得CBPeripheralManagerStatePoweredOn状态，所以做测试的时候请使用真机
*/

```

**3、在回调中初始化服务、特征等**

```
#pragma mark CBPeripheralManagerDelegate
-(void)peripheralManagerDidUpdateState:(CBPeripheralManager *)peripheral
{
    switch (peripheral.state) {
        case CBManagerStatePoweredOn:
        {
            NSLog(@"设备已打开....");
            [self setupBlueInfo];
            break;
        }
        default:
            NSLog(@"异常状态:%ld",(long)peripheral.state);
            break;
    }
}

-(void)setupBlueInfo
{
    CBUUID *CBUUIDCharacteristicUserDescriptionStringUUID = [CBUUID UUIDWithString:CBUUIDCharacteristicUserDescriptionString];

    //可以通知的Characteristic
    CBMutableCharacteristic *notiyCharacteristic = [[CBMutableCharacteristic alloc]initWithType:[CBUUID UUIDWithString:notiyCharacteristicUUID] properties:CBCharacteristicPropertyNotify value:nil permissions:CBAttributePermissionsReadable];
    
    //可读写的characteristics
    self.readwriteCharacteristic = [[CBMutableCharacteristic alloc]initWithType:[CBUUID UUIDWithString:readwriteCharacteristicUUID] properties:CBCharacteristicPropertyWrite | CBCharacteristicPropertyRead value:nil permissions:CBAttributePermissionsReadable | CBAttributePermissionsWriteable];
    
    //设置description
    CBMutableDescriptor *readwriteCharacteristicDescription1 = [[CBMutableDescriptor alloc]initWithType: CBUUIDCharacteristicUserDescriptionStringUUID value:@"readwrite"];
    [self.readwriteCharacteristic setDescriptors:@[readwriteCharacteristicDescription1]];

    
    //只读的Characteristic
    //只有只读属性的特征才能设置初始值，其余设置nil即可，否则在初始化的时候会报错
    self.readCharacteristic = [[CBMutableCharacteristic alloc]initWithType:[CBUUID UUIDWithString:readCharacteristicUUID] properties:CBCharacteristicPropertyRead value:[@"readCharacteristic" dataUsingEncoding:NSUTF8StringEncoding] permissions:CBAttributePermissionsReadable];

    //service1初始化并加入两个characteristics
    CBMutableService *service1 = [[CBMutableService alloc]initWithType:[CBUUID UUIDWithString:ServiceUUID1] primary:YES];
    [service1 setCharacteristics:@[notiyCharacteristic,self.readwriteCharacteristic]];
    
    //service2初始化并加入一个characteristics
    CBMutableService *service2 = [[CBMutableService alloc]initWithType:[CBUUID UUIDWithString:ServiceUUID2] primary:YES];
    [service2 setCharacteristics:@[self.readCharacteristic]];
    
    //添加服务
    [self.peripheralManager addService:service1];
    [self.peripheralManager addService:service2];
    //添加后就会调用代理的- (void)peripheralManager:(CBPeripheralManager *)peripheral didAddService:(CBService *)service error:(NSError *)error
}

-(void)peripheralManager:(CBPeripheralManager *)peripheral didAddService:(CBService *)service error:(NSError *)error
{
    if (error == nil) {
        self.servicesNum++;
    }
    //因为我们添加了2个服务，所以想两次都添加完成后才去发送广播
    if (self.servicesNum == 2) {
        //添加完成之后发送广播
        [peripheral startAdvertising:@{CBAdvertisementDataServiceUUIDsKey : @[[CBUUID UUIDWithString:ServiceUUID1],[CBUUID UUIDWithString:ServiceUUID2]],
        CBAdvertisementDataLocalNameKey : LocalNameKey}];
        //发送广播调用完这个方法后会调用代理的
        //(void)peripheralManagerDidStartAdvertising:(CBPeripheralManager *)peripheral error:(NSError *)error
    }
}

-(void)peripheralManagerDidStartAdvertising:(CBPeripheralManager *)peripheral error:(NSError *)error
{
    NSLog(@"peripheralManagerDidStartAdvertising......");
}

```

**4、处理读写和订阅的委托操作**

```
//订阅characteristics
-(void)peripheralManager:(CBPeripheralManager *)peripheral central:(CBCentral *)central didSubscribeToCharacteristic:(CBCharacteristic *)characteristic{
    NSLog(@"订阅了 %@的数据",characteristic.UUID);
    [self sendMsg:characteristic];
}

//取消订阅characteristics
-(void)peripheralManager:(CBPeripheralManager *)peripheral central:(CBCentral *)central didUnsubscribeFromCharacteristic:(CBCharacteristic *)characteristic{
    NSLog(@"取消订阅 %@的数据",characteristic.UUID);
    [self sendMsg:characteristic];
}

//读characteristics请求
- (void)peripheralManager:(CBPeripheralManager *)peripheral didReceiveReadRequest:(CBATTRequest *)request{
    NSLog(@"didReceiveReadRequest");
    //判断是否有读数据的权限
    if (request.characteristic.properties & CBCharacteristicPropertyRead) {
        NSData *data = request.characteristic.value;
        [request setValue:data];
        //对请求作出成功响应
        [self.peripheralManager respondToRequest:request withResult:CBATTErrorSuccess];
    }else{
        [self.peripheralManager respondToRequest:request withResult:CBATTErrorWriteNotPermitted];
    }
}

//写characteristics请求
- (void)peripheralManager:(CBPeripheralManager *)peripheral didReceiveWriteRequests:(NSArray *)requests{
    NSLog(@"didReceiveWriteRequests");
    CBATTRequest *request = requests[0];
    //判断是否有写数据的权限
    if (request.characteristic.properties & CBCharacteristicPropertyWrite) {
        //需要转换成CBMutableCharacteristic对象才能进行写值
        CBMutableCharacteristic *c =(CBMutableCharacteristic *)request.characteristic;
        c.value = request.value;
        [self.peripheralManager respondToRequest:request withResult:CBATTErrorSuccess];
    }else{
        [self.peripheralManager respondToRequest:request withResult:CBATTErrorWriteNotPermitted];
    }
}

//给外设发送消息
- (void)sendMsg:(CBCharacteristic *)characteristic {
    [self.peripheralManager updateValue:[@"success" dataUsingEncoding:NSUTF8StringEncoding] forCharacteristic:(CBMutableCharacteristic *)characteristic onSubscribedCentrals:nil];
}
```

####  三、中心模式

**1、引入框架，创建中心模式管理，初始化并设置CBCentralManagerDelegate代理**

```
#import <CoreBluetooth/CoreBluetooth.h>

@property (nonatomic,strong) CBCentralManager *centerManager;

-(CBCentralManager *)centerManager
{
    if (!_centerManager) {
        _centerManager = [[CBCentralManager alloc]initWithDelegate:self queue:nil];
        //回调-(void)centralManagerDidUpdateState:(CBCentralManager *)central
    }
    return _centerManager;
}

#pragma mark CBCentralManagerDelegate
-(void)centralManagerDidUpdateState:(CBCentralManager *)central
{
    switch (central.state) {
        case CBManagerStatePoweredOn:
        {
            //开始扫描，传nil表示不过滤，扫描所有设备
            [self.centerManager scanForPeripheralsWithServices:nil options:nil];
            break;
        }
        default:
            NSLog(@"异常状态:%ld",(long)central.state);
            break;
    }
}

//停止扫描
[self.centralManager stopScan];

```

**2、扫描到设备后连接设备**

```
//扫描到设备后调用
-(void)centralManager:(CBCentralManager *)central didDiscoverPeripheral:(CBPeripheral *)peripheral advertisementData:(NSDictionary<NSString *,id> *)advertisementData RSSI:(NSNumber *)RSSI
{
    [self.deviceListArr addObject:peripheral];
    [self.tableView reloadData];
}

//点击cell开始连接设备
-(void)tableView:(UITableView *)tableView didSelectRowAtIndexPath:(NSIndexPath *)indexPath
{
    [tableView deselectRowAtIndexPath:indexPath animated:YES];

    [self.centerManager stopScan];

    CBPeripheral *peripheral = [self.deviceListArr objectAtIndex:indexPath.row];
    NSLog(@"+++++++++++++++:%@",peripheral.identifier.UUIDString);

    // 连接外设 传入你搜索到的目标外设对象
    [self.centerManager connectPeripheral:peripheral options:nil];

}

//连接成功
-(void)centralManager:(CBCentralManager *)central didConnectPeripheral:(CBPeripheral *)peripheral
{
    NSLog(@"连接成功...");
    PeripheralVC *peripheralVC = [[UIStoryboard storyboardWithName:@"Main" bundle:nil] instantiateViewControllerWithIdentifier:@"PeripheralVC"];
    peripheralVC.peripheral = peripheral;
    [self.navigationController pushViewController:peripheralVC animated:YES];
}

//连接失败
-(void)centralManager:(CBCentralManager *)central didFailToConnectPeripheral:(CBPeripheral *)peripheral error:(NSError *)error
{
    NSLog(@"连接失败...");
}
```

**3、扫描服务**

设置选择设备的CBPeripheralDelegate并开始扫描服务

```
self.peripheral.delegate = self;
//扫描所有服务
[self.peripheral discoverServices:nil];

#pragma mark CBPeripheralDelegate
-(void)peripheral:(CBPeripheral *)peripheral didDiscoverServices:(NSError *)error
{
    if (error) {
        NSLog(@"didDiscoverServices:%@",error.localizedDescription);
        return;
    }
    else
    {
        for (CBService *services in peripheral.services) {
            NSLog(@"service UUID:%@",services.UUID.UUIDString);
            [self.servicesList addObject:services];
        }
    }
    [self.tableview reloadData];
}

//点击cell之后查找对应服务的特征
-(void)tableView:(UITableView *)tableView didSelectRowAtIndexPath:(NSIndexPath *)indexPath
{
    [tableView deselectRowAtIndexPath:indexPath animated:YES];

    CBService *service = [self.servicesList objectAtIndex:indexPath.row];

    CharacteristicVC *characteristicVC = [[UIStoryboard storyboardWithName:@"Main" bundle:nil] instantiateViewControllerWithIdentifier:@"CharacteristicVC"];
    characteristicVC.peripheral = self.peripheral;
    characteristicVC.service = service;
    [self.navigationController pushViewController:characteristicVC animated:YES];
}
```

**4、扫描特征**

```
//设置代理
self.peripheral.delegate = self;

#pragma mark CBPeripheralDelegate
-(void)peripheral:(CBPeripheral *)peripheral didDiscoverCharacteristicsForService:(nonnull CBService *)service error:(nullable NSError *)error
{
    if (error) {
        NSLog(@"didDiscoverCharacteristicsForService:%@",error.localizedDescription);
        return;
    }
    else
    {
        for (CBCharacteristic *cha in service.characteristics) {
            [self.characteristicList addObject:cha];
            switch (cha.properties) {
                case CBCharacteristicPropertyRead:
                {
                    NSLog(@"------------------读:%@",cha);
                    break;
                }
                case CBCharacteristicPropertyWrite:
                {
                    NSLog(@"------------------写:%@",cha);
                    break;
                }
                case CBCharacteristicPropertyNotify:
                {
                    NSLog(@"------------------订阅:%@",cha);
                    break;
                }
                default:
                    NSLog(@"------------------:%@",cha);
                    break;
            }
        }
        [self.tableView reloadData];
    }
}

//点击cell到特征操作页面，开始做数据传输
-(void)tableView:(UITableView *)tableView didSelectRowAtIndexPath:(NSIndexPath *)indexPath
{
    [tableView deselectRowAtIndexPath:indexPath animated:YES];
    CBCharacteristic *characteristic = [self.characteristicList objectAtIndex:indexPath.row];

    CharacteristicHandleVC *characteristicHandleVC = [[UIStoryboard storyboardWithName:@"Main" bundle:nil] instantiateViewControllerWithIdentifier:@"CharacteristicHandleVC"];
    characteristicHandleVC.peripheral = self.peripheral;
    characteristicHandleVC.service = self.service;
    characteristicHandleVC.characteristic = characteristic;
    [self.navigationController pushViewController:characteristicHandleVC animated:YES];
}
```

**5、数据传输**

数据传输都是在下面的回调中完成

```
//设置代理
self.peripheral.delegate = self;

//读数据的回调
-(void)peripheral:(CBPeripheral *)peripheral didUpdateValueForCharacteristic:(CBCharacteristic *)characteristic error:(NSError *)error
{
    self.msgShowLabel.text = [[NSString alloc] initWithData:characteristic.value encoding:NSUTF8StringEncoding];
}

//写数据的回调
-(void)peripheral:(CBPeripheral *)peripheral didWriteValueForCharacteristic:(CBCharacteristic *)characteristic error:(NSError *)error
{
    if (error) {
        self.msgShowLabel.text = @"didWriteValueForCharacteristic error";
    }
    else
    {
        //读数据
        [peripheral readValueForCharacteristic:characteristic];
    }
}

- (IBAction)sendMsgBtnClicked:(UIButton *)sender {
    //写数据
    NSString *msg = [self.msgTextField.text length] > 0 ? self.msgTextField.text : @"无";
    [self.peripheral writeValue:[msg dataUsingEncoding:NSUTF8StringEncoding] forCharacteristic:self.characteristic type:CBCharacteristicWriteWithResponse];
}
```

以上就是基于BLE实现的蓝牙通讯的基本介绍
































