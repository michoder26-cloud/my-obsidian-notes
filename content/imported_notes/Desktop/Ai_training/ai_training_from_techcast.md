# โน้ตเรียนรู้จากคลิป: รู้จัก AI ใน 10 นาที (แบบ Technical)

> แหล่งหลัก: YouTube `https://www.youtube.com/watch?v=843baKYqjGU`  
> ชื่อคลิป: `รู้จัก AI ใน 10 นาที (แบบ Technical)`  
> ช่อง: `Techcast`  
> ความยาวตามแหล่ง mirror: ประมาณ `00:11:28`  
> จุดประสงค์ไฟล์นี้: เก็บความรู้จากคลิปให้อยู่ในรูปแบบที่ AI ตัวอื่นอ่านต่อได้ เพื่อเข้าใจพื้นฐาน AI, Machine Learning, Deep Learning และแนวคิดการเทรนโมเดล

หมายเหตุ: ไฟล์นี้เป็นสรุปเชิงโครงสร้างและคู่มือเรียนรู้ ไม่ใช่ transcript คำต่อคำของคลิป

## สารบัญตามเวลาในคลิป

| เวลา | หัวข้อ |
|---|---|
| 0:00 | Intro |
| 0:15 | Sponsor: Grainey |
| 0:40 | ความหมายของ AI |
| 0:55 | GOFAI หรือ Symbolic AI |
| 1:11 | Machine Learning |
| 1:28 | Learning Types |
| 2:00 | Supervised Learning |
| 4:24 | Unsupervised Learning |
| 5:05 | Reinforcement Learning |
| 6:00 | Deep Learning |
| 7:24 | Underfitting และ Overfitting |
| 8:28 | ตัวอย่าง Model Architecture |
| 9:10 | ประเภท AI ในอุดมคติ |
| 10:02 | 5 Pillars of AI Ethics |

## ภาพรวมใหญ่ของคลิป

คลิปนี้อธิบาย AI แบบ technical ฉบับสั้น โดยไล่จากความหมายกว้างของ AI ไปสู่ Machine Learning, วิธีเรียนรู้ของโมเดล, ปัญหาการเทรน, Deep Learning, architecture บางแบบ และจริยธรรม AI

แกนหลักที่ต้องจำ:

- AI คือความพยายามทำให้เครื่องจักรหรือระบบคอมพิวเตอร์มีพฤติกรรมที่ดูฉลาด
- Machine Learning คือวิธีทำให้คอมพิวเตอร์เรียนรู้จากข้อมูลหรือประสบการณ์ แทนที่จะเขียนกฎทุกอย่างด้วยมือ
- การเทรน AI คือการปรับค่าภายในโมเดล เช่น weight และ bias ให้โมเดลทำนายหรือทำงานได้ดีขึ้น
- โมเดลที่ดีไม่ใช่แค่จำข้อมูลเทรนได้ แต่ต้อง generalize กับข้อมูลใหม่ได้
- Deep Learning เป็น Machine Learning แบบหนึ่งที่ใช้ neural network หลายชั้น และเป็นพื้นฐานสำคัญของ AI ยุคใหม่

## 1. AI คืออะไร

AI หรือ Artificial Intelligence คือศาสตร์และวิศวกรรมที่ทำให้เครื่องจักรฉลาด หรือทำให้ระบบคอมพิวเตอร์สามารถทำงานบางอย่างที่ต้องใช้ความสามารถเชิงสติปัญญาได้

นิยามนี้กว้างมาก จึงทำให้ระบบที่ใช้กฎง่าย ๆ เช่น `if-else` ก็อาจถูกมองเป็น AI แบบหนึ่งได้ ถ้าระบบนั้นทำงานคล้ายการตัดสินใจของมนุษย์

## 2. GOFAI / Symbolic AI

GOFAI ย่อมาจาก Good Old-Fashioned Artificial Intelligence หมายถึง AI ยุคดั้งเดิมที่ใช้กฎ สัญลักษณ์ ตรรกะ คณิตศาสตร์ และ algorithm ที่มนุษย์กำหนดไว้ล่วงหน้า

ตัวอย่างแนวคิด:

```text
ถ้าเงื่อนไข A เป็นจริง -> ทำ B
ถ้าเงื่อนไข A เป็นเท็จ -> ทำ C
```

ข้อดี:

- อธิบายได้ง่าย
- ควบคุมพฤติกรรมได้ตรง
- เหมาะกับปัญหาที่มีกฎชัดเจน

