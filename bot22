import json
import os
import telebot
import yt_dlp
import urllib.parse

TOKEN = '8833808424:AAFVkwdbarGpbsx8uH4DW9aaARQdNU5xsJE'
MAIN_ADMIN_ID = 8085880852
SECOND_ADMIN_ID = 1775270218

bot = telebot.TeleBot(TOKEN)

DATA_FILE = 'products.json'
AD_FILE = 'announcement.json'
ADMINS_FILE = 'admins.json'
USERS_FILE = 'users.json'
BANNED_FILE = 'banned.json'

user_temp_product = {}

def load_data(filename):
    if not os.path.exists(filename):
        if filename == DATA_FILE:
            return []
        elif filename == ADMINS_FILE:
            return [MAIN_ADMIN_ID, SECOND_ADMIN_ID]
        else:
            return [] if filename in [USERS_FILE, BANNED_FILE] else {}
    with open(filename, 'r', encoding='utf-8') as f:
        return json.load(f)

def save_data(filename, data):
    with open(filename, 'w', encoding='utf-8') as f:
        json.dump(data, f, ensure_ascii=False, indent=4)

def is_user_admin(user_id):
    if user_id in [MAIN_ADMIN_ID, SECOND_ADMIN_ID]:
        return True
    admins = load_data(ADMINS_FILE)
    return user_id in admins

def is_user_banned(user_id):
    banned_users = load_data(BANNED_FILE)
    return user_id in banned_users

def register_user(message):
    user_id = message.from_user.id
    full_name = f"{message.from_user.first_name or ''} {message.from_user.last_name or ''}".strip()
    username = f"@{message.from_user.username}" if message.from_user.username else "بدون معرف"
    users = load_data(USERS_FILE)
    if not any(u.get('id') == user_id for u in users):
        users.append({'id': user_id, 'name': full_name if full_name else 'مستخدم', 'username': username})
        save_data(USERS_FILE, users)

def get_persistent_reply_keyboard():
    keyboard = telebot.types.ReplyKeyboardMarkup(resize_keyboard=True, row_width=2)
    btn_main = telebot.types.KeyboardButton('🏠 القائمة الرئيسية')
    btn_contact = telebot.types.KeyboardButton('📞 تواصل مع الإدارة (ساجدة)')
    keyboard.add(btn_main, btn_contact)
    return keyboard

def get_main_menu_markup(user_id):
    is_admin = is_user_admin(user_id)
    markup = telebot.types.InlineKeyboardMarkup(row_width=2)
    btn1 = telebot.types.InlineKeyboardButton('🟢 ألبسة رجالية 👔', callback_data='cat_men')
    btn2 = telebot.types.InlineKeyboardButton('🔵 نسائي وبناتي 👗', callback_data='cat_women')
    btn3 = telebot.types.InlineKeyboardButton('🟢 ألبسة أطفال 👶', callback_data='cat_kids')
    btn4 = telebot.types.InlineKeyboardButton('🔵 الأحذية والفرشات 👟', callback_data='cat_shoes')
    btn5 = telebot.types.InlineKeyboardButton('🔥 عروض العيد والحسومات 🎁', callback_data='cat_offers')
    btn6 = telebot.types.InlineKeyboardButton('📍 معلومات وموقع المتجر 📌', callback_data='store_info')
    btn7 = telebot.types.InlineKeyboardButton('🟡 أسعار الذهب والدولار 📈', callback_data='prices')
    btn8 = telebot.types.InlineKeyboardButton('🟣 تحميل السوشيال ميديا 📥', callback_data='social_dl')
    
    markup.add(btn1, btn2)
    markup.add(btn3, btn4)
    markup.add(btn5, btn6)
    markup.add(btn7, btn8)
    
    if is_admin:
        btn_admin = telebot.types.InlineKeyboardButton('🔴 لوحة التحكم السرية للأدمين 🔐', callback_data='admin_panel')
        markup.add(btn_admin)
        
    return markup

