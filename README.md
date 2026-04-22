# ABCblockA24
TV

<!DOCTYPE html>
<html lang="pt-br">
<head>
    <meta charset="UTF-8">
    <style>
        /* Ajusta o container para ficar responsivo no post do WordPress */
        .tv-wrapper {
            position: relative;
            width: 100%;
            padding-bottom: 56.25%; /* Proporção 16:9 */
            background: #000;
            overflow: hidden;
            border-radius: 8px;
        }
        
        #player {
            position: absolute;
            top: 0; left: 0; width: 100%; height: 100%;
        }

        /* Camada invisível para bloquear cliques no vídeo */
        .tv-overlay {
            position: absolute;
            top: 0; left: 0; width: 100%; height: 100%;
            z-index: 10;
            cursor: default;
        }

        /* Tela de Início */
        #tv-screen-off {
            position: absolute;
            top: 0; left: 0; width: 100%; height: 100%;
            background: #1a1a1a;
            display: flex;
            justify-content: center;
            align-items: center;
            z-index: 20;
            cursor: pointer;
        }

        #tv-btn-play {
            padding: 15px 30px;
            background: #ff0000;
            color: #fff;
            border: none;
            border-radius: 5px;
            font-weight: bold;
            font-size: 18px;
            box-shadow: 0 4px 15px rgba(0,0,0,0.5);
        }
    </style>
</head>
<body>

<div class="tv-wrapper">
    <div id="tv-screen-off" onclick="ligarCanal()">
        <button id="tv-btn-play">📺 LIGAR TV AO VIVO</button>
    </div>
    <div class="tv-overlay"></div>
    <div id="player"></div>
</div>

<script src="https://www.youtube.com/iframe_api"></script>
<script>
    // --- CONFIGURAÇÃO DA PLAYLIST ---
    // IMPORTANTE: Coloque a duração exata de cada vídeo em segundos
    const playlist = [
        { id: '04GSON1Wffk', duracao: 201 }, 
        { id: 'wnXRkxffjJg', duracao: 185 }
    ];

    let player;
    const tempoTotal = playlist.reduce((acc, v) => acc + v.duracao, 0);

    // Calcula o ponto exato da "transmissão"
    function obterSincronia() {
        const agora = Math.floor(Date.now() / 1000);
        let resto = agora % tempoTotal;
        
        for (let video of playlist) {
            if (resto < video.duracao) {
                return { id: video.id, start: resto };
            }
            resto -= video.duracao;
        }
    }

    function onYouTubeIframeAPIReady() {
        const conf = obterSincronia();
        player = new YT.Player('player', {
            videoId: conf.id,
            playerVars: {
                'autoplay': 1,
                'controls': 0,      // Remove barras de tempo
                'disablekb': 1,    // Bloqueia teclado
                'modestbranding': 1,
                'rel': 0,
                'start': conf.start
            },
            events: {
                'onReady': (e) => { e.target.mute(); e.target.playVideo(); }
            }
        });
    }

    function ligarCanal() {
        // Remove a tela preta e libera o som
        document.getElementById('tv-screen-off').style.display = 'none';
        
        const statusAtual = obterSincronia();
        player.loadVideoById({
            videoId: statusAtual.id,
            startSeconds: statusAtual.start
        });
        
        player.unMute();
        player.playVideo();
    }

    // Monitora o fim do vídeo para pular para o próximo da grade
    setInterval(() => {
        if (player && player.getPlayerState() === YT.PlayerState.ENDED) {
            const prox = obterSincronia();
            player.loadVideoById({
                videoId: prox.id,
                startSeconds: prox.start
            });
        }
    }, 1000);
</script>
</body>
</html>