ข้อจำกัด:

- ต้องให้มนุษย์เขียนกฎเองจำนวนมาก
- ไม่ยืดหยุ่นกับสถานการณ์ใหม่
- ไม่เหมาะกับข้อมูลซับซ้อน เช่น ภาพ เสียง ภาษา หรือพฤติกรรมมนุษย์

จุดเปลี่ยนสำคัญคือ Machine Learning ซึ่งให้ระบบเรียนรู้ pattern จากข้อมูลเอง

## 3. Machine Learning คืออะไร

Machine Learning คือศาสตร์ที่ทำให้คอมพิวเตอร์เรียนรู้จากประสบการณ์หรือข้อมูล

การเขียนโปรแกรมแบบเดิม:

```text
Input + Rules/Program -> Output
```

Machine Learning:

```text
Input + Output ตัวอย่าง -> Algorithm เรียนรู้ Rules/Model
```

พูดง่าย ๆ คือเราไม่ได้เขียนกฎทั้งหมดเอง แต่ให้ algorithm หา pattern จากข้อมูล แล้วสร้าง model ที่ใช้ทำนายหรือตัดสินใจได้

## 4. Learning Types: วิธีเรียนรู้หลักของ Machine Learning

คลิปแบ่งการเรียนรู้หลักเป็น 3 แบบ:

- Supervised Learning
- Unsupervised Learning
- Reinforcement Learning

แต่ละแบบต่างกันที่ “ข้อมูลสอน” และ “สัญญาณ feedback” ที่โมเดลได้รับ

## 5. Supervised Learning

Supervised Learning คือการเรียนรู้จากข้อมูลที่มีคำตอบกำกับไว้แล้ว

รูปแบบข้อมูล:

```text
Input -> Correct Output
```

ตัวอย่าง:

- รูปภาพแมว -> label ว่า `cat`
- ขนาดบ้าน, ทำเล, จำนวนห้อง -> ราคาบ้าน
- ข้อความรีวิว -> positive/negative

เป้าหมาย:

```text
ให้โมเดลเรียนรู้ความสัมพันธ์ระหว่าง input และ output
```

### 5.1 Linear Regression

Linear Regression เป็นตัวอย่างพื้นฐานของ Supervised Learning ใช้ทำนายค่าต่อเนื่อง เช่น ราคา คะแนน ยอดขาย หรืออุณหภูมิ

สมการพื้นฐาน:

```text
y = wx + b
```

ความหมาย:

- `x` คือ input feature
- `y` คือ output ที่ต้องการทำนาย
- `w` คือ weight หรือค่าน้ำหนักของ feature
- `b` คือ bias หรือค่าคงที่ที่ช่วยเลื่อนเส้นทำนาย

การเทรน Linear Regression คือการหา `w` และ `b` ที่ทำให้เส้นทำนายใกล้ข้อมูลจริงมากที่สุด

### 5.2 Cost Function

Cost Function คือฟังก์ชันวัดว่าโมเดลผิดพลาดมากแค่ไหน

ถ้าโมเดลทำนายใกล้คำตอบจริง:

```text
cost ต่ำ
```

ถ้าโมเดลทำนายห่างจากคำตอบจริง:

```text
cost สูง
```

ใน Linear Regression มักใช้แนวคิด squared error หรือการยกกำลังสองของค่าความผิดพลาด เพื่อให้ความผิดพลาดมากถูกลงโทษมากขึ้น

### 5.3 Gradient Descent

Gradient Descent คือวิธีค่อย ๆ ปรับ parameter เช่น `w` และ `b` เพื่อลดค่า cost

แนวคิด:

```text
เริ่มจากค่า parameter สุ่มหรือค่าเริ่มต้น
คำนวณว่าโมเดลผิดเท่าไร
หาทิศทางที่ทำให้ error ลดลง
ขยับ parameter ไปทางนั้น
ทำซ้ำจน cost ต่ำพอ
```

คำว่า gradient หมายถึงความชันของ cost function ถ้ารู้ความชัน เราจะรู้ว่าควรปรับ parameter ไปทางไหน

### 5.4 Learning Rate

Learning Rate คือขนาดก้าวในการปรับ parameter แต่ละครั้ง

ถ้า learning rate สูงเกินไป:

- โมเดลอาจกระโดดข้ามจุดที่ดีที่สุด
- loss อาจแกว่งหรือไม่ลด

ถ้า learning rate ต่ำเกินไป:

- เทรนนานมาก
- โมเดลเรียนรู้ช้า