@bot.message_handler(commands=['start'])
def send_welcome(message):
    user_id = message.from_user.id
    if is_user_banned(user_id):
        bot.reply_to(message, "❌ عذراً ، تم حظرك ومنعك من استخدام هذا البوت من قبل الإدارة.", parse_mode='Markdown')
        return
        
    register_user(message)
    ad_data = load_data(AD_FILE)
    ad_text = ad_data.get('announcement', '')
    welcome_msg = (
        "╔═══════════════════════╗\n"
        "   👑 متجر ألبسة الريان 👑\n"
        "╚═══════════════════════╝\n\n"
        "🔥 أهلاً بكم في عالم الفخامة والأناقة لكل العيلة!\n"
    )
    if ad_text:
        welcome_msg += f"\n📢 [ إعلان هام جداً ]:\n{ad_text}\n"
    welcome_msg += "\n━━━━━━━━━━━━━━━━━━━\n👇 اختر القسم المطلوب من الأزرار الملونة أدناه:"
    
    bot.send_message(message.chat.id, welcome_msg, reply_markup=get_main_menu_markup(user_id), parse_mode='Markdown')
    bot.send_message(message.chat.id, "👇 الأزرار السريعة تحت أمرك يا انسة:", reply_markup=get_persistent_reply_keyboard())

@bot.message_handler(commands=['addadmin'])
def add_new_admin(message):
    user_id = message.from_user.id
    if not is_user_admin(user_id):
        bot.reply_to(message, "❌ عذراً، هذا الأمر مخصص للأدمين فقط يا خال!")
        return
    try:
        parts = message.text.split()
        if len(parts) < 2:
            bot.reply_to(message, "⚠️ الاستخدام الصحيح:\n/addadmin [أيدي_الشخص]", parse_mode='Markdown')
            return
        new_admin_id = int(parts[1])
        admins = load_data(ADMINS_FILE)
        if new_admin_id in admins or new_admin_id in [MAIN_ADMIN_ID, SECOND_ADMIN_ID]:
            bot.reply_to(message, "ℹ️ هذا الشخص أدمين بالفعل يا خال!")
            return
        admins.append(new_admin_id)
        save_data(ADMINS_FILE, admins)
        bot.reply_to(message, f"✅ تمت ترقية الشخص بنجاح!\nالأيدي: {new_admin_id} صار أدمين معنا بالمتجر 👑", parse_mode='Markdown')
    except Exception as e:
        bot.reply_to(message, "❌ حدث خطأ بالمعالجة: تأكد من كتابة الأيدي بشكل صحيح.")

@bot.message_handler(commands=['ban'])
def ban_user(message):
    user_id = message.from_user.id
    if not is_user_admin(user_id):
        return
    try:
        parts = message.text.split()
        if len(parts) < 2:
            bot.reply_to(message, "⚠️ الاستخدام الصحيح لطرد شخص:\n/ban [أيدي_الشخص]", parse_mode='Markdown')
            return
        banned_id = int(parts[1])
        if banned_id in [MAIN_ADMIN_ID, SECOND_ADMIN_ID]:
            bot.reply_to(message, "❌ ما فيك تطرد الأدمين يا زعيم!")
            return
        banned_users = load_data(BANNED_FILE)
        if banned_id in banned_users:
            bot.reply_to(message, "ℹ️ هذا الشخص مطرود ومحظور مسبقاً.")
            return
        banned_users.append(banned_id)
        save_data(BANNED_FILE, banned_users)
        bot.reply_to(message, f"🚫 تم طرد وحظر المستخدم بنجاح!\nالأيدي: {banned_id} لم يعد يستطيع استخدام البوت.", parse_mode='Markdown')
    except Exception as e:
        bot.reply_to(message, "❌ تأكد من كتابة الأيدي برقم صحيح يا خال.")

