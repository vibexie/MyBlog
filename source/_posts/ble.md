---
title: Android BLE开发
date: 2016-11-29 03:23:06
tags: android
---

#### 概述

* 什么是BLE?
BLE 是 Bluetooth Low Energy 的缩写，即蓝牙低功耗方案。Android 4.3 (API Level 18) 内置框架引入了BLE；
* 与蓝牙(Bluetooth)的区别
与传统的蓝牙对比, 蓝牙低功耗方案 (Bluetooth Low Energy) 是出于更低的电量消耗考虑而设计的. 这可以使 Android 应用可以与 BLE 设备进行交流, 这些设备需要很低的电量, 如 近距离传感器, 心率测量设备, 健康设备 等等.

<!-- more -->
#### 术语及概念
* Generic Attribute Profile (GATT) 通用属性规范
GATT 规范是一个针对 在 BLE 连接上的, 发送 和 接收 少量数据的一个规范, 所有的现有的低功耗应用的规范都是基于这个 GATT 规范制定的；蓝牙技术联盟 (Bluetooth SIG) 为低功耗设备定义了许多规范, 一个 规范 (Profile) 就是 设备如何在特定的应用中工作的详述;此外, 一个设备可以实现多个规范, 如 : 一个设备可以包含一个心率检测器, 和 电量检测器.
* Attribute Protocol (ATT) 属性协议
GATT 规范是建立在 ATT 的上一层的, 这套改改通常被称为 GATT/ATT;ATT 被用于优化 BLE 设备的运行, 为了这个目的, ATT (属性协议) 使用尽可能少的字节;ATT 中的每个属性都被 一个 UUID (Universally Unique Identifier) 独一无二的进行标识, UUID 是一个 128 比特的标准的字符串 ID, 用于信息的唯一标识;ATT 中定义的属性就是 Charicteristics (特性) 和 Services (服务);
* Characteristic 特性
一个 Characteristic 特性包含了一个值 和 多个 Descriptor (描述符) 用于描述这个特性的值;一个特性可以被认为是一个类型, 类似于一个类;
* Descriptor 描述符
描述符被定义为一些属性, 这些属性用于描述 Characteristic (特性) 的值; 例如, 一个 描述符 可以说明一个 可读的描述, 一个 特性值的可接受范围, 或者 一个特性值的测量单元;
* Service 服务
服务是 Characteristic (特性) 的集合;如, 你可以有一个 名称为 "Heart Rate Monitor (心率监控)" 的服务, 包含了特性 "Heart Rate Measurement (心率测量)";你可以在 bluetooth.org 官网查询到一个基于 GATT 服务 和 规范的列表;

#### BLE权限

* AndroidManifest.xml 声明蓝牙权限示例
``` xml
<uses-permission android:name="android.permission.BLUETOOTH"/>  
<uses-permission android:name="android.permission.BLUETOOTH_ADMIN"/>
```
如果你的APP只需要胜任BLE设备的工作, 只需要如下配置
``` xml
<uses-feature android:name="android.hardware.bluetooth_le" android:required="true"/>
```
* 动态控制BLE功能是否使用
如果你想要让你的 APP 可以当做 BLE 设备, 但是手机不支持这个操作, 你仍然可以进行如下配置, 只是将其中的 android:required 设置成 false. 此时在运行时, 你可以使用 "PackageManager.hasSystemFeature()" 方法决定 BLE 是否可用.
```java
// Use this check to determine whether BLE is supported on the device. Then
// you can selectively disable BLE-related features.
if (!getPackageManager().hasSystemFeature(PackageManager.FEATURE_BLUETOOTH_LE)) {
    Toast.makeText(this, R.string.ble_not_supported, Toast.LENGTH_SHORT).show();
    finish();
}
```

#### 创建BLE
在应用可以通过 BLE 交互之前, 你需要验证设备是否支持 BLE 功能, 如果支持, 确定它是可以使用的;注意这个检查只有在 下面的配置 设置为 false 时才是必须的;
``` xml
<uses-feature android:name="android.hardware.bluetooth_le" android:required="true"/>
```
如果 Android 手机不支持 BLE 功能, 你应该优雅的 关闭 BLE 相关功能;
如果 BLE 支持 BLE 功能, 但是设备的蓝牙是关闭的, 你可以在应用中请求打开设备的蓝牙模块;
创建 BLE 蓝牙的过程分成两个步骤, 1. 获取 BluetoothAdapter, 2. 打开 设备的蓝牙模块;

