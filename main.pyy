import os
import logging
import asyncio
from aiohttp import web
import yfinance as yt
import yt_dlp
from telegram import Update, InlineKeyboardButton, InlineKeyboardMarkup
from telegram.ext import ApplicationBuilder, CommandHandler, MessageHandler, CallbackQueryHandler, filters, ContextTypes

# توكن البوت
TOKEN = os.getenv("BOT_TOKEN", "8996385082:AAHctaC2YFM_6_fQmoIPdPDvUySVHtXL_E4")

logging.basicConfig(
    format='%(asctime)s - %(name)s - %(levelname)s - %(message)s',
    level=logging.INFO
)

# ----------------- 1. سيرفر HTTP لفحص الصحة (Render Health Check) -----------------
async def handle_health_check(request):
    return web.Response(text="Bot is running smoothly!")

async def start_web_server():
    app = web.Application()
    app.router.add_get('/', handle_health_check)
    runner = web.AppRunner(app)
    await runner.setup()
    port = int(os.environ.get("PORT", 8080))
    site = web.TCPSite(runner, "0.0.0.0", port)
    await site.start()
    logging.info(f"Web server started on port {port}")

# ----------------- 2. أوامر البوت -----------------
async def start(update: Update, context: ContextTypes.DEFAULT_TYPE):
    keyboard = [
        [InlineKeyboardButton("📈 تحليل أسهم تاسي", callback_data="tasi_info")],
        [InlineKeyboardButton("📥 تحميل فيديو (YouTube/TikTok/X/Insta)", callback_data="downloader_info")],
        [InlineKeyboardButton("📝 المساعد الشخصي والوظائف", callback_data="ai_info")]
    ]
    reply_markup = InlineKeyboardMarkup(keyboard)
    text = (
        "👋 **أهلاً بك في البوت الشامل الذكي!**\n\n"
        "إليك كيفية استخدام الخدمات:\n"
        "🔹 **تحليل أسهم تاسي:** أرسل `/stock 2222` (مثال لأرامكو)\n"
        "🔹 **تحميل المقاطع:** أرسل رابط الفيديو مباشرة في المحادثة\n"
        "🔹 **تحليل النصوص والوظائف:** أرسل أي نص أو استفسار عن الوظائف"
    )
    if update.message:
        await update.message.reply_text(text, reply_markup=reply_markup, parse_mode="Markdown")
    else:
        await update.callback_query.message.reply_text(text, reply_markup=reply_markup, parse_mode="Markdown")

async def stock_analyzer(update: Update, context: ContextTypes.DEFAULT_TYPE):
    if not context.args:
        await update.message.reply_text("⚠️ يرجى كتابة رمز السهم بعد الأمر، مثال:\n`/stock 2222` أو `/stock 1120`", parse_mode="Markdown")
        return
    
    symbol = context.args[0]
    ticker_symbol = f"{symbol}.SR" if not symbol.endswith(".SR") else symbol
    
    await update.message.reply_text("🔄 جاري جلب بيانات السهم من سوق تاسي...")
    try:
        stock = yt.Ticker(ticker_symbol)
        info = stock.info
        
        name = info.get('longName', info.get('shortName', 'غير متاح'))
        price = info.get('currentPrice', info.get('regularMarketPrice', 'N/A'))
        prev_close = info.get('previousClose', 'N/A')
        div_yield = info.get('dividendYield', 0)
        div_yield_pct = f"{div_yield * 100:.2f}%" if div_yield else "لا يوجد/غير متاح"
        
        report = (
            f"📊 **تقرير سهم سوق تاسي**\n"
            f"━━━━━━━━━━━━━━━━━━\n"
            f"🔹 **اسم السهم:** {name}\n"
            f"🔹 **رمز السهم:** `{symbol}`\n"
            f"💵 **السعر الحالي:** {price} ر.س\n"
            f"📉 **الإغلاق السابق:** {prev_close} ر.س\n"
            f"💰 **عائد توزيع الأرباح:** {div_yield_pct}\n"
        )
        await update.message.reply_text(report, parse_mode="Markdown")
    except Exception as e:
        await update.message.reply_text("❌ تعذر جلب بيانات السهم. تأكد من صحة الرمز.")

async def download_media(update: Update, context: ContextTypes.DEFAULT_TYPE):
    url = update.message.text
    if not url.startswith(("http://", "https://")):
        return

    msg = await update.message.reply_text("⏳ جاري معالجة وتحميل المقطع...")
    ydl_opts = {
        'format': 'best',
        'outtmpl': 'downloaded_video.mp4',
        'max_filesize': 50 * 1024 * 1024
    }
    
    try:
        with yt_dlp.YoutubeDL(ydl_opts) as ydl:
            ydl.download([url])
        
        with open('downloaded_video.mp4', 'rb') as video:
            await update.message.reply_video(video=video, caption="✅ تم التحميل بنجاح!")
        
        if os.path.exists('downloaded_video.mp4'):
            os.remove('downloaded_video.mp4')
        await msg.delete()
    except Exception as e:
        await msg.edit_text("❌ حدث خطأ أثناء التحميل. تأكد من صحة الرابط وأن حجم الفيديو لا يتجاوز 50 ميجابايت.")

async def text_assistant(update: Update, context: ContextTypes.DEFAULT_TYPE):
    text = update.message.text
    
    if "سيرة ذاتية" in text or "وظيفة" in text or "تقديم" in text:
        reply = (
            "💼 **مساعد التقديم على الوظائف:**\n\n"
            "1. **تحسين السيرة الذاتية (ATS):** تأكد من استخدام الكلمات المفتاحية الموجودة في الوصف الوظيفي.\n"
            "2. **خطاب التغطية (Cover Letter):** اجعل الخطاب مخصصاً للشركة ويشرح كيف تحل مشكلاتهم.\n"
            "3. **منصة LinkedIn:** اضبط المسمى الوظيفي وخانة (About) لتبدو كخبير في مجالك."
        )
    else:
        words_count = len(text.split())
        char_count = len(text)
        reply = (
            f"📝 **تحليل النص السريع:**\n\n"
            f"🔹 عدد الكلمات: {words_count}\n"
            f"🔹 عدد الحروف: {char_count}\n"
            f"💡 *نصيحة مساعدك الشخصي:* تم استلام النص بنجاح ويمكنك توجيهه لأي أمر تخصيص."
        )
    
    await update.message.reply_text(reply, parse_mode="Markdown")

# ----------------- 3. تشغيل الخادم والبوت معاً -----------------
async def main():
    await start_web_server()
    
    application = ApplicationBuilder().token(TOKEN).build()
    application.add_handler(CommandHandler("start", start))
    application.add_handler(CommandHandler("stock", stock_analyzer))
    application.add_handler(MessageHandler(filters.Regex(r'https?://'), download_media))
    application.add_handler(MessageHandler(filters.TEXT & ~filters.COMMAND, text_assistant))
    
    async with application:
        await application.initialize()
        await application.start()
        await application.updater.start_polling(drop_pending_updates=True)
        logging.info(">>> البوت يعمل الآن بنجاح <<<")
        await asyncio.Event().wait()

if __name__ == '__main__':
    asyncio.run(main())