@bot.message_handler(commands=['unban'])
def unban_user(message):
    user_id = message.from_user.id
    if not is_user_admin(user_id):
        return
    try:
        parts = message.text.split()
        if len(parts) < 2:
            bot.reply_to(message, "⚠️ الاستخدام:\n/unban [أيدي_الشخص]", parse_mode='Markdown')
            return
        unbanned_id = int(parts[1])
        banned_users = load_data(BANNED_FILE)
        if unbanned_id not in banned_users:
            bot.reply_to(message, "ℹ️ هذا الشخص ليس محظوراً أساساً.")
            return
        banned_users.remove(unbanned_id)
        save_data(BANNED_FILE, banned_users)
        bot.reply_to(message, f"✅ تم رفع الحظر عن المستخدم {unbanned_id} بنجاح يا خال.")
    except Exception as e:
        bot.reply_to(message, "❌ خطأ في المعالجة.")

@bot.message_handler(func=lambda message: message.text in ['🏠 القائمة الرئيسية', '📞 تواصل مع الإدارة (ساجدة)'])
def handle_persistent_buttons(message):
    user_id = message.from_user.id
    if is_user_banned(user_id):
        return
        
    if message.text == '🏠 القائمة الرئيسية':
        ad_data = load_data(AD_FILE)
        ad_text = ad_data.get('announcement', '')
        welcome_msg = (
            "╔═══════════════════════╗\n"
            "   👑 متجر ألبسة الريان 👑\n"
            "╚═══════════════════════╝\n\n"
            "🔥 أهلاً بكم  من جديد!\n"
        )
        if ad_text:
            welcome_msg += f"\n📢 [ إعلان هام جداً ]:\n{ad_text}\n"
        welcome_msg += "\n━━━━━━━━━━━━━━━━━━━\n👇 اختر القسم المطلوب من الأزرار أدناه:"
        bot.send_message(message.chat.id, welcome_msg, reply_markup=get_main_menu_markup(user_id), parse_mode='Markdown')
    elif message.text == '📞 تواصل مع الإدارة (ساجدة)':
        markup = telebot.types.InlineKeyboardMarkup(row_width=1)
        btn_whatsapp = telebot.types.InlineKeyboardButton("💬 تواصل عبر الواتساب", url="https://wa.me/963954689426?text=مرحباً، أود الاستفسار عن خدمات المتجر")
        btn_telegram = telebot.types.InlineKeyboardButton("📱 تواصل عبر واتساب (الآنسة ساجدة)", url="https://wa.me/963954689426?text=مرحبا انسة بيسان اريد استفار عن الملابس المنشورة في قناتك هل يمكنك تقديم السعر وشكرا لحضرتك انسة بيسان")
        markup.add(btn_whatsapp, btn_telegram)
        bot.send_message(message.chat.id, "👩‍💼 قسم خدمة العملاء والإدارة:\nلتثبيت الطلبات أو الاستفسارات، اختر إحدى الطرق أدناه يا خال:", reply_markup=markup, parse_mode='Markdown')

@bot.callback_query_handler(func=lambda call: call.data.startswith('pcat_'))
def save_product_category(call):
    user_id = call.from_user.id
    if not is_user_admin(user_id):
        return
    cat = call.data.replace('pcat_', '')
    if user_id not in user_temp_product:
        user_temp_product[user_id] = {}
    user_temp_product[user_id]['category'] = cat
    bot.answer_callback_query(call.id, "✅ تم اختيار القسم بنجاح")
    msg = bot.send_message(call.message.chat.id, "💰 هلق أكتب سعر المنتج (مثال: 450 ليرة):")
    bot.register_next_step_handler(msg, get_product_price_step)

@bot.callback_query_handler(func=lambda call: call.data.startswith('more_'))
def handle_more_details(call):
    prod_id = int(call.data.split('_')[1])
    products = load_data(DATA_FILE)
    product = next((p for p in products if p['id'] == prod_id), None)
    if product:
        bot.answer_callback_query(call.id, f"🏷️ المنتج: {product['name']}\n💰 السعر: {product['price']}", show_alert=True)
    else:
        bot.answer_callback_query(call.id, "❌ عذراً، المنتج غير موجود.", show_alert=True)

