import json
import os
import re
import time
import telebot
from telebot.types import InlineKeyboardMarkup, InlineKeyboardButton

BOT_TOKEN = "8633380651:AAFtXT38U5Y8KBt1XLM_lrZGgsh5GG1IRQQ"
ADMIN_USERNAME = "rbxrajababu786"  # Bina @ ke apna Telegram username
ADMIN_CHAT_ID = 7969092612  # Apna numeric Telegram User ID
UPI_ID = "rijwanbagwan40@okhdfcbank"
PAYEE_NAME = "rijwan bagwan"
REQUIRED_REFERRALS = 3
DATA_FILE = "bot_data.json"

# Social links & Telegram live verification chat ID
WHATSAPP_CHANNEL_URL = "https://whatsapp.com/channel/0029VbE3DyO0rGiDtvR17l1k"
TELEGRAM_LINK_URL = "https://t.me/rbxrajachainal786"
TELEGRAM_CHAT_ID = -1003559075457
YOUTUBE_LINK_URL = "https://youtube.com/@ziyatechnical111?si=C6dOcLG45EDM9Uzb"

bot = telebot.TeleBot(BOT_TOKEN, parse_mode="HTML")
BOT_USERNAME = bot.get_me().username

# Users waiting to send payment proof
WAITING_PROOF = set()

DEFAULT_CATALOG = {
    "cat_pk_wp": {
        "title": "PAKISTAN WP +92",
        "price": 20,
        "stock": 25,
        "desc": "Fresh WhatsApp +92 number ready for use.",
    },
    "cat_in_wp_new": {
        "title": "NEW WP +91",
        "price": 20,
        "stock": 40,
        "desc": "New +91 WhatsApp account with clean status.",
    },
    "cat_in_wp_hq": {
        "title": "+91 HIGH QUALITY NEW",
        "price": 20,
        "stock": 15,
        "desc": "High quality +91 account with better stability.",
    },
    "cat_us_wp": {
        "title": "USA +1 WP",
        "price": 20,
        "stock": 18,
        "desc": "USA +1 WhatsApp number with instant delivery.",
    },
    "cat_us_tg": {
        "title": "USA +1 TG",
        "price": 20,
        "stock": 12,
        "desc": "USA +1 Telegram account ready for login.",
    },
}


def parse_price(val):
  if isinstance(val, (int, float)):
    return int(val)
  digits = re.sub(r"[^\d]", "", str(val))
  return int(digits) if digits else 0


def load_data():
  if os.path.exists(DATA_FILE):
    try:
      with open(DATA_FILE, "r", encoding="utf-8") as f:
        raw = json.load(f)
      users = {}
      now_ts = int(time.time())
      for uid_str, udata in raw.get("users", {}).items():
        users[int(uid_str)] = {
            "referred_by": udata.get("referred_by"),
            "referrals": [int(x) for x in udata.get("referrals", [])],
            "approved": bool(udata.get("approved", False)),
            "joined_socials": bool(udata.get("joined_socials", False)),
            "last_active": int(udata.get("last_active", now_ts)),
            "balance": int(udata.get("balance", 0)),
            "orders": list(udata.get("orders", [])),
        }
      catalog = raw.get("catalog", DEFAULT_CATALOG)
      for item in catalog.values():
        item["price"] = parse_price(item.get("price", 0))
        item["stock"] = int(item.get("stock", 0))
      return users, catalog
    except Exception as e:
      print(f"Error loading {DATA_FILE}: {e}")

  return {}, {k: v.copy() for k, v in DEFAULT_CATALOG.items()}


def save_data():
  try:
    payload = {
        "users": {str(uid): udata for uid, udata in USERS.items()},
        "catalog": CATALOG,
    }
    with open(DATA_FILE, "w", encoding="utf-8") as f:
      json.dump(payload, f, indent=2, ensure_ascii=False)
  except Exception as e:
    print(f"Error saving {DATA_FILE}: {e}")


USERS, CATALOG = load_data()