* 获取 BluetoothAdapter (蓝牙适配器)
所有的蓝牙活动都需要 BluetoothAdapter, BluetoothAdapter 代表了设备本身的蓝牙适配器 (蓝牙无线设备). 整个系统中只有一个 蓝牙适配器, 应用可以使用 BluetoothAdapter 对象与 蓝牙适配器硬件进行交互;
获取 BluetoothAdapter 代码示例如下：
``` java
// Initializes Bluetooth adapter.
final BluetoothManager bluetoothManager =
        (BluetoothManager) getSystemService(Context.BLUETOOTH_SERVICE);
mBluetoothAdapter = bluetoothManager.getAdapter();
```
注意 : 这个方法使用了 getSystemService() 方法, 返回了一个 BluetoothManager 实例对象, 从 BluetoothManager 实例对象中可以获取 BluetoothAdapter 对象;

* 打开蓝牙功能
为了保证 蓝牙功能是打开的, 调用 BluetoothAdapter 的 isEnable() 方法, 检查蓝牙在当前是否可用. 如果返回 false, 说明当前蓝牙不可用.
``` java
private BluetoothAdapter mBluetoothAdapter;
...
// Ensures Bluetooth is available on the device and it is enabled. If not,
// displays a dialog requesting user permission to enable Bluetooth.
if (mBluetoothAdapter == null || !mBluetoothAdapter.isEnabled()) {
    Intent enableBtIntent = new Intent(BluetoothAdapter.ACTION_REQUEST_ENABLE);
    startActivityForResult(enableBtIntent, REQUEST_ENABLE_BT);
}

```

#### 查找 BLE 设备
为了搜索到 BLE 设备, 调用 BluetoothAdapter 的 startLeScan() 方法, 该方法需要一个 BluetoothAdapter.LeScanCallback 类型的参数. 你必须实现这个 LeScanCallback 接口, 因为 BLE 蓝牙设备扫描结果在这个接口中返回;
查找策略 : 蓝牙搜索是非常耗电的, 你需要遵守以下的 中断策略 和 不循环策略.
1. 中断策略 : 只要一发现蓝牙设备, 马上中断扫描.
2. 不循环策略 : 不要循环扫描, 设置一个扫描的最大时间限制. 一个设备在之前可用, 继续扫描可能会使设备不可用, 此外继续扫描会持续浪费电池电量.
``` java
/**
 * Activity for scanning and displaying available BLE devices.
 */
public class DeviceScanActivity extends ListActivity {

    private BluetoothAdapter mBluetoothAdapter;
    private boolean mScanning;
    private Handler mHandler;

    // Stops scanning after 10 seconds.
    private static final long SCAN_PERIOD = 10000;
    ...
    private void scanLeDevice(final boolean enable) {
        if (enable) {
            // Stops scanning after a pre-defined scan period.
            mHandler.postDelayed(new Runnable() {
                @Override
                public void run() {
                    mScanning = false;
                    mBluetoothAdapter.stopLeScan(mLeScanCallback);
                }
            }, SCAN_PERIOD);

            mScanning = true;
            mBluetoothAdapter.startLeScan(mLeScanCallback);
        } else {
            mScanning = false;
            mBluetoothAdapter.stopLeScan(mLeScanCallback);
        }
        ...
    }
...
}
```

当你需要查找特定类型的外围设备, 可以调用下面的方法, 这个方法需要提供一个 UUID 对象数组, 这个 UUID 数组是 APP 支持的 GATT 服务的特殊标识.
``` java
startLeScan(UUID[], BluetoothAdapter.LeScanCallback);
```
BluetoothAdapter.LeScanCallback 实现类, 在这个实现类的接口中返回 BLE 设备扫描结果;
``` java
private LeDeviceListAdapter mLeDeviceListAdapter;
...
// Device scan callback.
private BluetoothAdapter.LeScanCallback mLeScanCallback =
        new BluetoothAdapter.LeScanCallback() {
    @Override
    public void onLeScan(final BluetoothDevice device, int rssi,
            byte[] scanRecord) {
        runOnUiThread(new Runnable() {
           @Override
           public void run() {
               mLeDeviceListAdapter.addDevice(device);
               mLeDeviceListAdapter.notifyDataSetChanged();
           }
       });
   }
};
```