@bot.callback_query_handler(func=lambda call: True)
def callback_handler(call):
    user_id = call.from_user.id
    if is_user_banned(user_id):
        bot.answer_callback_query(call.id, "❌ تم حظرك من استخدام البوت.", show_alert=True)
        return
        
    is_admin = is_user_admin(user_id)
    
    if call.data == 'cat_men':
        show_products_by_category(call.message, 'men')
    elif call.data == 'cat_women':
        show_products_by_category(call.message, 'women')
    elif call.data == 'cat_kids':
        show_products_by_category(call.message, 'kids')
    elif call.data == 'cat_shoes':
        show_products_by_category(call.message, 'shoes')
        
    elif call.data == 'cat_offers':
        bot.answer_callback_query(call.id)
        markup = telebot.types.InlineKeyboardMarkup()
        markup.add(telebot.types.InlineKeyboardButton('🔙 العودة للقائمة الرئيسية', callback_data='main_menu'))
        bot.edit_message_text("🔥 [ عروض العيد والحسومات الكبرى ]\n\nترقبوا أقوى الحسومات العائلية والتنزيلات الحصرية قريباً !", call.message.chat.id, call.message.message_id, reply_markup=markup, parse_mode='Markdown')
        
    elif call.data == 'store_info':
        bot.answer_callback_query(call.id)
        markup = telebot.types.InlineKeyboardMarkup()
        markup.add(telebot.types.InlineKeyboardButton('🔙 العودة للقائمة الرئيسية', callback_data='main_menu'))
        bot.edit_message_text("📍 [ معلومات وموقع متجر بيسان الحمصية ]\n\n✨ متخصصون بأفخم أنواع الألبسة والأحذية.\n🚚 يتوفر شحن وتوصيل لكافة المناطق.", call.message.chat.id, call.message.message_id, reply_markup=markup, parse_mode='Markdown')
        
    elif call.data == 'prices':
        bot.answer_callback_query(call.id)
        prices_text = (
            "📊 [ أسعار الصرف والذهب اليوم ]\n\n"
            "💵 الدولار مقابل التركي: ~32.50 ليرة\n"
            "💱 الدولار مقابل السوري: ~14800 ليرة\n"
            "🥇 غرام الذهب عيار 24: ~2450 ليرة تركية"
        )
        markup = telebot.types.InlineKeyboardMarkup()
        markup.add(telebot.types.InlineKeyboardButton('🔙 العودة للقائمة الرئيسية', callback_data='main_menu'))
        bot.edit_message_text(prices_text, call.message.chat.id, call.message.message_id, reply_markup=markup, parse_mode='Markdown')
        
    elif call.data == 'social_dl':
        bot.answer_callback_query(call.id)
        markup = telebot.types.InlineKeyboardMarkup(row_width=2)
        markup.add(
            telebot.types.InlineKeyboardButton('📥 يوتيوب (YouTube)', callback_data='dl_youtube'),
            telebot.types.InlineKeyboardButton('📥 انستغرام (Instagram)', callback_data='dl_insta'),
            telebot.types.InlineKeyboardButton('📥 تيك توك (TikTok)', callback_data='dl_tiktok'),
            telebot.types.InlineKeyboardButton('📥 فيسبوك (Facebook)', callback_data='dl_facebook'),
            telebot.types.InlineKeyboardButton('📥 ساوند كلاود (SoundCloud)', callback_data='dl_soundcloud'),
            telebot.types.InlineKeyboardButton('🔙 العودة للقائمة الرئيسية', callback_data='main_menu')
        )
        bot.edit_message_text("📥 [ قسم التحميل الشامل من السوشيال ميديا ]\nاختر المنصة المطلوبة يا خال:", call.message.chat.id, call.message.message_id, reply_markup=markup, parse_mode='Markdown')
        
    elif call.data in ['dl_youtube', 'dl_insta', 'dl_tiktok', 'dl_facebook', 'dl_soundcloud']:
        bot.answer_callback_query(call.id)
        platform_names = {
            'dl_youtube': 'يوتيوب 📺',
            'dl_insta': 'انستغرام 📸',
            'dl_tiktok': 'تيك توك 🎵',
            'dl_facebook': 'فيسبوك 📘',
            'dl_soundcloud': 'ساوند كلاود 🎧'
        }
        p_name = platform_names.get(call.data, 'السوشيال ميديا')
        msg = bot.send_message(call.message.chat.id, f"🔗 أرسل رابط الفيديو أو الملف من {p_name} الآن يا خال:", parse_mode='Markdown')
        bot.register_next_step_handler(msg, download_media_from_link)
        
    elif call.data == 'admin_panel' and is_admin:
        bot.answer_callback_query(call.id)
        markup = telebot.types.InlineKeyboardMarkup(row_width=1)
        markup.add(
            telebot.types.InlineKeyboardButton('➕ إضافة منتج جديد للقائمة', callback_data='step_add'),
            telebot.types.InlineKeyboardButton('📦 إدارة وحذف المنتجات الحالية', callback_data='manage_products'),
            telebot.types.InlineKeyboardButton('📊 إحصائيات وأيدي المستخدمين 👥', callback_data='show_stats'),
            telebot.types.InlineKeyboardButton('📢 نشر إعلان عام جديد', callback_data='make_broadcast'),
            telebot.types.InlineKeyboardButton('🔙 العودة للقائمة الرئيسية', callback_data='main_menu')
        )
        bot.edit_message_text("🔐 [ لوحة التحكم السرية للأدمين ]\nتحكم بكل تفاصيل المتجر يا زعيم:", call.message.chat.id, call.message.message_id, reply_markup=markup, parse_mode='Markdown')
        
    elif call.data == 'show_stats' and is_admin:
        bot.answer_callback_query(call.id)
        users = load_data(USERS_FILE)
        banned = load_data(BANNED_FILE)
        total_users = len(users)
        total_banned = len(banned)
        stats_text = (
            f"📊 [ إحصائيات بوت متجر الريان ]\n\n"
            f"👥 إجمالي عدد المستخدمين: {total_users} شخص\n"
            f"🚫 عدد المستخدمين المحظورين (المطرودين): {total_banned} شخص\n\n"
            f"📋 قائمة آخر المستخدمين وأدياتهم (للطرد):\n"
        )
        for u in users[-15:]:
            stats_text += f"👤 {u.get('name')} | أيدي: {u.get('id')} | {u.get('username')}\n"
        stats_text += "\n💡 لطرد أي شخص، استخدم الأمر التالي بالدردشة:\n/ban [أيدي_الشخص]\n\n➕ لإضافة أدمين جديد:\n/addadmin [أيدي_الشخص]"
        markup = telebot.types.InlineKeyboardMarkup()
        markup.add(telebot.types.InlineKeyboardButton('🔙 عودة للوحة التحكم', callback_data='admin_panel'))
        bot.edit_message_text(stats_text, call.message.chat.id, call.message.message_id, reply_markup=markup, parse_mode='Markdown')
        
    elif call.data == 'main_menu':
        bot.answer_callback_query(call.id)
        ad_data = load_data(AD_FILE)
        ad_text = ad_data.get('announcement', '')
        welcome_msg = (
            "╔═══════════════════════╗\n"
            "   👑 متجر ألبسة الريان  👑\n"
            "╚═══════════════════════╝\n\n"
            "🔥 أهلاً بك يا خال في عالم الفخامة والأناقة لكل العيلة!\n"
        )
        if ad_text:
            welcome_msg += f"\n📢 [ الإعلان هام جداً ]:\n{ad_text}\n"
        welcome_msg += "\n━━━━━━━━━━━━━━━━━━━\n👇 اختر القسم المطلوب من الأزرار الملونة أدناه:"
        bot.edit_message_text(welcome_msg, call.message.chat.id, call.message.message_id, reply_markup=get_main_menu_markup(user_id), parse_mode='Markdown')
        
    elif call.data == 'step_add' and is_admin:
        bot.answer_callback_query(call.id)
        msg = bot.send_message(call.message.chat.id, "📸 عيوني يا خال، أرسل صورة المنتج الآن:")
        bot.register_next_step_handler(msg, get_product_image_step)
        
    elif call.data == 'manage_products' and is_admin:
        bot.answer_callback_query(call.id)
        show_manage_menu(call.message)
        
    elif call.data == 'make_broadcast' and is_admin:
        bot.answer_callback_query(call.id)
        msg = bot.send_message(call.message.chat.id, "✍️ أكتب نص الإعلان العام الجديد ليظهر بالواجهة:")
        bot.register_next_step_handler(msg, save_broadcast_step)

