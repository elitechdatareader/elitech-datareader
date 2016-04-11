Elitech RC4 / RC5 DataReader
====================================


Description
-----------

This software is a data collecting tool, written in python for Temperature data logger RC-4/RC-5.

[Elitech RC-4](http://www.elitech.uk.com/temperature_logger/Elitech_UK__Mini_USB_Temperature_Data_logger_URC_4_149.html) / 
[RC-5](http://www.e-elitech.com/jingChuang3/shouYe.do?operate=doProductDetail&chanpinId=156) 
is a reasonable data logger.


Enables to use RC-4/RC-5 on Mac, Linux, Windows.

Requirements
------------

- Python2.7 or Python3.4
- Serial Port Driver
    - (for RC-4) Silicon Labs CP210x USB-UART bridge VCP driver.  <http://www.silabs.com/products/mcu/Pages/USBtoUARTBridgeVCPDrivers.aspx>
    - (for RC-5) CH340 Serial Driver [MacOSX](http://www.wch.cn/download/CH341SER_MAC_ZIP.html)
- pySerial (data access library for serial port)


Setup
------------

1. Install Serial Port Driver.
    - for RC-4: CP210x USB-UART bridge VCP driver.  Download and install CP210x driver for your platform.
 <http://www.silabs.com/products/mcu/Pages/USBtoUARTBridgeVCPDrivers.aspx>
    - for RC-5: CH340 Serial Driver [MacOSX](http://www.wch.cn/download/CH341SER_MAC_ZIP.html)

2. install dependencies.

```
$ python setup.py install
```

OR pip

```
$ pip install elitech-datareader
```

Example(Script)
--------------

### Initialize device.

initialize. rec interval 10sec. set clock now.

```
$ elitech-datareader --command simple-set --interval=10 /dev/tty.SLAB_USBtoUART # RC-4 on macosx
                                                      # /dev/tty.wchusbserialfd1430 # RC-5 on macosx
```

### Get data

Press stop button for stop recording.

output to stdout.

```
$ elitech-datareader --command get /dev/tty.SLAB_USBtoUART
6
1	2015-06-07 13:53:36	25.0
2	2015-06-07 13:53:46	25.1
3	2015-06-07 13:53:56	25.1
4	2015-06-07 13:54:06	25.1
5	2015-06-07 13:54:16	25.1
6	2015-06-07 13:54:26	25.1
```

Elitech device gets the data in units called "Page size".
Page size is determined by the device model. RC4 is 100 and RC5 is 500.

You can directly specify the page size, with an optional argument --page_size. (for debug)

```
$ elitech-datareader --command get --page_size=500 /dev/tty.SLAB_USBtoUART
6
1	2015-06-07 13:53:36	25.0
2	2015-06-07 13:53:46	25.1
3	2015-06-07 13:53:56	25.1
4	2015-06-07 13:54:06	25.1
5	2015-06-07 13:54:16	25.1
6	2015-06-07 13:54:26	25.1
```

### Get latest data

```
$ elitech-datareader --command latest /dev/tty.SLAB_USBtoUART
6	2015-06-07 13:54:26	25.1

$ elitech-datareader --command latest --only-value /dev/tty.SLAB_USBtoUART
25.2
```


### Get device information

get device information.

```
$ elitech-datareader --command devinfo /dev/tty.SLAB_USBtoUART
station_no=3
last_online=2015-06-09 01:13:13
temp_unit=TemperatureUnit.C
alarm=AlarmSetting.NONE
work_sts=WorkStatus.STOP
lower_limit=-30.0
tone_set=ToneSet.NONE
rec_count=272
upper_limit=60.0
delay=0.0
stop_button=StopButton.ENABLE
current=2015-06-09 07:42:00
start_time=2015-06-07 13:53:36
rec_interval=00:00:10
temp_calibration=-1.5
user_info=RC-4 Data Logger
dev_num=9900112233
```

### Parameter set

set device parameter.

```
$ elitech-datareader --command set --interval=10 --upper_limit=60.0 --lower_limit=-30.0 \
--station_no=1 --stop_button=y --delay=0.0 --tone_set=y --alarm=x --temp_unit=C \
--temp_calibration=-1.5 --dev_num=1234567890 --user_info="RC4 Data Logger" /dev/tty.SLAB_USBtoUART
```

### Debug raw communication

Send raw request data. receive response data.

```
$ elitech-datareader --command raw --req="CC 00 06 00 D2" -res_len=4 /dev/tty.SLAB_USBtoUART

response length=4
55 01 01 32
```


Example(Python module)
-------

### Get device infomation.

```python
import elitech

device = elitech.Device("/dev/tty.SLAB_USBtoUART")
devinfo = device.get_devinfo()
print(devinfo.info)
```

### Get record data

```python
import elitech

device = elitech.Device("/dev/tty.SLAB_USBtoUART")
body = device.get_data()
for elm in body:
    print elm
    
```

### Update param

```python
device = elitech.Device("/dev/tty.SLAB_USBtoUART")
devinfo = device.get_devinfo()  # get current parameters.

param_put = devinfo.to_param_put()  #convart devinfo to parameter
param_put.rec_interval = datetime.time(0, 0, 10)    # update parameter
param_put.stop_button = elitech.StopButton.ENABLE   # update parameter

param_put_res = device.update(param_put)    # update device

```
