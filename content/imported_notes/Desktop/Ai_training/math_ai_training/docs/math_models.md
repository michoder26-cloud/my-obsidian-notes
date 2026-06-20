# สูตรคณิตศาสตร์พื้นฐานสำหรับ AI Trading

เอกสารนี้เก็บสูตรที่ใช้แปลงข้อมูลราคาเป็น feature ให้โมเดลเรียนรู้

## 1. Simple Return

วัดการเปลี่ยนแปลงของราคาจากช่วงก่อนหน้า

```text
return[t] = close[t] / close[t-1] - 1
```

ใช้เพื่อดูว่าราคาเพิ่มหรือลดกี่เปอร์เซ็นต์

## 2. Log Return

ใช้ logarithm เพื่อให้ return รวมกันตามเวลาได้สะดวกกว่า simple return

```text
log_return[t] = ln(close[t] / close[t-1])
```

เหมาะกับงาน time series และสถิติ

## 3. Rolling Mean

ค่าเฉลี่ยย้อนหลังตาม window

```text
rolling_mean[t] = mean(close[t-window+1 ... t])
```

ใช้ดูแนวโน้มระยะสั้นหรือระยะยาว

## 4. Rolling Volatility

ส่วนเบี่ยงเบนมาตรฐานของ return ย้อนหลัง

```text
volatility[t] = std(return[t-window+1 ... t])
```

ใช้ดูความผันผวนของตลาด

## 5. Momentum

วัดว่าราคาปัจจุบันเปลี่ยนจากอดีตมากแค่ไหน

```text
momentum[t] = close[t] / close[t-window] - 1
```

ถ้าค่าสูง แปลว่าราคาเพิ่มขึ้นจากอดีต

## 6. Z-Score

วัดว่าราคาปัจจุบันห่างจากค่าเฉลี่ยย้อนหลังมากแค่ไหนในหน่วยส่วนเบี่ยงเบนมาตรฐาน

```text
z_score[t] = (close[t] - rolling_mean[t]) / rolling_std[t]
```

ใช้ช่วยดูภาวะราคาสูงหรือต่ำผิดปกติเมื่อเทียบกับอดีต

## 7. Drawdown

วัดการลดลงจากจุดสูงสุดก่อนหน้า

```text
drawdown[t] = close[t] / max(close[0 ... t]) - 1
```

ใช้ประเมินความเสี่ยงและช่วงที่สินทรัพย์ตกจากยอดเดิม

## 8. Future Return Label

ใช้สร้างคำตอบให้โมเดลเรียนรู้ โดยดูผลตอบแทนในอนาคต

```text
future_return[t] = close[t+horizon] / close[t] - 1
label[t] = 1 ถ้า future_return[t] > threshold
label[t] = 0 ถ้า future_return[t] <= threshold
```

`horizon` คือจำนวนแท่งเวลาที่มองไปข้างหน้า เช่น 1, 3, 5, 10

ข้อควรระวัง: label ใช้ข้อมูลอนาคตได้เฉพาะตอนสร้างชุดฝึกเท่านั้น ห้ามเอา future_return มาเป็น feature ตอนใช้งานจริง

