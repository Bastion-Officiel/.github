# 🛡️ Guide Officiel : Créer et Connecter un Bot sur BASTION
### SDK Officiel : `@bastion-hub/bastion.js`

Bienvenue dans le guide complet de développement de bots pour la plateforme **BASTION**. Ce guide vous explique comment créer, configurer et exécuter un bot autonome avec commandes slash, embeds riches, boutons interactifs et modération.

---

## 📑 Sommaire
1. [Installation du SDK](#-1-installation-du-sdk)
2. [Création du Bot sur le Portail Développeur](#-2-création-du-bot-sur-le-portail-développeur)
3. [Configuration des Variables d'Environnement (.env)](#-3-configuration-des-variables-denvironnement-env)
4. [Architecture & Fonctionnement du SDK](#-4-architecture--fonctionnement-du-sdk)
5. [Guide de Code & Exemples Complets](#-5-guide-de-code--exemples-complets)
   * [1. Initialisation du Client & Événements (`index.js`)](#1-initialisation-du-client--événements-indexjs)
   * [2. Déploiement des Commandes Slash (`deploy-commands.js`)](#2-déploiement-des-commandes-slash-deploy-commandsjs)
   * [3. Embeds Riches & Boutons Interactifs](#3-embeds-riches--boutons-interactifs)
   * [4. Menus de Sélection Déroulants (Select Menus)](#4-menus-de-sélection-déroulants-select-menus)
   * [5. Modération & Actions Serveur (Kick, Ban, Purge)](#5-modération--actions-serveur-kick-ban-purge)

---

## 📦 1. Installation du SDK

Le SDK officiel est publié mondialement sur NPM. Pour l'installer dans votre projet :

```bash
npm install @bastion-hub/bastion.js
```

Dans votre `package.json` :
```json
{
  "name": "mon-bot-bastion",
  "version": "1.0.0",
  "main": "index.js",
  "dependencies": {
    "@bastion-hub/bastion.js": "^1.0.0",
    "dotenv": "^16.4.5"
  }
}
```

---

## 🌐 2. Création du Bot sur le Portail Développeur

1. Lancez **BASTION** et ouvrez le **Portail Développeur** via le menu de navigation ou en accédant à :
   `http://localhost:8080/developers`
2. Cliquez sur **« Nouvelle Application »** en haut à droite.
3. Donnez un nom et une description à votre bot (ex: `Protect`, `MusicBot`, `ModManager`).
4. Dans l'onglet **Bot & Token** :
   * Personnalisez l'avatar de votre bot (upload d'image PNG/JPG).
   * Cliquez sur **« Copier le Token »** (ou régénérez-en un si nécessaire).
5. Dans l'onglet **OAuth2 / URL Generator** :
   * Sélectionnez les permissions requises (Administrateur, Envoyer des messages, etc.).
   * Cliquez sur **Inviter sur un serveur** et choisissez le serveur cible.

---

## ⚙️ 3. Configuration des Variables d'Environnement (.env)

Créez un fichier `.env` à la racine de votre bot :

```env
# Token secret de votre bot copié sur le portail développeur
BASTION_TOKEN="votre_token_secret_ici"

# ID de l'application
CLIENT_ID="votre_application_id_ici"

# Endpoints Bastion (par défaut http://localhost:8080)
BASTION_API_URL="http://localhost:8080/api/v10"
BASTION_GATEWAY_URL="ws://localhost:8080/gateway"
```

---

## 🏗️ 4. Architecture & Fonctionnement du SDK

`@bastion-hub/bastion.js` communique en temps réel avec le Hub Bastion via deux canaux :

```
┌─────────────────────────────────────────────────────────────┐
│                          VOTRE BOT                          │
│        (const { Client } = require('@bastion-hub/bastion.js'))│
└──────────────┬───────────────────────────────┬──────────────┘
               │                               │
               │ (Gateway WebSocket)           │ (API REST v10)
               │ ws://localhost:8080/gateway   │ http://localhost:8080/api/v10
               ▼                               ▼
┌─────────────────────────────────────────────────────────────┐
│                    SERVEUR BASTION HUB                      │
│                                                             │
│  • Gateway : Handshake Opcodes (HELLO, IDENTIFY, HEARTBEAT) │
│  • Événements : READY, GUILD_CREATE, MESSAGE_CREATE,        │
│    INTERACTION_CREATE (Commandes Slash, Clics boutons)      │
│  • REST API : Déploiement commandes, Envoi messages/embeds, │
│    Callback d'interactions, Modération membres/serveurs     │
└──────────────────────────────┬──────────────────────────────┘
                               │
                               ▼
┌─────────────────────────────────────────────────────────────┐
│                   INTERFACE UTILISATEUR                     │
│  • Rendu visuel des Embeds avec couleurs et champs          │
│  • Boutons interactifs (Bleu, Gris, Vert, Rouge, Lien)      │
│  • Menus de sélection déroulants (Select Menus)             │
│  • Réponses éphémères (« Visible uniquement par vous »)     │
└─────────────────────────────────────────────────────────────┘
```

---

## 💻 5. Guide de Code & Exemples Complets

### 1. Initialisation du Client & Événements (`index.js`)

```javascript
const { Client, GatewayIntentBits, Partials, Events } = require('@bastion-hub/bastion.js');
require('dotenv').config();

const client = new Client({
    intents: [
        GatewayIntentBits.Guilds,
        GatewayIntentBits.GuildMessages,
        GatewayIntentBits.MessageContent,
        GatewayIntentBits.GuildMembers
    ],
    partials: [Partials.Message, Partials.Channel, Partials.User]
});

// Événement quand le bot est en ligne
client.on(Events.ClientReady, (c) => {
    console.log(`🤖 Bot opérationnel sur BASTION sous le nom : ${c.user.username}`);
    console.log(`🏰 Connecté à ${c.guilds.cache.size} serveur(s).`);
});

// Événement réception de message
client.on(Events.MessageCreate, async (message) => {
    // Ignorer les messages envoyés par les bots
    if (message.author.bot) return;

    if (message.content === '!ping') {
        await message.reply('🏓 **Pong !** Latence gateway : `12ms`');
    }
});

// Événement arrivée d'un nouveau membre
client.on(Events.GuildMemberAdd, async (member) => {
    console.log(`👋 Nouveau membre sur le serveur : ${member.user.username}`);
});

// Connexion du bot
client.login(process.env.BASTION_TOKEN);
```

---

### 2. Déploiement des Commandes Slash (`deploy-commands.js`)

Pour enregistrer vos commandes slash auprès de Bastion afin qu'elles s'affichent lors de la frappe de `/` :

```javascript
const { REST, Routes, SlashCommandBuilder } = require('@bastion-hub/bastion.js');
require('dotenv').config();

const commands = [
    new SlashCommandBuilder()
        .setName('help')
        .setDescription('Afficher le centre d\'aide et de sécurité'),
    new SlashCommandBuilder()
        .setName('stats')
        .setDescription('Afficher les statistiques du serveur'),
    new SlashCommandBuilder()
        .setName('warn')
        .setDescription('Avertir un utilisateur')
        .addUserOption(option => 
            option.setName('target').setDescription('Membre ciblé').setRequired(true))
        .addStringOption(option => 
            option.setName('reason').setDescription('Raison de l\'avertissement').setRequired(false))
].map(cmd => cmd.toJSON());

const rest = new REST().setToken(process.env.BASTION_TOKEN);

(async () => {
    try {
        console.log('🚀 Déploiement des commandes slash sur BASTION...');
        await rest.put(
            Routes.applicationCommands(process.env.CLIENT_ID),
            { body: commands }
        );
        console.log('✅ Commandes déployées avec succès !');
    } catch (error) {
        console.error('❌ Erreur lors du déploiement :', error);
    }
})();
```

---

### 3. Embeds Riches & Boutons Interactifs

```javascript
const { EmbedBuilder, ActionRowBuilder, ButtonBuilder, ButtonStyle, Events } = require('@bastion-hub/bastion.js');

client.on(Events.InteractionCreate, async (interaction) => {
    // 1. Commande Slash /help
    if (interaction.isChatInputCommand()) {
        if (interaction.commandName === 'help') {
            const embed = new EmbedBuilder()
                .setTitle('🛡️ Centre de Protection & Sécurité')
                .setDescription('Configurez les modules actifs sur votre communauté Bastion.')
                .setColor('#5865F2')
                .addFields(
                    { name: 'Système Anti-Raid', value: 'Activé ✅', inline: true },
                    { name: 'Vérification Captcha', value: 'Automatique 🛡️', inline: true },
                    { name: 'Niveau de Modération', value: 'Élevé ⚡', inline: true }
                )
                .setFooter({ text: 'Bastion Security Hub • Version 1.0' })
                .setTimestamp();

            const row = new ActionRowBuilder().addComponents(
                new ButtonBuilder()
                    .setCustomId('btn_toggle_antiraid')
                    .setLabel('Module Anti-Raid')
                    .setStyle(ButtonStyle.Primary),
                new ButtonBuilder()
                    .setCustomId('btn_view_logs')
                    .setLabel('Journaux d\'audit')
                    .setStyle(ButtonStyle.Secondary),
                new ButtonBuilder()
                    .setCustomId('btn_danger_lockdown')
                    .setLabel('Verrouiller le serveur')
                    .setStyle(ButtonStyle.Danger)
            );

            await interaction.reply({ embeds: [embed], components: [row] });
        }
    } 
    // 2. Gestion du Clic sur un Bouton
    else if (interaction.isButton()) {
        if (interaction.customId === 'btn_toggle_antiraid') {
            await interaction.reply({ 
                content: '🛡️ **Module Anti-Raid :** La sensibilité a été mise à jour.', 
                ephemeral: true // Message visible uniquement par l'auteur du clic
            });
        } else if (interaction.customId === 'btn_danger_lockdown') {
            await interaction.reply({ 
                content: '🚨 **Alerte :** Le salon a été placé en confinement temporaire.', 
                ephemeral: false 
            });
        }
    }
});
```

---

### 4. Menus de Sélection Déroulants (Select Menus)

```javascript
const { StringSelectMenuBuilder, ActionRowBuilder, Events } = require('@bastion-hub/bastion.js');

client.on(Events.InteractionCreate, async (interaction) => {
    if (interaction.isChatInputCommand() && interaction.commandName === 'settings') {
        const selectMenu = new StringSelectMenuBuilder()
            .setCustomId('select_category')
            .setPlaceholder('Choisissez une catégorie à configurer...')
            .addOptions([
                {
                    label: 'Modération & Filtres',
                    description: 'Gérer l\'anti-spam, anti-lien et censure',
                    value: 'cat_mod'
                },
                {
                    label: 'Rôles automatiques & Bienvenue',
                    description: 'Attribuer des rôles aux nouveaux membres',
                    value: 'cat_autorole'
                },
                {
                    label: 'Logs & Alertes',
                    description: 'Définir le salon des rapports d\'audit',
                    value: 'cat_logs'
                }
            ]);

        const row = new ActionRowBuilder().addComponents(selectMenu);
        await interaction.reply({ content: '⚙️ **Paramètres du Serveur :**', components: [row] });
    } 
    // Gestion du choix dans le menu déroulant
    else if (interaction.isStringSelectMenu()) {
        if (interaction.customId === 'select_category') {
            const selected = interaction.values[0];
            await interaction.reply({ 
                content: `🔧 Vous avez sélectionné la catégorie : \`${selected}\``, 
                ephemeral: true 
            });
        }
    }
});
```

---

### 5. Modération & Actions Serveur (Kick, Ban, Purge)

```javascript
const { Events } = require('@bastion-hub/bastion.js');

client.on(Events.InteractionCreate, async (interaction) => {
    if (!interaction.isChatInputCommand()) return;

    // Purge de messages
    if (interaction.commandName === 'purge') {
        await interaction.channel.send({ content: '🧹 Nettoyage du salon effectué avec succès.' });
        await interaction.reply({ content: 'Salon purgé !', ephemeral: true });
    }
});
```
