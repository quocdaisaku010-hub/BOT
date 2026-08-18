# BOT
# main.py
import discord
from discord.ext import commands, tasks
import os
from dotenv import load_dotenv
import logging

load_dotenv()

logging.basicConfig(level=logging.INFO)
logger = logging.getLogger(__name__)

intents = discord.Intents.default()
intents.message_content = True
intents.members = True

bot = commands.Bot(command_prefix="!", intents=intents)

notifications = []

@bot.event
async def on_ready():
    logger.info(f'Bot đã đăng nhập: {bot.user}')
    print(f'✅ Bot BONDMC {bot.user} đã kết nối!')
    send_notification_task.start()

@bot.command(name='thongbao')
async def send_notification(ctx, *, message):
    try:
        embed = discord.Embed(
            title="📢 THÔNG BÁO TỪ BONDMC",
            description=message,
            color=discord.Color.blue()
        )
        embed.set_footer(text=f"Gửi bởi: {ctx.author}")
        await ctx.send(embed=embed)
        notifications.append(message)
    except Exception as e:
        await ctx.send(f"❌ Lỗi: {str(e)}")

@bot.command(name='hello')
async def hello(ctx):
    await ctx.send(f"👋 Xin chào {ctx.author.mention}! Tôi là BONDMC")

@bot.command(name='help')
async def help_command(ctx):
    embed = discord.Embed(title="📖 DANH SÁCH LỆNH", color=discord.Color.green())
    embed.add_field(name="!thongbao <nội dung>", value="Gửi thông báo", inline=False)
    embed.add_field(name="!hello", value="Chào hỏi", inline=False)
    embed.add_field(name="!lichsu", value="Xem lịch sử", inline=False)
    await ctx.send(embed=embed)

@bot.command(name='lichsu')
async def history(ctx):
    if not notifications:
        await ctx.send("📭 Chưa có thông báo")
        return
    embed = discord.Embed(title="📜 LỊCH SỬ", color=discord.Color.purple())
    for i, notif in enumerate(notifications[-10:], 1):
        embed.add_field(name=f"{i}", value=notif, inline=False)
    await ctx.send(embed=embed)

@tasks.loop(hours=24)
async def send_notification_task():
    channel_id = int(os.getenv('CHANNEL_ID', '0'))
    if channel_id == 0:
        return
    try:
        channel = bot.get_channel(channel_id)
        if channel:
            embed = discord.Embed(title="📢 THÔNG BÁO HÀNG NGÀY", color=discord.Color.orange())
            await channel.send(embed=embed)
    except Exception as e:
        logger.error(f"Lỗi: {str(e)}")

@send_notification_task.before_loop
async def before_send_notification():
    await bot.wait_until_ready()

@bot.event
async def on_message(message):
    if message.author == bot.user:
        return
    await bot.process_commands(message)

def main():
    token = os.getenv('DISCORD_TOKEN')
    if not token:
        print("❌ Lỗi: DISCORD_TOKEN chưa được cấu hình")
        return
    bot.run(token)

if __name__ == "__main__":
    main()
