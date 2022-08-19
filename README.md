# PC Game Recommendations - a Content Based and Collaborative Recommender for Steam

## Overview

## Business Understanding
Steam is the most popular digital distribution service and store for PC games. While physical discs were once the norm for PC game sales, digital downloads have become the standard. Steam offers a platform for users to manage their library of game with access to services such as cloud saves and community features. In 2021, Steam averaged around 69 million daily active players. 

Recommender systems are important to online stores to engage consumers with products that would interest them and have the highest likelihood of being purchased. Steam has various recommender systems in their store. There are content based systems (based on what the user has played) as well as collaborative systems (based on similar users or friends of the user have played). 

My goal with this project was to leverage data from Steam in order to build recommender models. 

## Data Understanding
I used two different datasets to build my recommendation models. The content based recommender uses a dataset from Kaggle, while the collaborative recommender uses data I collected from Steam's Web API.

### Steam Store Games  - Kaggle Dataset
I used a dataset from Kaggle containing information on video game categories and genres. The dataset can be found [here](https://www.kaggle.com/datasets/nikdavis/steam-store-games?select=steamspy_tag_data.csv).

The author documented their process of data collection via Steam Web API calls and SteamSpy API calls. This [link](https://nik-davis.github.io/posts/2019/steam-data-collection/) goes into how the data was collected and cleaned.

### Steam User Libraries - Steam Web API
I used Steam's Web API service to collect information on users and their personal libraries of games. You need to have a Steam Account in order to request a Web API key. 
[Here](https://steamcommunity.com/dev) is a general overview of how to access Steam's Web API. This [link](https://developer.valvesoftware.com/wiki/Steam_Web_API) has more documentation on different types of API calls and what kinds of information is available.

## Modeling

!['popular_labels.png']

## Conclusion