# JSON-va-CSV-real-ma-lumotlar-bilan-ishlash
import json

talabalar = [
    {"ism": "Ali",   "ball": 87, "teglar": ["python", "ai"]},
    {"ism": "Vali",  "ball": 92, "teglar": ["js"]},
    {"ism": "Gulya", "ball": 78, "teglar": ["python", "web"]},
]

# Faylga yozish
with open("talabalar.json", "w", encoding="utf-8") as f:
    json.dump(talabalar, f, ensure_ascii=False, indent=2)

# Qayta o'qish
with open("talabalar.json", encoding="utf-8") as f:
    qaytarilgan = json.load(f)

print(qaytarilgan[0]["ism"])    # Ali
print(type(qaytarilgan))         # list
import csv

# Yozish — DictWriter dict elementlarini ustun-ustun qiladi
with open("talabalar.csv", "w", encoding="utf-8", newline="") as f:
    writer = csv.DictWriter(f, fieldnames=["ism", "ball"])
    writer.writeheader()
    for t in talabalar:
        writer.writerow({"ism": t["ism"], "ball": t["ball"]})

# O'qish — har qator dict bo'ladi
with open("talabalar.csv", encoding="utf-8") as f:
    reader = csv.DictReader(f)
    for qator in reader:
        print(qator["ism"], "->", qator["ball"])
# JSON string -> Python obyekt
matn = '{"ism": "Ali", "ball": 87}'
obj = json.loads(matn)
print(obj["ism"])

# Python obyekt -> JSON string
s = json.dumps({"ism": "Ali"}, ensure_ascii=False)
print(s)         # {"ism": "Ali"}

# CSV string'larini ham parse qilish mumkin
from io import StringIO
csv_str = "ism,ball\nAli,87\nVali,92"
reader = csv.DictReader(StringIO(csv_str))
print(list(reader))
import csv

with open("test.csv", "w", encoding="utf-8") as f:    # ⚠️ newline yo'q
    writer = csv.writer(f)
    writer.writerow(["a", "b", "c"])
    writer.writerow([1, 2, 3])

# Windows'da:
# a,b,c
#                   <-- ortiqcha bo'sh qator
# 1,2,3
json.dump(
    obj, f,
    ensure_ascii=False,     # Unicode harflarni o'zicha yozadi
    indent=2,               # Chiroyli formatlash (2 bo'shliq)
    sort_keys=True,         # Kalitlarni alifbo tartibida
    default=str,            # JSON tushunmaydigan turlar uchun (datetime ga str)
)
from dataclasses import asdict, dataclass
from datetime import datetime
import json

@dataclass
class Buyurtma:
    id: int
    sana: datetime
    summa: float

b = Buyurtma(1, datetime.now(), 1500.0)

# datetime — JSON tushunmaydi. default=str orqali stringga
s = json.dumps(asdict(b), default=str, ensure_ascii=False)
print(s)
# ─── JSON va CSV bilan ishlash — to'liq sweep ────────────────────────────
import json
import csv
from io import StringIO
from dataclasses import dataclass, asdict, field
from datetime import datetime
from pathlib import Path

# Test fayllarni shu skript joyiga yaratamiz
HERE = Path(__file__).parent

# 1) Dataclass + JSON
@dataclass
class Foydalanuvchi:
    id: int
    ism: str
    yosh: int
    teglar: list[str] = field(default_factory=list)

ali = Foydalanuvchi(1, "Ali", 21, ["python"])
vali = Foydalanuvchi(2, "Vali", 19, ["js", "react"])
gulya = Foydalanuvchi(3, "Gulya", 22, ["data"])

foydalanuvchilar = [ali, vali, gulya]

# Faylga JSON yozish
json_yo_l = HERE / "demo_foydalanuvchilar.json"
with open(json_yo_l, "w", encoding="utf-8") as f:
    json.dump(
        [asdict(u) for u in foydalanuvchilar],
        f, ensure_ascii=False, indent=2,
    )
print(f"JSON yozildi: {json_yo_l}")

# Qayta o'qish
with open(json_yo_l, encoding="utf-8") as f:
    qaytarilgan = json.load(f)
print(f"O'qildi: {len(qaytarilgan)} ta foydalanuvchi")
print("Birinchisi:", qaytarilgan[0])

# 2) JSON string bilan (fayl emas)
matn = '{"ism": "Doniyor", "yosh": 25}'
obj = json.loads(matn)
print(f"\nParse: {obj['ism']} — {obj['yosh']} yoshda")

# Python -> string
s = json.dumps({"x": [1, 2, 3], "y": None}, ensure_ascii=False)
print("Dump string:", s)

# 3) Maxsus turlar — datetime
buyurtma = {
    "id": 42,
    "sana": datetime.now(),
    "summa": 12500.0,
}

# datetime ni JSON tushunmaydi — default=str bilan stringga
buy_json = json.dumps(buyurtma, default=str, ensure_ascii=False, indent=2)
print(f"\nBuyurtma JSON:\n{buy_json}")

# 4) CSV yozish — DictWriter
csv_yo_l = HERE / "demo_foydalanuvchilar.csv"
with open(csv_yo_l, "w", encoding="utf-8-sig", newline="") as f:
    writer = csv.DictWriter(f, fieldnames=["id", "ism", "yosh", "teglar"])
    writer.writeheader()
    for u in foydalanuvchilar:
        row = asdict(u)
        row["teglar"] = ";".join(row["teglar"])    # list -> string
        writer.writerow(row)
print(f"\nCSV yozildi: {csv_yo_l}")