def download_media_from_link(message):
    url = message.text.strip()
    if not url.startswith('http'):
        bot.reply_to(message, "❌ الرابط غير صحيح يا خال، يرجى إرسال رابط صالح يبدأ بـ http")
        return
    processing_msg = bot.reply_to(message, "📥 جارٍ معالجة وسحب الملف من الرابط، ثواني يا خال...", parse_mode='Markdown')
    output_template = 'downloads/%(id)s.%(ext)s'
    os.makedirs('downloads', exist_ok=True)
    ydl_opts = {
        'format': 'best',
        'outtmpl': output_template,
        'max_filesize': 50 * 1024 * 1024,
    }
    try:
        with yt_dlp.YoutubeDL(ydl_opts) as ydl:
            info = ydl.extract_info(url, download=True)
            filename = ydl.prepare_filename(info)
        if os.path.exists(filename):
            with open(filename, 'rb') as f:
                if filename.endswith(('.mp3', '.m4a', '.wav')):
                    bot.send_audio(message.chat.id, f, caption="🎵 تم التحميل بنجاح بواسطة بوت ساجدة")
                else:
                    bot.send_video(message.chat.id, f, caption="🎬 تم التحميل بنجاح بواسطة بوت ساجدة")
            try:
                os.remove(filename)
            except:
                pass
            bot.delete_message(message.chat.id, processing_msg.message_id)
        else:
            bot.edit_message_text("❌ حدث خطأ ولم يتم العثور على الملف يا خال.", message.chat.id, processing_msg.message_id)
    except Exception as e:
        bot.edit_message_text(f"❌ عذراً لم يتم التحميل:\n{str(e)[:100]}", message.chat.id, processing_msg.message_id, parse_mode='Markdown')