def ensure_user(user_id):
  now_ts = int(time.time())
  if user_id not in USERS:
    USERS[user_id] = {
        "referred_by": None,
        "referrals": [],
        "approved": False,
        "joined_socials": False,
        "last_active": now_ts,
        "balance": 0,
        "orders": [],
    }
    save_data()
  else:
    USERS[user_id].setdefault("balance", 0)
    USERS[user_id].setdefault("orders", [])
    USERS[user_id].setdefault("last_active", now_ts)
  return USERS[user_id]


def touch_user(user_id):
  u = ensure_user(user_id)
  u["last_active"] = int(time.time())
  save_data()
  return u


def get_monthly_users_count(days=30):
  cutoff = int(time.time()) - (days * 86400)
  return sum(
      1 for udata in USERS.values() if int(udata.get("last_active", 0)) >= cutoff
  )


def is_admin(user_id):
  return int(user_id) == int(ADMIN_CHAT_ID)


def get_telegram_join_status(user_id):
  try:
    member = bot.get_chat_member(TELEGRAM_CHAT_ID, user_id)
    if member.status in ["member", "administrator", "creator"]:
      return True, None
    return False, f"Status: {member.status}"
  except Exception as e:
    print(f"Telegram membership check error: {e}")
    return False, str(e)


def is_user_in_telegram(user_id):
  joined, _ = get_telegram_join_status(user_id)
  return joined


def get_referral_link(user_id):
  return f"https://t.me/{BOT_USERNAME}?start={user_id}"


def build_join_gate_menu():
  markup = InlineKeyboardMarkup(row_width=1)
  markup.add(
      InlineKeyboardButton(text="📢 Join WhatsApp Channel", url=WHATSAPP_CHANNEL_URL)
  )
  markup.add(
      InlineKeyboardButton(text="✈️ Join Telegram", url=TELEGRAM_LINK_URL)
  )
  markup.add(
      InlineKeyboardButton(text="▶️ Subscribe YouTube", url=YOUTUBE_LINK_URL)
  )
  markup.add(
      InlineKeyboardButton(
          text="✅ I Have Joined All", callback_data="verify_socials"
      )
  )
  return markup


def send_join_gate(chat_id):
  text = (
      "🔐 <b>ACCESS LOCKED — JOIN REQUIRED</b>\n"
      "━━━━━━━━━━━━━━━\n\n"
      "Bot aur store kholne ke liye pehle teeno channels join/subscribe"
      " karein:\n"
      "1️⃣ WhatsApp Channel\n"
      "2️⃣ Telegram (Live check hoga)\n"
      "3️⃣ YouTube\n\n"
      "Sab join karne ke baad <b>✅ I Have Joined All</b> dabayein:"
  )
  bot.send_message(chat_id, text, reply_markup=build_join_gate_menu())


def build_dashboard_menu():
  markup = InlineKeyboardMarkup(row_width=2)
  markup.add(
      InlineKeyboardButton(text="🛒 Shop Now", callback_data="open_shop")
  )
  markup.add(
      InlineKeyboardButton(text="📦 My Orders", callback_data="my_orders"),
      InlineKeyboardButton(text="👤 Profile", callback_data="my_profile"),
  )
  markup.add(
      InlineKeyboardButton(text="💰 Add Fund", callback_data="add_fund"),
      InlineKeyboardButton(text="📄 Payment Proof", callback_data="payment_proof"),
  )
  markup.add(
      InlineKeyboardButton(text="📖 How To Use", callback_data="how_to_use"),
      InlineKeyboardButton(text="💬 Support", callback_data="support_info"),
  )
  markup.add(
      InlineKeyboardButton(text="👥 My Refer & Approval", callback_data="my_refer")
  )
  return markup


def send_dashboard(chat_id, user):
  u = ensure_user(user.id)
  text = (
      "✨ <b>WELCOME TO DIGITAL STORE</b> ✨\n"
      "━━━━━━━━━━━━━━━\n\n"
      f"👋 Hey <b>{user.first_name}</b>!\n"
      "<i>Your Trusted Digital Marketplace</i>\n\n"
      "━━━━━━━━━━━━━━━\n"
      f"💰 <b>Your Balance:</b> ₹{u['balance']}\n"
      "━━━━━━━━━━━━━━━\n\n"
      "👇 <i>Choose an option below to continue</i>"
  )
  bot.send_message(chat_id, text, reply_markup=build_dashboard_menu())