#### 连接到 GATT 服务
 与 BLE 设备交互的第一步是 连接到 BLE 设备中的 GATT 服务,调用 BluetoothDevice 的 connectGatt() 方法可以连接到 BLE 设备的 GATT 服务,connectGatt() 方法需要三个参数, 参数一 Context 上下文对象, 参数二 boolean autoConnect 是否自动连接扫描到的蓝牙设备, 参数三 BluetoothGattCallback 接口实现类.
 ``` java
 mBluetoothGatt = device.connectGatt(this, false, mGattCallback);
 ```
调用 connectGatt() 方法可以连接到 BLE 设备上的 GATT 服务, 返回一个 BluetoothGatt 实例对象, 你可以使用这个对象去 管理 GATT 客户端操作.
Android APP 可以调用 GATT Client (客户端). BluetoothGattCallback 可以用于传递结果到 GATT 客户端, 如 连接状态 和 更进一步的 GATT Client 操作.

在下面的示例中, BLE 应用提供了一个 Activity 界面, 该 Activity 界面用于 连接, 展示数据, 展示 GATT 服务 和 设备支持的特性;基于用户的输入, 这个 Activity 界面可以与一个 BluetoothLeService 的服务进行交流, 该交流的本质就是 BLE 设备的 GATT 服务 与 Android 的 BLE API 进行交流.
``` java
// A service that interacts with the BLE device via the Android BLE API.
public class BluetoothLeService extends Service {
    private final static String TAG = BluetoothLeService.class.getSimpleName();

    private BluetoothManager mBluetoothManager;
    private BluetoothAdapter mBluetoothAdapter;
    private String mBluetoothDeviceAddress;
    private BluetoothGatt mBluetoothGatt;
    private int mConnectionState = STATE_DISCONNECTED;

    private static final int STATE_DISCONNECTED = 0;
    private static final int STATE_CONNECTING = 1;
    private static final int STATE_CONNECTED = 2;

    public final static String ACTION_GATT_CONNECTED =
            "com.example.bluetooth.le.ACTION_GATT_CONNECTED";
    public final static String ACTION_GATT_DISCONNECTED =
            "com.example.bluetooth.le.ACTION_GATT_DISCONNECTED";
    public final static String ACTION_GATT_SERVICES_DISCOVERED =
            "com.example.bluetooth.le.ACTION_GATT_SERVICES_DISCOVERED";
    public final static String ACTION_DATA_AVAILABLE =
            "com.example.bluetooth.le.ACTION_DATA_AVAILABLE";
    public final static String EXTRA_DATA =
            "com.example.bluetooth.le.EXTRA_DATA";

    public final static UUID UUID_HEART_RATE_MEASUREMENT =
            UUID.fromString(SampleGattAttributes.HEART_RATE_MEASUREMENT);

    // Various callback methods defined by the BLE API.
    private final BluetoothGattCallback mGattCallback =
            new BluetoothGattCallback() {
        @Override
        public void onConnectionStateChange(BluetoothGatt gatt, int status,
                int newState) {
            String intentAction;
            if (newState == BluetoothProfile.STATE_CONNECTED) {
                intentAction = ACTION_GATT_CONNECTED;
                mConnectionState = STATE_CONNECTED;
                broadcastUpdate(intentAction);
                Log.i(TAG, "Connected to GATT server.");
                Log.i(TAG, "Attempting to start service discovery:" +
                        mBluetoothGatt.discoverServices());

            } else if (newState == BluetoothProfile.STATE_DISCONNECTED) {
                intentAction = ACTION_GATT_DISCONNECTED;
                mConnectionState = STATE_DISCONNECTED;
                Log.i(TAG, "Disconnected from GATT server.");
                broadcastUpdate(intentAction);
            }
        }

        @Override
        // New services discovered
        public void onServicesDiscovered(BluetoothGatt gatt, int status) {
            if (status == BluetoothGatt.GATT_SUCCESS) {
                broadcastUpdate(ACTION_GATT_SERVICES_DISCOVERED);
            } else {
                Log.w(TAG, "onServicesDiscovered received: " + status);
            }
        }

        @Override
        // Result of a characteristic read operation
        public void onCharacteristicRead(BluetoothGatt gatt,
                BluetoothGattCharacteristic characteristic,
                int status) {
            if (status == BluetoothGatt.GATT_SUCCESS) {
                broadcastUpdate(ACTION_DATA_AVAILABLE, characteristic);
            }
        }
     ...
    };
...
}
```
当一个特定的回调被触发, 它调用适当的 broadcastUpdate() 帮助方法, 将其当做一个 Action 操作传递出去,这部分的数据解析 与 蓝牙心率测量 是一起被执行的.
``` java
private void broadcastUpdate(final String action) {
    final Intent intent = new Intent(action);
    sendBroadcast(intent);
}

private void broadcastUpdate(final String action,
                             final BluetoothGattCharacteristic characteristic) {
    final Intent intent = new Intent(action);

    // This is special handling for the Heart Rate Measurement profile. Data
    // parsing is carried out as per profile specifications.
    if (UUID_HEART_RATE_MEASUREMENT.equals(characteristic.getUuid())) {
        int flag = characteristic.getProperties();
        int format = -1;
        if ((flag & 0x01) != 0) {
            format = BluetoothGattCharacteristic.FORMAT_UINT16;
            Log.d(TAG, "Heart rate format UINT16.");
        } else {
            format = BluetoothGattCharacteristic.FORMAT_UINT8;
            Log.d(TAG, "Heart rate format UINT8.");
        }
        final int heartRate = characteristic.getIntValue(format, 1);
        Log.d(TAG, String.format("Received heart rate: %d", heartRate));
        intent.putExtra(EXTRA_DATA, String.valueOf(heartRate));
    } else {
        // For all other profiles, writes the data formatted in HEX.
        final byte[] data = characteristic.getValue();
        if (data != null && data.length > 0) {
            final StringBuilder stringBuilder = new StringBuilder(data.length);
            for(byte byteChar : data)
                stringBuilder.append(String.format("%02X ", byteChar));
            intent.putExtra(EXTRA_DATA, new String(data) + "\n" +
                    stringBuilder.toString());
        }
    }
    sendBroadcast(intent);
}
```
在 DeviceControlActivity 中处理广播事件.
``` java
// Handles various events fired by the Service.
// ACTION_GATT_CONNECTED: connected to a GATT server.
// ACTION_GATT_DISCONNECTED: disconnected from a GATT server.
// ACTION_GATT_SERVICES_DISCOVERED: discovered GATT services.
// ACTION_DATA_AVAILABLE: received data from the device. This can be a
// result of read or notification operations.
private final BroadcastReceiver mGattUpdateReceiver = new BroadcastReceiver() {
    @Override
    public void onReceive(Context context, Intent intent) {
        final String action = intent.getAction();
        if (BluetoothLeService.ACTION_GATT_CONNECTED.equals(action)) {
            mConnected = true;
            updateConnectionState(R.string.connected);
            invalidateOptionsMenu();
        } else if (BluetoothLeService.ACTION_GATT_DISCONNECTED.equals(action)) {
            mConnected = false;
            updateConnectionState(R.string.disconnected);
            invalidateOptionsMenu();
            clearUI();
        } else if (BluetoothLeService.
                ACTION_GATT_SERVICES_DISCOVERED.equals(action)) {
            // Show all the supported services and characteristics on the
            // user interface.
            displayGattServices(mBluetoothLeService.getSupportedGattServices());
        } else if (BluetoothLeService.ACTION_DATA_AVAILABLE.equals(action)) {
            displayData(intent.getStringExtra(BluetoothLeService.EXTRA_DATA));
        }
    }
};

```
Android 应用连接到了 设备中的 GATT 服务, 并且发现了 各种服务 (特性集合), 可以读写其中的属性,遍历服务 (特性集合) 和 特性, 将其展示在 UI 界面中.
``` java
public class DeviceControlActivity extends Activity {
    ...
    // Demonstrates how to iterate through the supported GATT
    // Services/Characteristics.
    // In this sample, we populate the data structure that is bound to the
    // ExpandableListView on the UI.
    private void displayGattServices(List<BluetoothGattService> gattServices) {
        if (gattServices == null) return;
        String uuid = null;
        String unknownServiceString = getResources().
                getString(R.string.unknown_service);
        String unknownCharaString = getResources().
                getString(R.string.unknown_characteristic);
        ArrayList<HashMap<String, String>> gattServiceData =
                new ArrayList<HashMap<String, String>>();
        ArrayList<ArrayList<HashMap<String, String>>> gattCharacteristicData
                = new ArrayList<ArrayList<HashMap<String, String>>>();
        mGattCharacteristics =
                new ArrayList<ArrayList<BluetoothGattCharacteristic>>();

        // Loops through available GATT Services.
        for (BluetoothGattService gattService : gattServices) {
            HashMap<String, String> currentServiceData =
                    new HashMap<String, String>();
            uuid = gattService.getUuid().toString();
            currentServiceData.put(
                    LIST_NAME, SampleGattAttributes.
                            lookup(uuid, unknownServiceString));
            currentServiceData.put(LIST_UUID, uuid);
            gattServiceData.add(currentServiceData);

            ArrayList<HashMap<String, String>> gattCharacteristicGroupData =
                    new ArrayList<HashMap<String, String>>();
            List<BluetoothGattCharacteristic> gattCharacteristics =
                    gattService.getCharacteristics();
            ArrayList<BluetoothGattCharacteristic> charas =
                    new ArrayList<BluetoothGattCharacteristic>();
           // Loops through available Characteristics.
            for (BluetoothGattCharacteristic gattCharacteristic :
                    gattCharacteristics) {
                charas.add(gattCharacteristic);
                HashMap<String, String> currentCharaData =
                        new HashMap<String, String>();
                uuid = gattCharacteristic.getUuid().toString();
                currentCharaData.put(
                        LIST_NAME, SampleGattAttributes.lookup(uuid,
                                unknownCharaString));
                currentCharaData.put(LIST_UUID, uuid);
                gattCharacteristicGroupData.add(currentCharaData);
            }
            mGattCharacteristics.add(charas);
            gattCharacteristicData.add(gattCharacteristicGroupData);
         }
    ...
    }
...
}

```

