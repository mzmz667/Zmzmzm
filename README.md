# Zmzmzm
Digiview
const express = require('express');
const app = express();
const PORT = 3000;

app.use(express.json());

app.get('/', (req, res) => {
    res.send(`
    <!DOCTYPE html>
    <html lang="fr">
    <head>
        <meta charset="UTF-8">
        <title>Lecteur Tunnelisé Stable</title>
        <style>
            body { font-family: sans-serif; background: #121212; color: #fff; text-align: center; padding: 40px; }
            input { width: 60%; padding: 12px; background: #000; color: #fff; border: 1px solid #444; }
            button { padding: 12px 20px; background: #007bff; color: white; border: none; cursor: pointer; }
            video { width: 100%; max-width: 700px; margin-top: 20px; display: none; }
        </style>
    </head>
    <body>
        <h1>Lecteur Anti-Blocage (Moteur Cobalt)</h1>
        <input type="text" id="url" placeholder="Lien YouTube..." />
        <button onclick="charger()">Regarder</button>
        <p id="status"></p>
        <video id="player" controls></video>

        <script>
            async function charger() {
                const url = document.getElementById('url').value;
                const status = document.getElementById('status');
                const player = document.getElementById('player');
                
                status.innerText = "Génération du tunnel sécurisé...";
                
                // On demande à notre serveur de requêter l'API de contournement
                const response = await fetch('/get-stream?url=' + encodeURIComponent(url));
                const data = await response.json();
                
                if(data.streamUrl) {
                    status.innerText = "Vidéo débloquée !";
                    player.src = data.streamUrl; // L'URL du flux proxy
                    player.style.display = "block";
                } else {
                    status.innerText = "Erreur de décodage.";
                }
            }
        </script>
    </body>
    </html>
    `);
});

// Le point de contact qui traite la vidéo via une API intermédiaire externe
app.get('/get-stream', async (req, res) => {
    const videoUrl = req.query.url;
    try {
        // Nous contactons une instance publique de Cobalt qui sert de super-proxy
        const cobaltResponse = await fetch('https://cobalt.tools', {
            method: 'POST',
            headers: {
                'Accept': 'application/json',
                'Content-Type': 'application/json'
            },
            body: JSON.stringify({ url: videoUrl }) // Envoi de l'URL demandée
        });
        
        const result = await cobaltResponse.json();
        
        // Cobalt renvoie une URL "tunnel" qui stream la vidéo directement sans passer par l'IP de YouTube
        if (result.url) {
            res.json({ streamUrl: result.url });
        } else {
            res.status(500).json({ error: "L'intermédiaire n'a pas pu traiter cette vidéo" });
        }
    } catch (error) {
        res.status(500).json({ error: error.message });
    }
});

app.listen(PORT, () => console.log(`Serveur actif : http://localhost:${PORT}`));