หลักจำ:

```text
learning rate = ปุ่มควบคุมความเร็วในการเรียนรู้
```

### 5.5 ถ้าข้อมูลไม่เป็นเส้นตรง

Linear Regression เหมาะกับข้อมูลที่ความสัมพันธ์ใกล้เคียงเส้นตรง ถ้าข้อมูลซับซ้อนกว่า อาจต้องใช้ model แบบอื่น เช่น

- Polynomial Regression
- Naive Bayes
- Decision Tree
- K-Nearest Neighbors
- Neural Network

บทเรียนสำคัญ:

```text
ไม่ใช่แค่โยนข้อมูลเข้าโมเดลแล้วจะได้ผลลัพธ์ดี
ต้องเข้าใจธรรมชาติของข้อมูล และเลือก model architecture ให้เหมาะ
```

## 6. Unsupervised Learning

Unsupervised Learning คือการเรียนรู้จากข้อมูลที่ไม่มีคำตอบกำกับ

รูปแบบข้อมูล:

```text
Input เท่านั้น ไม่มี label
```

เป้าหมาย:

- หา pattern
- จัดกลุ่มข้อมูล
- ลดมิติข้อมูล
- ค้นหาโครงสร้างที่ซ่อนอยู่

### 6.1 Clustering

ตัวอย่างที่คลิปพูดถึงคือ K-Means Clustering

แนวคิด:

```text
ให้โมเดลแบ่งข้อมูลออกเป็นกลุ่มตามความใกล้เคียงกัน
```

โมเดลอาจไม่รู้ว่ากลุ่มนั้นคืออะไรในความหมายมนุษย์ แต่สามารถบอกได้ว่าข้อมูลบางชุดมี pattern คล้ายกัน

ตัวอย่าง:

- แบ่งกลุ่มลูกค้าตามพฤติกรรมซื้อสินค้า
- แบ่งกลุ่มภาพที่มีลักษณะคล้ายกัน
- แบ่งกลุ่มเอกสารตามหัวข้อ

## 7. Reinforcement Learning

Reinforcement Learning คือการเรียนรู้จากการลองผิดลองถูก โดยมีรางวัลเป็น feedback

องค์ประกอบหลัก:

- `Agent` = ตัวโมเดลหรือตัวตัดสินใจ
- `Environment` = สภาพแวดล้อมที่ agent เข้าไปกระทำ
- `Action` = สิ่งที่ agent เลือกทำ
- `Reward` = คะแนนหรือผลตอบแทนจากการกระทำ
- `State` = สถานะที่ environment ส่งกลับมา

วงจรการเรียนรู้:

```text
Environment ให้ state
Agent เลือก action
Environment ให้ reward และ state ใหม่
Agent ปรับพฤติกรรมเพื่อเพิ่ม reward ในอนาคต
```

เหมาะกับงานที่หาข้อมูลสอนแบบ label ยาก แต่สามารถจำลอง environment ได้ เช่น

- เกม
- หุ่นยนต์
- ระบบควบคุม
- การวางแผน

ข้อดีคือ agent สามารถเรียนรู้กลยุทธ์จากประสบการณ์ของตัวเองได้ แต่ข้อจำกัดคือการสร้าง environment ที่ดีอาจยาก และการเทรนอาจใช้ทรัพยากรมาก

## 8. Deep Learning

Deep Learning คือ Machine Learning ที่ใช้ Neural Network หลายชั้น

แนวคิด neural network ได้แรงบันดาลใจจากการทำงานของสมอง โดยมีหน่วยย่อยคล้าย neuron เชื่อมต่อกันเป็น layer

องค์ประกอบหลัก:

- Input layer
- Hidden layers
- Output layer
- Weight
- Bias
- Activation function

แต่ละ node รับ input หลายค่า คูณด้วย weight รวมกับ bias แล้วส่งผลต่อไปยัง layer ถัดไป

สมการในระดับ node มีแนวคิดคล้าย:

```text
output = activation(w1*x1 + w2*x2 + ... + b)
```

เมื่อ stack หลาย layer จะกลายเป็น Deep Neural Network ซึ่งสามารถเรียนรู้ pattern ซับซ้อนได้ เช่น ภาพ เสียง ภาษา และข้อมูลหลายมิติ

## 9. ทำไม Deep Learning สำคัญ

Deep Learning เป็นฐานของ AI ยุคใหม่จำนวนมาก เช่น

- Chatbot
- Image generation
- Speech recognition
- Translation
- Recommendation system
- Multimodal AI

