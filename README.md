# 📦 MatrixBot Pro by ریس ماتریکس 👑
from telegram import InlineKeyboardButton, InlineKeyboardMarkup, Update
from telegram.ext import Updater, CommandHandler, CallbackQueryHandler, MessageHandler, Filters, CallbackContext
from pdf2image import convert_from_path
from PIL import Image
import os

import logging
logging.basicConfig(format='%(asctime)s - %(name)s - %(levelname)s - %(message)s', level=logging.INFO)

# 👑 توکن ربات (ثابت شده با توکن جدید)
TOKEN = '7635984578:AAGOOK4yJZwNfdsoGsUa_XdwxTsldxzI1hE'

# 🌐 لیست زبان‌ها
LANGUAGES = {
    'fa': '🇮🇷 فارسی',
    'en': '🇺🇸 English',
    'ar': '🇸🇦 عربي'
}

user_language = {}

# 📑 شروع
def start(update: Update, context: CallbackContext):
    user = update.effective_user
    keyboard = [
        [InlineKeyboardButton("📑 مدیریت فایل‌ها", callback_data='menu')],
        [InlineKeyboardButton("🌐 تغییر زبان", callback_data='language')],
        [InlineKeyboardButton("ℹ️ درباره ما", callback_data='about')]
    ]
    reply_markup = InlineKeyboardMarkup(keyboard)
    update.message.reply_text(
        f"سلام {user.first_name} 👋\nبه MatrixBot خوش آمدید!\nیک گزینه را انتخاب کنید:",
        reply_markup=reply_markup
    )

# 🕹 هندلر دکمه‌ها
def button(update: Update, context: CallbackContext):
    query = update.callback_query
    query.answer()

    if query.data == 'menu':
        keyboard = [
            [InlineKeyboardButton("📥 PDF ➡️ عکس", callback_data='pdf_to_img')],
            [InlineKeyboardButton("🖼 عکس ➡️ PDF", callback_data='img_to_pdf')],
            [InlineKeyboardButton("🔙 بازگشت", callback_data='back')]
        ]
        query.edit_message_text("📑 مدیریت فایل‌ها:", reply_markup=InlineKeyboardMarkup(keyboard))

    elif query.data == 'language':
        keyboard = [
            [InlineKeyboardButton(LANGUAGES['fa'], callback_data='lang_fa')],
            [InlineKeyboardButton(LANGUAGES['en'], callback_data='lang_en')],
            [InlineKeyboardButton(LANGUAGES['ar'], callback_data='lang_ar')],
            [InlineKeyboardButton("🔙 بازگشت", callback_data='back')]
        ]
        query.edit_message_text("🌐 لطفاً یک زبان انتخاب کنید:", reply_markup=InlineKeyboardMarkup(keyboard))

    elif query.data == 'about':
        query.edit_message_text(
            "👑 ریس ماتریکس\n"
            "@nazeefmatrix369\n"
            "برای ارتباط بیشتر با ما در ارتباط باشید."
        )

    elif query.data.startswith('lang_'):
        lang_code = query.data.split('_')[1]
        user_language[query.from_user.id] = lang_code
        query.edit_message_text(f"✅ زبان تغییر یافت به: {LANGUAGES[lang_code]}")

    elif query.data == 'back':
        start(update, context)

# 📥 تبدیل PDF به عکس
def pdf_to_img(update: Update, context: CallbackContext):
    file = update.message.document.get_file()
    file.download('input.pdf')
    images = convert_from_path('input.pdf')
    for i, image in enumerate(images):
        image_path = f'page_{i+1}.jpg'
        image.save(image_path, 'JPEG')
        update.message.reply_photo(photo=open(image_path, 'rb'))
        os.remove(image_path)
    os.remove('input.pdf')

# 🖼 تبدیل عکس به PDF
def img_to_pdf(update: Update, context: CallbackContext):
    file = update.message.photo[-1].get_file()
    file.download('input.jpg')
    image = Image.open('input.jpg')
    pdf_path = 'output.pdf'
    image.save(pdf_path, 'PDF', resolution=100.0)
    update.message.reply_document(document=open(pdf_path, 'rb'))
    os.remove('input.jpg')
    os.remove(pdf_path)

# 🔥 شروع ربات
def main():
    updater = Updater(TOKEN)
    dp = updater.dispatcher
    dp.add_handler(CommandHandler('start', start))
    dp.add_handler(CallbackQueryHandler(button))
    dp.add_handler(MessageHandler(Filters.document.mime_type("application/pdf"), pdf_to_img))
    dp.add_handler(MessageHandler(Filters.photo, img_to_pdf))
    updater.start_polling()
    updater.idle()

if __name__ == '__main__':
    main()