#### 接收 GATT 通知
当 BLE 设备中的一些特殊的特性改变, 需要通知与之连接的 Android BLE 应用.使用 setCharacteristicNotification() 方法为特性设置通知.
``` java
private BluetoothGatt mBluetoothGatt;
BluetoothGattCharacteristic characteristic;
boolean enabled;
...
mBluetoothGatt.setCharacteristicNotification(characteristic, enabled);
...
BluetoothGattDescriptor descriptor = characteristic.getDescriptor(
        UUID.fromString(SampleGattAttributes.CLIENT_CHARACTERISTIC_CONFIG));
descriptor.setValue(BluetoothGattDescriptor.ENABLE_NOTIFICATION_VALUE);
mBluetoothGatt.writeDescriptor(descriptor);
```
一但特性开启了改变通知监听, 如果特性发生了改变, 就会回调 BluetoothGattCallback 接口中的 onCharacteristicChanged() 方法.
``` java
@Override
// Characteristic notification
public void onCharacteristicChanged(BluetoothGatt gatt,
        BluetoothGattCharacteristic characteristic) {
    broadcastUpdate(ACTION_DATA_AVAILABLE, characteristic);
}

```

#### 关闭 APP 中的 BLE 连接
一旦结束了 BLE 设备的使用, 调用 BluetoothGatt 的 close() 方法, 关闭 BLE 连接, 释放相关的资源.
``` java
public void close() {
    if (mBluetoothGatt == null) {
        return;
    }
    mBluetoothGatt.close();
    mBluetoothGatt = null;
}
```

