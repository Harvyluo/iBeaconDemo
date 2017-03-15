# iBeaconDemo
        项目中用到了ibeacon技术，一直想记录下来，写了一个关于扫描ibeacon设备的Demo，有错误之处，欢迎指正。
### 1、工程配置
在info.plist文件中添加如下参数
[image:CFAB167B-DBE3-4A9F-9E4F-488AE531F6CC-1204-000006C6A8852C7E/6B29BEFE-9345-477C-B28F-2A2BE5C450A3.png]
上图是添加位置定位使用。
[image:1F8DCA3A-B979-498A-9DDE-3A7183C0CF48-1204-000006CB4AE790F8/43C43486-44A4-4650-8206-AD5C5577C1D3.png]
上图可以不添加，我的项目中需要后台也扫描ibeacon并发送Ibeacon信息给服务器，所以采用播放一段无声音乐强制常驻后台，这个可以根据你的需求来，一般是不需要的。
### 2、ibeacon知识讲解
*什么是ibeacon*:通过使用低功耗蓝牙技术（Bluetooth Low Energy，也就是通常所说的Bluetooth 4.0或者Bluetooth Smart），iBeacon基站可以创建一个信号区域，当设备进入该区域时，相应的应用程序便会提示用户是否需要接入这个信号网络。通过能够放置在任何物体中的小型无线传感器和低功耗蓝牙技术，用户便能使用iPhone来传输数据。
*beacon可以用来做什么：*
* ibeacon做室内导航：
用户可以连接到最近的iBeacon基站，从而获得该基站的GPS位置信息，从而知道目前所处的地点。当用户进入或离开某个iBeacon基站的通信范围时都会收到相应的通知信息，从而实现导航的目的。
* 娱乐行业
使用者安装了了官方 iOS app 并打开蓝牙后，在检验员或者会场附近路过时，手机会自动收到提醒，使用者可以选择观看电影买票。
* 教育行业
白皙的移动开发公司开发了一款app BeHere，主要用于帮助老师帮助老师点名查看学生的考勤情况，当学生走进教室的时候，该APP自动签到，这个APP可以省去老师们点名的时间，APP实现了自动签到，该产品已经在大学试用，其实未来更应该借助诸如穿戴设备推广到中小学生，及时检测儿童的是否出勤，防止学生发生意外。
### 3、程序讲解
.m文件中声明相关对象示例，AVAudioSession、AVAudioPlayer是播放音乐时使用的对象。*BEACONUUID一定要换成自己的ibeacon模块的*，很多同学拿到Demo没有改代码，是扫描不到的。
```
#import "ViewController.h"
#define BEACONUUID @"FDA50693-A4E2-4FB1-AFCF-C6EB07647825"//iBeacon的uuid可以换成自己设备的uuid
#import <AVFoundation/AVFoundation.h>

@interface ViewController ()<AVAudioPlayerDelegate>{
    
    AVAudioSession *session;
    AVAudioPlayer *_player;
}

@property (strong, nonatomic) UITableView *tableView;

@end
```
下面是播放一段无声音乐的代码，保证应用强制常驻后台
```
- (void)playbackgroud
{
    /*
     这里是随便添加得一首音乐。
     真正的工程应该是添加一个尽可能小的音乐。。。
     0～1秒的
     没有声音的。
     循环播放就行。
     这个只是保证后台一直运行该软件。
     使得该软件一直处于活跃状态.
     你想操作的东西该在哪里操作就在哪里操作。
     */
    session = [AVAudioSession sharedInstance];
    /*打开应用会关闭别的播放器音乐*/
    //    [session setCategory:AVAudioSessionCategoryPlayback error:nil];
    /*打开应用不影响别的播放器音乐*/
    [[AVAudioSession sharedInstance] setCategory:AVAudioSessionCategoryPlayback withOptions:AVAudioSessionCategoryOptionMixWithOthers error:nil];
    [session setActive:YES error:nil];
    //设置代理 可以处理电话打进时中断音乐播放
    NSString *musicPath = [[NSBundle mainBundle] pathForResource:@"music" ofType:@"mp3"];
    NSURL *URLPath = [[NSURL alloc] initFileURLWithPath:musicPath];
    _player = [[AVAudioPlayer alloc] initWithContentsOfURL:URLPath error:nil];
    [_player prepareToPlay];
    [_player setDelegate:self];
    _player.numberOfLoops = -1;
    [_player play];
}
```
                                        
                                            /*下面是重点*/
