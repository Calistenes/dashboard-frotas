<!DOCTYPE html>
<html lang="pt-PT">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Painel de Rotas e Geolocalização</title>
    
    <link rel="stylesheet" href="https://unpkg.com/leaflet@1.9.4/dist/leaflet.css" />
    <link rel="stylesheet" href="https://unpkg.com/leaflet.markercluster@1.5.3/dist/MarkerCluster.css" />
    <link rel="stylesheet" href="https://unpkg.com/leaflet.markercluster@1.5.3/dist/MarkerCluster.Default.css" />
    
    <style>
        body { margin: 0; font-family: Arial, sans-serif; display: flex; flex-direction: column; height: 100vh; }
        header { background-color: #1a73e8; color: white; padding: 15px; text-align: center; }
        .controles { background: #f1f3f4; padding: 15px; text-align: center; border-bottom: 1px solid #ddd; }
        #btnAtualizar {
            background-color: #34a853; color: white; border: none; padding: 10px 20px;
            font-size: 16px; border-radius: 5px; cursor: pointer; font-weight: bold;
        }
        #btnAtualizar:hover { background-color: #2c8c45; }
        #mapa { flex: 1; width: 100%; }
    </style>
</head>
<body>

    <header>
        <h2>Painel de Rotas Regularizadas</h2>
        <p>A sua localização e os dados do sistema</p>
    </header>

    <div class="controles">
        <button id="btnAtualizar">Atualizar Base de Dados (CSV)</button>
        <input type="file" id="csvFileInput" accept=".csv" style="display: none;" />
    </div>

    <div id="mapa"></div>

    <script src="https://unpkg.com/leaflet@1.9.4/dist/leaflet.js"></script>
    <script src="https://unpkg.com/leaflet.markercluster@1.5.3/dist/leaflet.markercluster.js"></script>
    <script src="https://cdnjs.cloudflare.com/ajax/libs/PapaParse/5.4.1/papaparse.min.js"></script>

    <script>
        var mapa = L.map('mapa').fitWorld();
        L.tileLayer('https://{s}.tile.openstreetmap.org/{z}/{x}/{y}.png', {
            attribution: '&copy; OpenStreetMap'
        }).addTo(mapa);

        mapa.locate({setView: true, maxZoom: 13});
        mapa.on('locationfound', function(e) {
            L.circleMarker(e.latlng, { radius: 8, fillColor: "#4285F4", color: "#fff", weight: 2, opacity: 1, fillOpacity: 0.9 })
                .addTo(mapa)
                .bindPopup("<b>Você está aqui!</b>")
                .openPopup();
            L.circle(e.latlng, e.accuracy / 2).addTo(mapa);
        });
        mapa.on('locationerror', function(e) {
            console.warn("Aviso: Não foi possível aceder à sua localização.");
            mapa.setView([-7.115, -34.861], 10);
        });

        var marcadores = L.markerClusterGroup();
        mapa.addLayer(marcadores);

        function obterIconePorIndicador(indicador) {
            var cor = "blue"; 
            
            if (indicador) {
                var indTexto = indicador.toUpperCase();
                if (indTexto.includes("VERMELHO")) cor = "red";
                else if (indTexto.includes("VERDE")) cor = "green";
                else if (indTexto.includes("AMARELO")) cor = "yellow";
                else if (indTexto.includes("LARANJA")) cor = "orange";
            }

            return new L.Icon({
                iconUrl: 'https://raw.githubusercontent.com/pointhi/leaflet-color-markers/master/img/marker-icon-' + cor + '.png',
                shadowUrl: 'https://cdnjs.cloudflare.com/ajax/libs/leaflet/1.9.4/images/marker-shadow.png',
                iconSize: [25, 41],
                iconAnchor: [12, 41],
                popupAnchor: [1, -34],
                shadowSize: [41, 41]
            });
        }

        document.getElementById('btnAtualizar').addEventListener('click', function() {
            var senha = prompt("Digite a palavra-passe para importar um novo ficheiro:");
            if (senha === "Brisa@1020") {
                document.getElementById('csvFileInput').click();
            } else if (senha !== null) {
                alert("Palavra-passe incorreta. Acesso negado.");
            }
        });

        document.getElementById('csvFileInput').addEventListener('change', function(evento) {
            var arquivo = evento.target.files[0];
            if (!arquivo) return;

            Papa.parse(arquivo, {
                header: true,
                skipEmptyLines: true,
                complete: function(resultados) {
                    var dados = resultados.data;
                    marcadores.clearLayers();

                    dados.forEach(function(linha) {
                        var localizacao = linha['LOCALIZACAO'];
                        
                        if (localizacao && localizacao.includes(',')) {
                            var coords = localizacao.split(',');
                            var lat = parseFloat(coords[0].trim());
                            var lng = parseFloat(coords[1].trim());
                            
                            if (!isNaN(lat) && !isNaN(lng)) {
                                var popupContent = 
                                    "<b>Rota:</b> " + linha['CIDADE_ROTA'] + "<br>" +
                                    "<b>Técnico:</b> " + linha['TECNICO'] + "<br>" +
                                    "<b>Indicador:</b> " + linha['INDICADOR'];

                                var iconeCustomizado = obterIconePorIndicador(linha['INDICADOR']);
                                var pino = L.marker([lat, lng], { icon: iconeCustomizado }).bindPopup(popupContent);
                                
                                marcadores.addLayer(pino);
                            }
                        }
                    });

                    if (marcadores.getBounds().isValid()) {
                        mapa.fitBounds(marcadores.getBounds());
                    }
                    
                    document.getElementById('csvFileInput').value = ""; 
                    alert("Rotas atualizadas com sucesso!");
                }
            });
        });
    </script>
</body>
</html>
