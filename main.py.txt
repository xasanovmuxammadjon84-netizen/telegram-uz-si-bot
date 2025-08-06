import telebot
import datetime
import openai
import os

API_KEY = os.getenv("YOUR_OPENAI_API_KEY")
BOT_TOKEN = os.getenv("YOUR_TELEGRAM_BOT_TOKEN")

openai.api_key = API_KEY
bot = telebot.TeleBot(BOT_TOKEN)

USERS = {}
ADMIN_CODE = "muxammadjon_0798"
USER_CODE = "xasanov_0798"
MAX_USERS = 15

@bot.message_handler(commands=['start'])
def handle_start(message):
    chat_id = message.chat.id
    bot.send_message(chat_id, "Kirish uchun kodni kiriting:")
    bot.register_next_step_handler(message, check_code)

def check_code(message):
    chat_id = message.chat.id
    text = message.text.strip()

    if text == ADMIN_CODE:
        USERS[chat_id] = {"role": "admin", "expires": None}
        bot.send_message(chat_id, "Admin sifatida muvaffaqiyatli kirdingiz.")
    elif text == USER_CODE:
        user_count = sum(1 for user in USERS.values() if user.get("role") == "user")
        if user_count >= MAX_USERS:
            bot.send_message(chat_id, "Kechirasiz, maksimal foydalanuvchilar soniga yetildi.")
            return
        expires = datetime.datetime.now() + datetime.timedelta(days=30)
        USERS[chat_id] = {"role": "user", "expires": expires}
        bot.send_message(chat_id, "Xush kelibsiz! Sizga 1 oylik kirish huquqi berildi.")
    else:
        bot.send_message(chat_id, "Noto'g'ri kod. Iltimos, qaytadan urinib ko'ring.")

bot.infinity_polling()