def show_products_by_category(message, category):
    products = load_data(DATA_FILE)
    cat_products = [p for p in products if p.get('category') == category]
    markup_back = telebot.types.InlineKeyboardMarkup()
    markup_back.add(telebot.types.InlineKeyboardButton('🔙 العودة للقائمة الرئيسية', callback_data='main_menu'))
    if not cat_products:
        bot.edit_message_text("🛒 لا توجد منتجات مضافة في هذا القسم حالياً يا خال.", message.chat.id, message.message_id, reply_markup=markup_back)
        return
    try:
        bot.delete_message(message.chat.id, message.message_id)
    except:
        pass
    for p in cat_products:
        markup = telebot.types.InlineKeyboardMarkup(row_width=2)
        
        whatsapp_text = f"مرحباً أستاذة ساجدة أريد شراء هذا المنتج:\n🏷️ المنتج: {p['name']}\n💰 السعر: {p['price']}\n✨ متجر ألبسة الريان"
        encoded_text = urllib.parse.quote(whatsapp_text)
        whatsapp_url = f"https://wa.me/963951557642?text={encoded_text}"
        
        btn_buy = telebot.types.InlineKeyboardButton('✅ شراء الآن', url=whatsapp_url)
        btn_more = telebot.types.InlineKeyboardButton('🔍 تفاصيل أكثر', callback_data=f'more_{p["id"]}')
        markup.add(btn_buy, btn_more)
        markup.add(telebot.types.InlineKeyboardButton('🔙 العودة للقائمة الرئيسية', callback_data='main_menu'))
        
        caption = f"🏷 المنتج: {p['name']}\n💰 السعر: {p['price']}\n✨ متجر ألبسة بيسان الحمصية"
        if p.get('image'):
            try:
                bot.send_photo(message.chat.id, p['image'], caption=caption, reply_markup=markup, parse_mode='Markdown')
            except:
                bot.send_message(message.chat.id, caption, reply_markup=markup, parse_mode='Markdown')
        else:
            bot.send_message(message.chat.id, caption, reply_markup=markup, parse_mode='Markdown')

