# Google Maps Eco-Friendly Routing

- **url** = https://www.gstatic.com/gumdrop/sustainability/google-maps-eco-friendly-routing.pdf , [copie locale](LOCALCOPIES/google-maps-eco-friendly-routing.pdf)
- **source** = peu claire
- **auteur** = Google
- **date de publication** = novembre 2021
- **date de rédaction initiale de ces notes** = 21 septembre 2023

## Introduction

- eco-friendly routing pour répondre aux enjeux climatiques
- optimise à la fois le fuel consumption et l'EAT (d'après l'intro, quasiment aucune dégradation sur l'EAT)
- machine learning model

## Introducing eco-friendly routing on Google Maps

- google maps = 1 milliard d'utilisateurs mensuels
- les deux routes sont affichées (meilleur ETA + meilleur fuel efficiency), pour que l'utilisateur puisse choisir celle qu'il préfère

## How eco-friendly routing works

- hum... ils ont l'air de dire qu'ils définissent un set de routes candidates, puis qu'ils appliquent les conditions traffic pour choisir laquelle
    - (ce qui est différent de calculer dynamiquement la meilleure route)
- pour la fuel-consumption, ils incorporent en plus un critère sur le fuel consommé par les routes
- problème = c'est difficile à estimer → c'est le NREL (= National Renewable Energy Laboratory)qui l'a fait
- donc en gros, ils mélangent la façon de faire de google (définir un set de route-candidates + les traiter avec les critères temps-réel) avec les infos NREL (d'estimation de fuel-consumption)
- derrière, ils semblent avoir des règles expertes pour définir à partir de quand le trade-off vaut la peine d'être présenté à l'utilisateur
    - l'idée est de ne pas garder de routes qui dégradent trop l'ETA tout en n'économisant que peu le fuel
- un point important = ils mélangent les modèles NREL de consommation de fuel en fonction de route+vitesse avec les modèles de traffic google et avec les infos géographique des routes pour déduire la fuel-consumption exacte, et identifer les routes qui sont fuel-efficient
- d'après leurs testing, les utilisateurs n'acceptent qu'une faible dégradation de l'ETA pour économiser du temps

## Global roll-out and next steps

- je ne sais pas si ça a été fait au final, mais ils prévoyaient de lancer ça en Europe en 2022
- attention que les paramètres utilisés dans les modèles ne sont pas les mêmes en fonction des régions du globe
