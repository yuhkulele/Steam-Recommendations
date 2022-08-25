# PC Game Recommendations - a Content Based and Collaborative Recommender for Steam

## Overview
My goal with this project was to leverage data from Steam in order to tackle two approaches to provide PC game recommendations for Steam users. I used content labels - categories, genres, and tags - to provide content based recommendations. For a collaborative recommender, I utilized playtime as an implicit rating of a user's engagement with a game. I compared user profiles based on their game playtimes to come up with a collaborative recommender that predicts how much time a player might spend on a game. 

## Business Understanding
Steam is the most popular digital distribution service and store for PC games. While physical discs were once the norm for PC game sales, digital downloads have become the standard means of owning and accessing PC games. Steam offers a platform for users to manage their library of games with access to services such as cloud saves and community features. In 2021, Steam averaged around [69 million daily active players](https://store.steampowered.com/news/group/4145017/view/3133946090937137590). Steam saw a 27% increase in user spending and a 21% increase in playtime from 2020 to 2021. 

Recommender systems are important to online stores to engage consumers with relevant products that have the highest likelihood of being purchased. Steam already uses various recommender systems in their store. There are content based systems (comparing games to games) as well as collaborative systems (comparing user or friend profiles). By using systems that produce favorable recommendations, Steam can maintain customer satisfaction and encourage users to stick with Steam's platform. Users can benefit from good recommendation systems that offer compelling suggestions so that their time is not spent on playing disagreeable games.

Since users have to go through Steam's client to access their games, Steam easily collects vast amounts of user data on game playtimes. Player data, along with community features such as friends lists, are used to fuel Steam's collaborative recommendation systems. Additionally, Steam's shop listings utilizes user-voted labels to help compare game content and provide strong content-based recommendations.

## Data Understanding
I used two different datasets to build my recommendation models. The content based recommender uses a dataset from Kaggle, while the collaborative recommender uses data I collected from Steam's Web API.

The Kaggle dataset contains information about ~27,000 Steam apps/games that were available on Steam's store when the data was collected in 2019. Games were identified by their unique 'appid'. The key information from this dataset is categories, genres, and tags - a collection of labels that describe a game's content. Additional information such as the game's price or the name of the developer is also present in the dataset.

The second dataset consists of Steam users and their playtimes for games in their Steam library. I collected this data in August 2022 through Steam's Web API. The data is stored in a JSON format that connects Steam users, identified by their unique Steam Account ID, to information about which games they own and how much time they've spent playing each one. You need to have a Steam Account in order to request a Web API key. [Here](https://steamcommunity.com/dev) is a general overview of how to access Steam's Web API. This [link](https://developer.valvesoftware.com/wiki/Steam_Web_API) has more documentation on different types of API calls and what kinds of information is available.

### Data Instructions
The Kaggle dataset can be found under the repository's data folder as the file within the Steam_store_data subfolder. The  csv table I used is named 'steam.csv'. Ther other csv files in the subfolder were not used, but originated from the same Kaggle source. Alternatively, the Kaggle dataset can be downloaded ![here](https://www.kaggle.com/datasets/nikdavis/steam-store-games?select=steam.csv).

The Steam Web API data is under the repository's data folder as a JSON file ('Steam_API_Owned_Games.json'). This can be read in using json.load(filename). My notebook includes several functions I used to make API calls and save the JSON data. 

## Data Preparation

### Steam Store Games  - Kaggle Dataset
I used pandas to extract content labels consisting of the 'categories', 'genres', and 'steamspy_tag' columns. There was overlap within the label groups. I filtered out these duplicate labels as well as labels that I reasoned were not strong influences on a Steam user's purchasing decision. Some apps in the dataset were not games and were removed. The labels were combined into a single column in a dataframe along with additional identifying information of games.

Here are the most common labels in the data:
!['popular_labels.png']

### Steam User Libraries - Steam Web API
This data was used for my collaborative filtering model. I removed any users with fewer than 5 games played as well as any games with fewer than 5 users to improve user comparisons. The recommender had to account for if a user owned but had not yet played a game. For each user, I extracted out just the games with recorded playtime for modeling. A separate dictionary contained information on user libraries including unplayed games for my recommender to reference.

## Modeling

### Content-based Approach:
I took the dataframe containing information of labels and applied scikit-learn's MultiLabelBinazer to the labels column. This encoded each label as a 0 or 1 for every game in the dataframe. The result was each row was now a vector of many dimensions(labels) describing a game. I used cosine similarity to calculate similarity between the vectorized game. My content-based recommender returns the top n-recommendations for a user-specified game based on highest similarity scores.

### Collaborative-filtering Approach:
For each game in a user's library, I scaled their playtime against the maximum playtime of that game across all users. This bounded each game's playtime 'rating' between 0 and 1. Using those playtime ratings, I used surprise to implement a collaborative-filtering model to compare user profiles between each other to predict a user's playtimes. The playtimes were predicted as the scaled rating based on how similar users spent their playtime. I iterated through model algorithms, using grid searches to optimize hyperparameters. My best performing model was svd_gs3. This model had the lowest Root Mean Square Error (RMSE) at 0.218 on the test set while also having faster runtimes for the SVD++ model that produced a near-ideentical RMSE. This SVD model can be found as a .sav file under the models folder in the repository.

My collaborative-filtering recommender took in the Steam ID of a user within the dataset and returned n-recommendations. The SVD algorithm determined predictions for games that the user hadn't played based on how their playtime ratings compared to those of other users. The predictions with the highest scaled playtime ratings were returned as the recommendations.

## Conclusion

### Content-based Approach
The content-based recommender defined a game's content by its labels - an agglomeration of categories, genres, and tags from its Steam Store listing. The system was highly dependent on user-voted tags which greatly outnumbered the categories and genres. These labels were encoded into binary features to translate games into a multi-dimensional vectors. Cosine similarity was used to compare vectors against a user-suggested game and return the closest games as recommendations.

The recommender was limited by the labels available in the data. More data about game content would help improve recommendations by extracting the details of how a game plays and what genres it covers. One method of engineering more of these descriptive features is to use natural language processing (NLP) to skim game descriptions or reviews. By pulling out key phrases that players and developers use to describe the game, NLP could provide additional features to better categorize games. Extracting information from game descriptions could provide additional information on how developers or Steam describe a game's contents. Review language would be an additional source of player definition of what a game is about. 

A content-based recommendation system allows anybody to look for recommendations. The person does not have to be a Steam user. This approach can be used to help new Steam users who have not yet purchased many PC games. The binary encoding of labels can accomodate the influx of additional data to refine the way the recommender determines game content. 

It is difficult to assess a content-based recommendation system without user feedback. The recommender has no target and cannot judge by itself if its recommendations are a good fit. Deployment of this model on an app such as Streamlit is one method for me to get feedback from users.

### Collaborative-filtering Approach
Collaborative-filtering recommenders can utilize player's explicit ratings to compare users and return recommendations based on player similarity. However, Steam's review system does not have a robust scale due to its binary Recommended/Not Recommended ratings. Additionally, a collaborative recommender that uses reviews would only work for users who have rated items before. As a Steam user who does not leave product reviews, I wanted to use playtime as a stand-in for ratings in order to create a collaborative-filtering model. 

The amount of time that player spends on a game could be a reflection of how much enjoyment they derive from playing. Therefore, higher playtimes can reflect a positive rating for a game. Since games are not designed to be equally played for the same length of time, my model scaled playtime by the maximum playtime of a game to prevent inflation of ratings for longer games. The trained SVD model compared user playtime profiles and produced estimated playtimes based on latent features in the data. Unfortunately, the approach used to scale playtime seemed to bias the model's recommendations towards a particular set of games. More investigation would have to be done to see user and playtime information about those games to address the issue.

Playtime is a dynamic measure. As such, collaborative-filtering recommendations based on playtime need to be updated and re-trained frequently in order to maintain recommendation quality. The size of the user base and the algorithm used can lead to a computational problem. This was seen with the long training and fit times of the SVD++ model. 

A user must have playtime logged on Steam in order to use this collaborative-filtering model. My model cannot make any predictions for users that were not in the dataset. However, this 'cold-start' problem can be addressed with the use of my content-based approach where new users can build up their library with content-based recommendations until they have accrued enough playtime. Since players have to log in through Steam to access their games, Steam can readily gather playtime data to maintain a robust collaborative recommender. 

### Final Thoughts
The PC game market continues to expand with more platforms offering competition to the Steam storefront. Quality PC game recommendations encourage Steam users to discover new products and provide increase sales for both Steam and game developers. While a single recommender approach will have drawbacks, a content-based system and collaborative-filtering system can work together to provide options that are sure to keep users playing.

## Next Steps
* Try other playtime scalings/implicit ratings
* Use NLP to extract more content features
* Deploy an app with Streamlit 

## Repository Structure

├── data
│   ├── Steam_store_data
│   │   ├── steam.csv
│   │   ├── steam._description.csv
│   │   ├── steam_media_data.csv
│   │   ├── steam_requirements_data.csv
│   │   ├── steam_support_info.csv
│   │   ├── steamspy_tag_data.csv
│   ├── Steam_API_Owned_Games.json
├── images
├── models
│   ├── collab_rec.sav
├── .gitignore
├── README.md
├── notebook.ipynb
├── notebook.pdf
└── presentation.pdf