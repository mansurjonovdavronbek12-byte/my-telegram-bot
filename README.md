import telebot
import qrcode
from io import BytesIO
from PIL import Image

TOKEN = "8020625917:AAEhRbr6OPZQeDu2BI9CtrnjtKB7lIP8qZU"  # BotFatherdan olgan token
bot = telebot.TeleBot(TOKEN)

# Logo rasm manzili (mahalliy fayl)
LOGO_PATH = "logo.png"  # o'zingizning logo.png faylingizni shu papkaga qo'ying

# /start komandasi
@bot.message_handler(commands=['start'])
def start(message):
    bot.send_message(message.chat.id,
                     "Salom! Men sizga chiroyli, gradientli QR code yaratib beraman. "
                     "Shunchaki matningizni yoki havolani yuboring.")

# Matnni qabul qilish va QR code yaratish
@bot.message_handler(func=lambda message: True)
def generate_qr(message):
    text = message.text

    # QR code yaratish
    qr = qrcode.QRCode(
        version=1,
        error_correction=qrcode.constants.ERROR_CORRECT_H,
        box_size=5,  # kichkina o'lcham
        border=2,
    )
    qr.add_data(text)
    qr.make(fit=True)

    # QR code rasmini gradient rang bilan yaratish
    qr_img = qr.make_image(fill_color="black", back_color="white").convert('RGB')

    # Gradient qo'shish
    width, height = qr_img.size
    gradient = Image.new('RGB', (width, height), color=0)
    for y in range(height):
        r = int(255 * (y / height))
        g = int(128 * (y / height))
        b = 255 - int(255 * (y / height))
        for x in range(width):
            gradient.putpixel((x, y), (r, g, b))

    qr_img = Image.blend(qr_img, gradient, alpha=0.3)  # 0.3 – gradient kuchi

    # Logo qo'shish
    try:
        logo = Image.open(LOGO_PATH)
        logo_size = width // 5  # logo o'lchami QR kodga nisbatan kichik
        logo = logo.resize((logo_size, logo_size))
        pos = ((width - logo_size) // 2, (height - logo_size) // 2)
        qr_img.paste(logo, pos, mask=logo if logo.mode == 'RGBA' else None)
    except Exception as e:
        print("Logo qo'shilmadi:", e)

    # Rasmni xotirada saqlash
    bio = BytesIO()
    bio.name = 'qr.png'
    qr_img.save(bio, 'PNG')
    bio.seek(0)

    bot.send_photo(message.chat.id, photo=bio, caption="Sizning chiroyli QR code tayyor! ✅")

bot.infinity_polling()
