# แผนเรียนสำหรับ AI คณิตศาสตร์สาย Trading

เอกสารนี้คือแผนเรียนแบบค่อยเป็นค่อยไป จุดประสงค์คือทำให้ AI เข้าใจข้อมูลราคาในเชิงคณิตศาสตร์ก่อน ยังไม่ต้องเทรดจริง

## Phase 0: กฎความปลอดภัย

ตอนนี้ห้ามทำสิ่งต่อไปนี้:

- ห้ามต่อ API เพื่อส่งคำสั่งซื้อขาย
- ห้ามใช้เงินจริง
- ห้ามสร้างระบบ auto trade
- ห้ามเชื่อผล backtest โดยไม่มีการทดสอบแยกชุดข้อมูล

## Phase 1: เข้าใจข้อมูลราคา

ข้อมูลราคาพื้นฐานมักมีคอลัมน์:

```text
open
high
low
close
volume
```

ความหมาย:

- `open` = ราคาเปิดของช่วงเวลา
- `high` = ราคาสูงสุดของช่วงเวลา
- `low` = ราคาต่ำสุดของช่วงเวลา
- `close` = ราคาปิดของช่วงเวลา
- `volume` = ปริมาณการซื้อขาย

## Phase 2: สร้าง feature พื้นฐาน

Feature คือค่าที่คำนวณจากข้อมูลดิบ เพื่อให้ AI เห็น pattern ง่ายขึ้น

Feature ที่ควรเริ่ม:

- simple return
- log return
- rolling mean
- rolling volatility
- momentum
- z-score
- drawdown
- volume change

## Phase 3: สร้าง label แบบไม่เทรดจริง

Label คือคำตอบที่ใช้สอนโมเดล เช่น:

```text
ถ้าราคา close ในอนาคตสูงกว่าปัจจุบัน -> 1
ถ้าไม่สูงกว่า -> 0
```

ตัวอย่าง:

```text
future_return = close[t+1] / close[t] - 1
label = 1 ถ้า future_return > 0
label = 0 ถ้า future_return <= 0
```

## Phase 4: เทรนโมเดลง่าย

เมื่อมี feature และ label แล้ว ค่อยใช้โมเดลง่าย ๆ เช่น:

- Logistic Regression
- Random Forest
- Gradient Boosting

ยังไม่ต้องใช้ deep learning ในช่วงแรก เพราะโมเดลง่ายช่วยให้ตรวจ error ได้ง่ายกว่า

## Phase 5: ประเมินผล

การวัดผลไม่ควรดู accuracy อย่างเดียว ต้องดู:

- precision
- recall
- confusion matrix
- performance แยกช่วงเวลา
- ความเสถียรของผลลัพธ์

เมื่อเริ่มมี strategy ในอนาคต ค่อยเพิ่ม:

- fee
- slippage
- max drawdown
- Sharpe ratio
- Sortino ratio

## Phase 6: ป้องกันการหลอกตัวเอง

ต้องระวัง:

- data leakage: เผลอใช้ข้อมูลอนาคต
- overfitting: จำอดีตมากเกินไป
- survivorship bias: ใช้เฉพาะสินทรัพย์ที่รอด
- regime change: ตลาดเปลี่ยนนิสัย

หลักจำ:

```text
อย่าสร้าง AI ที่ชนะอดีตอย่างเดียว ต้องสร้าง AI ที่รอดกับข้อมูลที่ไม่เคยเห็น
```

