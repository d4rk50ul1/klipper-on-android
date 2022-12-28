# How to add support for the telegram bot

1. Follow the instructions [here](https://github.com/nlef/moonraker-telegram-bot/wiki/installation) to install the bot itself
2. Run the following commands:
```
sudo wget -O /etc/default/telegram https://raw.githubusercontent.com/d4rk50ul1/klipper-on-android/main/scripts/etc_default_telegram
sudo wget -O /etc/init.d/telegram https://raw.githubusercontent.com/d4rk50ul1/klipper-on-android/main/scripts/etc_init.d_telegram

sudo chmod +x /etc/init.d/telegram

sudo update-rc.d telegram defaults

sudo /etc/init.d/telegram start
```

The bot should be active now and auto start when you start the container
