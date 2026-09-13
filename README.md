# 🤖 AI-Powered Auto Task Allocator

ระบบช่วยย่อยโปรเจกต์ใหญ่ให้เป็นงานย่อย (**Micro-tasks**) ด้วย AI (LLM Structured Outputs) พร้อมระบบ **Automated Workload Allocation Algorithm** จัดสรรงานให้สมาชิกในทีมอัตโนมัติ โดยอิงจากความถนัด (Skill Matching Score) และช่วงเวลาว่าง (Capacity) เพื่อขจัดปัญหาความเหลื่อมล้ำ การเกี่ยงงาน และความขัดแย้งในองค์กร

---

## ✨ Features

- **Project Decomposition**: แปลงความต้องการโปรเจกต์ภาษามนุษย์ให้กลายเป็นโครงสร้าง Micro-tasks พร้อมประเมินเวลา (Hours) และระดับความสำคัญ (Priority)
- **Smart Skill Matching**: เปรียบเทียบ ทักษะที่งานต้องการ กับ ทักษะของสมาชิกในทีม เพื่อจับคู่คนที่เหมาะสมที่สุด
- **Conflict-Free Capacity Planning**: คำนวณชั่วโมงงานที่เหลือของแต่ละคน ป้องกันการกระจายงานหนักเกินไป (Overload) ให้ใครคนใดคนหนึ่ง
- **Structured JSON Engine**: พึ่งพา Pydantic เพื่อรับประกัน Schema ข้อมูลที่แม่นยำ ไร้ข้อผิดพลาด

---

## 🚀 Quick Start

### 1. Clone Repository & Install Dependencies

```bash
git clone [https://github.com/your-username/ai-task-allocator.git](https://github.com/your-username/ai-task-allocator.git)
cd ai-task-allocator
pip install -r requirements.txt
```

### 2. Set OpenAI API Key

```bash
# สำหรับ Linux / macOS
export OPENAI_API_KEY="your-api-key-here"

# สำหรับ Windows (PowerShell)
$env:OPENAI_API_KEY="your-api-key-here"
```

*หมายเหตุ: หากไม่ได้ใส่ API Key ระบบจะรันในรูปแบบ **Mock Mode** อัตโนมัติ เพื่อให้ทดสอบ Logic การจัดสรรงานได้ทันที*

### 3. Run the Application

```bash
python main.py
```

---

## ⚙️ How It Works (การทำงานของระบบ)

1. **Decomposition Phase**: ระบบส่งคำอธิบายโปรเจกต์ไปยัง LLM (`gpt-4o-mini`) เพื่อสกัดข้อมูลออกมาเป็นชุด `MicroTask`
2. **Scoring Phase**: อัลกอริทึมคำนวณคะแนน Match Score จากสูตร:
   $$\text{Score} = (\text{Skill Match Ratio} \times 0.7) + (\text{Availability Score} \times 0.3)$$
3. **Allocation Phase**: มอบหมายงานโดยเรียงลำดับจากความสำคัญสูงลงมา (Priority-based) และตัดยอดชั่วโมงว่างของสมาชิกแบบ Real-time

---

## 📄 License

Distributed under the MIT License. See `LICENSE` for more information.