def get_product_image_step(message):
    if not message.photo:
        bot.reply_to(message, "❌ يجب إرسال صورة حصراً للمنتج!")
        return
    file_id = message.photo[-1].file_id
    user_temp_product[message.from_user.id] = {'image': file_id}
    msg = bot.send_message(message.chat.id, "🏷 تمام، هلق أكتب اسم المنتج:")
    bot.register_next_step_handler(msg, get_product_name_step)

def get_product_name_step(message):
    user_temp_product[message.from_user.id]['name'] = message.text
    markup = telebot.types.InlineKeyboardMarkup(row_width=2)
    markup.add(
        telebot.types.InlineKeyboardButton('🟢 ألبسة رجالية', callback_data='pcat_men'),
        telebot.types.InlineKeyboardButton('🔵 نسائي وبناتي', callback_data='pcat_women'),
        telebot.types.InlineKeyboardButton('🟢 ألبسة أطفال', callback_data='pcat_kids'),
        telebot.types.InlineKeyboardButton('🔵 الأحذية', callback_data='pcat_shoes')
    )
    bot.send_message(message.chat.id, "📁 اختر القسم المناسب للمنتج:", reply_markup=markup)

def get_product_price_step(message):
    user_id = message.from_user.id
    price = message.text
    if user_id not in user_temp_product:
        return
    prod = user_temp_product[user_id]
    products = load_data(DATA_FILE)
    new_p = {
        'id': len(products) + 1,
        'name': prod.get('name'),
        'category': prod.get('category'),
        'price': price,
        'image': prod.get('image')
    }
    products.append(new_p)
    save_data(DATA_FILE, products)
    bot.send_message(message.chat.id, "✅ تم حفظ ونشر المنتج بنجاح!", reply_markup=get_main_menu_markup(user_id))

def show_manage_menu(message):
    products = load_data(DATA_FILE)
    if not products:
        bot.send_message(message.chat.id, "🛒 لا توجد منتجات حالياً للحذف.")
        return
    for p in products:
        markup = telebot.types.InlineKeyboardMarkup()
        markup.add(telebot.types.InlineKeyboardButton(f'❌ حذف ({p["name"]})', callback_data=f'del_{p["id"]}'))
        bot.send_message(message.chat.id, f"🏷 {p['name']} | 💰 {p['price']} | 📁 {p['category']}", reply_markup=markup)

@bot.callback_query_handler(func=lambda call: call.data.startswith('del_'))
def handle_delete_callback(call):
    if not is_user_admin(call.from_user.id):
        return
    prod_id = int(call.data.split('_')[1])
    products = load_data(DATA_FILE)
    products = [p for p in products if p['id'] != prod_id]
    save_data(DATA_FILE, products)
    bot.send_message(call.message.chat.id, "🗑️ تم حذف المنتج بنجاح.")

def save_broadcast_step(message5):
    ad_text = message5.text
    save_data(AD_FILE, {'announcement': ad_text})
    bot.send_message(message5.chat.id, f"✅ تم نشر الإعلان العام بنجاح:\n\n📢 {ad_text}", reply_markup=get_main_menu_markup(message5.from_user.id))

if __name__ == '__main__':
    print("Al-Bisan Bot with Multi-Admins & Stats System is running successfully...")
    bot.infinity_polling()
