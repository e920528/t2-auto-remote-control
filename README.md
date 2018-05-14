# t2-auto-remote-control

## 簡介
本專案為使用tessel2實現的冷氣自動控制器，對應的冷氣型號為Hitachi的RAS-63QD</br>
當環境溫度過高時(預設為>30度C)，此控制器將送出開機訊號以開啟冷氣</br>
當環境溫度過低時(預設為<26度C)，此控制器將送出關機訊號以關閉冷氣</br>



## 使用說明

1. 使用`npm install`安裝所有的node_modules
2. 設定好你的tessel2，並將IR module插入port A，將climate module插入port B
</br>

## 系統架構
### 讀取環境溫度
```JavaScript
   setImmediate(function loop () {
      climate.readTemperature('f', function (err, temp) {
      climate.readHumidity(function (err, humid) {
      temperature = ((temp-32)/9*5).toFixed(1);
      console.log("Current temperature: " + temperature);
      setTimeout(loop, 60000);
      });
    });
```
本系統依靠tessel2官方的climate module來偵測環境的溫度，並將其存在一變數`temperature`中</br>
由於climate module預設的單位為華氏溫標，因此需要將module傳來的訊號換算求得攝氏溫標</br>
利用`setTimeout`使系統每分鐘更新一次`temperature`儲存的數值，以作為是否送出開(關)機訊號的判斷依據</br>

### 開關機訊號
```JavaScript
// poweron buffer
  var powerBufferOn = new Buffer([0x48,0xf9,0x5c,0x1,0x90,0xfa,0xec,0x1,0x90,0xfe,0xc,0x1,0x90,0xfe,0xc,0x1,0x90,0xfe,0xc,0x1,0x90,0xfe,0xc,0x1,0x90,0xfe,0xc,0x1,0x90,0xfe,0xc,0x1,0xc2,0xfe,0xc,0x1,0x90,0xfe,0xc,0x1,0x90,0xfe,0xc,0x1,0x90,0xfe,0xc,0x1,0x90,0xfe,0xc,0x1,0x90,0xfa,0xec,0x1,0x90,0xfe,0xc,0x1,0x90,0xfe,0xc,0x1,0x90,0xfe,0xc,0x1,0x90,0xfe,0xc,0x1,0x90,0xfe,0xc,0x1,0xc2,0xfe,0x3e,0x1,0xc2,0xfe,0xc,0x1,0x90,0xfe,0xc,0x1,0x90,0xfe,0xc,0x1,0x90,0xfe,0xc,0x1,0x90,0xfe,0xc,0x1]);
  //poweroff buffer
  var powerBufferOff = new Buffer([0x48,0xf9,0x8e,0x1,0xc2,0xfb,0x1e,0x1,0x90,0xfe,0xc,0x1,0x90,0xfe,0xc,0x1,0x90,0xfd,0xda,0x1,0x90,0xfe,0xc,0x1,0x90,0xfe,0xc,0x1,0x90,0xfe,0xc,0x1,0x90,0xfe,0xc,0x1,0x90,0xfe,0xc,0x1,0x90,0xfe,0xc,0x1,0x90,0xfe,0xc,0x1,0xc2,0xfe,0xc,0x1,0x90,0xfb,0x1e,0x1,0x90,0xfe,0xc,0x1,0xc2,0xfe,0x3e,0x1,0xc2,0xfe,0xc,0x1,0x90,0xfe,0xc,0x1,0x90,0xfe,0xc,0x1,0x90,0xfe,0xc,0x1,0x90,0xfe,0xc,0x1,0x90,0xfe,0xc,0x1,0x90,0xfe,0xc,0x1,0xc2,0xfe,0xc,0x1,0x90,0xfe,0xc,0x1]);
    
  setInterval(function () {
    //open airconditioner if current temperature is higher than 30 celsius degree
    if(temperature > 30){
      infrared.sendRawSignal(38, powerBufferOn, function(err) {
        if (err) {
          console.log("Unable to send signal: ", err);
        } else {
          console.log("power on Signal sent!");
        }
      });
    }
    //close airconditioner if current temperature is lower than 26 celsius degree
    if(temperature < 26){
      infrared.sendRawSignal(38, powerBufferOff, function(err) {
        if (err) {
          console.log("Unable to send signal: ", err);
        } else {
          console.log("power off Signal sent!");
        }
      });
    }
   
  }, 3000); // Every 3 seconds
  ```
  以tessel2官方提供的ir-module程式碼，測得冷氣遙控器開(關)機訊號，將之儲存於`powerBufferOn(powerBufferOff)`變數中。</br>
  當環境溫度高於30度時，系統會每隔三秒鐘送出開機訊號，直到溫度低於30度。</br>
  反之，當環境溫度低於26度時，系統會每隔三秒鐘送出關機訊號，直到溫度高於26度。</br>
  
