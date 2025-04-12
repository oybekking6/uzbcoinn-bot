import logging from aiogram import Bot, Dispatcher, types from aiogram.types import InlineKeyboardMarkup, InlineKeyboardButton from aiogram.utils import executor import sqlite3

API_TOKEN = '8007238931:AAG1-3Gui43p39M7T3XQxccqVIbNWaMkbCg'

logging.basicConfig(level=logging.INFO) bot = Bot(token=API_TOKEN) dp = Dispatcher(bot)

conn = sqlite3.connect('uzbcoinn.db') cursor = conn.cursor() cursor.execute('''CREATE TABLE IF NOT EXISTS users (id INTEGER PRIMARY KEY, balance INTEGER DEFAULT 0, ref INTEGER DEFAULT 0)''') conn.commit()

ADMIN_ID = 884998981  # Sizning ID

Start komandasi

@dp.message_handler(commands=['start']) async def start(message: types.Message): user_id = message.from_user.id cursor.execute("SELECT * FROM users WHERE id=?", (user_id,)) if not cursor.fetchone(): ref = message.get_args() cursor.execute("INSERT INTO users (id, ref) VALUES (?, ?)", (user_id, ref if ref.isdigit() else 0)) conn.commit() if ref.isdigit(): cursor.execute("UPDATE users SET balance = balance + 50 WHERE id = ?", (ref,)) conn.commit() keyboard = InlineKeyboardMarkup() keyboard.add(InlineKeyboardButton("Tanga yig'ish", callback_data="collect")) keyboard.add(InlineKeyboardButton("Do‘st taklif qilish", callback_data="ref")) await message.answer("UzbCoinn botiga xush kelibsiz!", reply_markup=keyboard)

Callbacklar

@dp.callback_query_handler(lambda c: c.data == 'collect') async def collect(callback_query: types.CallbackQuery): user_id = callback_query.from_user.id cursor.execute("UPDATE users SET balance = balance + 1 WHERE id = ?", (user_id,)) conn.commit() cursor.execute("SELECT balance FROM users WHERE id = ?", (user_id,)) bal = cursor.fetchone()[0] await callback_query.answer(f"1 tanga qo‘shildi. Balans: {bal}")

@dp.callback_query_handler(lambda c: c.data == 'ref') async def ref(callback_query: types.CallbackQuery): user_id = callback_query.from_user.id await callback_query.message.answer(f"Sizning referal havolangiz:\nhttps://t.me/uzbcoinn_bot?start={user_id}")

Admin uchun

@dp.message_handler(commands=['panel']) async def admin_panel(message: types.Message): if message.from_user.id == ADMIN_ID: cursor.execute("SELECT COUNT(*) FROM users") count = cursor.fetchone()[0] await message.answer(f"Bot foydalanuvchilari soni: {count}") else: await message.answer("Siz admin emassiz.")

if name == 'main': executor.start_polling(dp, skip_updates=True)
