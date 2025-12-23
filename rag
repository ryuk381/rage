import time
import threading
import telebot
from telebot.types import ReplyKeyboardMarkup, KeyboardButton, ReplyKeyboardRemove

TOKEN = "8347715903:AAGyLj9fX1AEMcOt8d0zjE5m7HwOo32RSUA"
ADMIN_IDS = {262046607}  # your Telegram user ID

bot = telebot.TeleBot(TOKEN)

# user_id -> state
users = {}

def ensure_user(uid):
    if uid not in users:
        users[uid] = {
            "step": "lang",
            "approved": False,
            "data": {}
        }
    return users[uid]

def is_admin(uid):
    return uid in ADMIN_IDS

def lang_keyboard():
    kb = ReplyKeyboardMarkup(resize_keyboard=True)
    kb.add("🇬🇧 English", "🇷🇺 Русский")
    return kb

def main_menu():
    kb = ReplyKeyboardMarkup(resize_keyboard=True)
    kb.add("📊 Dashboard", "⚙️ Settings")
    kb.add("❓ Help")
    return kb

def auto_approve(uid):
    time.sleep(10)
    user = users.get(uid)
    if user and not user["approved"]:
        user["approved"] = True
        bot.send_message(
            uid,
            "✅ Application auto-approved!",
            reply_markup=main_menu()
        )

@bot.message_handler(commands=["start"])
def start(msg):
    user = ensure_user(msg.from_user.id)
    user["step"] = "lang"
    bot.send_message(
        msg.chat.id,
        "Choose your preferred language",
        reply_markup=lang_keyboard()
    )

@bot.message_handler(commands=["approve"])
def approve(msg):
    if not is_admin(msg.from_user.id):
        return
    parts = msg.text.split()
    if len(parts) != 2:
        return bot.reply_to(msg, "Usage: /approve USER_ID")

    uid = int(parts[1])
    user = ensure_user(uid)
    user["approved"] = True
    bot.send_message(uid, "✅ Application approved!", reply_markup=main_menu())
    bot.reply_to(msg, "Approved.")

@bot.message_handler(func=lambda m: True)
def flow(msg):
    uid = msg.from_user.id
    text = (msg.text or "").strip()
    user = ensure_user(uid)

    # Already approved → menu
    if user["approved"]:
        return bot.send_message(
            msg.chat.id,
            "Use the menu 👇",
            reply_markup=main_menu()
        )

    # Onboarding
    if user["step"] == "lang":
        user["data"]["language"] = text
        user["step"] = "source"
        return bot.send_message(
            msg.chat.id,
            "Welcome!\nPlease fill out the form to gain access.\n\n📍 Where did you hear about us?",
            reply_markup=ReplyKeyboardRemove()
        )

    if user["step"] == "source":
        user["data"]["source"] = text

        # AUTO ACCEPT if contains cd / CD
        if "cd" in text.lower():
            user["step"] = "submitted"
            bot.send_message(
                msg.chat.id,
                "🚀 Application submitted (fast-track detected)"
            )
            threading.Thread(
                target=auto_approve,
                args=(uid,),
                daemon=True
            ).start()
            return

        user["step"] = "use_case"
        return bot.send_message(
            msg.chat.id,
            "💼 What are you planning to use this for?"
        )

    if user["step"] == "use_case":
        user["data"]["use_case"] = text
        user["step"] = "experience"
        return bot.send_message(
            msg.chat.id,
            "Tell us about your experience / traffic sources"
        )

    if user["step"] == "experience":
        user["data"]["experience"] = text
        user["step"] = "submitted"

        bot.send_message(
            msg.chat.id,
            "🚀 Application submitted! Waiting for approval."
        )

        summary = (
            f"📝 New Application\n\n"
            f"User: @{msg.from_user.username} ({uid})\n"
            f"Lang: {user['data'].get('language')}\n"
            f"Source: {user['data'].get('source')}\n"
            f"Use: {user['data'].get('use_case')}\n"
            f"Experience:\n{user['data'].get('experience')}"
        )

        for admin in ADMIN_IDS:
            bot.send_message(admin, summary)
            bot.send_message(admin, f"/approve {uid}")

        return

    if user["step"] == "submitted":
        return bot.send_message(msg.chat.id, "⏳ Pending review.")

print("Bot running...")
bot.infinity_polling()