จุดแข็ง:

- เรียนรู้ feature ซับซ้อนเองได้
- ยิ่งมีข้อมูลและ compute มาก มักยิ่งทำงานได้ดีขึ้น
- รองรับข้อมูลหลายประเภท เช่น text, image, audio, video

จุดอ่อน:

- ใช้ข้อมูลเยอะ
- ใช้ compute สูง
- อธิบายการตัดสินใจได้ยาก
- อาจเกิด bias จากข้อมูล
- เทรนผิดอาจ overfit หรือได้โมเดลที่ใช้จริงไม่ดี

## 10. Underfitting และ Overfitting

ปัญหาสำคัญในการเทรนโมเดลคือโมเดลอาจเรียนรู้น้อยไปหรือมากไป

### 10.1 Underfitting

Underfitting คือโมเดลเรียนรู้ pattern ไม่พอ

อาการ:

- ทำนายข้อมูลเทรนก็ยังไม่ดี
- ทำนายข้อมูลใหม่ก็ไม่ดี
- model ง่ายเกินไป
- train loss สูง

สาเหตุที่เป็นไปได้:

- model capacity ต่ำเกิน
- feature ไม่พอ
- เทรนน้อยเกิน
- learning rate ไม่เหมาะ

### 10.2 Overfitting

Overfitting คือโมเดลจำข้อมูลเทรนมากเกินไป แต่ใช้งานกับข้อมูลใหม่ไม่ดี

อาการ:

- train loss ต่ำมาก
- validation loss เริ่มสูงขึ้น
- performance บนข้อมูลใหม่แย่

สาเหตุที่เป็นไปได้:

- model ซับซ้อนเกิน
- parameter เยอะเกินเมื่อเทียบกับข้อมูล
- train นานเกิน
- dataset เล็กหรือไม่หลากหลาย

## 11. การแบ่งข้อมูลเพื่อเทรนและทดสอบ

เพื่อป้องกันการหลอกตัวเองว่าโมเดลเก่ง ควรแบ่งข้อมูลเป็น 3 ชุด

### 11.1 Training Set

ใช้สำหรับให้โมเดลเรียนรู้และปรับ parameter

### 11.2 Validation Set

ใช้วัดระหว่างเทรน เพื่อดูว่าโมเดล generalize กับข้อมูลที่ไม่เคยเห็นได้ดีแค่ไหน

ถ้า training loss ลด แต่ validation loss เพิ่ม:

```text
มีแนวโน้ม overfitting
```

### 11.3 Test Set

ใช้ประเมินครั้งสุดท้ายหลังเลือกโมเดลแล้ว ไม่ควรใช้ test set ในการปรับโมเดลระหว่างทาง

## 12. วิธีคิดการเทรน AI แบบเป็นขั้นตอน

ใช้ flow นี้เพื่อให้ AI ตัวอื่นเข้าใจวิธีเทรนโมเดล:

```text
1. กำหนดปัญหา
2. นิยาม input และ output
3. รวบรวมข้อมูล
4. ทำความสะอาดข้อมูล
5. แบ่ง train / validation / test
6. เลือก learning type
7. เลือก model architecture
8. เลือก loss หรือ cost function
9. เลือก optimizer เช่น gradient descent
10. ตั้ง hyperparameters เช่น learning rate, batch size, epoch
11. เทรนโมเดล
12. วัดผลบน validation set
13. แก้ underfitting หรือ overfitting
14. ประเมินครั้งสุดท้ายด้วย test set
15. นำไปใช้งานจริงและ monitor ต่อ
```

## 13. ตัวอย่าง Model Architecture ที่คลิปพูดถึง

### 13.1 GAN: Generative Adversarial Network

GAN ใช้โมเดลสองตัวแข่งกัน:

- Generator = สร้างข้อมูลปลอม เช่น ภาพ
- Discriminator = แยกว่าข้อมูลจริงหรือปลอม

แนวคิด:

```text
Generator พยายามหลอก Discriminator
Discriminator พยายามจับให้ได้ว่าอะไรจริงอะไรปลอม
ทั้งสองตัวพัฒนาขึ้นจากการแข่งขันกัน
```

เหมาะกับงาน generation เช่น สร้างภาพหรือข้อมูลสังเคราะห์

### 13.2 Encoder / Decoder

Encoder / Decoder เป็น architecture ที่แปลงข้อมูลเป็น representation ภายใน แล้วแปลงกลับเป็น output

