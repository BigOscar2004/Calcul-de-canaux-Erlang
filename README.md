<!DOCTYPE html>
<html lang="fr">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Calculateur Erlang B – Canaux & Probabilité de rejet</title>
    <style>
        body {
            font-family: 'Segoe UI', Roboto, sans-serif;
            max-width: 550px;
            margin: 2rem auto;
            padding: 0 1.2rem;
            background: #f4f6f9;
            color: #1e2b3c;
        }
        .card {
            background: white;
            border-radius: 24px;
            padding: 2rem 2rem 2.2rem;
            box-shadow: 0 12px 28px rgba(0,0,0,0.06);
            border: 1px solid #e9eef3;
        }
        h1 {
            margin: 0 0 0.25rem;
            font-weight: 500;
            font-size: 2rem;
            letter-spacing: -0.01em;
            display: flex;
            align-items: center;
            gap: 8px;
        }
        h1 span {
            background: #1e3a5f;
            color: white;
            font-size: 0.9rem;
            padding: 2px 12px;
            border-radius: 30px;
            font-weight: 400;
        }
        .sub {
            color: #546e7a;
            margin-bottom: 1.8rem;
            border-left: 4px solid #2c7da0;
            padding-left: 1rem;
        }
        label {
            font-weight: 600;
            display: block;
            margin: 1.4rem 0 0.3rem;
            color: #0b2b40;
        }
        input {
            width: 100%;
            padding: 0.9rem 1rem;
            font-size: 1.1rem;
            border: 1.5px solid #cfdde9;
            border-radius: 18px;
            box-sizing: border-box;
            background: white;
            transition: 0.15s;
        }
        input:focus {
            border-color: #1e5f8e;
            outline: none;
            box-shadow: 0 0 0 4px rgba(30,95,142,0.1);
        }
        .hint {
            font-size: 0.85rem;
            color: #5e7e9c;
            margin-top: 4px;
        }
        button {
            margin: 2.2rem 0 1rem;
            width: 100%;
            padding: 1rem;
            font-size: 1.3rem;
            font-weight: 600;
            background: #1e5f8e;
            color: white;
            border: none;
            border-radius: 50px;
            cursor: pointer;
            transition: 0.2s;
            box-shadow: 0 4px 8px rgba(0,20,40,0.1);
        }
        button:hover {
            background: #0e3e5e;
            transform: scale(1.01);
        }
        .result {
            margin-top: 1.8rem;
            padding: 1.4rem 1.6rem;
            background: #e2edf7;
            border-radius: 20px;
            border-left: 8px solid #1e5f8e;
        }
        .result p {
            margin: 0.4rem 0;
            font-size: 1.2rem;
        }
        .result .big-number {
            font-size: 2.8rem;
            font-weight: 700;
            color: #003153;
            line-height: 1.2;
        }
        .footer-note {
            margin-top: 1.2rem;
            font-size: 0.9rem;
            color: #5b7a95;
            text-align: center;
        }
        .error-msg {
            color: #b13e3e;
            font-weight: 500;
        }
    </style>
</head>
<body>
<div class="card">
    <h1>📡 Erlang B <span>dimensionnement</span></h1>
    <div class="sub">Nombre de canaux nécessaire pour une probabilité de rejet donnée</div>

    <label>📊 Trafic offert (Erlangs)</label>
    <input type="number" id="trafic" value="10.0" step="0.1" min="0.001" lang="en">
    <div class="hint">Exemple : 10 Erlangs = 10 appels simultanés en moyenne</div>

    <label>🚫 Probabilité de rejet maximale (ex: 0.01 = 1%)</label>
    <input type="number" id="rejet" value="0.01" step="0.001" min="0.000001" max="1" lang="en">
    <div class="hint">Valeur entre 0 et 1 (0.02 = 2% de blocage)</div>

    <button onclick="calculerErlangB()">Calculer le nombre de canaux</button>

    <div class="result" id="resultat">
        <p>👆 Saisissez les valeurs et cliquez</p>
    </div>
    <div class="footer-note">
        Formule d'Erlang B — perte de trafic sans file d'attente.
    </div>
</div>

<script>
    // --- Fonction utilitaire : factorielle (itératif) ---
    function factorielle(n) {
        if (n === 0 || n === 1) return 1;
        let result = 1;
        for (let i = 2; i <= n; i++) result *= i;
        return result;
    }

    // --- Calcul de la probabilité de blocage Erlang B ---
    // A = trafic en Erlangs, N = nombre de canaux
    function erlangB(A, N) {
        if (N === 0) return 1.0;       // aucun canal → blocage certain
        if (A === 0) return 0.0;

        // Méthode stable numériquement : calcul itératif
        let prob = 1.0;
        for (let i = 1; i <= N; i++) {
            prob = (A * prob) / (i + A * prob);
        }
        return prob;
    }

    // --- Recherche du nombre minimum de canaux pour P_blocage ≤ P_cible ---
    function trouverCanaux(A, P_cible) {
        if (P_cible <= 0 || P_cible >= 1) return "⛔ Probabilité cible doit être entre 0 et 1 (exclus)";
        if (A <= 0) return 0; // aucun trafic → 0 canal suffit

        let N = 1;
        const maxIter = 10000; // sécurité pour éviter boucle infinie
        while (N <= maxIter) {
            const blocage = erlangB(A, N);
            if (blocage <= P_cible) {
                return N;
            }
            N++;
        }
        return "⚠️ Trop grand, vérifiez les valeurs.";
    }

    function calculerErlangB() {
        const traficInput = document.getElementById('trafic').value;
        const rejetInput = document.getElementById('rejet').value;

        const A = parseFloat(traficInput);
        const Pcible = parseFloat(rejetInput);

        const resultDiv = document.getElementById('resultat');

        // Validation
        if (isNaN(A) || A < 0) {
            resultDiv.innerHTML = `<p class="error-msg">❌ Le trafic doit être un nombre positif.</p>`;
            return;
        }
        if (isNaN(Pcible) || Pcible <= 0 || Pcible >= 1) {
            resultDiv.innerHTML = `<p class="error-msg">❌ La probabilité de rejet doit être entre 0 et 1 (ex: 0.01).</p>`;
            return;
        }

        // Calcul du nombre de canaux
        const canaux = trouverCanaux(A, Pcible);

        if (typeof canaux === 'number') {
            // Calculer la probabilité de blocage exacte pour ce nombre de canaux
            const blocageReel = erlangB(A, canaux);
            const pourcentageBlocage = (blocageReel * 100).toFixed(4);

            // Affichage
            resultDiv.innerHTML = `
                <p style="margin-bottom: 0.2rem;">🔢 <strong>Nombre de canaux nécessaires</strong></p>
                <div class="big-number">${canaux}</div>
                <p style="margin-top: 0.8rem;">📉 Probabilité de rejet obtenue : <strong>${pourcentageBlocage}%</strong></p>
                <p style="font-size:0.95rem; opacity:0.8;">(≤ ${ (Pcible*100).toFixed(3) }% demandé)</p>
                <p style="margin-top: 0.8rem;">📞 Trafic écoulé ≈ ${ (A * (1 - blocageReel)).toFixed(3) } Erlang</p>
            `;
        } else {
            // Message d'erreur
            resultDiv.innerHTML = `<p class="error-msg">${canaux}</p>`;
        }
    }

    // Lancer un calcul initial au chargement de la page
    window.onload = function() {
        calculerErlangB();
    };
</script>
</body>
</html>