def build_store_menu():
  markup = InlineKeyboardMarkup(row_width=1)
  for cat_id, item in CATALOG.items():
    label = f"{item['title']} — ₹{item['price']} ({item['stock']} left)"
    markup.add(InlineKeyboardButton(text=label, callback_data=cat_id))
  markup.add(
      InlineKeyboardButton(text="⬅ Back to Dashboard", callback_data="go_back")
  )
  return markup


def build_product_menu(cat_id):
  markup = InlineKeyboardMarkup(row_width=1)
  markup.add(
      InlineKeyboardButton(text="🛒 Buy Now", callback_data=f"buy_{cat_id}")
  )
  markup.add(
      InlineKeyboardButton(
          text="⬅ Back to Categories", callback_data="back_to_store"
      )
  )
  return markup


def format_refer_status(user_id):
  u = ensure_user(user_id)
  ref_count = len(u["referrals"])
  status_text = (
      "✅ <b>APPROVED</b>"
      if u["approved"]
      else f"⏳ <b>PENDING ({ref_count}/{REQUIRED_REFERRALS} Refers)</b>"
  )
  link = get_referral_link(user_id)
  return (
      "👥 <b>REFERRAL & APPROVAL STATUS</b>\n"
      "━━━━━━━━━━━━━━━\n"
      f"📌 <b>Status:</b> {status_text}\n"
      f"📊 <b>Total Refers:</b> {ref_count} / {REQUIRED_REFERRALS}\n\n"
      f"🔗 <b>Your Referral Link:</b>\n<code>{link}</code>\n\n"
      f"💡 Jab <b>{REQUIRED_REFERRALS} log</b> aapke link se bot start karenge,"
      " tabhi aapko <b>APPROVAL</b> milega."
  )


@bot.message_handler(commands=["addbal"])
def add_balance_command(message):
  if not is_admin(message.from_user.id):
    bot.send_message(message.chat.id, "❌ Yeh command sirf Admin ke liye hai.")
    return
  parts = message.text.split()
  if len(parts) != 3:
    bot.send_message(
        message.chat.id, "Usage: <code>/addbal &lt;user_id&gt; &lt;amount&gt;</code>"
    )
    return
  try:
    target_id = int(parts[1])
    amount = int(parts[2])
    u = ensure_user(target_id)
    u["balance"] = max(0, u["balance"] + amount)
    save_data()
    bot.send_message(
        message.chat.id,
        f"✅ Updated User <code>{target_id}</code> balance: <b>₹{u['balance']}</b>",
    )
    try:
      bot.send_message(
          target_id,
          f"💰 <b>BALANCE UPDATED</b>\nAapke wallet mein <b>₹{amount}</b> add"
          f" kiya gaya hai.\nNew Balance: <b>₹{u['balance']}</b>",
      )
    except Exception:
      pass
  except ValueError:
    bot.send_message(message.chat.id, "❌ User ID aur Amount number mein dein.")


@bot.message_handler(commands=["stats"])
def stats_command(message):
  if not is_admin(message.from_user.id):
    bot.send_message(message.chat.id, "❌ Yeh command sirf Admin ke liye hai.")
    return

  total_users = len(USERS)
  monthly_users = get_monthly_users_count(30)
  approved_users = sum(1 for u in USERS.values() if u.get("approved"))

  bot.send_message(
      message.chat.id,
      "📈 <b>BOT USER STATS</b>\n"
      "━━━━━━━━━━━━━━━\n"
      f"📅 <b>Monthly Users (30d):</b> {monthly_users}\n"
      f"👥 <b>Total Users:</b> {total_users}\n"
      f"✅ <b>Approved Users:</b> {approved_users}",
  )


