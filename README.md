import telebot
import requests
from api import get_social_media_data  # API faylidan funksiyani import qilamiz

TOKEN = '7444815384:AAFoKqZBxts4UH0ebD-zqE74hvrytiSgfwM'  # Telegram bot tokeningizni kiriting
bot = telebot.TeleBot(TOKEN)

@bot.message_handler(commands=['start'])
def send_welcome(message):
    bot.reply_to(message, "Salom! Men sizga ijtimoiy tarmoqdan video yuklab olishda yordam bera olaman. Link yuboring yoki /download buyrug'idan foydalaning.")

@bot.message_handler(commands=['download'])
def download_content(message):
    bot.reply_to(message, "Iltimos, yuklamoqchi bo'lgan video havolasini yuboring.")

@bot.message_handler(content_types=['text'])
def handle_text_message(message):
    url = message.text  # Foydalanuvchi yuborgan havolani olamiz
    data = get_social_media_data(url)  # API yordamida havola bo'yicha ma'lumot olishga harakat qilamiz

    if data and not data.get('error'):
        video_url = data['medias'][0]['url']  # API'dan video URL'ni olamiz

        # Videoni vaqtinchalik faylga yuklab olamiz
        video_response = requests.get(video_url, stream=True)
        if video_response.status_code == 200:
            with open("video.mp4", "wb") as video_file:
                for chunk in video_response.iter_content(chunk_size=1024):
                    video_file.write(chunk)

            # Videoni foydalanuvchiga yuboramiz
            with open("video.mp4", "rb") as video_file:
                bot.send_video(message.chat.id, video_file)

            # Video yuborilgandan so'ng faylni o'chirish
            import os
            os.remove("video.mp4")

        else:
            bot.reply_to(message, "Videoni yuklashda xatolik yuz berdi.")
    else:
        bot.reply_to(message, "Ma'lumot olishda xatolik yuz berdi yoki havola noto'g'ri.")

if __name__ == '__main__':
    bot.polling(none_stop=True)

