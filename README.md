import os
import logging
from dotenv import load_dotenv
from telegram.ext import Updater, CommandHandler, MessageHandler, Filters
from pytube import YouTube

# Charger les variables d'environnement depuis le fichier .env
load_dotenv()

# Configurer les logs
logging.basicConfig(format='%(asctime)s - %(name)s - %(levelname)s - %(message)s', level=logging.INFO)
logger = logging.getLogger(__name__)

# Fonction pour gérer la commande /start
def start(update, context):
    context.bot.send_message(chat_id=update.effective_chat.id, text="Bonjour! Envoyez-moi le lien de la vidéo YouTube que vous souhaitez télécharger en tant que mp3.")

# Fonction pour télécharger le song
def download_song(update, context):
    video_link = update.message.text
    # Appeler la fonction de téléchargement de vidéo avec le lien fourni
    download_song_from_link(video_link, update, context)

# Fonction pour télécharger la vidéo à partir du lien
def download_song_from_link(video_link, update, context):
    try:
        # Télécharger la vidéo YouTube
        youtube = YouTube(video_link)
        video = youtube.streams.filter(only_audio=True).first()
        filename = video.download()

        # Convertir la vidéo en mp3
        mp3_filename = f"{filename[:-4]}.mp3"
        os.rename(filename, mp3_filename)

        # Envoyer le fichier mp3
        context.bot.send_message(chat_id=update.effective_chat.id, text="Le mp3 est prêt à être téléchargé!")
        context.bot.send_audio(chat_id=update.effective_chat.id, audio=open(mp3_filename, 'rb'))

        # Supprimer les fichiers temporaires
        os.remove(mp3_filename)
    except Exception as e:
        context.bot.send_message(chat_id=update.effective_chat.id, text="Erreur lors du téléchargement du mp3.")

# Create an instance of the Updater class and pass in your bot token
updater = Updater(token='5767020015:AAGpP1hM2JA39DQBRNK9qkP5VJmFqh_czDQ', use_context=True)

# Get the dispatcher to register handlers
dispatcher = updater.dispatcher

# Register the start command handler
dispatcher.add_handler(CommandHandler('start', start))

# Register the video link message handler
dispatcher.add_handler(MessageHandler(Filters.text & ~Filters.command, download_song))

# Start the bot
updater.start_polling()

# Run the bot until you press Ctrl-C or the process receives SIGINT, SIGTERM, or SIGABRT
updater.idle()
# IVIII