---
```
//初始化ibeacon方法
- (void)initIbeacon{
    
    self.beaconArr = [[NSArray alloc] init];
    
    self.locationmanager = [[CLLocationManager alloc] init];//初始化
    
    self.locationmanager.delegate = self;
    //初始化监测的iBeacon信息
    self.beacon1 = [[CLBeaconRegion alloc] initWithProximityUUID:[[NSUUID alloc] initWithUUIDString:BEACONUUID] identifier:@"tencent1"];
    [self.locationmanager requestAlwaysAuthorization];//设置location是一直允许
    [self.locationmanager startRangingBeaconsInRegion:self.beacon1];//开始
    //[self playbackgroud]; 
}
//CLLocationManager总是使用位置
- (void)locationManager:(CLLocationManager *)manager didChangeAuthorizationStatus:(CLAuthorizationStatus)status{
    
    if (status == kCLAuthorizationStatusAuthorizedAlways) {
        
        [self.locationmanager startMonitoringForRegion:self.beacon1];//开始MonitoringiBeacon
    }
}
```
手机作为中心设备，扫描周边的ibeacon设备，会执行下面的代理方法。
CLBeacon对象有以下几个属性
//ibeacon的UUID
@property (readonly, nonatomic, copy) NSUUID *proximityUUID;
//ibeacon的major,可以通过软件修改
@property (readonly, nonatomic, copy) NSNumber *major;
//ibeacon的minor,可以通过软件修改
@property (readonly, nonatomic, copy) NSNumber *minor;
@property (readonly, nonatomic) CLProximity proximity;
//ibeacon的accuracy,ibeacon距离中心设备的距离，转换成浮点型即可
@property (readonly, nonatomic) CLLocationAccuracy accuracy;
//ibeacon的rssi,ibeacon信号辐射强度，这个强度也可以反应距离
@property (readonly, nonatomic) NSInteger rssi;
```
//找的iBeacon后扫描它的信息
- (void)locationManager:(CLLocationManager *)manager didRangeBeacons:(NSArray *)beacons inRegion:(CLBeaconRegion *)region{
        
        //如果存在不是我们要监测的iBeacon那就停止扫描他 
        if (![[region.proximityUUID UUIDString] isEqualToString:BEACONUUID]){
            
            [self.locationmanager stopMonitoringForRegion:region];
            [self.locationmanager stopRangingBeaconsInRegion:region];
            
        }
        //打印所有iBeacon的信息
        
        for (CLBeacon* beacon in beacons) {
            
            NSLog(@"rssi is :%ld",beacon.rssi);
            
            NSLog(@"beacon.proximity %ld",beacon.proximity);
            
            NSLog(@"beacon.proximity %@",beacon.major);
            
            NSLog(@"beacon.proximity %@",beacon.minor);
        }
        
        self.beaconArr = beacons;
        
        [self.tableView reloadData];
}
```
在tableview中显示出来，包括扫描到的Ibeacon UUID,信号强度、距离等信息
```
- (UITableViewCell *)tableView:(UITableView *)tableView cellForRowAtIndexPath:(NSIndexPath *)indexPath{
        
        static NSString *ident = @"cell";
        UITableViewCell *cell = [tableView dequeueReusableCellWithIdentifier:ident];
        
        if (!cell) {
            
            cell = [[UITableViewCell alloc] initWithStyle:UITableViewCellStyleSubtitle reuseIdentifier:ident];
            
        }
        CLBeacon *beacon = [self.beaconArr objectAtIndex:indexPath.row];
        cell.textLabel.text = [beacon.proximityUUID UUIDString];
        NSString *str;
        switch (beacon.proximity) {
            case CLProximityNear:
                
                str = @"近";
                break;
                
            case CLProximityImmediate:
                
                str = @"超近";
                break;
                
            case CLProximityFar:
                
                str = @"远";
                break;
                
            case CLProximityUnknown:
                
                str = @"不见了";
                break;
                
            default:
                
                break;
                
        }
        cell.detailTextLabel.text = [NSString stringWithFormat:@"%@ %ld %@ %@ %.2f",str,beacon.rssi,beacon.major,beacon.minor,(double)(beacon.accuracy)];
        
        return cell;
        
    }
```

### 4、界面截图

[image:78BBA11A-2463-4C68-BB1C-E8CCD99D5149-1204-0000088E8CA10373/IMG_4451.PNG]

如有错误之处，欢迎指正，有疑问可以联系QQ：623778119
Github演示Demo： [https://github.com/Harvyluo/iBeaconDemo](https://github.com/Harvyluo/iBeaconDemo)
