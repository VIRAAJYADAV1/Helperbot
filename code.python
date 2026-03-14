import telebot
import requests
import json
import sqlite3
from telebot import types
import os
from datetime import datetime, timedelta
import threading
import time
import random

# Bot Token - Apna token yahan daale
BOT_TOKEN = "YOUR_BOT_TOKEN_HERE"
bot = telebot.TeleBot(BOT_TOKEN)

# Database setup
def init_db():
    conn = sqlite3.connect('youtube_growth.db')
    c = conn.cursor()
    c.execute('''CREATE TABLE IF NOT EXISTS users 
                 (user_id INTEGER PRIMARY KEY, channel_id TEXT, channel_name TEXT, 
                  subscribers INTEGER, join_date TEXT)''')
    c.execute('''CREATE TABLE IF NOT EXISTS analytics 
                 (user_id INTEGER, date TEXT, subscribers INTEGER, views INTEGER, 
                  engagement_rate REAL)''')
    conn.commit()
    conn.close()

init_db()

# YouTube API Key (free tier se milega)
YOUTUBE_API_KEY = "YOUR_YOUTUBE_API_KEY"

# Main menu keyboard
def main_menu():
    markup = types.ReplyKeyboardMarkup(resize_keyboard=True)
    btn1 = types.KeyboardButton("📊 Channel Analysis")
    btn2 = types.KeyboardButton("🚀 Growth Strategies")
    btn3 = types.KeyboardButton("💡 Content Ideas")
    btn4 = types.KeyboardButton("📈 Analytics")
    btn5 = types.KeyboardButton("👥 Engagement Booster")
    btn6 = types.KeyboardButton("⚙️ Settings")
    markup.add(btn1, btn2, btn3, btn4)
    markup.add(btn5, btn6)
    return markup

# Channel analysis
def analyze_channel(channel_id):
    url = f"https://www.googleapis.com/youtube/v3/channels?part=statistics,snippet&id={channel_id}&key={YOUTUBE_API_KEY}"
    response = requests.get(url)
    data = response.json()
    
    if 'items' in data and len(data['items']) > 0:
        channel = data['items'][0]
        stats = channel['statistics']
        snippet = channel['snippet']
        
        subscribers = stats.get('subscriberCount', 'Hidden')
        views = stats.get('viewCount', 0)
        videos = stats.get('videoCount', 0)
        
        analysis = f"""
📊 **CHANNEL ANALYSIS**

👤 **Name**: {snippet['title']}
🆔 **ID**: {channel_id}
📺 **Videos**: {videos}
👥 **Subscribers**: {subscribers}
👁️ **Total Views**: {views:,}

💎 **Growth Score**: {calculate_growth_score(subscribers, views, videos)}
        """
        return analysis
    return "❌ Channel not found!"

def calculate_growth_score(subs, views, videos):
    score = 0
    if isinstance(subs, str): subs = 0
    subs = int(subs)
    views = int(views)
    videos = int(videos)
    
    if subs > 10000: score += 100
    elif subs > 1000: score += 80
    elif subs > 100: score += 50
    
    avg_views = views / videos if videos > 0 else 0
    if avg_views > 10000: score += 50
    elif avg_views > 5000: score += 30
    elif avg_views > 1000: score += 20
    
    return min(score, 100)

# Growth strategies generator
growth_strategies = {
    "new": ["Shorts banaye daily", "Trending topics pe videos", "Collaborations kare", "Cross-promote on Instagram"],
    "mid": ["SEO optimize titles", "Custom thumbnails", "Playlists banaye", "Community posts"],
    "pro": ["Live streams", "Merchandise", "Paid promotions", "Email list building"]
}

def get_growth_plan(subscribers):
    if subscribers < 1000:
        return "🚀 **Beginner Growth Plan**\n" + "\n".join(growth_strategies["new"])
    elif subscribers < 10000:
        return "📈 **Intermediate Plan**\n" + "\n".join(growth_strategies["mid"])
    else:
        return "💎 **Pro Growth Plan**\n" + "\n".join(growth_strategies["pro"])

# Content ideas generator
content_ideas = [
    "Top 10 Tips for [Topic]",
    "How to [Problem] in 5 Minutes",
    "2024 Guide to [Trend]",
    "[Popular Topic] Mistakes to Avoid",
    "Beginner Tutorial: [Skill]"
]

def generate_content_ideas(topic=""):
    ideas = []
    for i in range(5):
        idea = random.choice(content_ideas)
        if topic:
            idea = idea.replace("[Topic]", topic).replace("[Trend]", topic).replace("[Problem]", topic).replace("[Skill]", topic)
        ideas.append(f"{i+1}. {idea}")
    return "💡 **Viral Content Ideas**:\n" + "\n".join(ideas)

