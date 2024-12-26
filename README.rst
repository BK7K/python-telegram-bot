from telegram import Update, ChatPermissions
from telegram.ext import ApplicationBuilder, CommandHandler, MessageHandler, filters, ContextTypes

# التوكن الخاص بالبوت
BOT_TOKEN = "7877583021:AAFmM9RlfP8turM3FAdMKkjVPkjj5dwG-ng"

# قائمة المشرفين
ADMINS = [123456789]  # ضع معرفات المشرفين هنا

# الكلمات الممنوعة
BAD_WORDS = ["spam", "كلمة1", "كلمة2"]

# خوارزمية التحقق من الرسائل المشبوهة
def is_spam(message: str) -> bool:
    return any(word in message.lower() for word in BAD_WORDS)

# التحقق من إذا كان المستخدم مشرفاً
def is_admin(user_id: int) -> bool:
    return user_id in ADMINS

# أمر /start للترحيب
async def start(update: Update, context: ContextTypes.DEFAULT_TYPE) -> None:
    await update.message.reply_text("مرحباً، أنا بوت حماية! أعمل على حماية المجموعة 🚨")

# فلترة الرسائل
async def filter_messages(update: Update, context: ContextTypes.DEFAULT_TYPE) -> None:
    message_text = update.message.text
    user_id = update.effective_user.id

    if is_spam(message_text):
        await update.message.delete()
        await update.message.reply_text("🚫 تم حذف الرسالة لأنها تحتوي على محتوى ممنوع.")

# أمر لإضافة مشرف
async def add_admin(update: Update, context: ContextTypes.DEFAULT_TYPE) -> None:
    if is_admin(update.effective_user.id):
        if update.message.reply_to_message:
            new_admin_id = update.message.reply_to_message.from_user.id
            ADMINS.append(new_admin_id)
            await update.message.reply_text("✅ تم إضافة المستخدم إلى قائمة المشرفين.")
        else:
            await update.message.reply_text("❌ يرجى الرد على رسالة المستخدم لإضافته مشرفاً.")
    else:
        await update.message.reply_text("❌ ليس لديك الصلاحية لتنفيذ هذا الأمر.")

# أمر طرد المستخدم
async def ban_user(update: Update, context: ContextTypes.DEFAULT_TYPE) -> None:
    if is_admin(update.effective_user.id):
        if update.message.reply_to_message:
            user_id = update.message.reply_to_message.from_user.id
            await context.bot.ban_chat_member(chat_id=update.effective_chat.id, user_id=user_id)
            await update.message.reply_text("🚫 تم طرد المستخدم.")
        else:
            await update.message.reply_text("❌ يرجى الرد على رسالة المستخدم لطرده.")
    else:
        await update.message.reply_text("❌ ليس لديك الصلاحية لتنفيذ هذا الأمر.")

if __name__ == "__main__":
    app = ApplicationBuilder().token(BOT_TOKEN).build()

    # إضافة الأوامر والوظائف
    app.add_handler(CommandHandler("start", start))
    app.add_handler(CommandHandler("addadmin", add_admin))
    app.add_handler(CommandHandler("ban", ban_user))
    app.add_handler(MessageHandler(filters.TEXT & ~filters.COMMAND, filter_messages))

    # تشغيل البوت
    print("🚀 البوت يعمل الآن!")
    app.run_polling()
