import requests
from bs4 import BeautifulSoup
from telegram import Update, InlineKeyboardButton, InlineKeyboardMarkup
from telegram.ext import ApplicationBuilder, CommandHandler, ContextTypes, CallbackQueryHandler

# আপনার টোকেন
TOKEN = "8602432128:AAHCrgMefaaUlFp3Q9Jj0D_c5gsWEKEfgRw"

# ফ্রি এসএমএস সাইটগুলোর লিস্ট
SITES = {
    "Site 1 (Global)": "https://receive-smss.com/",
    "Site 2 (USA/UK)": "https://www.receivesms.co/",
    "Site 3 (Temporary)": "https://sms24.me/en/"
}

async def start(update: Update, context: ContextTypes.DEFAULT_TYPE):
    user_name = update.effective_user.first_name
    keyboard = [
        [InlineKeyboardButton("🌍 ফ্রি নম্বর দেখুন (Get Free Numbers)", callback_data='get_nums')],
        [InlineKeyboardButton("📢 চ্যানেল জয়েন করুন", url='https://t.me/your_channel_link')], # আপনার চ্যানেলের লিংক দিতে পারেন
        [InlineKeyboardButton("❓ সাহায্য (Help)", callback_data='help')]
    ]
    reply_markup = InlineKeyboardMarkup(keyboard)
    
    welcome_text = (
        f"হ্যালো {user_name}!\n\n"
        "স্বাগতম আমাদের **ফ্রি টেম্পোরারি নম্বর বটে**।\n"
        "এখানে আপনি বিভিন্ন দেশের পাবলিক নম্বর পাবেন যা দিয়ে ওটিপি (OTP) ভেরিফাই করা সম্ভব।"
    )
    await update.message.reply_text(welcome_text, reply_markup=reply_markup, parse_mode='Markdown')

async def button(update: Update, context: ContextTypes.DEFAULT_TYPE):
    query = update.callback_query
    await query.answer()

    if query.data == 'get_nums':
        await query.edit_message_text(text="🔍 সার্ভার থেকে নম্বরগুলো সংগ্রহ করছি... দয়া করে ১০-১৫ সেকেন্ড অপেক্ষা করুন।")
        
        try:
            # প্রথম সাইট থেকে স্ক্র্যাপিং (উদাহরণস্বরূপ)
            response = requests.get(SITES["Site 1 (Global)"], timeout=10)
            soup = BeautifulSoup(response.text, 'html.parser')
            
            # নম্বরগুলো খুঁজে বের করা (ওয়েবসাইটের স্ট্রাকচার অনুযায়ী)
            numbers = soup.find_all('div', class_='number-boxes-item-number')
            
            if numbers:
                msg = "✅ **বর্তমানে সচল ফ্রি নম্বরগুলো নিচে দেওয়া হলো:**\n\n"
                for i, num in enumerate(numbers[:8], 1): # প্রথম ৮টি নম্বর দেখাবে
                    clean_num = num.text.strip()
                    msg += f"{i}. 📞 `{clean_num}`\n"
                
                msg += "\n⚠️ **সতর্কতা:** এগুলো পাবলিক নম্বর। ওটিপি (OTP) দেখতে এই লিংকে যান: [Receive-SMSS](https://receive-smss.com/)"
                await query.edit_message_text(text=msg, parse_mode='Markdown', disable_web_page_preview=True)
            else:
                await query.edit_message_text(text="❌ এই মুহূর্তে কোনো ফ্রি নম্বর পাওয়া যায়নি। কিছুক্ষণ পর আবার চেষ্টা করুন।")
        
        except Exception as e:
            await query.edit_message_text(text=f"⚠️ এরর: সার্ভার রেসপন্স দিচ্ছে না।\nবিস্তারিত: {str(e)[:50]}")

    elif query.data == 'help':
        help_text = (
            "**কিভাবে ব্যবহার করবেন?**\n"
            "১. 'Get Free Numbers' বাটনে ক্লিক করুন।\n"
            "২. নম্বরটি কপি করে আপনার কাঙ্ক্ষিত অ্যাপে বসান।\n"
            "৩. ওটিপি দেখতে ওই ওয়েবসাইটের লিংকে গিয়ে নম্বরটি সার্চ করুন।"
        )
        await query.edit_message_text(text=help_text, parse_mode='Markdown')

if __name__ == '__main__':
    try:
        app = ApplicationBuilder().token(TOKEN).build()
        
        # হ্যান্ডলার সেটআপ
        app.add_handler(CommandHandler('start', start))
        app.add_handler(CallbackQueryHandler(button))
        
        print("-------------------------------")
        print("বটটি সফলভাবে চালু হয়েছে!")
        print("বন্ধ করতে চাইলে স্টপ বাটন চাপুন।")
        print("-------------------------------")
        
        app.run_polling()
    except Exception as e:
        print(f"বট চালু হতে সমস্যা হয়েছে: {e}")
        
