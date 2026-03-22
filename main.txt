import operator
from typing import Annotated, TypedDict, List
from langgraph.graph import StateGraph, END

# 1. State Management (แยก Key ชัดเจนตามโจทย์รูปที่ 2)
class AgentState(TypedDict):
    topic: str
    pos_data: Annotated[list, operator.add] 
    neg_data: Annotated[list, operator.add] 
    verdict: str
    # ตัด loop_count ออก แล้วใช้ len(pos_data) เช็คแทนจะเสถียรกว่า

# 2. Agent A (The Admirer)
def agent_admirer(state: AgentState):
    topic = state['topic']
    # โจทย์สั่ง: ปรับปรุง Instruction ให้เติมคำค้นหา (เช่น achievements)
    search_query = f"{topic} achievements" 
    print(f"--- Agent A ค้นหาด้านบวก: {search_query} ---")
    
    # จำลองการได้ข้อมูลใหม่ (ในงานจริงตรงนี้ต้องใช้ Google Search/Wiki Tool)
    return {"pos_data": [f"พบความสำเร็จของ {topic} เพิ่มเติม"]}

# 3. Agent B (The Critic)
def agent_critic(state: AgentState):
    topic = state['topic']
    # โจทย์สั่ง: ปรับปรุง Instruction ให้เติมคำค้นหา (เช่น controversy)
    search_query = f"{topic} controversy"
    print(f"--- Agent B ค้นหาด้านลบ: {search_query} ---")
    
    return {"neg_data": [f"พบข้อโต้แย้งของ {topic} เพิ่มเติม"]}

# 4. Agent C (The Judge) - จุดตัดสินชะตากรรม Loop
def judge_logic(state: AgentState):
    print(f"--- Judge ตรวจสอบ: บวก({len(state['pos_data'])}) ลบ({len(state['neg_data'])}) ---")
    
    # เงื่อนไข: ถ้าฝั่งใดฝั่งหนึ่งน้อยไป (สมมติว่าต้องมีอย่างละ 2 ข้อมูลถึงจะพอ)
    if len(state['pos_data']) < 2 or len(state['neg_data']) < 2:
        print("!!! ข้อมูลไม่สมดุล สั่ง Researcher ค้นหาใหม่เจาะจงขึ้น !!!")
        return "continue_research"
    else:
        # โจทย์สั่ง: ต้องเรียกใช้ tool exit_loop (ใน Logic คือการไปโหมดสรุป)
        print("✔ ข้อมูลสมบูรณ์แล้ว เรียกใช้ tool: exit_loop")
        return "exit_loop"

# 5. สรุปผลและบันทึกไฟล์ (The Verdict)
def final_verdict(state: AgentState):
    print("--- กำลังจัดทำรายงานสรุปเป็นไฟล์ .txt ---")
    report = f"รายงานเปรียบเทียบข้อเท็จจริง: {state['topic']}\n"
    report += "="*30 + "\n"
    report += f"ข้อมูลด้านบวก: {state['pos_data']}\n"
    report += f"ข้อมูลด้านลบ: {state['neg_data']}\n"
    
    with open("verdict_report.txt", "w", encoding="utf-8") as f:
        f.write(report)
    return {"verdict": "Success"}

# --- สร้าง Workflow ---
builder = StateGraph(AgentState)
builder.add_node("admirer", agent_admirer)
builder.add_node("critic", agent_critic)
builder.add_node("verdict_node", final_verdict)

builder.set_entry_point("admirer")
builder.add_edge("admirer", "critic")

# เชื่อม Loop ตามโจทย์ Step 3
builder.add_conditional_edges(
    "critic", 
    judge_logic, 
    {
        "continue_research": "admirer", 
        "exit_loop": "verdict_node"
    }
)
builder.add_edge("verdict_node", END)

app = builder.compile()

# --- Step 1: รับชื่อจาก User ---
if __name__ == "__main__":
    target = input("ระบุชื่อบุคคลหรือเหตุการณ์ (เช่น Genghis Khan): ")
    app.invoke({"topic": target, "pos_data": [], "neg_data": []})
    print("เสร็จสิ้น! ตรวจสอบไฟล์ 'verdict_report.txt' ในโฟลเดอร์ของคุณ")