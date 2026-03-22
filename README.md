# Historical Court AI System
โปรเจกต์วิเคราะห์ประวัติศาสตร์ด้วยระบบ Multi-Agent (Google ADK)

## รายชื่อ Agent
- **Agent A (Admirer):** เน้นหาด้านบวกและความสำเร็จ
- **Agent B (Critic):** เน้นหาด้านลบและข้อโต้แย้ง
- **Agent C (Judge):** ตรวจสอบความสมดุลและสรุปรายงาน

## เทคนิคที่ใช้
- การทำงานแบบ Parallel (Agent A & B)
- การทำ Loop ตรวจสอบข้อมูลใน Session State
- การจบงานด้วย tool exit_loop