ตัวอย่างกับภาพ:

```text
ภาพ input -> Encoder -> vector/latent representation -> Decoder -> ภาพ output
```

การใช้งาน:

- ลด noise ภาพหรือเสียง
- compression
- image reconstruction
- translation
- sequence-to-sequence task

แนวคิดสำคัญคือ encoder บีบข้อมูลให้กลายเป็น vector ที่เก็บสาระสำคัญ ส่วน decoder ขยายกลับเป็นรูปแบบ output ที่ต้องการ

## 14. ประเภท AI ในอุดมคติ

คลิปพูดถึงการแบ่ง AI ตามระดับความสามารถ

### 14.1 ANI: Artificial Narrow Intelligence

AI ที่เก่งเฉพาะงานแคบ ๆ อย่างใดอย่างหนึ่ง

ตัวอย่าง:

- โมเดลแยกภาพ
- โมเดลแปลภาษา
- โมเดลแนะนำสินค้า
- โมเดลเล่นเกมบางเกม

AI ส่วนใหญ่ในปัจจุบันอยู่ในกลุ่มนี้ แม้จะดูเก่งมากในบางงาน

### 14.2 AGI: Artificial General Intelligence

AI ที่มีความสามารถทั่วไป ทำงานได้หลายด้าน ยืดหยุ่น และเรียนรู้ข้ามงานได้คล้ายมนุษย์

ตัวอย่างแนวคิด:

- รับข้อความแล้วสร้างภาพ
- วิเคราะห์ปัญหาใหม่
- เรียนรู้งานใหม่โดยใช้ข้อมูลน้อย
- ใช้เหตุผลข้าม domain

### 14.3 ASI: Artificial Super Intelligence

AI ที่ฉลาดเหนือมนุษย์ในระดับกว้าง เป็นแนวคิดเชิงอนาคตและยังมีประเด็นถกเถียงมาก

ต้องระวังการใช้คำว่า ASI เพราะเกี่ยวข้องกับความเสี่ยงด้านความปลอดภัย จริยธรรม และการควบคุม

## 15. 5 Pillars of AI Ethics

การสร้าง AI ไม่ใช่แค่ทำให้โมเดลแม่น แต่ต้องรับผิดชอบผลกระทบด้วย

### 15.1 Transparency

ความโปร่งใส เช่น

- ข้อมูลมาจากไหน
- โมเดลตัดสินใจอย่างไร
- ระบบมีข้อจำกัดอะไร
- ผู้ใช้ควรรู้อะไรก่อนเชื่อผลลัพธ์

### 15.2 Fairness

ความเป็นธรรม เช่น

- ไม่เลือกปฏิบัติจากเพศ สีผิว เชื้อชาติ อายุ หรือสถานะทางสังคม
- ตรวจ bias ใน dataset
- ประเมิน performance แยกตามกลุ่มผู้ใช้

### 15.3 Privacy

ความเป็นส่วนตัว เช่น

- ไม่ใช้ข้อมูลส่วนบุคคลโดยไม่มีเหตุผล
- จำกัดข้อมูลที่เก็บ
- ปกป้องข้อมูลระหว่าง train และ deploy
- ให้ผู้ใช้ควบคุมข้อมูลของตนเอง

### 15.4 Accountability

ความรับผิดชอบ เช่น

- ใครรับผิดชอบเมื่อ AI ตัดสินใจผิด
- มีระบบตรวจสอบหรือ appeal หรือไม่
- มี human oversight ในงานเสี่ยงสูงหรือไม่

### 15.5 Sustainability

ความยั่งยืน เช่น

- การเทรนใช้พลังงานเท่าไร
- มี carbon footprint สูงหรือไม่
- คุ้มค่ากับผลลัพธ์หรือไม่
- ใช้ทรัพยากรอย่างรับผิดชอบหรือไม่

## 16. Cheat Sheet: คำศัพท์สำคัญสำหรับ AI ตัวอื่น

