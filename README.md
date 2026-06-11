import os
import logging
from telegram import Update
from telegram.ext import Application, MessageHandler, filters, ContextTypes
import yt_dlp

TOKEN = "8909877890:AAFykT8PST1SJwnV8DSUWZLYzQ0X7tiIR-A"

logging.basicConfig(format='%(asctime)s - %(name)s - %(levelname)s - %(message)s', level=logging.INFO)

async def download_and_send(update: Update, context: ContextTypes.DEFAULT_TYPE):
    url = update.message.text
    chat_id = update.message.chat_id
    
    status_message = await update.message.reply_text("🔍 लिंक चेक किया जा रहा है और वीडियो डाउनलोड हो रही है, कृपया थोड़ा इंतजार करें...")

    ydl_opts = {
        'format': 'bestvideo[height<=720]+bestaudio/best[height<=720]',
        'outtmpl': '/tmp/%(id)s.%(ext)s',
        'merge_output_format': 'mp4',
        'quiet': True,
    }

    try:
        with yt_dlp.YoutubeDL(ydl_opts) as ydl:
            info = ydl.extract_info(url, download=True)
            filename = ydl.prepare_filename(info)
            if not os.path.exists(filename):
                filename = os.path.splitext(filename)[0] + ".mp4"

        await status_message.edit_text("🚀 वीडियो मिल गई! अब टेलीग्राम पर अपलोड की जा रही है...")
        with open(filename, 'rb') as video_file:
            await context.bot.send_video(chat_id=chat_id, video=video_file, caption="Done! ✨")
        
        if os.path.exists(filename):
            os.remove(filename)
        await status_message.delete()

    except Exception as e:
        await status_message.edit_text("❌ माफी चाहता हूँ, इस वेबसाइट से वीडियो डाउनलोड नहीं हो सकी।")
        print(f"Error: {e}")

def main():
    app = Application.builder().token(TOKEN).build()
    app.add_handler(MessageHandler(filters.TEXT & ~filters.COMMAND, download_and_send))
    print("बॉट चालू हो गया है...")
    app.run_polling()

if __name__ == '__main__':
  
    main()