@bot.message_handler(commands=["checkid"])
def check_id_command(message):
  touch_user(message.from_user.id)
  uid = message.from_user.id
  try:
    chat_info = bot.get_chat(TELEGRAM_CHAT_ID)
    member = bot.get_chat_member(TELEGRAM_CHAT_ID, uid)
    bot.send_message(
        message.chat.id,
        "✅ <b>CHECK SUCCESS</b>\n"
        f"• Chat Title: <b>{chat_info.title}</b>\n"
        f"• Chat ID: <code>{TELEGRAM_CHAT_ID}</code>\n"
        f"• Your Status: <b>{member.status}</b>",
    )
  except Exception as e:
    bot.send_message(
        message.chat.id,
        "❌ <b>CHECK ERROR</b>\n"
        f"<code>{e}</code>",
    )


@bot.message_handler(commands=["broadcast"])
def broadcast_command(message):
  if not is_admin(message.from_user.id):
    bot.send_message(message.chat.id, "❌ Yeh command sirf Admin ke liye hai.")
    return

  parts = message.text.split(maxsplit=1)
  broadcast_text = parts[1].strip() if len(parts) == 2 else ""
  reply_msg = message.reply_to_message

  if not broadcast_text and not reply_msg:
    bot.send_message(
        message.chat.id,
        "📣 <b>BROADCAST USAGE</b>\n"
        "━━━━━━━━━━━━━━━\n"
        "• Text bhejne ke liye:\n"
        "<code>/broadcast Naya stock aa gaya hai!</code>\n\n"
        "• Ya kisi message par reply karke likhein:\n"
        "<code>/broadcast</code>",
    )
    return

  all_users = list(USERS.keys())
  if not all_users:
    bot.send_message(message.chat.id, "⚠️ Abhi koi user saved nahi hai.")
    return

  status_msg = bot.send_message(
      message.chat.id,
      f"⏳ Broadcasting to <b>{len(all_users)}</b> users...",
  )

  sent_count = 0
  failed_count = 0

  for uid in all_users:
    try:
      if reply_msg:
        bot.copy_message(
            chat_id=uid,
            from_chat_id=reply_msg.chat.id,
            message_id=reply_msg.message_id,
        )
      else:
        formatted_msg = (
            "📢 <b>ANNOUNCEMENT</b>\n━━━━━━━━━━━━━━━\n\n" f"{broadcast_text}"
        )
        bot.send_message(uid, formatted_msg)
      sent_count += 1
      time.sleep(0.05)
    except Exception:
      failed_count += 1

  bot.edit_message_text(
      chat_id=message.chat.id,
      message_id=status_msg.message_id,
      text=(
          "✅ <b>BROADCAST COMPLETE</b>\n"
          "━━━━━━━━━━━━━━━\n"
          f"👥 Total Users: <b>{len(all_users)}</b>\n"
          f"✉️ Sent: <b>{sent_count}</b>\n"
          f"⚠️ Failed / Blocked: <b>{failed_count}</b>"
      ),
  )


@bot.message_handler(commands=["refer"])
def refer_command(message):
  u = touch_user(message.from_user.id)
  if not u["joined_socials"] or not is_user_in_telegram(message.from_user.id):
    u["joined_socials"] = False
    save_data()
    send_join_gate(message.chat.id)
    return
  bot.send_message(message.chat.id, format_refer_status(message.from_user.id))


