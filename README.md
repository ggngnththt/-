from pyrogram import Client, filters
from pyrogram.types import InlineKeyboardMarkup, InlineKeyboardButton
import random, time, json, os

API_ID = 123456
API_HASH = "ضع_الهاش_هنا"
BOT_TOKEN = "ضع_التوكن_هنا"
ADMIN_ID = 123456789
DATA_FILE = "data.json"

if os.path.exists(DATA_FILE):
    with open(DATA_FILE, "r") as f:
        users = json.load(f)
else:
    users = {}

def save_data():
    with open(DATA_FILE, "w") as f:
        json.dump(users, f)

app = Client("khdmatk_bot", api_id=API_ID, api_hash=API_HASH, bot_token=BOT_TOKEN)

main_menu = InlineKeyboardMarkup([
    [InlineKeyboardButton("💼 الخدمات", callback_data="services"),
     InlineKeyboardButton("💰 التمويل", callback_data="finance")],
    [InlineKeyboardButton("🎁 هدية يومية", callback_data="daily"),
     InlineKeyboardButton("🔗 رابط دعوة", callback_data="invite")],
    [InlineKeyboardButton("📢 انضمام لقنوات", callback_data="join"),
     InlineKeyboardButton("🎟️ كود هدية", callback_data="code")],
    [InlineKeyboardButton("👤 حسابك", callback_data="profile"),
     InlineKeyboardButton("📦 الطلبات المكتملة", callback_data="orders")]
])

services_menu = InlineKeyboardMarkup([
    [InlineKeyboardButton("👁️‍🗨️ 200 مشاهدة - 5 نقاط", callback_data="views")],
    [InlineKeyboardButton("🤖 تفاعل عشوائي - 15 نقطة", callback_data="react")],
    [InlineKeyboardButton("🔙 رجوع", callback_data="back_main")]
])

@app.on_message(filters.command("start"))
def start(client, message):
    uid = str(message.from_user.id)
    if uid not in users:
        users[uid] = {"points": 0, "last_daily": 0, "orders": 0}
        save_data()
    message.reply_text("👋 أهلاً بك في بوت خدماتك.", reply_markup=main_menu)

@app.on_callback_query()
def cb(client, cq):
    uid = str(cq.from_user.id)
    if uid not in users:
        users[uid] = {"points": 0, "last_daily": 0, "orders": 0}
        save_data()
    u = users[uid]
    d = cq.data

    if d == "services":
        cq.message.edit_text("💼 اختر الخدمة المطلوبة 👇", reply_markup=services_menu)

    elif d == "finance":
        cq.message.edit_text("💰 قسم التمويل", reply_markup=main_menu)

    elif d == "daily":
        now = int(time.time())
        if now - u["last_daily"] >= 86400:
            u["points"] += 10
            u["last_daily"] = now
            save_data()
            cq.answer("🎁 حصلت على 10 نقاط!", show_alert=True)
        else:
            cq.answer("⏰ بعد 24 ساعة تقدر تاخذ الهدية.", show_alert=True)

    elif d == "invite":
        cq.message.reply_text(f"🔗 رابط دعوتك:\nhttps://t.me/username_bot?start={uid}\nتحصل 30 نقطة عن كل شخص!")

    elif d == "join":
        u["points"] += 15
        save_data()
        cq.answer("✅ حصلت على 15 نقطة!", show_alert=True)

    elif d == "code":
        cq.message.reply_text("🎟️ أرسل الكود هنا:")

    elif d == "profile":
        cq.message.reply_text(f"👤 حسابك:\nرصيدك: {u['points']} نقطة\nالطلبات: {u['orders']}")

    elif d == "orders":
        cq.message.reply_text(f"📦 عدد الطلبات المكتملة: {u['orders']}")

    elif d == "views":
        if u["points"] >= 5:
            u["points"] -= 5
            u["orders"] += 1
            save_data()
            client.send_message("@fjtdhhr", f"📢 طلب جديد من {uid}\nمشاهدات 200 🔹")
            cq.answer("✅ تم تنفيذ الطلب!", show_alert=True)
        else:
            cq.answer("❌ رصيدك غير كافٍ.", show_alert=True)

    elif d == "react":
        if u["points"] >= 15:
            u["points"] -= 15
            u["orders"] += 1
            save_data()
            client.send_message("@fjtdhhr", f"🤖 طلب تفاعل عشوائي من {uid}")
            cq.answer("✅ تم تنفيذ الطلب!", show_alert=True)
        else:
            cq.answer("❌ لا يوجد رصيد كافٍ.", show_alert=True)

    elif d == "back_main":
        cq.message.edit_text("القائمة الرئيسية:", reply_markup=main_menu)

print("✅ بوت خدماتك يعمل الآن...")
app.run()
نبن
