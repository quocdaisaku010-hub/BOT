# BOT
import discord
from discord.ext import commands, tasks
import os
from dotenv import load_dotenv
import logging

# Tải biến môi trường
load_dotenv()

# Cấu hình logging
logging.basicConfig(level=logging.INFO)
logger = logging.getLogger(__name__)

# Tạo bot
intents = discord.Intents.default()
intents.message_content = True
intents.members = True

bot = commands.Bot(command_prefix="!", intents=intents)

# Lưu trữ thông báo
notifications = []

@bot.event
async def on_ready():
    """Sự kiện khi bot sẵn sàng"""
    logger.info(f'Bot đã đăng nhập thành công: {bot.user}')
    print(f'✅ Bot BONDMC {bot.user} đã kết nối!')
    print(f'🚀 Bot sẵn sàng phục vụ!')
    
    # Bắt đầu task gửi thông báo định kỳ
    send_notification_task.start()

@bot.command(name='thongbao')
async def send_notification(ctx, *, message):
    """
    Gửi thông báo đến kênh
    Cách dùng: !thongbao <nội dung thông báo>
    """
    try:
        embed = discord.Embed(
            title="📢 THÔNG BÁO TỪ BONDMC",
            description=message,
            color=discord.Color.blue()
        )
        embed.set_footer(text=f"Gửi bởi: {ctx.author}")
        
        await ctx.send(embed=embed)
        notifications.append(message)
        logger.info(f"Thông báo được gửi: {message}")
    except Exception as e:
        await ctx.send(f"❌ Lỗi: {str(e)}")
        logger.error(f"Lỗi khi gửi thông báo: {str(e)}")

@bot.command(name='hello')
async def hello(ctx):
    """Chào hỏi"""
    await ctx.send(f"👋 Xin chào {ctx.author.mention}! Tôi là **BONDMC** - Bot thông báo của bạn.")

@bot.command(name='help')
async def help_command(ctx):
    """Hiển thị danh sách lệnh"""
    embed = discord.Embed(
        title="📖 DANH SÁCH LỆNH - BONDMC",
        description="Các lệnh có sẵn:",
        color=discord.Color.green()
    )
    embed.add_field(name="!thongbao <nội dung>", value="Gửi thông báo", inline=False)
    embed.add_field(name="!hello", value="Chào hỏi bot", inline=False)
    embed.add_field(name="!help", value="Hiển thị danh sách lệnh", inline=False)
    embed.add_field(name="!lichsu", value="Xem lịch sử thông báo", inline=False)
    embed.add_field(name="!pingbot", value="Kiểm tra bot còn sống không", inline=False)
    
    await ctx.send(embed=embed)

@bot.command(name='lichsu')
async def history(ctx):
    """Xem lịch sử thông báo"""
    if not notifications:
        await ctx.send("📭 Chưa có thông báo nào.")
        return
    
    embed = discord.Embed(
        title="📜 LỊCH SỬ THÔNG BÁO - BONDMC",
        color=discord.Color.purple()
    )
    
    for i, notif in enumerate(notifications[-10:], 1):  # Hiển thị 10 thông báo gần nhất
        embed.add_field(name=f"{i}. ", value=notif, inline=False)
    
    await ctx.send(embed=embed)

@bot.command(name='pingbot')
async def ping(ctx):
    """Kiểm tra độ trễ của bot"""
    latency = round(bot.latency * 1000)
    embed = discord.Embed(
        title="🏓 PING - BONDMC",
        description=f"Độ trễ: **{latency}ms**",
        color=discord.Color.yellow()
    )
    await ctx.send(embed=embed)

@tasks.loop(hours=24)
async def send_notification_task():
    """Gửi thông báo định kỳ mỗi 24 giờ"""
    # Lấy channel để gửi thông báo
    channel_id = int(os.getenv('CHANNEL_ID', '0'))
    
    if channel_id == 0:
        logger.warning("CHANNEL_ID chưa được cấu hình")
        return
    
    try:
        channel = bot.get_channel(channel_id)
        if channel:
            embed = discord.Embed(
                title="📢 THÔNG BÁO ĐỊNH KỲ TỪ BONDMC",
                description="Đây là thông báo hàng ngày từ BONDMC!",
                color=discord.Color.orange()
            )
            await channel.send(embed=embed)
            logger.info("Thông báo định kỳ đã được gửi")
    except Exception as e:
        logger.error(f"Lỗi khi gửi thông báo định kỳ: {str(e)}")

@send_notification_task.before_loop
async def before_send_notification():
    """Đợi bot sẵn sàng trước khi bắt đầu task"""
    await bot.wait_until_ready()

@bot.event
async def on_message(message):
    """Xử lý tin nhắn"""
    if message.author == bot.user:
        return
    
    # Xử lý lệnh
    await bot.process_commands(message)

def main():
    """Hàm chính để khởi chạy bot"""
    token = os.getenv('DISCORD_TOKEN')
    
    if not token:
        print("❌ Lỗi: DISCORD_TOKEN chưa được cấu hình trong file .env")
        return
    
    try:
        bot.run(token)
    except Exception as e:
        logger.error(f"Lỗi khi khởi chạy bot: {str(e)}")

if __name__ == "__main__":
    main()