| คำ | ความหมายสั้น |
|---|---|
| AI | ระบบที่ทำงานคล้ายต้องใช้ความฉลาด |
| GOFAI | AI แบบใช้กฎและตรรกะที่มนุษย์เขียนไว้ |
| Machine Learning | ให้คอมพิวเตอร์เรียนรู้จากข้อมูล |
| Model | ฟังก์ชันหรือระบบที่เรียนรู้ pattern แล้วใช้ทำนาย |
| Training | กระบวนการปรับ parameter ให้โมเดลดีขึ้น |
| Parameter | ค่าภายในโมเดลที่เรียนรู้ได้ เช่น weight, bias |
| Hyperparameter | ค่าที่มนุษย์ตั้งก่อนเทรน เช่น learning rate |
| Feature | คุณสมบัติของข้อมูล input |
| Label | คำตอบที่ใช้สอนใน supervised learning |
| Cost/Loss | ค่าความผิดพลาดของโมเดล |
| Gradient Descent | วิธีปรับ parameter เพื่อลด loss |
| Learning Rate | ขนาดก้าวในการปรับ parameter |
| Epoch | การเทรนครบ dataset หนึ่งรอบ |
| Overfitting | จำข้อมูลเทรนมากเกิน ใช้จริงไม่ดี |
| Underfitting | เรียนรู้น้อยเกิน ใช้ทั้ง train/test ไม่ดี |
| Validation Set | ข้อมูลใช้ประเมินระหว่างเลือกโมเดล |
| Test Set | ข้อมูลใช้ประเมินสุดท้าย |
| Neural Network | โครงข่ายคำนวณหลาย node คล้าย neuron |
| Deep Learning | Neural network หลายชั้น |
| Encoder | ส่วนที่บีบข้อมูลเป็น representation |
| Decoder | ส่วนที่แปลง representation กลับเป็น output |
| GAN | โมเดลสร้างข้อมูลที่ใช้ generator แข่งกับ discriminator |

## 17. สรุปสำหรับ AI Agent ที่ต้องเรียนรู้วิธีเทรน AI

ถ้าต้องสอน AI ตัวอื่นจากคลิปนี้ ให้เน้น 7 ประโยคนี้:

1. AI คือระบบที่ทำให้เครื่องจักรทำงานเหมือนมีความฉลาด
2. Machine Learning คือวิธีให้ระบบเรียนรู้กฎจากข้อมูล แทนการเขียนกฎเองทั้งหมด
3. การเทรนคือการปรับ parameter เพื่อลด error หรือ loss
4. Supervised Learning ใช้ข้อมูลที่มีคำตอบ, Unsupervised Learning ใช้ข้อมูลไม่มีคำตอบ, Reinforcement Learning ใช้ reward จาก environment
5. Gradient Descent คือวิธีค่อย ๆ ปรับ parameter ไปทางที่ทำให้ loss ลดลง
6. โมเดลที่ดีต้องไม่ underfit และไม่ overfit จึงต้องใช้ train/validation/test split
7. Deep Learning ใช้ neural network หลายชั้นและเป็นฐานของ AI สมัยใหม่ แต่ต้องคำนึงถึงจริยธรรม ความโปร่งใส ความเป็นธรรม ความเป็นส่วนตัว ความรับผิดชอบ และความยั่งยืน

## 18. Prompt สำหรับให้ AI ตัวอื่นเรียนจากไฟล์นี้

ใช้ prompt นี้ได้:

```text
อ่านไฟล์นี้เป็น knowledge base เรื่องพื้นฐาน AI และการเทรนโมเดล

หน้าที่ของคุณ:
1. เข้าใจความต่างระหว่าง AI, Machine Learning, Deep Learning
2. อธิบาย learning types ทั้ง 3 แบบได้
3. อธิบาย training loop, cost function, gradient descent, learning rate ได้
4. รู้วิธีตรวจ underfitting และ overfitting
5. รู้ว่าควรแบ่ง train/validation/test อย่างไร
6. เมื่อตอบคำถามเรื่องเทรน AI ให้ตอบเป็นขั้นตอน ใช้ภาษาง่าย และเตือนเรื่อง ethics เมื่อเหมาะสม

ข้อห้าม:
- อย่าบอกว่าแค่ใส่ข้อมูลเข้าโมเดลแล้วจะได้ AI ที่ดีทันที
- อย่าข้าม validation/test set
- อย่าอ้างว่าโมเดลแม่นโดยไม่มี metric
- อย่าลืมพิจารณา bias, privacy และ accountability
```

## 19. แหล่งอ้างอิงที่ใช้สร้างโน้ตนี้

- YouTube: `https://www.youtube.com/watch?v=843baKYqjGU`
- Metadata จาก YouTube oEmbed: ชื่อคลิปและช่อง `Techcast`
- Rutube mirror/description: timestamp และคำอธิบายคลิปเดียวกัน
- Medium summary: `สรุปสิ่งที่ได้จากคลิป รู้จัก AI ใน 10 นาที (แบบ Technical)` โดย MRPROPZ

