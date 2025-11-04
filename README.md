# farm_system_union(IoTogether) / 6주차 기술보고서
####   - (산업시스템공학과 2022112386 김호준)


## 진정한 시작

 본 조의 산출물 주제는 대학생들을 위한 강의실 환경 파악 시스템을 구성하는 것이다. 그만큼 공부를 하기 위한 환경이 좋은 곳을 찾아야 하는데, 해당 정보들을 수집하는 알맞은 센서들을 장착하여 작동시켜야 한다.

 그래서 이번 주는 실질적인 정보를 주는 온습도 센서(DHT11)를 장착하여 그 결과를 확인해보는 활동을 진행해보았다.

 참고로 온습도 센서를 이용하는 이유는, 일단 공부를 위한 장소로 소음 측정도 가능하지만, 공부를 하는 데에 온도와 습도도 영향을 줄 것이라 판단하였기 때문이다.

 필자도 너무나 더울 때 또는 추울 때 집중력이 줄어들어 공부하기가 어려웠던 경험이 있었고 습도도 마찬가지로 공부에 영향을 받은 적이 있었다.

---

## 온습도 센서 활용

 먼저 우리의 시스템과 결합하기 전에, 아두이노에 온습도 센서 하나만 먼저 온습도 센서가 잘 작동되는지의 여부를 먼저 확인해보았다. 이번 활동은 교재의 도움을 많이 받았다.

 우선 해당 센서를 이용하기 위해서는 "DHT sensor library"라는 라이브러리를 먼저 IDE 내에서 다운로드 받아줘야 한다.

 <div align="center">
  <img width="241" src="https://github.com/user-attachments/assets/dd76172d-6df2-423c-ae2e-9c3598de12c8" />
 </div>

 그다음 코드 내에서 해당 라이브러리를 이전 lcd 설치 작업과 비슷하게 "include"로 삽입해주고 적절한 코드를 작성하여 온도와 습도 모두 파악할 수 있다.

 <div align="center">
  <img width="700" src="https://github.com/user-attachments/assets/4625043e-e681-40cb-b98b-b29eab0e54c2" />
 </div>

 위의 코드에서 확인할 수 있듯이, 헤더 이름은 "DHT.h"로 확인되고 온습도는 각각의 코드로 파악할 수 있는 것으로 확인된다.

 온도 : "dht.readTemperature()"   /    습도 : "dht.readHumidity()"

 실행 결과는 다음과 같이 잘 나오는 것을 확인해볼 수 있다.
 <div align="center">
  <img width="700" src="https://github.com/user-attachments/assets/001e6f93-be98-4232-aa59-efe895a91dbe" />
 </div>
 
---

## 응용(기존 시스템과의 결합)

 이제는 거리를 측정해야하는 일이 사라졌으니 초음파 센서를 제거하고 대신 온습도 센서를 장착하여 제대로 연동이 되는지 알아보도록 하자.

 추가해야 하는 코드들은 다음과 같다.

 ```
#include <DHT.h>

#define DHTPIN D8
#define DHTTYPE DHT11

DHT dht(DHTPIN, DHTTYPE);

void setup() {

  // LCD 디스플레이 출력 내용 수정
  lcd.init();
  lcd.backlight();
  lcd.setCursor(0, 0);
  lcd.print("Tem C : ");
  lcd.setCursor(0, 1);
  lcd.print("Hum % : ");

  dht.begin();

}

void loop() {

  // 데이터 수집
  float humi = dht.readHumidity();
  float temp = dht.readTemperature();

  Serial.print(temp);Serial.print("C ");
  Serial.print(humi);Serial.println("%");

  // 데이터 출력
  if (temp >= 30 || humi >= 70){ # 온도는 30도 이상, 습도는 70% 이상
    Serial.println("공부하기에 어렵습니다.");
    lcd.setCursor(9, 0);
    lcd.print(temp);
    lcd.setCursor(9, 1);
    lcd.print(humi);

    if (WiFi.status() != WL_CONNECTED){
      ~~~;
    }
    else {
      sprintf(message, "공부하기에 어렵습니다. : %f C / %f %", temp, humi);
    }

 ```

 기존 시스템의 거리와 관련된 모든 항목들을 모두 온도와 습도로 수정하는 과정이 필요하다.

 우선 LCD 디스플레이에 첫 번째 줄은 온도를, 두 번째 줄은 습도를 출력할 수 있도록 작성하였다.

 이번에는 텔레그램에 출력하는 내용으로 만약 온도 또는 습도가 특정 수치 이상 올라간다면 텔레그램에 알림이 갈 수 있도록 작성된 모습을 확인할 수 있다.

 - 초음파 센서 대신 온습도 센서가 결합된 아두이노의 모습이다.
   
   <div align="center">
    <img width="720" src="https://github.com/user-attachments/assets/64b717c0-6e99-4316-9578-0e01b983b8fd" />
   </div>
   
---

 ## 실행 결과

  큰 환경 자체에 큰 이상이 없을 시, 다음과 같이 수집된 온도와 습도 데이터가 출력되는 모습을 확인할 수 있다.
  
  <div align="center">
   <img width="1000" src="https://github.com/user-attachments/assets/09141a6b-e571-40ab-8f53-81661798d389" />
  </div>

  특정 조건(온도는 30도 이상, 습도는 70% 이상)이 되면 LCD 디스플레이, 시리얼 모니터, 텔레그램 알림의 모습이다. 다행히 모두 같은 수치를 잘 시각화해주는 모습을 확인해 줄 수 있었다.

  텔레그램의 문자를 보니 각 출력문은 습도가 70%를 넘겼기에 문구가 출력됨을 알 수 있다.

- LCD 디스플레이

  <div align="center">
   <img width="750" src="https://github.com/user-attachments/assets/02397d78-06d9-41d4-ae15-27cfa956a2a6" />
  </div>
  
- 시리얼 모니터
  
  <div align="center">
   <img width="750" src="https://github.com/user-attachments/assets/7ff7605c-667c-40f9-885d-3cd36789ccdf" />
  </div>
  
- 텔레그램
  
  <div align="center">
   <img width="720" src="https://github.com/user-attachments/assets/03b76e46-d8fd-40ff-9587-cde0871bf386" />
  </div>

 ## 느낀 점

  DHT11 센서도 또한 필자가 어드벤처디자인 과목에서 한 번 이용해보고 더 이상 사용하지 않았던 센서이기에, 이번 결과가 제대로 나올 수 있었을지 걱정을 많이 하였다.

  다행히 출력되는 모든 기기에서 정상적으로 수치를 잘 받을 수 있었고, 이를 위하여 코드를 다시 수정해보고 적절한 결과를 얻기 위해 노력하는 과정이 재미가 있었다.
