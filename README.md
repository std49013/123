import json
from dataclasses import dataclass, field
from typing import List, Dict, Optional


@dataclass
class TeamMember:
    name: str
    skills: Dict[str, int]  # เช่น {"Python": 5, "Database": 3} (คะแนน 1-5)
    total_available_hours: float
    assigned_hours: float = 0.0
    assigned_tasks: List[str] = field(default_factory=list)

    @property
    def remaining_hours(self) -> float:
        return self.total_available_hours - self.assigned_hours

    @property
    def workload_percentage(self) -> float:
        if self.total_available_hours == 0:
            return 0.0
        return (self.assigned_hours / self.total_available_hours) * 100


@dataclass
class MicroTask:
    task_id: str
    name: str
    required_skill: str
    min_skill_level: int
    est_time: float  # หน่วย: ชั่วโมง (0.5 - 3.0)
    priority: str  # High, Medium, Low
    definition_of_done: str


@dataclass
class AssignmentResult:
    task: MicroTask
    assigned_to: Optional[str]
    match_score: float
    reason: str


class TaskAllocatorEngine:
    """ระบบประมวลผลจัดสรรงานอัตโนมัติอย่างเป็นธรรม"""

    def __init__(self, members: List[TeamMember]):
        self.members = {m.name: m for m in members}

    def allocate_task(self, task: MicroTask) -> AssignmentResult:
        best_candidate = None
        best_score = -1.0
        assignment_reason = ""

        for member in self.members.values():
            # 1. เงื่อนไขการกรอง (Hard Constraint): เวลาว่างต้องพอ และ ทักษะต้องผ่านเกณฑ์ขั้นต่ำ
            member_skill_level = member.skills.get(task.required_skill, 0)

            if member.remaining_hours < task.est_time:
                continue

            if member_skill_level < task.min_skill_level:
                continue

            # 2. คำนวณคะแนน (Soft Constraint / Optimization)
            # - Skill Score (60%): ยิ่งทักษะตรงและสูงยิ่งได้คะแนนมาก
            # - Workload Score (40%): ยิ่งมีสัดส่วนเวลาว่างเหลือมาก ยิ่งได้คะแนนมาก (เพื่อกระจายงาน)
            skill_score = member_skill_level / 5.0
            capacity_score = member.remaining_hours / member.total_available_hours

            total_score = (skill_score * 0.6) + (capacity_score * 0.4)

            if total_score > best_score:
                best_score = total_score
                best_candidate = member
                assignment_reason = (
                    f"ทักษะ {task.required_skill} ระดับ {member_skill_level}/{task.min_skill_level} "
                    f"+ เวลาว่างคงเหลือ {member.remaining_hours:.1f} ชม. "
                    f"(ภาระงานปัจจุบัน {member.workload_percentage:.1f}%)"
                )

        # 3. ยืนยันการจัดสรรงาน
        if best_candidate:
            best_candidate.assigned_hours += task.est_time
            best_candidate.assigned_tasks.append(task.task_id)
            return AssignmentResult(
                task=task,
                assigned_to=best_candidate.name,
                match_score=round(best_score, 2),
                reason=assignment_reason,
            )
        else:
            return AssignmentResult(
                task=task,
                assigned_to="Unassigned (Risk)",
                match_score=0.0,
                reason="ไม่มีสมาชิกที่มีทักษะตรง หรือเวลาว่างไม่เพียงพอ",
            )


class ProjectDecomposer:
    """ระบบจำลองการย่อยฟีเจอร์โปรเจกต์เป็น Micro-tasks"""

    @staticmethod
    def create_sample_micro_tasks() -> List[MicroTask]:
        return [
            MicroTask(
                task_id="T-01",
                name="ออกแบบ Database Schema สำหรับระบบสมาชิก",
                required_skill="Database",
                min_skill_level=3,
                est_time=2.0,
                priority="High",
                definition_of_done="ER-Diagram และ DDL Script พร้อมรัน",
            ),
            MicroTask(
                task_id="T-02",
                name="เขียน REST API authentication (JWT)",
                required_skill="Python",
                min_skill_level=3,
                est_time=3.0,
                priority="High",
                definition_of_done="API Endpoint ผ่าน Unit Test 100%",
            ),
            MicroTask(
                task_id="T-03",
                name="สร้างหน้าจอ UI สำหรับ Login / Register",
                required_skill="Frontend",
                min_skill_level=2,
                est_time=2.5,
                priority="Medium",
                definition_of_done="หน้าเว็บตาม Mockup รองรับ Responsive",
            ),
            MicroTask(
                task_id="T-04",
                name="เขียน Dockerfile และ docker-compose setup",
                required_skill="DevOps",
                min_skill_level=2,
                est_time=1.5,
                priority="Medium",
                definition_of_done="Container spin up ได้สำเร็จด้วยคำสั่งเดียว",
            ),
            MicroTask(
                task_id="T-05",
                name="ทำ API Documentation ด้วย Swagger",
                required_skill="Python",
                min_skill_level=1,
                est_time=1.0,
                priority="Low",
                definition_of_done="เอกสาร API อัปเดตครอบคลุมทุก Endpoint",
            ),
        ]


def generate_markdown_reports(
    results: List[AssignmentResult], members: List[TeamMember]
):
    """ส่งออกรายงานผลการจัดสรรงานและสรุป Workload เป็น Markdown"""

    print("\n### ขั้นตอนที่ 3: ตารางจัดสรรงาน (Smart Assignment Matrix)\n")
    print(
        "| Task ID | Micro-Task Name | Est. Time | Required Skill | Assigned To | Match Score | Reason for Assignment |"
    )
    print(
        "|---|---|---|---|---|---|---| "
    )
    for res in results:
        t = res.task
        print(
            f"| {t.task_id} | {t.name} | {t.est_time} ชม. | {t.required_skill} | **{res.assigned_to}** | {res.match_score} | {res.reason} |"
        )

    print(
        "\n--- \n"
    )
    print("### ขั้นตอนที่ 4: สรุปภาระงานและความโปร่งใส (Workload Transparency Summary)\n")
    print("| สมาชิก | เวลาว่างทั้งหมด | เวลาที่ถูกใช้ไป | เวลาคงเหลือ | % Workload |")
    print("|---|---|---|---|---|")
    for m in members:
        print(
            f"| {m.name} | {m.total_available_hours} ชม. | {m.assigned_hours} ชม. | {m.remaining_hours} ชม. | **{m.workload_percentage:.1f}%** |"
        )


# ==========================================
# Execution Entry Point
# ==========================================
if __name__ == "__main__":
    # 1. กำหนดข้อมูลทีม (Skill Matrix & Available Hours)
    team_members = [
        TeamMember(
            name="Alice",
            skills={"Python": 4, "Database": 4, "DevOps": 1},
            total_available_hours=8.0,
        ),
        TeamMember(
            name="Bob",
            skills={"Frontend": 4, "Python": 2, "Database": 1},
            total_available_hours=6.0,
        ),
        TeamMember(
            name="Charlie",
            skills={"DevOps": 3, "Python": 3, "Database": 2},
            total_available_hours=5.0,
        ),
    ]

    # 2. ย่อยงานโปรเจกต์
    tasks = ProjectDecomposer.create_sample_micro_tasks()

    # 3. ประมวลผลการจัดสรรงาน
    allocator = TaskAllocatorEngine(team_members)
    assignment_results = [allocator.allocate_task(task) for task in tasks]

    # 4. แสดงผลลัพธ์
    generate_markdown_reports(assignment_results, team_members)