@bot.message_handler(commands=["start"])
def send_welcome(message):
  user = message.from_user
  is_new_user = user.id not in USERS
  u = touch_user(user.id)

  if is_new_user and not is_admin(user.id):
    username_display = f"@{user.username}" if user.username else "No Username"
    new_user_alert = (
        "🙂 <b>New User Started Bot</b>\n\n"
        f"👉 <b>User:</b> <code>{user.id}</code>\n"
        f"👤 <b>Name:</b> {user.first_name}\n"
        f"🔗 <b>Username:</b> {username_display}\n"
        f"📅 <b>Monthly Users:</b> {get_monthly_users_count(30)}\n"
        f"📊 <b>Total Users:</b> {len(USERS)}"
    )
    try:
      bot.send_message(ADMIN_CHAT_ID, new_user_alert)
    except Exception as e:
      print(f"New user alert failed: {e}")

  parts = message.text.split(maxsplit=1)
  if len(parts) == 2 and is_new_user:
    try:
      referrer_id = int(parts[1].strip())
      if referrer_id != user.id and referrer_id in USERS:
        ref_user = ensure_user(referrer_id)
        if user.id not in ref_user["referrals"]:
          u["referred_by"] = referrer_id
          ref_user["referrals"].append(user.id)
          current_refs = len(ref_user["referrals"])

          if current_refs >= REQUIRED_REFERRALS and not ref_user["approved"]:
            ref_user["approved"] = True

          save_data()

          try:
            bot.send_message(
                referrer_id,
                "🎉 <b>NEW REFER JOINED!</b>\n"
                f"👤 User: {user.first_name}\n"
                f"📊 Progress: <b>{current_refs}/{REQUIRED_REFERRALS}</b>",
            )
          except Exception:
            pass

          if ref_user["approved"] and current_refs == REQUIRED_REFERRALS:
            try:
              bot.send_message(
                  referrer_id,
                  "🏆 <b>CONGRATULATIONS! APPROVAL GRANTED</b>\n"
                  "━━━━━━━━━━━━━━━\n"
                  f"Aapke <b>{REQUIRED_REFERRALS} refers</b> poore ho gaye"
                  " hain!\n"
                  "Ab aapka account <b>APPROVED</b> hai.",
              )
              bot.send_message(
                  ADMIN_CHAT_ID,
                  "✅ <b>USER AUTO-APPROVED BY REFERRAL</b>\n"
                  f"🆔 User ID: <code>{referrer_id}</code>\n"
                  f"👥 Refers Completed: {current_refs}/{REQUIRED_REFERRALS}",
              )
            except Exception:
              pass
    except ValueError:
      pass

  if not u["joined_socials"] or not is_user_in_telegram(user.id):
    u["joined_socials"] = False
    save_data()
    send_join_gate(message.chat.id)
    return

  send_dashboard(message.chat.id, user)