# 5) CSV o'qish — DictReader
print("\nCSV dan o'qildi:")
with open(csv_yo_l, encoding="utf-8-sig") as f:
    reader = csv.DictReader(f)
    for qator in reader:
        teglar = qator["teglar"].split(";") if qator["teglar"] else []
        print(f"  #{qator['id']:>2}  {qator['ism']:<8}  {qator['yosh']} yoshda  teglar={teglar}")

# 6) CSV string (fayl yaratmasdan)
csv_matn = (
    "mahsulot,narx,soni\n"
    "Olma,12000,5\n"
    "Non,4000,12\n"
    "Sut,9000,3\n"
)
reader = csv.DictReader(StringIO(csv_matn))
jami = sum(int(r["narx"]) * int(r["soni"]) for r in reader)
print(f"\nJami summa: {jami:,} so'm")

# 7) Real transformatsiya — JSON dan CSV ga
data = [
    {"ism": "Ali",   "ball": 87, "teglar": ["python"]},
    {"ism": "Vali",  "ball": 92, "teglar": ["js"]},
    {"ism": "Gulya", "ball": 78, "teglar": ["python", "web"]},
]

with open(HERE / "demo_top.csv", "w", encoding="utf-8-sig", newline="") as f:
    w = csv.DictWriter(f, fieldnames=["ism", "ball", "asosiy_teg"])
    w.writeheader()
    for t in sorted(data, key=lambda x: x["ball"], reverse=True):
        w.writerow({
            "ism": t["ism"],
            "ball": t["ball"],
            "asosiy_teg": t["teglar"][0] if t["teglar"] else "",
        })
print("Top CSV yozildi.")
# ─── JSON va CSV bilan ishlash — to'liq sweep ────────────────────────────
import json
import csv
from io import StringIO
from dataclasses import dataclass, asdict, field
from datetime import datetime
from pathlib import Path

# Test fayllarni shu skript joyiga yaratamiz
HERE = Path(__file__).parent

# 1) Dataclass + JSON
@dataclass
class Foydalanuvchi:
    id: int
    ism: str
    yosh: int
    teglar: list[str] = field(default_factory=list)

ali = Foydalanuvchi(1, "Ali", 21, ["python"])
vali = Foydalanuvchi(2, "Vali", 19, ["js", "react"])
gulya = Foydalanuvchi(3, "Gulya", 22, ["data"])

foydalanuvchilar = [ali, vali, gulya]

# Faylga JSON yozish
json_yo_l = HERE / "demo_foydalanuvchilar.json"
with open(json_yo_l, "w", encoding="utf-8") as f:
    json.dump(
        [asdict(u) for u in foydalanuvchilar],
        f, ensure_ascii=False, indent=2,
    )
print(f"JSON yozildi: {json_yo_l}")

# Qayta o'qish
with open(json_yo_l, encoding="utf-8") as f:
    qaytarilgan = json.load(f)
print(f"O'qildi: {len(qaytarilgan)} ta foydalanuvchi")
print("Birinchisi:", qaytarilgan[0])

# 2) JSON string bilan (fayl emas)
matn = '{"ism": "Doniyor", "yosh": 25}'
obj = json.loads(matn)
print(f"\nParse: {obj['ism']} — {obj['yosh']} yoshda")

# Python -> string
s = json.dumps({"x": [1, 2, 3], "y": None}, ensure_ascii=False)
print("Dump string:", s)

# 3) Maxsus turlar — datetime
buyurtma = {
    "id": 42,
    "sana": datetime.now(),
    "summa": 12500.0,
}

# datetime ni JSON tushunmaydi — default=str bilan stringga
buy_json = json.dumps(buyurtma, default=str, ensure_ascii=False, indent=2)
print(f"\nBuyurtma JSON:\n{buy_json}")

# 4) CSV yozish — DictWriter
csv_yo_l = HERE / "demo_foydalanuvchilar.csv"
with open(csv_yo_l, "w", encoding="utf-8-sig", newline="") as f:
    writer = csv.DictWriter(f, fieldnames=["id", "ism", "yosh", "teglar"])
    writer.writeheader()
    for u in foydalanuvchilar:
        row = asdict(u)
        row["teglar"] = ";".join(row["teglar"])    # list -> string
        writer.writerow(row)
print(f"\nCSV yozildi: {csv_yo_l}")

# 5) CSV o'qish — DictReader
print("\nCSV dan o'qildi:")
with open(csv_yo_l, encoding="utf-8-sig") as f:
    reader = csv.DictReader(f)
    for qator in reader:
        teglar = qator["teglar"].split(";") if qator["teglar"] else []
        print(f"  #{qator['id']:>2}  {qator['ism']:<8}  {qator['yosh']} yoshda  teglar={teglar}")

# 6) CSV string (fayl yaratmasdan)
csv_matn = (
    "mahsulot,narx,soni\n"
    "Olma,12000,5\n"
    "Non,4000,12\n"
    "Sut,9000,3\n"
)
reader = csv.DictReader(StringIO(csv_matn))
jami = sum(int(r["narx"]) * int(r["soni"]) for r in reader)
print(f"\nJami summa: {jami:,} so'm")

# 7) Real transformatsiya — JSON dan CSV ga
data = [
    {"ism": "Ali",   "ball": 87, "teglar": ["python"]},
    {"ism": "Vali",  "ball": 92, "teglar": ["js"]},
    {"ism": "Gulya", "ball": 78, "teglar": ["python", "web"]},
]

with open(HERE / "demo_top.csv", "w", encoding="utf-8-sig", newline="") as f:
    w = csv.DictWriter(f, fieldnames=["ism", "ball", "asosiy_teg"])
    w.writeheader()
    for t in sorted(data, key=lambda x: x["ball"], reverse=True):
        w.writerow({
            "ism": t["ism"],
            "ball": t["ball"],
            "asosiy_teg": t["teglar"][0] if t["teglar"] else "",
        })
print("Top CSV yozildi.")
