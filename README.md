# BLS-Espagne-Alg-rie---Alerte-de-Disponibilit-Visa
Vérifie la disponibilité des RDV sur le site BLS après connexion et alerte si un calendrier apparaît.


// ==UserScript==
// @name         BLS Espagne (Algérie) - Alerte de Disponibilité Visa
// @namespace    http://tampermonkey.net/
// @version      1.0
// @description  Vérifie la disponibilité des RDV sur le site BLS après connexion et alerte si un calendrier apparaît.
// @author       Gemini
// @match        https://algeria.blsspainvisa.com/*
// @grant        none
// @run-at       document-idle
// ==/UserScript==

(function() {
    'use strict';

    // --- 1. CONFIGURATION ---

    // L'intervalle de vérification en millisecondes (30 secondes)
    const INTERVALLE_VERIFICATION_MS = 30000;

    // Le SÉLECTEUR CSS de l'élément que vous recherchez.
    // **ATTENTION :** C'est l'élément qui prouverait que les RDV sont disponibles.
    // Si la page affiche un message "Actuellement indisponible", trouvez le SÉLECTEUR du CALENDRIER qui devrait apparaître.
    const SELECTEUR_CALENDRIER = '#calendar-container'; // ID/Classe hypothétique du calendrier

    // Le SÉLECTEUR CSS d'un message d'indisponibilité (pour savoir où vous êtes).
    // Si le message d'indisponibilité est présent, le script sait qu'il doit attendre.
    const SELECTEUR_MESSAGE_INDISPONIBLE = '.no-slots-available'; // ID/Classe hypothétique du message

    // --- 2. FONCTIONS D'ALERTE ---

    /**
     * Joue un son d'alerte fort.
     */
    function jouerAlerteSonore() {
        const audio = new Audio('https://www.soundjay.com/buttons/beep-07a.mp3'); // Un son de bip en ligne
        audio.loop = true; // Joue en boucle jusqu'à ce que l'utilisateur arrête.
        audio.play();
        return audio;
    }

    /**
     * Lance l'alerte visuelle et sonore.
     */
    function declencherAlerte() {
        const alerteSonore = jouerAlerteSonore();

        // Alerte visuelle
        alert("🎉 ALERTE RDV VISA BLS DISPONIBLE ! Cliquer sur OK pour arrêter le son.");

        // Quand l'utilisateur clique sur OK, arrêter le son
        alerteSonore.pause();
        alerteSonore.currentTime = 0;

        // Mettre fin au script après l'alerte
        clearInterval(verificateurIntervalle);
    }

    // --- 3. LOGIQUE PRINCIPALE ---

    /**
     * Vérifie la page pour détecter le calendrier.
     */
    function verifierDisponibilite() {
        // 1. Vérifier si l'utilisateur est bien sur la page du calendrier/prise de RDV
        if (window.location.href.includes('appointment')) {
            console.log("BLS Alerte: Vérification de la disponibilité...");

            // 2. Recherche l'élément du calendrier (disponible)
            const calendrierElement = document.querySelector(SELECTEUR_CALENDRIER);

            // 3. Recherche le message d'indisponibilité
            const indisponibleMessage = document.querySelector(SELECTEUR_MESSAGE_INDISPONIBLE);

            if (calendrierElement && calendrierElement.offsetHeight > 0) {
                // Le calendrier est visible (il existe et n'est pas caché)
                console.log("BLS Alerte: Calendrier de RDV TROUVÉ !");
                declencherAlerte();

            } else if (indisponibleMessage) {
                // Le message d'indisponibilité est toujours présent
                console.log("BLS Alerte: Toujours indisponible. Attente...");

            } else {
                // Si ni le calendrier n'est trouvé, ni le message d'indisponibilité,
                // il y a peut-être eu un changement sur la page ou une erreur.
                // On considère cela comme une POTENTIELLE DISPONIBILITÉ.
                console.log("BLS Alerte: État Inconnu / Potentielle Disponibilité !");
                declencherAlerte();
            }
        }
    }

    // Lancer la vérification à intervalles réguliers
    const verificateurIntervalle = setInterval(verifierDisponibilite, INTERVALLE_VERIFICATION_MS);

    // Première vérification immédiate
    verifierDisponibilite();

})();