@bot.callback_query_handler(func=lambda call: True)
def handle_callback(call):
  data = call.data
  user = call.from_user
  user_id = user.id
  u = touch_user(user_id)

  if data == "verify_socials":
    joined, reason = get_telegram_join_status(user_id)
    if not joined:
      msg = (
          "❌ Pehle Telegram channel/group join karein, phir dobara try karein!"
      )
      if reason and not reason.startswith("Status:"):
        msg = f"⚠️ Check error: {reason[:90]}"
      bot.answer_callback_query(call.id, msg, show_alert=True)
      return

    bot.answer_callback_query(call.id, "✅ Verification complete!")
    u["joined_socials"] = True
    save_data()
    try:
      bot.delete_message(call.message.chat.id, call.message.message_id)
    except Exception:
      pass
    send_dashboard(call.message.chat.id, user)
    return

  bot.answer_callback_query(call.id)

  if not u["joined_socials"] or not is_user_in_telegram(user_id):
    u["joined_socials"] = False
    save_data()
    send_join_gate(call.message.chat.id)
    return

  if data == "open_shop":
    store_text = (
        "🛒 <b>STORE — SHOP</b>\n━━━━━━━━━━━━━━━\n\n📱 Select a category to"
        " browse:"
    )
    bot.edit_message_text(
        chat_id=call.message.chat.id,
        message_id=call.message.message_id,
        text=store_text,
        reply_markup=build_store_menu(),
    )

  elif data == "my_orders":
    orders = u.get("orders", [])
    if not orders:
      msg = (
          "📦 <b>MY ORDERS</b>\n━━━━━━━━━━━━━━━\n\nAbhi aapka koi order nahi"
          " hai."
      )
    else:
      lines = ["📦 <b>MY ORDERS</b>\n━━━━━━━━━━━━━━━\n"]
      for idx, od in enumerate(orders[-10:], start=1):
        lines.append(f"{idx}. <b>{od['title']}</b> — ₹{od['price']}")
      msg = "\n".join(lines)
    bot.send_message(call.message.chat.id, msg)

  elif data == "my_profile":
    status_txt = "✅ APPROVED" if u["approved"] else "⏳ PENDING"
    msg = (
        "👤 <b>USER PROFILE</b>\n"
        "━━━━━━━━━━━━━━━\n"
        f"• <b>Name:</b> {user.first_name}\n"
        f"• <b>User ID:</b> <code>{user_id}</code>\n"
        f"• <b>Balance:</b> ₹{u['balance']}\n"
        f"• <b>Refers:</b> {len(u['referrals'])}/{REQUIRED_REFERRALS}\n"
        f"• <b>Approval:</b> {status_txt}"
    )
    bot.send_message(call.message.chat.id, msg)

  elif data == "add_fund":
    msg = (
        "💰 <b>ADD FUND</b>\n"
        "━━━━━━━━━━━━━━━\n\n"
        "💳 <b>UPI Payment Info:</b>\n"
        f"• <b>UPI ID:</b> <code>{UPI_ID}</code>\n"
        f"• <b>Name:</b> {PAYEE_NAME}\n\n"
        "Payment karne ke baad <b>Payment Proof</b> button dabayein aur"
        " screenshot/UTR bhejein."
    )
    bot.send_message(call.message.chat.id, msg)

  elif data == "payment_proof":
    WAITING_PROOF.add(user_id)
    bot.send_message(
        call.message.chat.id,
        "📄 <b>PAYMENT PROOF</b>\n"
        "━━━━━━━━━━━━━━━\n\n"
        "Apna payment <b>screenshot ya transaction details</b> abhi is chat"
        " mein bhejein. Admin ko forward kar diya jayega.",
    )

  elif data == "how_to_use":
    msg = (
        "📖 <b>HOW TO USE</b>\n"
        "━━━━━━━━━━━━━━━\n"
        "1. Sabhi channels join karke verify karein.\n"
        "2. <b>3 refers</b> poore karke <b>APPROVAL</b> lein (/refer).\n"
        "3. <b>Shop Now</b> se item chunein aur <b>Buy Now</b> dabayein.\n"
        "4. Payment karke <b>Payment Proof</b> bhejein."
    )
    bot.send_message(call.message.chat.id, msg)

  elif data == "support_info":
    bot.send_message(
        call.message.chat.id,
        "💬 <b>SUPPORT</b>\n"
        "━━━━━━━━━━━━━━━\n"
        f"Kisi bhi madad ya delivery ke liye @{ADMIN_USERNAME} se sampark"
        " karein.",
    )

  elif data == "my_refer":
    bot.send_message(call.message.chat.id, format_refer_status(user_id))

  elif data == "go_back":
    bot.delete_message(call.message.chat.id, call.message.message_id)
    send_dashboard(call.message.chat.id, user)

  elif data == "back_to_store":
    store_text = (
        "🛒 <b>STORE — SHOP</b>\n━━━━━━━━━━━━━━━\n\n📱 Select a category to"
        " browse:"
    )
    bot.edit_message_text(
        chat_id=call.message.chat.id,
        message_id=call.message.message_id,
        text=store_text,
        reply_markup=build_store_menu(),
    )

  elif data in CATALOG:
    item = CATALOG[data]
    if item["stock"] <= 0:
      bot.edit_message_text(
          chat_id=call.message.chat.id,
          message_id=call.message.message_id,
          text=(
              f"❌ <b>{item['title']}</b>\n━━━━━━━━━━━━━━━\n\nOut of stock!"
              " Kripya baad mein check karein."
          ),
          reply_markup=build_product_menu(data),
      )
      return

    details_text = (
        f"📦 <b>{item['title']}</b>\n"
        "━━━━━━━━━━━━━━━\n"
        f"💰 <b>Price:</b> ₹{item['price']}\n"
        f"📊 <b>Available Stock:</b> {item['stock']}\n"
        f"📝 <b>Info:</b> {item['desc']}\n\n"
        "Order place karne ke liye neeche <b>Buy Now</b> dabayein:"
    )
    bot.edit_message_text(
        chat_id=call.message.chat.id,
        message_id=call.message.message_id,
        text=details_text,
        reply_markup=build_product_menu(data),
    )

  elif data.startswith("buy_"):
    if not u["approved"]:
      ref_count = len(u["referrals"])
      bot.send_message(
          call.message.chat.id,
          "🔒 <b>APPROVAL REQUIRED</b>\n"
          "━━━━━━━━━━━━━━━\n"
          f"Order karne ke liye <b>{REQUIRED_REFERRALS} referrals</b> zaroori"
          " hain.\n"
          f"📊 Aapke abhi <b>{ref_count}/{REQUIRED_REFERRALS}</b> refers"
          " hue hain.\n\n"
          f"🔗 Apna link share karein:\n<code>{get_referral_link(user_id)}</code>",
      )
      return

    cat_id = data.replace("buy_", "", 1)
    item = CATALOG.get(cat_id)

    if not item or item["stock"] <= 0:
      bot.send_message(
          call.message.chat.id, "❌ Sorry, yeh item abhi stock mein nahi hai."
      )
      return

    price_num = parse_price(item["price"])
    paid_from_wallet = False

    if u["balance"] >= price_num and price_num > 0:
      u["balance"] -= price_num
      paid_from_wallet = True

    item["stock"] -= 1
    u["orders"].append({
        "title": item["title"],
        "price": price_num,
        "ts": int(time.time()),
    })
    save_data()

    username_display = f"@{user.username}" if user.username else user.first_name

    if paid_from_wallet:
      buyer_msg = (
          "✅ <b>ORDER CONFIRMED (WALLET PAID)</b>\n"
          "━━━━━━━━━━━━━━━\n"
          f"📦 <b>Item:</b> {item['title']}\n"
          f"💰 <b>Amount Paid:</b> ₹{price_num}\n"
          f"💼 <b>Remaining Balance:</b> ₹{u['balance']}\n"
          f"📉 <b>Remaining Stock:</b> {item['stock']}\n\n"
          f"Delivery ke liye @{ADMIN_USERNAME} se sampark karein."
      )
    else:
      buyer_msg = (
          "✅ <b>ORDER CREATED — PAYMENT DETAILS</b>\n"
          "━━━━━━━━━━━━━━━\n"
          f"📦 <b>Item:</b> {item['title']}\n"
          f"💰 <b>Amount:</b> ₹{price_num}\n"
          f"📉 <b>Remaining Stock:</b> {item['stock']}\n\n"
          "💳 <b>UPI Payment Info:</b>\n"
          f"• <b>UPI ID:</b> <code>{UPI_ID}</code>\n"
          f"• <b>Name:</b> {PAYEE_NAME}\n\n"
          f"Payment ke baad screenshot @{ADMIN_USERNAME} ko bhejein."
      )
    bot.send_message(call.message.chat.id, buyer_msg)

    admin_alert = (
        "🔔 <b>NEW ORDER ALERT!</b>\n"
        "━━━━━━━━━━━━━━━\n"
        f"👤 <b>User:</b> {username_display}\n"
        f"🆔 <b>User ID:</b> <code>{user.id}</code>\n"
        f"📦 <b>Item:</b> {item['title']}\n"
        f"💰 <b>Price:</b> ₹{price_num}\n"
        f"💳 <b>Mode:</b> {'Wallet' if paid_from_wallet else 'UPI'}\n"
        f"📊 <b>Stock Left:</b> {item['stock']}"
    )
    try:
      bot.send_message(ADMIN_CHAT_ID, admin_alert)
    except Exception as e:
      print(f"Admin alert failed: {e}")


@bot.message_handler(content_types=["photo", "text", "document"])
def handle_user_messages(message):
  if message.text and message.text.startswith("/"):
    return
  uid = message.from_user.id
  if uid in WAITING_PROOF:
    WAITING_PROOF.remove(uid)
    username_display = (
        f"@{message.from_user.username}"
        if message.from_user.username
        else message.from_user.first_name
    )
    try:
      bot.send_message(
          ADMIN_CHAT_ID,
          "📄 <b>PAYMENT PROOF RECEIVED</b>\n"
          f"👤 User: {username_display} (<code>{uid}</code>)",
      )
      bot.forward_message(ADMIN_CHAT_ID, message.chat.id, message.message_id)
      bot.send_message(
          message.chat.id,
          "✅ Aapka payment proof admin ko bhej diya gaya hai!",
      )
    except Exception as e:
      print(f"Proof forward failed: {e}")


print("Bot is running...")
bot.infinity_polling()