#### 简单实践（小米note与小米手环进行数据交互）
我们需要实现手机蓝牙的控制，以及对小米手环的连接和数据传输。
先看看效果:
![](http://qiniu.vibexie.com/blog/ble-1?imageView2/2/w/300)
实例代码部分有点乱：
``` java
package com.vibexie.bletest;

import java.util.ArrayList;
import java.util.List;
import java.util.UUID;


import android.app.Activity;
import android.bluetooth.BluetoothAdapter;
import android.bluetooth.BluetoothDevice;
import android.bluetooth.BluetoothGatt;
import android.bluetooth.BluetoothGattCallback;
import android.bluetooth.BluetoothGattCharacteristic;
import android.bluetooth.BluetoothManager;
import android.bluetooth.BluetoothProfile;
import android.content.Context;
import android.content.Intent;
import android.os.Bundle;
import android.os.Handler;
import android.util.Log;
import android.view.LayoutInflater;
import android.view.View;
import android.view.View.OnClickListener;
import android.view.ViewGroup;
import android.widget.BaseAdapter;
import android.widget.LinearLayout;
import android.widget.ListView;
import android.widget.TextView;

public class MainActivity extends Activity {

	BluetoothManager bluetoothManager;
	BluetoothAdapter mBluetoothAdapter;
	private ListView mListView;
	private MyAdapter myAdapter;
	private TextView mTextView;
	private List<BluetoothDevice> mData = new ArrayList<BluetoothDevice>();
	
	private boolean mScanning;
    private Handler mHandler;
    // Stops scanning after 10 seconds.
    private static final long SCAN_PERIOD = 10000;
	
	@Override
	protected void onCreate(Bundle savedInstanceState) {
		super.onCreate(savedInstanceState);
		setContentView(R.layout.activity_main);
		
		bluetoothManager = (BluetoothManager) getSystemService(Context.BLUETOOTH_SERVICE);
		mBluetoothAdapter = bluetoothManager.getAdapter();
		mHandler = new Handler();

		mListView = (ListView) this.findViewById(R.id.listview);
		mTextView = (TextView) this.findViewById(R.id.tv);
		myAdapter = new MyAdapter(this, mData);
		mListView.setAdapter(myAdapter);
	}
	
	public void doClick(View view) {
		switch (view.getId()) {
		case R.id.open_bluetooth:
			enableBluetooth();
			break;
			
		case R.id.open_discover:
			enableDiscover();
			break;
			
		case R.id.search_nearby:
			mData.clear();
			myAdapter.notifyDataSetChanged();
			scanLeDevice(true);
			break;

		default:
			break;
		}
	}

	public void enableBluetooth() {
		if (mBluetoothAdapter == null || !mBluetoothAdapter.isEnabled()) {
		    Intent enableBtIntent = new Intent(BluetoothAdapter.ACTION_REQUEST_ENABLE);
		    startActivity(enableBtIntent);
		}
	}
	
	public void enableDiscover() {
		if (mBluetoothAdapter == null || !mBluetoothAdapter.isDiscovering()) {
		    Intent enableBtIntent = new Intent(BluetoothAdapter.ACTION_REQUEST_DISCOVERABLE);
		    startActivity(enableBtIntent);
		}
	}
	
	private void scanLeDevice(final boolean enable) {
        if (enable) {
            // Stops scanning after a pre-defined scan period.
            mHandler.postDelayed(new Runnable() {
                @Override
                public void run() {
                    mScanning = false;
                    mBluetoothAdapter.stopLeScan(mLeScanCallback);
                }
            }, SCAN_PERIOD);

            mScanning = true;
            mBluetoothAdapter.startLeScan(mLeScanCallback);
        } else {
            mScanning = false;
            mBluetoothAdapter.stopLeScan(mLeScanCallback);
        }
    }
	
	private BluetoothAdapter.LeScanCallback mLeScanCallback =
	        new BluetoothAdapter.LeScanCallback() {
	    @Override
	    public void onLeScan(final BluetoothDevice device, int rssi,
	            byte[] scanRecord) {
	        		runOnUiThread(new Runnable() {
	        			@Override
	        			public void run() {
	        				myAdapter.addDevice(device);
	        			}
	       });
	   }
	};
	
	private class MyAdapter extends BaseAdapter {
		LayoutInflater layoutInflater;
		List<BluetoothDevice> data;
		Context mContext;
		
		public MyAdapter(Context context, List<BluetoothDevice> data) {
			// TODO Auto-generated constructor stub
			this.data = data;
			mContext = context;
			layoutInflater = LayoutInflater.from(context);
		}
		
		public void addDevice(BluetoothDevice device) {
			for (BluetoothDevice tmp : data) {
				tmp.getAddress().equals(device.getAddress());
				return;
			}
			data.add(device);
			notifyDataSetChanged();
		}

		@Override
		public int getCount() {
			// TODO Auto-generated method stub
			return data.size();
		}

		@Override
		public BluetoothDevice getItem(int position) {
			// TODO Auto-generated method stub
			return data.get(position);
		}

		@Override
		public long getItemId(int position) {
			// TODO Auto-generated method stub
			return position;
		}

		@Override
		public View getView(int position, View convertView, ViewGroup parent) {
			// TODO Auto-generated method stub
			ViewHolder holder = null;
			if (convertView == null) {
				convertView = layoutInflater.inflate(R.layout.item, null);
				holder = new ViewHolder(convertView);
				convertView.setTag(holder);
			} else {
				holder = (ViewHolder) convertView.getTag();
			}
			
			holder.fill(position);
			
			return convertView;
		}
		
		class ViewHolder implements OnClickListener{
			TextView tv;
			LinearLayout ll;
			int position;
			
			public ViewHolder(View view) {
				// TODO Auto-generated constructor stub
				tv = (TextView) view.findViewById(R.id.item_tv);
				ll = (LinearLayout) view.findViewById(R.id.item_ll);
			}

			public void fill(int position){
				this.position = position;
				tv.setText(getItem(position).getName());
				ll.setOnClickListener(this);
			}
			
			@Override
			public void onClick(View v) {
				// TODO Auto-generated method stub
				mBluetoothGatt = getItem(position).connectGatt(mContext, true, mGattCallback);
				mTextView.setText("连接到蓝牙:" + getItem(position).getName());
				
				//读数据
				new Handler().postDelayed(new Runnable() {
					
					@Override
					public void run() {
						// TODO Auto-generated method stub
						mTextView.setText(mTextView.getText().toString() + "\n" + "开始读取手环版本号");
						mBluetoothGatt.readCharacteristic(mBluetoothGatt.getService(UUID.fromString("0000180a-0000-1000-8000-00805F9B34FB")).getCharacteristic(UUID.fromString("00002A28-0000-1000-8000-00805F9B34FB")));
					}
				}, 5000);
				
				//写数据
				new Handler().postDelayed(new Runnable() {
					
					@Override
					public void run() {
						// TODO Auto-generated method stub
						mTextView.setText(mTextView.getText().toString() + "\n" + "向手环写入数据使手环震动");
						BluetoothGattCharacteristic chara = mBluetoothGatt.getService(UUID.fromString("00001802-0000-1000-8000-00805f9b34fb")).getCharacteristic(UUID.fromString("00002a06-0000-1000-8000-00805f9b34fb"));
						if (null == chara) {
							return;
						}
						chara.setValue(new byte[] {2});
						mBluetoothGatt.writeCharacteristic(chara);
					}
						
				}, 10000);
			}
		}
	}
	
	private BluetoothGatt mBluetoothGatt;

 // Various callback methods defined by the BLE API.
    private final BluetoothGattCallback mGattCallback =
            new BluetoothGattCallback() {
        @Override
        public void onConnectionStateChange(BluetoothGatt gatt, int status,
                int newState) {
            String intentAction;
            if (newState == BluetoothProfile.STATE_CONNECTED) {
//                broadcastUpdate(intentAction);
                Log.i("test", "Connected to GATT server.");
                Log.i("test", "Attempting to start service discovery:" +
                        mBluetoothGatt.discoverServices());
            } else if (newState == BluetoothProfile.STATE_DISCONNECTED) {
                Log.i("test", "Disconnected from GATT server.");
//                broadcastUpdate(intentAction);
            }
        }

        @Override
        // New services discovered
        public void onServicesDiscovered(BluetoothGatt gatt, int status) {
            if (status == BluetoothGatt.GATT_SUCCESS) {
//                broadcastUpdate(ACTION_GATT_SERVICES_DISCOVERED);
            } else {
                Log.w("test", "onServicesDiscovered received: " + status);
            }
        }

        //读取数据
        @Override
        public void onCharacteristicRead(BluetoothGatt gatt,
                final BluetoothGattCharacteristic characteristic,
                int status) {
            if (status == BluetoothGatt.GATT_SUCCESS) {
            	
            		runOnUiThread(new Runnable() {
					
					@Override
					public void run() {
						// TODO Auto-generated method stub
						mTextView.setText(mTextView.getText().toString() + "\n" + "读取到手环版本号为:" + characteristic.getStringValue(0));
					}
				});
            }
        }
    };
}

```

#### BLE的相关学习链接
http://www.freebuf.com/news/88281.html
http://www.zhaoxiaodan.com/android/%E5%B0%8F%E7%B1%B3%E6%89%8B%E7%8E%AF%E8%93%9D%E7%89%99%E5%8D%8F%E8%AE%AE%E7%A0%94%E7%A9%B6.html
http://www.race604.com/android-ble-tips/
http://www.cnblogs.com/Free-Thinker/p/4675388.html
https://developer.android.com/guide/topics/connectivity/bluetooth-le.html
https://github.com/pangliang/miband-sdk-android



