# projet-finance-1

Voici un projet codé sur python qui permet à l'utilisateur de saisir une action, un prix min, un prix max et un laps de temps au bout duquel le cours de l'action est actualisé. Si l'action n'est plus dans la tranche de prix, l'utilisateur reçoit un mail qui l'informe. Il pourra alors agir en conséquence en la vendant ou en rachetant d'autres.

# -*- coding: utf-8 -*-
"""
Created on Thu Aug 14 23:19:07 2025

@author: louis
"""

import yfinance as yf
import tkinter as tk
from tkinter import messagebox
import threading
import time
import smtplib
from email.mime.text import MIMEText
from email.mime.multipart import MIMEMultipart

# Fonction pour envoyer un email
def envoyer_email(destinataire, sujet, message_corps, email_expediteur, mot_de_passe):
    msg = MIMEMultipart()
    msg['From'] = email_expediteur
    msg['To'] = destinataire
    msg['Subject'] = sujet
    msg.attach(MIMEText(message_corps, 'plain'))

    try:
        serveur = smtplib.SMTP('smtp.gmail.com', 587)
        serveur.starttls()
        serveur.login(email_expediteur, mot_de_passe)
        serveur.send_message(msg)
        serveur.quit()
        print("Email envoyé !")
    except Exception as e:
        print("Erreur envoi email :", e)

# Fonction de surveillance
def surveiller_action(symbol, prix_min, prix_max, intervalle, stop_flag, email_utilisateur, email_expediteur, mot_de_passe):
    while not stop_flag[0]:
        try:
            data = yf.Ticker(symbol)
            prix_actuel = data.history(period="1d")['Close'][-1]

            if prix_actuel < prix_min or prix_actuel > prix_max:
                message = f"⚠️ {symbol} est à {prix_actuel:.2f}, hors de la tranche !"
                print(message)
                # Alerte pop-up
                messagebox.showwarning("Alerte action", message)
                # Envoi email
                envoyer_email(
                    destinataire=email_utilisateur,
                    sujet=f"Alerte pour {symbol}",
                    message_corps=message,
                    email_expediteur=email_expediteur,
                    mot_de_passe=mot_de_passe
                )
            else:
                print(f"{symbol} est à {prix_actuel:.2f}, dans la tranche.")
        except Exception as e:
            print("Erreur récupération prix :", e)
        
        time.sleep(intervalle)

# Fonction appelée par le bouton "Démarrer"
def demarrer_surveillance():
    symbol = entry_symbol.get().upper()
    try:
        prix_min = float(entry_min.get())
        prix_max = float(entry_max.get())
        intervalle = int(entry_intervalle.get())
    except ValueError:
        messagebox.showerror("Erreur", "Veuillez entrer des nombres valides pour les prix et l'intervalle.")
        return

    email_utilisateur = entry_email_utilisateur.get()
    email_expediteur = entry_email_expediteur.get()
    mot_de_passe = entry_mdp.get()

    stop_flag[0] = False
    # Lancer la surveillance dans un thread séparé pour ne pas bloquer l'interface
    thread = threading.Thread(target=surveiller_action, args=(symbol, prix_min, prix_max, intervalle, stop_flag, email_utilisateur, email_expediteur, mot_de_passe))
    thread.daemon = True
    thread.start()

# Fonction appelée par le bouton "Arrêter"
def arreter_surveillance():
    stop_flag[0] = True

# Création de la fenêtre
root = tk.Tk()
root.title("Surveillance Action avec Email")

# Labels
tk.Label(root, text="Symbole action :").grid(row=0, column=0, sticky="e")
tk.Label(root, text="Prix minimum :").grid(row=1, column=0, sticky="e")
tk.Label(root, text="Prix maximum :").grid(row=2, column=0, sticky="e")
tk.Label(root, text="Intervalle (s) :").grid(row=3, column=0, sticky="e")
tk.Label(root, text="Email utilisateur :").grid(row=4, column=0, sticky="e")
tk.Label(root, text="Email expéditeur :").grid(row=5, column=0, sticky="e")
tk.Label(root, text="Mot de passe app :").grid(row=6, column=0, sticky="e")

# Entrées
entry_symbol = tk.Entry(root)
entry_min = tk.Entry(root)
entry_max = tk.Entry(root)
entry_intervalle = tk.Entry(root)
entry_email_utilisateur = tk.Entry(root)
entry_email_expediteur = tk.Entry(root)
entry_mdp = tk.Entry(root, show="*")  # cacher le mot de passe

entry_symbol.grid(row=0, column=1)
entry_min.grid(row=1, column=1)
entry_max.grid(row=2, column=1)
entry_intervalle.grid(row=3, column=1)
entry_email_utilisateur.grid(row=4, column=1)
entry_email_expediteur.grid(row=5, column=1)
entry_mdp.grid(row=6, column=1)

# Boutons
tk.Button(root, text="Démarrer", command=demarrer_surveillance).grid(row=7, column=0, pady=10)
tk.Button(root, text="Arrêter", command=arreter_surveillance).grid(row=7, column=1, pady=10)

stop_flag = [False]  # drapeau pour arrêter le thread

root.mainloop()
