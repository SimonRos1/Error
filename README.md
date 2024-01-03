require('dotenv').config();
const { Client, GatewayIntentBits } = require('discord.js');
const { REST } = require('@discordjs/rest');
const { Routes } = require('discord-api-types/v9');
const axios = require('axios');
const cheerio = require('cheerio');


const client = new Client({
    intents: [
        GatewayIntentBits.Guilds,
        GatewayIntentBits.GuildMessages
    ]
});

const targetChannelName = 'bot'; // Ersetze dies durch den tatsächlichen Kanalnamen

client.on('ready', () => {
    console.log(`${client.user.username} hat sich bei Discord angemeldet!`);
});

client.on('messageCreate', async (message) => {
    console.log('Nachricht empfangen:', message.content);

    // Überprüfe, ob die Nachricht im angegebenen Kanal ist
    if (message.channel.name !== targetChannelName) {
        console.log('Ignoriere Nachrichten aus anderen Kanälen');
        return; // Ignoriere Nachrichten aus anderen Kanälen
    }

    // Überprüfe, ob der Befehl "!vacation" ist
    if (message.content === '!vacation') {
        console.log('Befehl !vacation empfangen');

        // Hole Urlaubsideen von Check24
        const check24_url = "https://www.check24.de/reisen/";

        try {
            const response = await axios.get(check24_url);
            const $ = cheerio.load(response.data);
            const vacation_ideas = $('.reisen-hotelname a');

            if (vacation_ideas.length > 0) {
                const random_idea = vacation_ideas.eq(Math.floor(Math.random() * vacation_ideas.length)).attr('title');
                await message.channel.send(`Wie wäre es mit einem Urlaub in ${random_idea}?`);
            } else {
                await message.channel.send("Entschuldigung, konnte keine Urlaubsideen abrufen.");
            }
        } catch (error) {
            console.error('Fehler beim Abrufen von Urlaubsideen:', error.message);
            await message.channel.send("Entschuldigung, konnte keine Verbindung zu Check24 herstellen.");
        }
    }

    // Deine bereits vorhandene Nachrichtenverarbeitung
    if (message.content === "ping") {
        console.log('Befehl ping empfangen');
        await message.channel.send("pong");
    }
});

// Melde dich bei Discord an
client.login(process.env.DISCORD_BOT_TOKEN);
