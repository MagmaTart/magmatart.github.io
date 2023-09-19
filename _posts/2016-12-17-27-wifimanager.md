---
layout: article
title:  "WifiManager.getScanResult()의 Empty List 반환"
date:   2016-12-17 12:00:00 -0400
modify_date: 2016-12-17 12:00:00 -0400
tags:
- Android
category: 
- Development
use_math: true
---

`WifiManager.getScanResult()` 메소드가 빈 리스트를 반환하는 문제를 알아봅니다.

<!--more-->
-----
주변 Wi-Fi를 스캔하는 어플리케이션을 만들고 있었습니다. 가장 보편적으로 보이는 방법인 `WifiManager`를 통한 스캐닝을 구현 중이었습니다.

```java
mWifiManager = 
(WifiManager)getApplicationContext().getSystemService(Context.WIFI_SERVICE)
WifiManager 객체를 만들어 주었고, onResume() 메소드에서 Broadcast Receiver를 등록해 주었습니다.
protected void onResume(){
    super.onResume();
    mIntentFilter = new IntentFilter(WifiManager.SCAN_RESULTS_AVAILABLE_ACTION);
    mIntentFilter.addAction(WifiManager.NETWORK_STATE_CHANGED_ACTION);
    getApplicationContext().registerReceiver(wifiReceiver, mIntentFilter);

    mWifiManager.startScan();
}
```

와이파이를 감지하기 위한 `IntentFilter`를 브로드캐스트 리시버에 같이 등록하고, `WifiManager.startScan()`을 호출하여 와이파이 스캔을 시작했습니다.

그리고 리시버를 구현했습니다.

```java
private BroadcastReceiver wifiReceiver = new BroadcastReceiver() {
        @Override
        public void onReceive(Context context, Intent intent) {
            final String action = intent.getAction();

            if(action.equals(WifiManager.SCAN_RESULTS_AVAILABLE_ACTION)){
                scanDatas = mWifiManager.getScanResults();
            }
        }
    };
```

일단 리시버가 호출되면 전달받은 intent에서 발생 액션을 가져옵니다. 와이파이 스캔 결과를 받아올 수 있는 경우 `WifiManager.getScanResults()`를 통해 `List<ScanResult> scanDatas`에 스캔된 AP 리스트를 넣어줍니다.

그런데 여기서 문제가 발생합니다. `getScanResults()`가 텅텅 빈 List를 반환했던 것이죠. 처음에는 퍼미션 문제인 줄 알았습니다. 안드로이드 6.0 에서 변경된 퍼미션 정책 때문인가 하고 런타임 퍼미션을 받아오는 부분을 구현했으나, 그 문제가 아니었습니다.

해결책을 찾아냈는데, 바로 와이파이 스캔 결과를 받기 위해서는 GPS 정보가 필요하다는 것이었습니다. 하지만 코드에 직접 GPS를 이용하는 부분은 구현하지 않습니다. `AndroidManifest.xml`에 다음 퍼미션을 추가하고, 이 퍼미션을 받아오기만 하면 됩니다.

```xml
<uses-permission android:name="android.permission.ACCESS_COARSE_LOCATION"/>
```

익히 아시듯이, 저 퍼미션은 GPS를 통해 현재 디바이스의 대략적인 위치를 가져올 수 있게 하는 퍼미션입니다.

그런데, 대체 왜 Wifi 스캔 결과에 GPS 정보가 필요할까요?
이건 제 추측이지만, `ScanResult` 클래스의 필드 중 하나인 `venueName`과 연관이 있는 것 같습니다.

<p align="center"><image src="/assets/posts/images/27/figure1.png"/></p>

`venueName` 필드는 AP가 위치한 대략적인 장소의 이름을 나타낸다고 이야기합니다. `getScanResult()` 메소드에서 이 필드에 값을 채우는 과정에서 GPS가 필요할 것이라고 추측 할 수 있습니다.