# Handlers
@bot.message_handler(commands=['start'])
def start(message):
    user_id = message.from_user.id
    conn = sqlite3.connect('youtube_growth.db')
    c = conn.cursor()
    c.execute("INSERT OR IGNORE INTO users (user_id, join_date) VALUES (?, ?)", 
              (user_id, datetime.now().strftime('%Y-%m-%d')))
    conn.commit()
    conn.close()
    
    welcome_msg = """
🤖 **YouTube Growth Bot** activated! 🚀

Main aapke YouTube channel ko grow karne mein madad karunga:
✅ Channel analysis
✅ Personalized growth strategies  
✅ Viral content ideas
✅ Engagement boosters
✅ Analytics tracking

Apna channel ID share kare ya menu se start kare!
    """
    bot.send_message(message.chat.id, welcome_msg, reply_markup=main_menu(), parse_mode='Markdown')

@bot.message_handler(func=lambda message: message.text == "📊 Channel Analysis")
def channel_analysis(message):
    msg = bot.send_message(message.chat.id, "🔍 Apna **YouTube Channel ID** bheje (ya channel URL se ID nikale):\n\nExample: UC_x5XG1OV2P6uZZ5FSM9Ttw")
    bot.register_next_step_handler(msg, process_channel_id)

def process_channel_id(message):
    channel_id = message.text.strip()
    
    # Extract channel ID from URL if needed
    if "youtube.com/channel/" in channel_id:
        channel_id = channel_id.split("channel/")[1].split("?")[0]
    elif "youtube.com/c/" in channel_id or "youtube.com/user/" in channel_id:
        # Handle username - need to convert to channel ID
        channel_id = get_channel_id_from_username(channel_id)
    
    analysis = analyze_channel(channel_id)
    bot.send_message(message.chat.id, analysis, parse_mode='Markdown')
    
    # Save channel for user
    user_id = message.from_user.id
    conn = sqlite3.connect('youtube_growth.db')
    c = conn.cursor()
    c.execute("UPDATE users SET channel_id=?, channel_name=? WHERE user_id=?", 
              (channel_id, "User Channel", user_id))
    conn.commit()
    conn.close()

def get_channel_id_from_username(username):
    # Simplified - in production use YouTube API
    return username.replace("youtube.com/c/", "").replace("youtube.com/user/", "")

@bot.message_handler(func=lambda message: message.text == "🚀 Growth Strategies")
def growth_strategies_handler(message):
    user_id = message.from_user.id
    conn = sqlite3.connect('youtube_growth.db')
    c = conn.cursor()
    c.execute("SELECT subscribers FROM users WHERE user_id=?", (user_id,))
    result = c.fetchone()
    conn.close()
    
    subs = result[0] if result and result[0] else 0
    plan = get_growth_plan(subs)
    bot.send_message(message.chat.id, plan, parse_mode='Markdown')

@bot.message_handler(func=lambda message: message.text == "💡 Content Ideas")
def content_ideas_handler(message):
    markup = types.InlineKeyboardMarkup()
    btn1 = types.InlineKeyboardButton("General Ideas", callback_data="ideas_general")
    btn2 = types.InlineKeyboardButton("Topic Specific", callback_data="ideas_topic")
    markup.add(btn1, btn2)
    bot.send_message(message.chat.id, "💡 Kya ideas chahiye?", reply_markup=markup)

@bot.callback_query_handler(func=lambda call: call.data.startswith("ideas_"))
def content_callback(call):
    if call.data == "ideas_general":
        ideas = generate_content_ideas()
        bot.edit_message_text(ideas, call.message.chat.id, call.message.message_id, parse_mode='Markdown')
    else:
        msg = bot.send_message(call.message.chat.id, "Topic bheje (jaise: tech, cooking, gaming):")
        bot.register_next_step_handler(msg, process_topic)

def process_topic(message):
    topic = message.text
    ideas = generate_content_ideas(topic)
    bot.send_message(message.chat.id, ideas, parse_mode='Markdown')

# Engagement booster
@bot.message_handler(func=lambda message: message.text == "👥 Engagement Booster")
def engagement_booster(message):
    booster_tips = """
🔥 **Engagement Booster Tips**:

1. **Community Posts**: Daily polls/questions
2. **Call-to-Actions**: "Like if you agree!"
3. **Reply to ALL comments** (first 1 hour)
4. **Pin best comments**
5. **Stories/Reels crosspost**
6. **End screens + Cards**

⚡ **Pro Tip**: First 30 mins = 80% algorithm boost!
    """
    bot.send_message(message.chat.id, booster_tips, parse_mode='Markdown')

# Run bot
if __name__ == "__main__":
    print("🤖 YouTube Growth Bot started!")
    bot.polling(none_stop=True)
